# 14 · API Specification

Aplikasi **tidak** membuka REST API publik. Komunikasi memakai:

1. **Server Components → `lib/data/*`**: query baca langsung ke Supabase (server client, RLS berlaku).
2. **Server Actions `lib/actions/*`**: semua mutasi dari UI.
3. **Route Handlers `app/api/*`**: unduhan file (export), auth callback, webhook.
4. **Supabase RPC**: agregasi dashboard & operasi atomik.
5. **Edge Functions**: kirim WhatsApp/email, undang pengguna (service role).

## Konvensi

| Aspek | Aturan |
|---|---|
| Autentikasi | Cookie sesi Supabase; setiap action memanggil `requireUser()` |
| Otorisasi | `requireSchoolRole(schoolId, roles)` di action **dan** RLS di DB |
| Validasi | Zod di `lib/validation/*`, dipakai bersama client & server |
| Hasil action | `type ActionResult<T> = { ok: true; data: T } \| { ok: false; error: { code: ErrorCode; message: string; fieldErrors?: Record<string,string[]> } }` |
| ErrorCode | `UNAUTHENTICATED` · `FORBIDDEN` · `NOT_FOUND` · `VALIDATION` · `CONFLICT` · `RATE_LIMITED` · `INTERNAL` |
| Paging | `page` (1-based), `pageSize` ∈ {10, 12, 15}; response `{ rows, total }` |
| Tanggal | ISO 8601 UTC di API; format ke Asia/Jakarta di UI |

## 1. Query baca (`lib/data`)

| Fungsi | Input | Output | Sumber DB |
|---|---|---|---|
| `getAccessibleSchools()` | — | `{group:{code,name,brand,logo}, schools:{id,code,name,applicantCount}[]}[]` | `schools` + count, RLS |
| `getSchoolByCode(code)` | code | `School \| null` | `schools` |
| `getMyRoles(schoolId)` | schoolId | `AppRole[]` | `user_school_roles` |
| `getNavCounts(schoolId, yearId)` | | jumlah per menu + macet + bermasalah | RPC `nav_counts` |
| `getDashboard(schoolId, yearId)` | | `{kpis, funnel, levels, sources}` | RPC `dashboard_kpis`, `applicant_funnel`, view |
| `getTrend(schoolId, yearId, granularity)` | `'week' \| 'day'` | `{current:number[], previous:number[], labels:string[]}` | RPC `applicant_trend` |
| `getActionItems(schoolId, yearId)` | | `{late[], schedule[], payments[]}` | `applicant_list_v`, `psychology_tests`, `parent_interviews` |
| `listApplicants(params)` | lihat skema | `{rows: ApplicantRow[], total}` | `applicant_list_v` |
| `countApplicantsByFlow(params)` | filter tanpa tab | `Record<Flow, number>` | RPC `applicant_flow_counts` |
| `getApplicantDetail(id)` | id | `ApplicantDetail` (120 field + dokumen + riwayat) | `applicants` + join tabel anak |
| `listTransactions(params)` | `state, method, q, page` | `{rows,total,kpis}` | `transactions` |
| `listPsychologyTests(params)` | `recommendation \| 'no_note', q, page` | `{rows,total,kpis}` | `psychology_tests` join |
| `getPsychologyTest(id)` | id | detail + psikolog + lembaga | |
| `listPsychologists(params)` | `active \| incomplete` | | `psychologists` + count tes per sekolah |
| `listClasses(params)` | `levelId, yearId, q` | | `class_overview_v` |
| `listStudents(params)` | `levelId, status, q` | | `students` + `class_enrollments` |
| `listTeachers(params)` | `position, q` | | `teacher_overview_v` |

### `listApplicants` params (Zod)

```ts
export const applicantListParams = z.object({
  schoolId: z.string().uuid(),
  flow: z.enum(['all','leads','register','payment','tuition','waiting_list','invalid']).default('all'),
  yearId: z.string().uuid().or(z.literal('all')),
  levelId: z.string().uuid().optional(),
  status: z.enum(['all','late','issue','form','psy','itv','decision','tuition','done','stale']).default('all'),
  rangeDays: z.enum(['all','7','30','90']).default('all'),
  officerId: z.string().uuid().optional(),
  q: z.string().trim().max(100).optional(),
  sort: z.enum(['created.desc','created.asc','days.desc','days.asc']).default('created.desc'),
  page: z.coerce.number().int().min(1).default(1),
  pageSize: z.union([z.literal(10), z.literal(12), z.literal(15)]).default(10),
});
```

