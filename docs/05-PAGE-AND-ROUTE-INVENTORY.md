# 05 · Page & Route Inventory

Prototipe tidak memakai URL; halaman berpindah lewat state `page`. Route di bawah adalah **REKOMENDASI** pemetaan ke Next.js App Router. Status implementasi saat ini: **semua belum diimplementasikan** (hanya prototipe).

## Halaman yang didesain

| # | Halaman | Route | Tujuan | Role | Komponen utama | Sumber prototipe |
|---|---|---|---|---|---|---|
| P1 | Ringkasan | `/[school]/dashboard` | KPI, funnel, tren, tindakan | Semua kecuali psikolog | KpiRow, FunnelChart, TrendChart, ActionPanel, LevelQuotaChart, SourceChart | `isDash` |
| P2 | Pendaftar | `/[school]/applicants` | Kelola pendaftar per tahap | officer, principal, finance, admin, viewer | DataToolbar, StageTabs, ApplicantTable, ApplicantCardGrid, KanbanBoard, BulkActionBar, Pager | `isList` |
| P3 | Detail pendaftar | `/[school]/applicants?detail=<id>` (slide-over); fallback `/[school]/applicants/[id]` | Semua data 1 pendaftar | sama dengan P2 | ApplicantDrawer, 7 tab | `hasDetail`, `appForm`, `appView` |
| P4 | Transaksi | `/[school]/transactions` | Pembayaran & rekonsiliasi | finance, admin, officer (R) | DataToolbar, TransactionTable/Card | `isTrx` |
| P5 | Psikotes | `/[school]/psychology` | Hasil tes | officer, principal, admin, psikolog | PsychologyTable/Card, PsychologyDrawer | `isPsy`, `xd` |
| P6 | Detail psikotes | `/[school]/psychology?detail=<id>` | Hasil tes 1 siswa | sama dengan P5 | PsychologyDrawer | `hasXD` |
| P7 | Psikolog | `/[school]/psychologists` | Mitra psikolog | admin, officer, principal | PsychologistCard/Table | `isPsyc` |
| P8 | Kelas | `/[school]/classes` | Rombel per TA | admin, officer, principal | ClassTable/Card | `isClass` |
| P9 | Siswa | `/[school]/students` | Siswa terdaftar | admin, officer, principal | StudentTable/Card | `isStudent` |
| P10 | Guru | `/[school]/teachers` | Guru & staf | admin, officer, principal | TeacherCard/Table | `isTeacher` |

`[school]` = `schools.code` (mis. `smp-tj-bdg`). REKOMENDASI: sekolah di path, bukan query, karena memengaruhi semua data dan brand (lihat [ADR-003](25-TECHNICAL-DECISIONS.md#adr-003--sekolah-aktif-sebagai-segmen-route)).

### Mobile (`design/Mobile.dc.html`)

| Layar | Setara desktop | Catatan |
|---|---|---|
| Ringkasan | P1 | 4 KPI, tuition, funnel, 3 macet |
| Pendaftar | P2 | Kartu, filter bottom sheet, mode Pilih |
| Tindakan | Panel "Perlu tindakan" P1 | Segmented Macet / Jadwal / Pembayaran |
| Detail | P3 | Layar penuh, tab pil sticky |
| P4–P10 | — | **Belum didesain untuk mobile** |

### State global (semua halaman)

Loading, kosong, tidak ada hasil, error ("Gagal memuat data"), tanpa hak akses. Sumber: `showLoading`, `listEmpty`, `dashEmpty`, `showError`, `showNoAccess`.

## Halaman yang dibutuhkan tetapi belum didesain

| # | Halaman | Route usulan | Alasan | Status |
|---|---|---|---|---|
| N1 | Login | `/login` | Supabase Auth | `DECISION REQUIRED` desain (usul: pakai token & logo dari 04) |
| N2 | Lupa password | `/forgot-password` | Reset via email | Belum didesain |
| N3 | Reset password | `/reset-password` | Target link email | Belum didesain |
| N4 | Auth callback | `/auth/callback` | Exchange code PKCE | Tanpa UI |
| N5 | Pilih sekolah awal | `/` | Redirect ke sekolah default pengguna | Tanpa UI (redirect) |
| N6 | Tanpa akses | `/[school]/…` state | Sudah ada sebagai state | ✅ didesain |
| N7 | Pendaftar baru | `/[school]/applicants/new` | Tombol "Pendaftar baru" | Belum didesain |
| N8 | Edit pendaftar | dalam P3 (mode edit per kartu) | Field perlu diisi | Belum didesain |
| N9 | Input hasil psikotes | `/[school]/psychology/new` | Tombol "Input hasil tes" | Belum didesain |
| N10 | Tambah kelas / guru / psikolog | dialog/route `new` | Tombol "Tambah …" | Belum didesain |
| N11 | Pengaturan sekolah | `/[school]/settings` | Kuota, target, ambang macet, tahun ajaran aktif | Belum didesain |
| N12 | Manajemen pengguna | `/admin/users` | Role per sekolah | Belum didesain |
| N13 | Profil saya | `/account` | Ganti password, nama | Belum didesain |
| N14 | 404 / error global | `not-found.tsx`, `error.tsx` | Next.js | Pakai EmptyState/ErrorState |
| N15 | Riwayat level (tab) | dalam P3 | Field belum diketahui | `OPEN QUESTION` Q-08 |
