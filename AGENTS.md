# AGENTS.md

> Konvensi teknis untuk semua agen AI & developer. **Aturan izin/keamanan ada di [`CLAUDE.md`](CLAUDE.md) dan selalu didahulukan.** File ini tidak mengizinkan apa pun yang dilarang CLAUDE.md.

## Struktur project

```text
app/                 # Next.js App Router (lihat docs/09 §3)
  (auth)/            # login, forgot-password, reset-password
  (app)/[school]/    # semua halaman per sekolah
  api/               # route handlers: export, webhooks
  auth/callback/
components/
  ui/                # shadcn (generated, boleh diubah seperlunya)
  shell/ data/ admission/ dashboard/ modules/ states/
lib/
  supabase/          # server.ts, client.ts, middleware.ts, database.types.ts
  data/              # query baca (server only)
  actions/           # server actions ('use server')
  domain/            # logika murni, tanpa I/O
  validation/        # Zod schemas
  format.ts brand.ts url-state.ts
supabase/
  migrations/ seed.sql tests/ functions/
scripts/migrate-from-odoo/
design/              # prototipe (read-only)
docs/
```

## Konvensi penamaan

| Hal | Konvensi | Contoh |
|---|---|---|
| Komponen | PascalCase, 1 komponen per file | `ProgressStepper.tsx` |
| Hook | `useX` | `useDataPage.ts` |
| Fungsi data | kata kerja + entitas | `listApplicants`, `getApplicantDetail` |
| Server Action | kata kerja + entitas | `moveApplicantStage` |
| Zod schema | `xxxSchema` | `applicantListParamsSchema` |
| Tabel DB | snake_case jamak | `applicant_documents` |
| Kolom | snake_case | `stage_entered_at` |
| Enum DB | snake_case | `applicant_flow` |
| Migration | `YYYYMMDDHHMMSS_deskripsi.sql` (via `supabase migration new`) | |
| Route segment | kebab-case Inggris | `/psychologists` |
| Teks UI | Bahasa Indonesia | "Pendaftar baru" |
| Kode & komentar | Bahasa Inggris (komentar boleh Indonesia untuk aturan bisnis) | |

## Aturan komponen

- Server Component secara default; tambahkan `'use client'` hanya bila butuh state, event, efek, atau browser API.
- Props bertipe eksplisit; tidak menerima objek DB mentah, tetapi view model dari `lib/data`.
- Styling hanya via Tailwind + token; tanpa inline hex.
- Setiap komponen interaktif: elemen asli, `aria-*` sesuai `docs/04` §8, bisa dioperasikan keyboard.
- Teks status selalu disertai ikon/teks (bukan warna saja).

## Aturan TypeScript

- `strict: true`, `noUncheckedIndexedAccess: true`.
- Dilarang `any`, `@ts-ignore` (pakai `@ts-expect-error` + alasan bila benar-benar perlu).
- Tipe DB dari `database.types.ts` (generated, jangan diedit manual).
- Hasil action bertipe `ActionResult<T>`.

## Aturan database

- Mengikuti `docs/10` & `docs/11`. Tabel baru wajib: PK uuid, `school_id` atau relasi ke tabel ber-`school_id`, timestamps, RLS, index FK, test pgTAP.
- Tidak ada query SQL string dari input pengguna.
- Derivasi status: view SQL + `lib/domain` diuji dengan kasus yang sama (ADR-006).

## Aturan Supabase

- Server: `createServerClient` (cookies). Browser: `createBrowserClient` hanya bila perlu.
- Storage: bucket privat, signed URL dibuat di server.
- Edge Functions untuk operasi yang butuh service role.

## Aturan testing

| Perubahan | Test wajib |
|---|---|
| `lib/domain` | Unit Vitest |
| Komponen bersama | Testing Library |
| Server Action | Integrasi dengan Supabase lokal |
| Migration / RLS | pgTAP |
| Alur halaman | Playwright (+ axe) |

## Aturan dokumentasi

Perubahan perilaku, skema, route, atau keputusan wajib memperbarui dokumen `docs/` terkait + `docs/CHANGELOG.md`. ADR baru untuk keputusan arsitektur di `docs/25`.

## Workflow

**Sebelum coding**
1. Baca `docs/00` dan dokumen fitur terkait.
2. Cek fase aktif di `docs/22` & Open Questions terkait.
3. Untuk tugas besar, buat rencana singkat dan minta persetujuan.

**Sesudah coding**
1. `pnpm lint && pnpm typecheck && pnpm test` (+ `supabase test db`, `pnpm test:e2e` bila relevan).
2. Bandingkan tampilan dengan prototipe.
3. Perbarui dokumentasi + CHANGELOG.
4. Ringkas apa yang dikerjakan, apa yang sudah diverifikasi, dan apa yang belum.
