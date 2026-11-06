# 24 · Open Questions

Prioritas: 🔴 memblokir Fase 2 · 🟠 memblokir Fase 4–5 · 🟢 bisa ditunda

## Pertanyaan

| ID | Prio | Pertanyaan | Dampak | Asumsi sementara |
|---|---|---|---|---|
| Q-01 | 🔴 | Bagaimana data pendaftar masuk? Lewat formulir publik orang tua (aplikasi `admission.kreativaglobal.sch.id`?) atau diinput staf? Apakah formulir publik ikut pindah ke Supabase? **Catatan:** teks prototipe menyebut "orang tua mengisi formulir online" dan "tambahkan pendaftar secara manual dari data leads Open House" | Skema insert, RLS anon, roadmap | Hanya staf yang menginput; formulir publik di luar scope |
| Q-02 | 🔴 | Akses & volume data Odoo untuk migrasi (DB, attachment, jumlah file), cutover kapan? | Fase 6 | Migrasi penuh, satu kali, dengan freeze |
| Q-03 | 🔴 | Daftar nilai resmi `acceptance_status` & aturan transisi `flow`. Apakah "Diterima" = flow tuition saja? | Enum, KPI, RPC | Lihat [15 §1](15-BUSINESS-LOGIC.md#1-tahap-flow) |
| Q-04 | 🟠 | Apakah psikolog eksternal perlu login sendiri? | Role, RLS | Tidak, staf yang menginput |
| Q-05 | 🟠 | Provider WhatsApp (Qontak, Wablas, Meta Cloud API?) & email (SMTP sekolah, Resend?) | Edge Function | Belum dipilih |
| Q-06 | 🟢 | Apakah transaksi dibuat otomatis oleh payment gateway (VA/QRIS)? Gateway apa? (Odoo "Id Via Bayar" = bank) | Webhook | Hanya pencatatan/impor |
| Q-07 | 🔴 | Nominal tuition per level atau per pendaftar? Kapasitas kelas & kuota per level: dari mana? Target tuition per sekolah? | `level_quotas`, `classes.capacity` | Angka di prototipe adalah contoh |
| Q-08 | 🟠 | Field apa saja di tab **Level History** Odoo? | `student_level_history` | Tabel draf minimal |
| Q-09 | 🟢 | Aspek psikotes (verbal, numerik, dll.) benar-benar dicatat? Rekomendasi diinput psikolog atau dihitung dari IQ? | `psychology_tests` | Tidak ada kolom aspek; rekomendasi diinput |
| Q-10 | 🟢 | Kontras teks putih di sidebar TJ `#00A7E1` (2,7:1) diterima, atau digelapkan ke `#0077A3`? | Token | Dipertahankan sesuai desain |
| Q-11 | 🟢 | Status Online guru: dari apa? (aktivitas login aplikasi ini?) | `profiles.last_seen_at` | Dari aktivitas aplikasi |
| Q-12 | 🟠 | Versi Next.js, React, @supabase/ssr yang akan dipakai (belum ada package.json) | Setup | Versi stabil terbaru saat Fase 1 |
| Q-13 | 🔴 | Di Odoo ada field **Company** dan **School** terpisah. Apakah bisa berbeda (mis. company yayasan, school unit)? | `schools`, `school_groups` | Sama dengan unit sekolah |
| Q-14 | 🟢 | Apakah staf memakai Google Workspace (untuk SSO)? | Auth | Email + password |
| Q-15 | 🟢 | Ejaan kelas yang berbeda di Odoo ("9 Ibu Majid" vs "9 Ibnu Majid", "Ibnu Batuta" vs "Battuta", "Wahyudin" vs "Wahyudi"): dirapikan saat migrasi? | Data | Disimpan apa adanya + dilaporkan |
| Q-16 | 🟠 | Daftar nilai Odoo untuk: Info (sumber), Religion, Father/Mother Job, Education, Income, Housing Type, Transport, Student Status | Enum vs text | Text bebas + daftar pilihan di UI |
| Q-17 | 🟠 | Field wajib saat membuat pendaftar & field wajib per tahap | Validasi | Belum ada |
| Q-18 | 🟢 | Retensi data pendaftar yang tidak diterima & log notifikasi | Kebijakan PDP | 1 tahun untuk log |
| Q-19 | 🟢 | Breakpoint pasti antara layout desktop & mobile | UI | ≤ 640 px = mobile |
| Q-20 | 🔴 | Definisi "Tidak dilanjutkan": TA berakhir (`ends_on`) atau TA tidak lagi membuka pendaftaran (`is_admission_open`)? Prototipe menganggap hanya 2027/2028 & 2028/2029 aktif, sehingga 2026/2027 sudah stale. TA mana default filter? | 15 §3, `applicant_list_v`, filter, KPI | Ikuti prototipe: stale = TA tanpa `is_admission_open` |
| Q-21 | 🔴 | Format no. registrasi pendaftar: prototipe memakai `REG-YYYY-NNNN` (daftar, header, cari) **dan** `{kode}.{YYYYMMDD}.{seq}` (tab Seleksi, Transaksi) | `applicants.registration_number`, pencarian, migrasi | Format Odoo `{reg_prefix}.{YYYYMMDD}.{seq}` |
| Q-22 | 🔴 | Matriks izin final: konflik 03 ↔ 13/14/05/12 (K-11). Mekanisme principal mengubah tahap & keputusan (RPC security definer?) | RLS, RPC, test R7 | Belum ada |
| Q-23 | 🔴 | Tahap yang dipakai per sekolah (FR-20): konfigurasi tersimpan atau turunan data? Dialog pindah tahap menyembunyikan tahap yang tidak dipakai? | Skema (kolom/tabel baru), dialog | Turunan data |
| Q-24 | 🟠 | Pendaftar ditolak: tahap apa? Apakah `acceptance_status = rejected` dipakai (tidak ada di prototipe)? | Enum, funnel | Belum ada |
| Q-25 | 🟠 | Riwayat tahap (FR-30) ditampilkan di mana? Dialog menyebut "dicatat di timeline" tetapi 7 tab tidak punya timeline | Detail | Sketsa kartu di Ringkasan |
| Q-26 | 🟠 | Letak tombol logout & "Profil saya" (footer sidebar hanya nama + email) | Shell, N13 | Menu di footer sidebar |
| Q-27 | 🟠 | Teks state error/tanpa akses menyebut "Odoo" dan "grup Admission / Officer"; desktop ≠ mobile. Boleh disesuaikan? | Teks UI | Belum ada |
| Q-28 | 🟢 | Filter mobile hanya 6 status, tanpa rentang/petugas, tanpa email/export massal: disengaja? | Mobile | Ikuti prototipe |
| Q-29 | 🟢 | Angka menu sidebar: semua TA (prototipe) atau TA aktif (`nav_counts(p_year)`)? | RPC `nav_counts` | Ikuti prototipe |
| Q-30 | 🟢 | Pendaftar stale boleh dipindah tahap? Prototipe mengaktifkannya kembali | Transisi | Belum ada |
| Q-31 | 🟢 | Persen kelengkapan memakai dasar 107 field (prototipe; Informasi pendaftaran tidak dihitung) atau 119? | Domain, E6 | 107 |
| Q-32 | 🟢 | Legenda grafik level "bar ungu / merah muda" hanya cocok untuk KGS; ganti teks netral? | Teks UI | Belum ada |

## Konflik / ketidakkonsistenan yang ditemukan

| ID | Temuan | Rekomendasi |
|---|---|---|
| K-01 | Brief awal: "data tetap di Odoo"; keputusan sekarang: **Supabase penuh** | Dokumentasi lama (`docs/legacy`) bagian Odoo tidak berlaku (ADR-001) |
| K-02 | Prompt dokumentasi menyebut UI berupa TSX; kenyataannya prototipe `.dc.html` | UI di-port ulang ke TSX di Fase 1, mengikuti prototipe |
| K-03 | NISN muncul di 2 tab form Odoo | Satu kolom (sudah diputuskan di prototipe) |
| K-04 | Status Psikotes/Wawancara ada di form pendaftar **dan** model Psychology terpisah | Sumber tunggal: `psychology_tests` / `parent_interviews` |
| K-05 | Prototipe: pindah tahap mengubah status psikotes/wawancara | REKOMENDASI hanya ubah `flow` (Q-03) |
| K-06 | Rekomendasi psikotes di prototipe dihitung dari IQ ≥ 95; di Odoo berupa teks bebas | Q-09 |
| K-07 | Tren pembanding TA sebelumnya di prototipe memakai angka acak | Wajib data asli |
| K-08 | Mobile belum punya halaman Transaksi … Guru | `DECISION REQUIRED`: desain atau pakai menu "Lainnya" + kartu |
| K-09 | Tidak ada desain login, form tambah/edit, pengaturan, manajemen pengguna | `DECISION REQUIRED` desain sebelum Fase 3/5 |
| K-10 | Odoo list Transaction kolom "Id Transaction" berisi nama siswa | Q-02: cek makna sebenarnya |
| K-11 | Matriks 03 bertentangan dengan 13/14/05/12: principal ubah tahap (RLS menolak, R7 gagal), viewer psikotes, finance guru/psikolog, delete transaksi, rekonsiliasi school_admin, officer hapus dokumen & ubah psikotes, export viewer, delete role, guard settings | Q-22 |
| K-12 | `storage_path` di 10 berprefiks nama bucket, sedangkan policy di 16 membandingkan dengan `storage.objects.name` (tanpa bucket) | Diperbaiki di 10: path tanpa prefiks bucket |
| K-13 | Pindah ke tahap sama: 14 `CONFLICT` vs 15 §9 "diabaikan" | Diperbaiki di 14: diabaikan, hasil = jumlah yang berubah |
| K-14 | `yearId` uuid (14) vs param URL `year` = nama TA (17) | URL pakai nama TA, dipetakan ke id di `lib/data` |
| K-15 | Pindah tahap tidak mengirim notifikasi ke orang tua (teks prototipe), belum tercatat di 08/15 | Tambahkan ke 15 §1 |

## Keputusan yang dibutuhkan (ringkas)

1. Q-01, Q-02, Q-03, Q-07, Q-13 sebelum Fase 2.
2. Desain: login, form pendaftar/edit, input psikotes, pengaturan sekolah, manajemen pengguna, halaman modul pendukung versi mobile.
3. Provider WhatsApp & email.
4. Role final & matriks izin ([03](03-USER-ROLES-AND-PERMISSIONS.md)).
