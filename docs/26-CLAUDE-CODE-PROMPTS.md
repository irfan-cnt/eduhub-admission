# 26 · Prompt Step-by-Step untuk Claude Code

Urutan prompt untuk membangun **Edu Hub · Modul Admission** (Next.js + Supabase) dengan Claude Code, mengikuti [22-IMPLEMENTATION-ROADMAP](22-IMPLEMENTATION-ROADMAP.md). Setiap prompt merujuk ke dokumen di `docs/`. Claude Code membaca aturan kerjanya dari [`CLAUDE.md`](../CLAUDE.md) dan [`AGENTS.md`](../AGENTS.md).

---

## Cara pakai

| Aturan | Alasan |
|---|---|
| **1 prompt = 1 sesi.** Jalankan `/clear` sebelum prompt berikutnya | Konteks bersih; aturan tetap terbaca dari `CLAUDE.md` |
| **Plan mode dulu** (`Shift+Tab` 2×) untuk prompt bertanda 🧭 | Setujui rencana sebelum Claude menulis kode |
| **Commit setelah "Selesai bila" terpenuhi** | Titik kembali yang aman |
| **Jangan lompat fase** | Fase berikutnya bergantung pada hasil fase sebelumnya |
| **Gerbang 🔒** | Prompt bertanda ini baru boleh dijalankan setelah syaratnya terpenuhi |
| Hasil meleset? | Pakai [prompt perbaikan](#prompt-perbaikan), jangan ulang prompt fase dari awal |

### Peta prompt

| Fase | Prompt | Hasil |
|---|---|---|
| 0 | 0.1 – 0.2 | Repo siap, dokumentasi dicek Claude |
| 1 | 1.1 – 1.10 | Semua halaman tampil persis prototipe dengan data mock |
| 2 🔒 | 2.1 – 2.5 | Skema, RLS, storage, seed di Supabase lokal |
| 3 🔒 | 3.1 – 3.4 | Login, sesi, akses per sekolah |
| 4 | 4.1 – 4.6 | Dashboard, Pendaftar, Detail memakai data Supabase |
| 5 🔒 | 5.1 – 5.7 | Modul pendukung, form, notifikasi, pengaturan, pengguna |
| 6 🔒 | 6.1 – 6.4 | Migrasi Odoo, audit keamanan, UAT |
| 7 | 7.1 | Optimasi performa |
| 8 | 8.1 | Persiapan deploy (eksekusi production oleh manusia) |

---

## Fase 0 · Persiapan

### Langkah manual

```bash
unzip edu-hub-admission-docs.zip && cd edu-hub-admission
git init && git add . && git commit -m "docs: dokumentasi awal v1.0.0"
claude
```

### Prompt 0.1 · Orientasi & pengecekan dokumentasi

```text
Baca CLAUDE.md, AGENTS.md, lalu docs/00-DOCUMENTATION-INDEX.md dan semua dokumen yang diindeksnya.
Buka juga design/Main.dc.html dan design/Mobile.dc.html (prototipe Claude Design: markup <x-dc> + logika JS di <script data-dc-script>).

JANGAN menulis kode. Laporkan:
1. Ringkasan aplikasi dalam 5 kalimat.
2. Ketidaksesuaian antara dokumentasi dan prototipe (halaman, field, perilaku) yang belum tercatat di docs/24.
3. Ketidaksesuaian antar-dokumen (mis. nama kolom di 10 vs 14, role di 03 vs 13).
4. Pertanyaan tambahan yang perlu masuk docs/24.

Bila menemukan hal baru, usulkan perubahan dokumen (diff), tapi jangan terapkan sebelum saya setuju.
```

### Prompt 0.2 · Mencatat jawaban Open Questions

Jalankan setiap kali Anda punya jawaban untuk pertanyaan di [24-OPEN-QUESTIONS](24-OPEN-QUESTIONS.md).

```text
Berikut jawaban saya untuk Open Questions:
- Q-01: …
- Q-03: …
- Q-07: …
- Q-13: …

Perbarui docs/24 (tandai terjawab + jawabannya), lalu perbarui semua dokumen yang terdampak (10, 13, 14, 15, 22, 25 bila perlu ADR baru) agar konsisten. Tambahkan entri di docs/CHANGELOG.md.
Tampilkan ringkasan perubahan per file. Jangan menulis kode aplikasi.
```

---

## Fase 1 · Setup + porting UI (data mock)

> Tujuan: seluruh UI tampil persis prototipe **tanpa Supabase**. Data diambil dari adapter mock yang kontraknya sama dengan `lib/data` di [14](14-API-SPECIFICATION.md), sehingga Fase 4 cukup mengganti adapter.

### Prompt 1.1 · Scaffold proyek 🧭

```text
Baca CLAUDE.md, AGENTS.md, docs/09 §2–3, docs/25 ADR-004/005/010.

Siapkan proyek Next.js di folder ini (jangan hapus docs/, design/, CLAUDE.md, AGENTS.md, README.md):
1. Next.js App Router versi stabil terbaru, TypeScript strict + noUncheckedIndexedAccess, pnpm, alias "@/".
2. Tailwind CSS + shadcn/ui. Tambahkan komponen: button, input, select, checkbox, dialog, sheet, popover, dropdown-menu, tabs, tooltip, badge, skeleton, separator, scroll-area, sonner.
3. Dependency (ADR-010): zod, nuqs, @tanstack/react-table, @tanstack/react-query, recharts, @dnd-kit/core, @dnd-kit/sortable, lucide-react, date-fns.
4. Dev: vitest, @vitest/coverage-v8, @testing-library/react, @testing-library/user-event, @testing-library/jest-dom, jsdom, @playwright/test, @axe-core/playwright, prettier.
5. Script: dev, build, start, lint, typecheck (tsc --noEmit), test, test:coverage, test:e2e.
6. Struktur folder persis docs/09 §3, setiap page.tsx berisi judul halaman sementara.
7. Font Source Sans 3 via next/font (400/500/600/700).
8. Catat versi persis semua dependency di docs/25 (ADR-010, tabel versi) dan tandai Q-12 terjawab.

Selesai bila: pnpm dev jalan, semua route di docs/05 (P1–P10, dengan [school]=kg-sec) bisa dibuka, pnpm lint && pnpm typecheck && pnpm test lulus.
```

### Prompt 1.2 · Design tokens & brand

```text
Baca docs/04 §1–4 dan §9.

1. app/globals.css: CSS variable untuk 4 kombinasi [data-theme=light|dark] × [data-brand=kgs|tj] dengan nilai PERSIS tabel docs/04 §2.1–2.3. Bandingkan dengan blok <style> di design/Main.dc.html (.t-light, .t-dark, .b-tj.*) dan pastikan identik.
2. tailwind.config: warna memetakan ke CSS var (bg, surface, surface2, border, text, muted, primary.*, secondary.*, side.*, status.*), radius 6/8/10/12.
3. lib/brand.ts: 2 grup + 9 sekolah (code, name, group, brand, levels) dari fungsi SCHOOLS() di prototipe; logo dari design/assets (salin ke public/assets/).
4. BrandProvider + ThemeToggle: tema dari cookie "theme" (agar SSR tidak berkedip) + fallback prefers-color-scheme.
5. lib/format.ts: formatDateTime ("26 Okt 2026, 15:47", Asia/Jakarta), formatDate, formatRupiah ("Rp 12.500.000"), formatRupiahShort ("Rp 1,50 M", "Rp 352 jt"), formatPercent ("29,9%"), plus unit test.
6. Halaman /dev/tokens (hanya NODE_ENV=development): swatch semua token untuk 4 kombinasi, tombol primer, badge 5 tone.
7. Unit test rasio kontras: --on-primary vs --primary ≥ 4.5 untuk 4 kombinasi; laporkan (jangan ubah) --side-fg vs --side-bg TJ light (Q-10).

Selesai bila: /dev/tokens cocok dengan prototipe; test lulus.
```

### Prompt 1.3 · Logika domain + unit test

```text
Baca docs/15 SELURUHNYA dan docs/21 §2. Bandingkan dengan fungsi derive(), matchStatus(), appView() di design/Main.dc.html.

Buat lib/domain/ (fungsi murni, tanpa React/Supabase):
- types.ts: tipe domain (Flow, Applicant view model, Step, StepState, IssueCode, StatusFilterKey).
- steps.ts: deriveCurrentStep, deriveSteps (5 titik + state + tooltip + tanggal).
- stage.ts: isAllDone, isStale, isLate(stuckDays), daysInStage, daysLabel.
- issues.ts: dataIssues → kode + label Indonesia.
- filters.ts: matchesStatusFilter untuk 9 key docs/15 §4.
- actions.ts: primaryAction (10 baris docs/15 §5).
- funnel.ts: computeFunnel (Invalid dikecualikan, drop-off, terbesar).
- psychology.ts: parseIqScore + recommendationFromIq.
- completeness.ts: konfigurasi 7 tab detail (tab → kartu → field, sesuai docs/10 "Pemetaan 120 field" dan docs/04/06) + hitung terisi/total + daftar "Perlu dilengkapi".

Unit test: semua kasus U1–U12 docs/21, plus test bahwa konfigurasi detail berisi tepat 119 tampilan field dan setiap field Odoo muncul tepat sekali (NISN sekali).

Selesai bila: pnpm test:coverage menunjukkan lib/domain ≥ 95% line coverage.
```

### Prompt 1.4 · Kontrak data + adapter mock

```text
Baca docs/14 §1–2 dan docs/10 (untuk nama field).

1. lib/data/types.ts: tipe hasil setiap fungsi baca di docs/14 §1 (ApplicantRow, ApplicantDetail dengan 120 field, DashboardData, TransactionRow, dst.), dan Zod schema params (applicantListParams persis docs/14).
2. lib/data/index.ts: interface DataSource dengan semua fungsi baca docs/14 §1. Pilih implementasi lewat env DATA_SOURCE=mock|supabase (default mock).
3. lib/data/mock/: generator deterministik (seed per sekolah) yang memporting buildData() dan buildExtra() prototipe, termasuk:
   - konfigurasi SCHOOLS() (flows, years, levels, kuota, target, nominal),
   - data kelas FIXED (6 sekolah, data asli Odoo) PERSIS dari prototipe,
   - nama orang karangan; beberapa nama siswa kosong & tuition Rp 0.
   Mock menjalankan filter/sort/paging di "server" dan memakai lib/domain untuk derivasi.
4. lib/actions/ versi mock untuk moveApplicantStage & sendReminders (in-memory per proses) dengan ActionResult (docs/14 Konvensi).

Test: jumlah per flow mock KG Secondary = 11/55/74/87/11 (TA semua) dan SMP TJ Bandung = 59/91/62/431/3; paging 10/12/15; data kelas 6 sekolah snapshot.

Selesai bila: test lulus, tidak ada import mock di luar lib/data/index.ts.
```

### Prompt 1.5 · App shell 🧭

```text
Baca docs/04 §5–8, docs/05, docs/06 §1, docs/08 §1, docs/17. Lihat blok <aside class="shell-side"> dan <header> di design/Main.dc.html.

Bangun app/(app)/[school]/layout.tsx + components/shell/:
1. Sidebar 272px bg var(--side-bg): logo sekolah aktif + "Edu Hub · Admission PSB"; SchoolSwitcher (popover berkelompok TJ 5 unit / KGS 4 unit, jumlah pendaftar, centang aktif, item 13px weight 400); NavGroup Admission/Akademik/SDM & Mitra dengan jumlah (padding 10px 14px, min-height 44, radius 10, 14px); pintasan Macet/Data bermasalah/Waiting List/Invalid (hanya bila tahap dipakai); profil di bawah.
2. TopBar 64px: toggle sidebar, breadcrumb, "Cari pendaftar" (shortcut "/"), notifikasi macet, ThemeToggle.
3. Hanya area konten yang scroll; < 820px bertumpuk.
4. Pindah sekolah → /[code]/<halaman sama>, filter direset (docs/17 aturan).
5. Gunakan data dari DataSource (mock), dan NoAccessState bila kode sekolah tidak ada.

Selesai bila: tampilan sidebar & top bar sama dengan prototipe untuk smp-tj-bdg dan kg-sec (light & dark), navigasi keyboard penuh, E2E kecil: ganti sekolah mengubah data-brand.
```

### Prompt 1.6 · Komponen data bersama

```text
Baca docs/06 §2, docs/08 §7, docs/17 (nama param URL).

Buat components/data/ dan components/states/: PageHeader, ViewToggle (ikon tabel/kartu/kanban, aria-pressed), FilterTabs (badge jumlah), FilterPills, SearchInput (debounce 250ms), FilterSelect, ResetFilterButton, DataTable (TanStack Table, aria-sort, seleksi, scroll horizontal di kotak), CardGrid, GenericInfoCard, Pager (a–b dari n, Awal/‹/›/Akhir, scroll ke atas), KpiRow/KpiCard, StatusBadge, IssueFlag (tooltip hover+focus), EmptyState, ErrorState, NoAccessState, Skeletons.
Hook lib/url-state.ts berbasis nuqs untuk semua param docs/17 (filter apa pun → page=1).

Halaman /dev/components (development) memamerkan semuanya di 4 kombinasi tema.

Selesai bila: test Testing Library per komponen (render + keyboard), axe bersih di /dev/components.
```

### Prompt 1.7 · Halaman Pendaftar (tabel & kartu) 🧭

```text
Baca docs/02 FR-20..28, docs/08 §3, docs/15 §2–5. Bandingkan dengan blok isList di design/Main.dc.html.

Bangun app/(app)/[school]/applicants/page.tsx (Server Component baca searchParams → DataSource):
- FilterTabs tahap + jumlah; SearchInput; filter TA (default TA admission aktif), level, status (9), rentang, petugas; Reset.
- ViewToggle table/card/kanban (kanban di prompt 1.8, sementara tampilkan placeholder).
- ApplicantTable: kolom persis prototipe (Siswa & orang tua + ⚠, Level, Tahap, Progress stepper + ringkasan, Lama di tahap (sort), Tanggal daftar (sort), Aksi). 10/halaman.
- ApplicantCard grid: 12/halaman.
- BulkActionBar: Pengingat WhatsApp, Email, Ubah tahap (ConfirmDialog), Export Excel (sementara toast), Batal pilih.
- ProgressStepper (varian sm) dari lib/domain/steps.
- Klik nama → ?detail=<id> (drawer di 1.9, sementara Sheet placeholder).
- Semua state: loading.tsx, kosong, tidak ada hasil, error.tsx.

Selesai bila: E2E E3 & E4 (docs/21) lulus dengan mock; tampilan dibandingkan screenshot prototipe.
```

### Prompt 1.8 · Kanban

```text
Baca docs/08 §3 (baris kanban) dan blok showKanban di prototipe.

KanbanBoard dengan dnd-kit (Pointer + Keyboard sensor, announcements Bahasa Indonesia): satu kolom per tahap yang dipakai sekolah, 8 kartu per kolom urut terlama, "+N lainnya — buka di tabel", drop di kolom lain → ConfirmDialog → moveApplicantStage dengan useOptimistic + rollback + toast. Lazy load (next/dynamic).

Selesai bila: E2E E5 (mouse & keyboard) lulus.
```

### Prompt 1.9 · Detail pendaftar (7 tab) 🧭

```text
Baca docs/02 FR-40..46, docs/08 §4, docs/10 "Pemetaan 120 field", konfigurasi tab di lib/domain/completeness.ts, dan fungsi appForm()/appView() + markup drawer di prototipe.

Bangun ApplicantDrawer (Sheet kanan 780px; X/overlay/Esc; versi halaman penuh di /applicants/[id]):
- Header: kode, avatar inisial, nama (merah miring bila kosong), level·sekolah·TA, tanggal daftar, badge tahap/lama/flag, aksi utama, WhatsApp, Ubah tahap, ProgressStepper varian lg.
- Tab sticky (?dtab=): Ringkasan, Profil siswa, Seleksi & pembayaran, Dokumen, Keluarga, Dapodik, Riwayat level, masing-masing dengan badge x/y.
- Ringkasan: CompletenessCard (klik bar → tab), TodoList, Informasi pendaftaran + Chat WhatsApp/Kirim email.
- Seleksi: SelectionTimeline 5 tahap + kartu Rekening & VA + Notifikasi.
- Dokumen: progress x/9, No. KK & NIK, 9 DocumentTile (Unggah/Ganti: sementara toast).
- FieldView (teks, "Belum diisi", badge, Ya/Tidak, file, foto) + toggle "Semua field / Belum diisi saja".
- BackNav "← Kembali ke …" berbasis riwayat (dikosongkan saat ditutup).
- Riwayat level: EmptyState (Q-08).

Selesai bila: test bahwa drawer merender 119 field; E2E E6 lulus; tampilan sama dengan prototipe untuk pendaftar tahap Leads, Payment, Tuition.
```

### Prompt 1.10 · Dashboard, modul pendukung & mobile

Kerjakan dalam 3 sesi terpisah.

```text
[Sesi A · Dashboard] Baca docs/02 FR-10..15, docs/15 §6, blok isDash prototipe.
Bangun /dashboard: AcademicYearSelect, KpiRow 5 KPI, FunnelChart, TrendChart (Recharts, lazy, toggle minggu/hari, aria-label ringkasan), ActionPanel 3 kolom (klik → drawer), LevelQuotaChart, SourceChart. Setiap widget dalam <Suspense> dengan skeleton sendiri.
Selesai bila: unit test agregasi = angka manual dari mock; tampilan sama dengan prototipe.
```

```text
[Sesi B · Modul pendukung] Baca docs/02 FR-50..56, docs/08 §5, blok isX/extraVM prototipe.
Bangun /transactions, /psychology (+ PsychologyDrawer 600px dengan "Data pendaftar →" ke ApplicantDrawer tab seleksi + BackNav "Kembali ke Hasil psikotes"), /psychologists, /classes ("Lihat siswa"), /students, /teachers. Semua dengan KPI, FilterTabs, pil filter, SearchInput, ViewToggle tabel/kartu (default kartu untuk Guru & Psikolog), Pager 10/12.
Selesai bila: E2E E8 & E9 lulus; snapshot kelas 6 sekolah sama dengan data Odoo di mock.
```

```text
[Sesi C · Mobile] Baca docs/04 §5 mobile, docs/05 tabel mobile, docs/06 §6, docs/08 §6, design/Mobile.dc.html.
Di ≤ 640px: MobileAppBar (sekolah → SchoolSheet), BottomNav Ringkasan/Pendaftar/Tindakan, Ringkasan mobile, Pendaftar mobile (pil tab, FilterSheet, mode Pilih, LoadMore 15 via TanStack useInfiniteQuery, ADR-004), Tindakan (segmented), Detail layar penuh dengan PillTabs sticky tinggi tetap 56px. Halaman modul pendukung: tampil sebagai kartu (K-08 belum didesain, beri tahu saya bagaimana Anda menanganinya).
Selesai bila: E2E E10 di viewport 390×844 lulus, tanpa scroll horizontal.
```

**Checkpoint akhir Fase 1:**

```text
Jalankan pnpm lint, typecheck, test:coverage, test:e2e (desktop + mobile) dan axe di semua route, light/dark, kedua brand.
Laporkan: hasil setiap perintah, perbedaan visual yang masih ada dengan prototipe (per halaman), dan apa yang belum diverifikasi.
Perbarui docs/05 kolom status implementasi dan docs/CHANGELOG.md.
```

---

## Fase 2 · Supabase foundation

> 🔒 **Gerbang:** Q-01, Q-02, Q-03, Q-07, Q-13 sudah dijawab dan dicatat lewat Prompt 0.2.

### Prompt 2.1 · Inisialisasi & migration skema 🧭

```text
Baca docs/10 dan docs/11 seluruhnya, docs/24 (jawaban terbaru), docs/25 ADR-002/006/007/008.

1. supabase init; aktifkan extension pg_trgm, pgcrypto.
2. Buat migration terpisah (supabase migration new) berurutan:
   a) enum, b) master sekolah & pengguna, c) applicants + tabel anak, d) psikotes, wawancara, transaksi, e) akademik, f) log & audit, g) trigger set_updated_at, track_stage_change, konsistensi lintas tabel (docs/11), h) view applicant_list_v, class_overview_v, teacher_overview_v (security_invoker), i) RPC dashboard_kpis, applicant_funnel, applicant_trend, applicant_flow_counts, nav_counts, move_applicant_stage.
3. Sesuaikan dengan jawaban Open Questions. Bila jawaban menyebabkan skema berbeda dari docs/10, perbarui docs/10 & docs/11 dulu dan tunjukkan diff-nya.
4. JANGAN buat RLS di prompt ini (prompt 2.2), tapi aktifkan `enable row level security` di setiap tabel.
5. supabase gen types → lib/supabase/database.types.ts.

Selesai bila: supabase db reset bersih dari nol; pgTAP struktur (tabel, kolom, FK, unique, check) lulus.
```

### Prompt 2.2 · Row Level Security

```text
Baca docs/03 dan docs/13 seluruhnya.

1. Migration: fungsi helper is_super_admin, has_school_role, applicant_school (security definer, search_path=public, grant ke authenticated).
2. Policy untuk SEMUA tabel sesuai tabel docs/13 §2. Tidak boleh ada policy using(true) untuk role authenticated pada tabel ber-school_id.
3. pgTAP di supabase/tests/rls/: buat user uji per role (super_admin, school_admin, admission_officer, principal, finance, viewer) di 2 sekolah berbeda; uji SELECT/INSERT/UPDATE/DELETE per tabel. Wajib mencakup R1–R10 docs/21.
4. Laporkan setiap tabel yang policy-nya Anda nilai tidak cocok dengan permission matrix docs/03.

Selesai bila: supabase test db lulus seluruhnya.
```

### Prompt 2.3 · Storage

```text
Baca docs/16.

Migration: 4 bucket (public-assets publik; applicant-documents, psychology-results, staff-photos privat, dengan file_size_limit & allowed_mime_types), policy storage.objects persis docs/16, trigger/queue untuk cleanup objek saat baris applicant_documents dihapus.
pgTAP/integration test: officer bisa upload ke folder sekolahnya, ditolak ke sekolah lain (R9); finance hanya bisa membaca bukti bayar.

Selesai bila: test lulus.
```

### Prompt 2.4 · Seed development

```text
Buat supabase/seed.sql (atau scripts/seed.ts yang dipanggil setelah reset) yang mengisi data contoh SETARA dengan mock Fase 1:
2 grup, 9 sekolah, level, tahun ajaran 2022/2023–2028/2029, kuota & settings, lead_sources, pendaftar dengan distribusi flow/TA yang sama dengan mock, tabel anak sesuai tahap (seperti appForm()), psikotes, wawancara, transaksi, guru, kelas (data asli 6 sekolah), siswa & enrollment, psikolog.
Semua nama orang karangan. Tidak ada data pribadi asli.
Buat juga 6 user uji (satu per role) dengan password dari env lokal, bukan hardcode.

Selesai bila: supabase db reset menghasilkan jumlah per flow yang sama dengan test mock di prompt 1.4.
```

### Prompt 2.5 · Paritas logika SQL ↔ domain

```text
Baca docs/25 ADR-006.

Buat test yang menjalankan dataset seed yang sama melalui (a) applicant_list_v + RPC dan (b) lib/domain, lalu membandingkan per pendaftar: days_in_stage, is_late, is_stale, all_done, issue_flags, hasil 9 filter status, funnel, KPI.
Perbaiki perbedaan di SQL atau domain (jelaskan mana yang benar menurut docs/15).

Selesai bila: 0 perbedaan.
```

---

## Fase 3 · Authentication & authorization

> 🔒 **Gerbang:** desain halaman login disetujui (atau setujui sketsa dari Prompt 3.1), dan Q-04 dijawab.

### Prompt 3.1 · Sketsa halaman auth

```text
Baca docs/04, docs/05 (N1–N4), docs/12.
Halaman login, lupa password, dan reset password BELUM didesain. Buat sketsa memakai token, font, logo, dan komponen yang ada (kartu terpusat, logo grup netral, input, tombol primer). Jangan membuat gaya baru.
Tampilkan screenshot (Playwright) light & dark, lalu TUNGGU persetujuan saya sebelum menyambungkannya ke Supabase.
```

### Prompt 3.2 · Supabase SSR, middleware, auth flow 🧭

```text
Baca docs/12 seluruhnya dan docs/19 §1. Cek dokumentasi @supabase/ssr untuk VERSI yang terpasang sebelum menulis kode.

1. lib/supabase/server.ts, client.ts, middleware.ts (pola cookie sesuai versi).
2. middleware.ts: refresh sesi via auth.getUser(), route publik vs privat, redirect /login?next= (hanya path internal).
3. Server Actions signIn, signOut, requestPasswordReset (anti-enumerasi), updatePassword (min 12) dengan Zod + ActionResult.
4. app/auth/callback/route.ts (exchangeCodeForSession).
5. Trigger on auth.users insert → profiles.
6. Konfigurasi supabase/config.toml: signup dimatikan, redirect URL lokal, template email Bahasa Indonesia.

Selesai bila: E2E E1 lulus pada Supabase lokal (login user uji → redirect ke sekolah default); login salah menampilkan pesan generik; service role tidak dipakai di kode Next.js (cek dengan grep).
```

### Prompt 3.3 · Guard sekolah & role

```text
Baca docs/03, docs/12 §5, docs/14 Konvensi.

1. lib/auth/: requireUser(), getMyRoles(schoolId), requireSchoolRole(schoolId, roles) → melempar FORBIDDEN.
2. app/(app)/[school]/layout.tsx: getSchoolByCode (RLS) → NoAccessState 403 bila tidak ada akses; context read-only {user, roles, school, brand}.
3. "/" redirect ke profiles.default_school_id atau sekolah pertama yang diizinkan.
4. getAccessibleSchools untuk SchoolSwitcher (hanya sekolah yang diizinkan).
5. UI menyembunyikan aksi yang tidak diizinkan per role (permission matrix docs/03), dan server tetap memvalidasi.

Selesai bila: E2E: user officer SMP TJ Bandung hanya melihat sekolahnya, dan URL sekolah lain menampilkan NoAccess; test unit requireSchoolRole.
```

### Prompt 3.4 · Undang pengguna (Edge Function)

```text
Baca docs/12 §2 (Undangan), docs/14 §5.
Edge Function invite-user (service role di Supabase secrets): validasi pemanggil school_admin/super admin, role yang diberikan tidak melebihi pemanggil, inviteUserByEmail + profiles + user_school_roles. Server Action inviteUser memanggilnya.
Jangan membuat UI manajemen pengguna (belum didesain, Fase 5).

Selesai bila: test integrasi lokal (supabase functions serve) lulus; R10 lulus.
```

---

## Fase 4 · Core features dengan data Supabase

### Prompt 4.1 · Adapter Supabase untuk lib/data 🧭

```text
Baca docs/14 §1 & §4, docs/17 (cache), docs/20 (strategi query).

Implementasikan lib/data/supabase/ untuk SEMUA fungsi DataSource yang dipakai Dashboard, Pendaftar, Detail, Shell (getAccessibleSchools, getSchoolByCode, getMyRoles, getNavCounts, getDashboard, getTrend, getActionItems, listApplicants, countApplicantsByFlow, getApplicantDetail).
- Server client per request, select kolom yang perlu saja, paging range(), count exact hanya untuk total.
- Pencarian: escape % dan _ sebelum ilike (docs/19 §3).
- Detail: query tabel anak paralel, dokumen dengan signed URL 5 menit.
- Mapping DB → view model persis tipe lib/data/types.ts (validasi Zod di dev).

Test kontrak: mock dan supabase (seed) mengembalikan bentuk data yang sama untuk params yang sama.
Selesai bila: DATA_SOURCE=supabase menampilkan Dashboard, Pendaftar, Detail dari seed tanpa perubahan komponen.
```

### Prompt 4.2 · Ubah tahap (tunggal, massal, kanban)

```text
Baca docs/07 §5, docs/13 §4, docs/15 §1.
Server Action moveApplicantStage: Zod (1–200 id), requireSchoolRole, rpc move_applicant_stage, mapping error (42501 → FORBIDDEN, tahap sama → CONFLICT), revalidatePath halaman & jumlah sidebar. Terapkan aturan transisi sesuai jawaban Q-03 (catatan wajib untuk mundur/invalid bila disetujui).
Sambungkan ke tabel, aksi massal, drawer, kanban (optimistic + rollback).

Selesai bila: E2E E4 & E5 lulus di Supabase lokal; applicant_stage_history tercatat dengan changed_by benar.
```

### Prompt 4.3 · Upload & lihat dokumen

```text
Baca docs/16 seluruhnya, docs/18 §2 (File).
Server Action uploadApplicantDocument (FormData; validasi MIME magic bytes + ukuran; path {school}/{applicant}/{type}/{uuid}.{ext}; upsert applicant_documents; hapus file lama) dan deleteApplicantDocument. Thumbnail/resize foto ≤ 1200px. DocumentTile menampilkan progress & error + Coba lagi.

Selesai bila: E2E E7 lulus; upload file .svg atau > 10 MB ditolak dengan pesan Indonesia.
```

### Prompt 4.4 · Export Excel

```text
Baca docs/14 §3 (export), docs/19 §3 (formula injection), docs/20 (export besar).
Route handler GET /api/export/applicants dengan filter yang sama seperti list (+ ids opsional): requireSchoolRole, exceljs, kolom sesuai tabel (tanpa data sensitif kecuali role SENS, konfirmasi ke saya kolom mana saja), header Bahasa Indonesia, escape sel diawali = + - @, batas 10.000 baris (413). Rate limit 10/jam/pengguna.

Selesai bila: unduhan dari toolbar & aksi massal berhasil; test formula injection.
```

### Prompt 4.5 · Edit data pendaftar

> 🔒 Butuh keputusan desain FR-47 (edit per kartu atau mode edit tab).

```text
Baca docs/02 FR-47, docs/18 §2, konfigurasi tab di lib/domain/completeness.ts.
Usulkan dulu pola edit yang konsisten dengan prototipe (mis. tombol "Ubah" di setiap kartu → field menjadi input, Simpan/Batal), tampilkan screenshot, TUNGGU persetujuan.
Setelah disetujui: Server Action updateApplicantSection per kartu (Zod per section, satu tabel per aksi, audit_logs nama kolom), role EDIT; kartu sensitif hanya untuk SENS.
```

### Prompt 4.6 · Checkpoint Fase 4

```text
Jalankan semua test (unit, pgTAP, integrasi, E2E desktop+mobile, axe) dengan DATA_SOURCE=supabase.
Verifikasi acceptance criteria US-01..US-04 (docs/02 §4) satu per satu dan laporkan lulus/gagal.
Perbarui docs/05 status, docs/CHANGELOG.md.
```

---

## Fase 5 · Advanced features

> 🔒 Desain form (FR-92..95) dan provider WA/email (Q-05) tersedia sebelum prompt yang memerlukannya.

| Prompt | Isi |
|---|---|
| 5.1 | Adapter Supabase untuk Transaksi, Psikotes, Psikolog, Kelas, Siswa, Guru |
| 5.2 | Rekonsiliasi transaksi (`setReconciled`) |
| 5.3 | Psikotes: jadwal, input hasil + PDF, keputusan penerimaan |
| 5.4 | Notifikasi WA/email (Edge Function + log) |
| 5.5 | Pendaftar baru |
| 5.6 | CRUD kelas, guru, psikolog, siswa |
| 5.7 | Pengaturan sekolah & manajemen pengguna |

### Prompt 5.1 · Modul pendukung

```text
Baca docs/14 §1 (fungsi list modul pendukung), docs/15 §7.
Implementasikan adapter Supabase untuk listTransactions, listPsychologyTests, getPsychologyTest, listPsychologists, listClasses, listStudents, listTeachers (memakai class_overview_v & teacher_overview_v) + KPI masing-masing. Test kontrak mock vs supabase.
Selesai bila: semua halaman modul pendukung tampil dari seed; E2E E8 & E9 lulus.
```

### Prompt 5.2 · Rekonsiliasi

```text
Baca docs/07 §8, docs/03 (finance). Desain aksi rekonsiliasi belum ada: usulkan (mis. checkbox baris + tombol "Tandai cocok" di BulkActionBar), TUNGGU persetujuan, lalu implementasikan setReconciled (role finance/school_admin, reconciled_at/by, audit).
```

### Prompt 5.3 · Psikotes & keputusan

```text
Baca docs/02 FR-51/FR-93, docs/15 §5, ADR-008, jawaban Q-04 & Q-09.
1. scheduleTest & scheduleInterview (aksi utama "Jadwalkan psikotes/wawancara").
2. savePsychologyResult (skor IQ, rekomendasi, catatan, PDF ke psychology-results) dan halaman/dialog "Input hasil tes" (sketsa dulu, tunggu persetujuan).
3. setAcceptanceDecision (principal/school_admin) dari aksi "Input keputusan".
4. requestPsychologistNote.
Selesai bila: alur Payment → psikotes → wawancara → keputusan → Tuition bisa dijalankan end-to-end di UI.
```

### Prompt 5.4 · Notifikasi WhatsApp & email

```text
Baca docs/14 §2 (sendReminders) & §5, docs/19 §4, jawaban Q-05.
1. Template (form_link, payment_reminder, tuition_invoice, psychology_result) di tabel/konfigurasi, teks Indonesia, saya review isinya.
2. sendReminders: validasi nomor/email, maks 200 per aksi, insert notification_logs (queued).
3. Edge Function send-notifications (provider dari Q-05, API key di Supabase secrets) + webhook status /api/webhooks/notifications dengan verifikasi signature.
4. Toast "N dari M terkirim / gagal".
Gunakan mode sandbox provider; JANGAN mengirim pesan ke nomor asli.
```

### Prompt 5.5 – 5.7 · Form & administrasi

```text
[5.5] Pendaftar baru: buat sketsa form (bertahap per kartu tab Profil/Keluarga, mengikuti field docs/10 dan field wajib dari Q-17), TUNGGU persetujuan, lalu createApplicant via RPC transaksional (applicants + tabel anak + nomor registrasi {reg_prefix}.{YYYYMMDD}.{seq}).
```

```text
[5.6] CRUD kelas, guru, psikolog, siswa & enrollment: dialog form memakai komponen yang ada (sketsa dulu, tunggu persetujuan), Server Actions role school_admin, validasi docs/18, test RLS untuk operasi baru.
```

```text
[5.7] /[school]/settings (kuota per level, nominal tuition, target, biaya formulir, ambang macet, TA admission aktif) dan /admin/users (daftar pengguna, undang via inviteUser, ubah role per sekolah). Sketsa dulu, tunggu persetujuan. Hanya school_admin / super admin.
```

---

## Fase 6 · Migrasi Odoo, keamanan & UAT

> 🔒 Akses Odoo **staging/salinan** (Q-02). Script hanya dijalankan ke Supabase **lokal atau staging**, tidak ke production.

### Prompt 6.1 · Script migrasi (dry-run) 🧭

```text
Baca docs/10 (pemetaan 120 field), docs/16 §Migrasi, ADR-001 & ADR-007, docs/24 Q-02/Q-15/Q-16.
Saya akan memberi: URL Odoo staging, nama database, user, dan daftar nama model & field teknis Odoo (lewat env, jangan commit).

Buat scripts/migrate-from-odoo/ (Node/TS):
1. Ekstrak via JSON-RPC: sekolah/company, TA, level, pendaftar + semua field, attachment, psikotes, psikolog, transaksi, kelas, siswa, guru.
2. Transformasi ke skema Supabase (mapping di satu file mapping.ts; parse skor IQ; isi stage_entered_at dari mail.tracking.value bila ada, selain itu write_date).
3. Mode --dry-run (default) yang hanya menulis laporan; mode --apply ke target dari env (tolak bila URL terlihat seperti production).
4. Idempoten (upsert dengan external_id Odoo).
5. Laporan: jumlah per sekolah × TA × flow (Odoo vs Supabase), headcount kelas vs enrollment, file (jumlah, ukuran, checksum), record ditolak + alasan, ejaan mencurigakan (Q-15).

JANGAN menjalankan --apply. Tunjukkan laporan dry-run.
```

### Prompt 6.2 · Apply ke staging

```text
Saya menyetujui laporan dry-run. Jalankan --apply ke Supabase STAGING (env yang saya siapkan), lalu jalankan ulang laporan rekonsiliasi.
Selesai bila: jumlah per sekolah × TA × flow sama dengan Odoo, atau setiap selisih dijelaskan.
```

### Prompt 6.3 · Audit keamanan

```text
Baca docs/19 seluruhnya. Audit codebase dan laporkan temuan per item OWASP di docs/19 §6, plus:
- grep service role / secret di kode client,
- semua tabel public punya RLS & test,
- semua Server Action memanggil requireSchoolRole + Zod,
- header keamanan & CSP (docs/19 §7),
- log tidak memuat PII,
- pnpm audit --prod.
Perbaiki temuan rendah/sedang langsung; temuan tinggi laporkan dulu.
```

### Prompt 6.4 · Persiapan UAT

```text
Buat docs/uat/UAT-CHECKLIST.md dari acceptance criteria docs/02 §4 dan skenario E1–E10 docs/21, berupa langkah yang bisa diikuti staf sekolah (bahasa non-teknis) untuk 2 sekolah (1 per brand) di staging, dengan kolom Lulus/Gagal/Catatan.
```

---

## Fase 7 · Performance

### Prompt 7.1

```text
Baca docs/20. Di staging (data migrasi):
1. explain analyze untuk listApplicants (berbagai filter), countApplicantsByFlow, RPC dashboard, detail; tambahkan index bila perlu (migration).
2. Ukur LCP/INP/CLS dengan Playwright + web-vitals untuk Dashboard, Pendaftar, Detail (desktop & mobile throttled).
3. Bundle analyzer: pastikan Recharts, dnd-kit, drawer lazy; first-load JS ≤ 200 KB/route.
Laporkan sebelum/sesudah per target docs/20.
```

---

## Fase 8 · Deployment

### Prompt 8.1 · Persiapan (tanpa eksekusi production)

```text
Baca docs/23. Siapkan (JANGAN eksekusi ke production):
1. .github/workflows/ci.yml sesuai docs/23 §5 (lint, typecheck, test, supabase start + db reset + test db, e2e + axe, build, audit).
2. Workflow deploy migration ke STAGING saat merge ke branch staging.
3. docs/runbooks/GO-LIVE.md: urutan langkah manual (buat proyek Supabase production region Singapore, set Auth, secrets, migration push oleh manusia, migrasi data final dengan freeze Odoo, Vercel env, domain, smoke test E1–E10, rencana rollback docs/23 §6).
4. .env.example lengkap tanpa nilai.
Laporkan apa yang harus saya lakukan secara manual.
```

---

## Prompt perbaikan

Gunakan prompt pendek yang menyebut **lokasi**, **masalah**, dan **hasil yang diharapkan**:

```text
Di [halaman/komponen], [masalah]. Seharusnya [hasil] sesuai prototipe design/Main.dc.html (blok …) dan docs/[nomor]. Ubah seminimal mungkin dan tambahkan test yang mencegah regresi.
```

Contoh:

```text
Di detail pendaftar mobile, tab pil tertimpa konten saat scroll. Seharusnya sticky dengan tinggi tetap 56px seperti design/Mobile.dc.html. Tambahkan test Playwright.
```

```text
Filter "Terlambat" menampilkan pendaftar Waiting List. Menurut docs/15 §3 Waiting List tidak pernah late. Cek lib/domain dan applicant_list_v, perbaiki keduanya, tambah kasus ke test paritas.
```

## Prompt review (sebelum merge)

```text
Review perubahan di branch ini terhadap CLAUDE.md, AGENTS.md, dan dokumen docs/ yang relevan.
Laporkan per temuan: file:baris, aturan yang dilanggar, dampak, usulan perbaikan. Fokus pada: penyimpangan dari prototipe, RLS/otorisasi, validasi, data sensitif, test yang hilang, dokumentasi yang belum diperbarui. Jangan mengubah kode.
```

## Prompt sinkronisasi dokumentasi

```text
Bandingkan implementasi saat ini dengan docs/05, 06, 10, 14, 15. Daftar semua perbedaan, lalu perbarui dokumentasi agar sesuai kenyataan (bukan sebaliknya), kecuali perbedaan yang merupakan bug; laporkan itu terpisah. Tambahkan entri CHANGELOG.
```

---

## Checklist akhir proyek

- [ ] Semua prompt Fase 0–7 selesai dengan "Selesai bila" terpenuhi
- [ ] Tampilan semua halaman sama dengan prototipe (desktop & mobile, light & dark, 2 brand)
- [ ] 119 tampilan field di detail pendaftar; 120 field Odoo termigrasi
- [ ] pgTAP RLS lulus untuk semua tabel × role
- [ ] Paritas SQL ↔ domain = 0 perbedaan
- [ ] Laporan migrasi: jumlah per sekolah × TA × flow cocok dengan Odoo
- [ ] Axe 0 serious/critical
- [ ] Target performa docs/20 tercapai
- [ ] UAT ditandatangani 2 sekolah
- [ ] Runbook go-live disetujui; production dieksekusi oleh manusia
