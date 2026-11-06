# 01 · Project Overview

## Deskripsi

**Edu Hub · Modul Admission** adalah aplikasi web internal untuk mengelola penerimaan siswa baru (PSB) di dua grup sekolah:

- **Kreativa Global School** (4 unit: Kindergarten, Primary, Secondary, High School)
- **Sekolah Talenta Juara** (5 unit: SD Bandung, SD Cimahi, SD Pekanbaru, SMP Bandung, SMP Pekanbaru)

Aplikasi mencakup alur pendaftar dari *Leads* sampai *Tuition lunas*, serta data pendukung: transaksi pembayaran, hasil psikotes, psikolog mitra, kelas, siswa, dan guru.

## Latar belakang

| | |
|---|---|
| **FAKTA** | Sistem lama adalah modul "Edu Hub" di Odoo 16 dengan list view berisi 5 kolom status, semuanya merah "Incomplete" |
| **FAKTA** | Prototipe redesign dikembangkan melalui 16 iterasi bersama pemilik produk (lihat [CHANGELOG](CHANGELOG.md)) |
| **KEPUTUSAN** | Backend baru memakai **Supabase penuh**, menggantikan Odoo (lihat [ADR-001](25-TECHNICAL-DECISIONS.md#adr-001--supabase-penuh-menggantikan-odoo)) |

## Problem statement

1. Tim admission tidak bisa melihat prioritas, karena semua status tampil merah tanpa perbedaan antara "belum" dan "bermasalah".
2. Kolom *Flow* dan *Acceptance Status* tumpang tindih sehingga membingungkan.
3. Tidak ada ringkasan konversi, pendaftar yang macet, atau sisa kuota.
4. Data tidak valid (nama siswa kosong, tuition Rp 0) tidak ditandai.
5. Banyak orang tua bertanya status lewat WhatsApp karena tim admission sulit menindaklanjuti secara proaktif.

## Solusi

| Masalah | Solusi di prototipe |
|---|---|
| Status tanpa prioritas | Progress stepper 5 titik, warna semantik, merah hanya untuk masalah |
| Tidak ada insight | Dashboard: KPI, funnel, tren, "Perlu tindakan", distribusi |
| Data tidak valid | Penanda ⚠, filter "Data bermasalah", penghitung kelengkapan field |
| Tindak lanjut lambat | Aksi utama per tahap, aksi massal (pengingat WhatsApp/email), daftar macet |
| Multi-sekolah | Dropdown pindah sekolah + tema brand otomatis |

## Target pengguna

| Pengguna | Sumber | Kebutuhan |
|---|---|---|
| Tim admission / officer | **FAKTA** (brief, UI "Admin Admission") | Proses pendaftar per tahap, lengkapi data & dokumen |
| Kepala sekolah | **FAKTA** (brief) | Pantau pendaftar, konversi, kuota |
| Tim keuangan | **FAKTA** (brief) | Pantau pembayaran & rekonsiliasi |
| Orang tua / calon siswa | **ASUMSI**: pengisi formulir, bukan pengguna aplikasi ini | Lihat `OPEN QUESTION` Q-01 |

## Ruang lingkup

### Termasuk (berdasarkan prototipe)

- Dashboard Ringkasan per sekolah & tahun ajaran
- Pendaftar: daftar (tabel/kartu/kanban), filter, pencarian, aksi massal, ubah tahap
- Detail pendaftar: 7 tab, 120 field form
- Transaksi, Psikotes (+ detail), Psikolog, Kelas, Siswa, Guru
- Multi-sekolah dengan dua brand, mode terang/gelap, desktop & mobile

### Dibutuhkan tetapi belum didesain (lihat [05](05-PAGE-AND-ROUTE-INVENTORY.md#halaman-yang-dibutuhkan-tetapi-belum-didesain))

- Login, lupa/reset password, halaman tanpa akses
- Form tambah/edit pendaftar, input hasil psikotes, tambah kelas/guru/psikolog
- Upload dokumen sungguhan, export Excel sungguhan
- Pengiriman WhatsApp/email
- Manajemen pengguna & role
- Pengaturan sekolah: kuota, target, ambang "macet"

### Di luar ruang lingkup

- Portal orang tua (formulir publik) **kecuali** diputuskan lain (Q-01)
- Akademik penuh (nilai, absensi, jadwal)
- Payment gateway (hanya pencatatan transaksi, kecuali diputuskan lain, Q-06)

## Teknologi

| Lapisan | Teknologi | Status |
|---|---|---|
| Framework | Next.js (App Router), TypeScript strict | REKOMENDASI |
| UI | Tailwind CSS, shadcn/ui, lucide-react | REKOMENDASI |
| Tabel / data | TanStack Table, TanStack Query (opsional, lihat ADR-004) | REKOMENDASI |
| Chart | Recharts | REKOMENDASI |
| Drag & drop | dnd-kit | REKOMENDASI |
| Validasi | Zod | REKOMENDASI |
| Backend | Supabase: Postgres, Auth, Storage, RLS, Edge Functions | KEPUTUSAN |
| Testing | Vitest, Testing Library, Playwright, pgTAP | REKOMENDASI |
| Hosting | Vercel + Supabase Cloud | REKOMENDASI |
| Font | Source Sans 3 | FAKTA (prototipe) |
