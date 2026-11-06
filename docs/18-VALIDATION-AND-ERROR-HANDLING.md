# 18 · Validation & Error Handling

## 1. Lapisan validasi

| Lapisan | Alat | Tujuan |
|---|---|---|
| Client | Zod schema yang sama + react-hook-form (form) | Umpan balik cepat |
| Server Action | Zod `safeParse` (**wajib**, client tidak dipercaya) | Keamanan & integritas |
| Database | `check`, `not null`, `unique`, FK, trigger konsistensi | Pertahanan terakhir |

## 2. Aturan validasi field

| Field | Aturan | Pesan |
|---|---|---|
| Email | format email | "Format email tidak valid" |
| WhatsApp | dinormalisasi ke `62…`, 10–15 digit | "Nomor WhatsApp tidak valid" |
| NIK, No. KK | 16 digit | "NIK harus 16 digit" |
| NISN | 10 digit | "NISN harus 10 digit" |
| NPSN | 8 digit | "NPSN harus 8 digit" |
| Kode pos | 5 digit | "Kode pos harus 5 digit" |
| Tanggal lahir | ≤ hari ini, usia 2–20 tahun | "Tanggal lahir tidak wajar" |
| Nominal (biaya, tuition, transaksi) | ≥ 0, maks 2 desimal | "Nominal tidak valid" |
| Skor IQ | 40–200 | "Skor IQ di luar rentang" |
| Kapasitas kelas | 1–60 | |
| Ambang macet | 1–60 hari | |
| Pencarian | trim, ≤ 100 karakter, escape `%` `_` | — |
| File | lihat [16](16-FILE-STORAGE.md) | "Format harus PDF/JPG/PNG, maks 10 MB" |
| Ubah tahap massal | 1–200 id | "Pilih maksimal 200 pendaftar" |

Field wajib saat membuat pendaftar baru: `DECISION REQUIRED` (form belum didesain). Data Odoo menunjukkan `student_name` bisa kosong, jadi DB mengizinkan null, tetapi form baru REKOMENDASI mewajibkannya.

## 3. Error handling

| Sumber | Penanganan | Tampilan |
|---|---|---|
| Validasi | `ActionResult { ok:false, code:'VALIDATION', fieldErrors }` | Pesan di bawah field |
| Tidak login | middleware redirect | `/login?next=` |
| Tidak punya role | `FORBIDDEN` / layout | `NoAccessState` |
| Tidak ditemukan | `notFound()` | `EmptyState` / toast di drawer |
| Konflik (no. registrasi ganda, tahap sama) | `CONFLICT` | Toast |
| Postgres RLS `42501` | dipetakan ke `FORBIDDEN` | Toast |
| Jaringan / Supabase down | `error.tsx` | `ErrorState` "Gagal memuat data" + Coba lagi |
| Mutasi optimistic gagal | rollback | Toast error |
| Upload gagal | — | Status di kotak dokumen + Coba lagi |
| Kirim WA/email gagal | `notification_logs.status = failed` | Toast "N dari M gagal terkirim" |

Pesan error untuk pengguna selalu dalam Bahasa Indonesia, tanpa detail teknis atau stack trace.

## 4. Feedback pengguna

| Kejadian | Feedback |
|---|---|
| Aksi sukses | Toast (`role="status"`) ±2,8 detik, mis. *"Siti Aisyah dipindahkan ke tahap Payment"* |
| Aksi berjalan | Tombol disabled + spinner |
| Aksi destruktif | Dialog konfirmasi |
| Loading halaman | Skeleton sesuai pola |

## 5. Logging

| Apa | Di mana | Aturan |
|---|---|---|
| Error server | Vercel logs / Sentry (REKOMENDASI, `DECISION REQUIRED`) | **Tanpa PII**: jangan log nama siswa, NIK, KK, nomor telepon, isi form |
| Perubahan data penting | `audit_logs` (nama kolom saja) | Perubahan tahap ada di `applicant_stage_history` |
| Pengiriman notifikasi | `notification_logs` | Simpan penerima (dibutuhkan untuk audit); retensi 1 tahun (`DECISION REQUIRED`) |