## 2. Server Actions (`lib/actions`)

| Action | Input | Role | Efek | Error khusus |
|---|---|---|---|---|
| `signIn` | `{email, password}` | publik | Sesi | `VALIDATION`, kredensial salah → pesan generik |
| `signOut` | — | user | Hapus sesi | |
| `requestPasswordReset` | `{email}` | publik | Email reset | Selalu sukses (anti enumerasi) |
| `updatePassword` | `{password}` | sesi recovery | | Min 12 karakter |
| `moveApplicantStage` | `{schoolId, ids: uuid[1..200], to: Flow, note?}` | EDIT (+principal untuk keputusan) | RPC `move_applicant_stage` | Id yang sudah di tahap tujuan diabaikan (15 §9); hasil = jumlah yang berubah |
| `createApplicant` | skema form (**belum didesain**) | EDIT | insert applicants + tabel anak (transaksi RPC) | `CONFLICT` no. registrasi |
| `updateApplicantSection` | `{id, section, values}` | EDIT | update 1 tabel anak | |
| `setAcceptanceDecision` | `{id, status, date, note}` | principal, school_admin | update kolom keputusan | |
| `uploadApplicantDocument` | `{applicantId, docType, file}` | EDIT | Storage + upsert `applicant_documents` | `VALIDATION` tipe/ukuran |
| `deleteApplicantDocument` | `{documentId}` | EDIT | Hapus objek + baris | |
| `sendReminders` | `{schoolId, ids, channel, templateKey}` | EDIT, finance (tagihan) | Insert `notification_logs` (queued) → Edge Function | `RATE_LIMITED` |
| `scheduleTest` / `scheduleInterview` | `{applicantId, at, psychologistId?}` | EDIT | insert | |
| `savePsychologyResult` | `{testId, iqScore, recommendation, description, file?}` | EDIT, psychologist | update + storage | |
| `requestPsychologistNote` | `{testId}` | EDIT | notifikasi | |
| `setReconciled` | `{transactionIds, reconciled: boolean}` | finance | update | |
| `inviteUser` | `{email, fullName, schoolId, role}` | school_admin | Edge Function `invite-user` | `FORBIDDEN` jika role > pengundang |
| CRUD kelas / guru / psikolog / siswa | skema masing-masing | school_admin | | |
| `updateSchoolSettings` | `{schoolId, yearId, tuitionTarget, stuckDays, registrationFee, quotas[]}` | school_admin | upsert | |

Setelah mutasi: `revalidatePath('/[school]/…')` untuk halaman terkait.

## 3. Route Handlers

| Method | Path | Fungsi | Auth | Response |
|---|---|---|---|---|
| GET | `/auth/callback?code&next` | Exchange code → sesi | publik | 302 ke `next` (hanya path internal) |
| GET | `/api/export/[entity]?<filter yang sama dengan list>&ids=` | Export .xlsx (applicants, transactions, psychology, classes, students, teachers) | user + role baca | `200 application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`; `413` bila > 10.000 baris |
| POST | `/api/webhooks/notifications` | Status kirim dari provider WA/email | signature provider | `204` |

Contoh error JSON:

```json
{ "error": { "code": "FORBIDDEN", "message": "Anda tidak memiliki akses ke sekolah ini." } }
```

## 4. RPC Postgres

| RPC | Parameter | Return |
|---|---|---|
| `dashboard_kpis` | `p_school uuid, p_year uuid` | `total, new_7d, prev_7d, accepted, conversion, tuition_collected, tuition_target, quota_total, quota_remaining, quota_by_level jsonb` |
| `applicant_funnel` | sama | `step text, count int` (5 baris, Invalid dikecualikan) + `invalid_count` |
| `applicant_trend` | `+ p_granularity text` | `bucket_start date, current int, previous int` |
| `applicant_flow_counts` | filter list | `flow, count` |
| `nav_counts` | `p_school, p_year` | jumlah per modul, macet, bermasalah |
| `move_applicant_stage` | `p_ids uuid[], p_to applicant_flow, p_note text` | `int` (jumlah diubah) |

## 5. Edge Functions

| Function | Trigger | Fungsi |
|---|---|---|
| `send-notifications` | Cron 1 menit / insert `notification_logs` | Kirim antrean WA/email, update status |
| `invite-user` | Server Action | `auth.admin.inviteUserByEmail` + profil + role |
| `cleanup-storage` | Hapus `applicant_documents` | Hapus objek Storage yatim |
