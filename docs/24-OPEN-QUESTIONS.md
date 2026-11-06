# 24 · Open Questions

Prioritas: 🔴 memblokir Fase 2 · 🟠 memblokir Fase 4–5 · 🟢 bisa ditunda

## Pertanyaan

| ID | Prio | Pertanyaan | Dampak | Asumsi sementara |
|---|---|---|---|---|
| Q-01 | 🔴 | Bagaimana data pendaftar masuk? Lewat formulir publik orang tua (aplikasi `admission.kreativaglobal.sch.id`?) atau diinput staf? Apakah formulir publik ikut pindah ke Supabase? | Skema insert, RLS anon, roadmap | Hanya staf yang menginput; formulir publik di luar scope |
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

## Keputusan yang dibutuhkan (ringkas)

1. Q-01, Q-02, Q-03, Q-07, Q-13 sebelum Fase 2.
2. Desain: login, form pendaftar/edit, input psikotes, pengaturan sekolah, manajemen pengguna, halaman modul pendukung versi mobile.
3. Provider WhatsApp & email.
4. Role final & matriks izin ([03](03-USER-ROLES-AND-PERMISSIONS.md)).
