# 03 · User Roles & Permissions

## Temuan dari UI

| Temuan | Sumber | Status |
|---|---|---|
| Satu profil pengguna "Admin Admission" di sidebar | Prototipe: footer sidebar | FAKTA |
| Pengguna bisa berpindah antar 9 sekolah dari 2 grup | Dropdown sekolah | FAKTA |
| State "Tanpa hak akses": *"Akun Anda tidak termasuk grup Admission / Officer untuk {sekolah}"* | `showNoAccess` | FAKTA, menunjukkan hak akses **per sekolah** |
| Pengguna: tim admission, kepala sekolah, tim keuangan | Brief awal | FAKTA |
| Tidak ada perbedaan tampilan per role di prototipe | — | FAKTA |

## Role yang diusulkan (REKOMENDASI, `DECISION REQUIRED`)

Role diberikan **per sekolah** (tabel `user_school_roles`). Satu pengguna bisa punya role berbeda di sekolah berbeda.

| Role | Kode | Deskripsi |
|---|---|---|
| Super admin | `super_admin` | Akses semua sekolah, kelola pengguna & pengaturan. Flag global di `profiles.is_super_admin` |
| Admin sekolah | `school_admin` | Kelola pengaturan & pengguna di sekolahnya |
| Officer admission | `admission_officer` | Proses pendaftar: ubah tahap, edit data, upload dokumen, kirim pengingat |
| Kepala sekolah | `principal` | Baca semua data sekolahnya, input keputusan penerimaan |
| Keuangan | `finance` | Baca pendaftar, kelola transaksi & rekonsiliasi |
| Psikolog | `psychologist` | *(opsional, Q-04)* Input hasil psikotes pendaftar yang ditugaskan |
| Viewer | `viewer` | Baca saja |

## Permission matrix

Legenda: **R** baca · **C** buat · **U** ubah · **D** hapus · — tidak ada akses

| Resource | super_admin | school_admin | admission_officer | principal | finance | psychologist | viewer |
|---|---|---|---|---|---|---|---|
| Dashboard | R | R | R | R | R | — | R |
| Pendaftar (list & detail) | CRUD | CRUD | CRU | R | R | R (yang ditugaskan) | R |
| Ubah tahap (`flow`) | ✓ | ✓ | ✓ | ✓ (Keputusan) | — | — | — |
| Keputusan penerimaan | ✓ | ✓ | — | ✓ | — | — | — |
| Data sensitif (NIK, KK, kesehatan, penghasilan) | R/U | R/U | R/U | R | — | — | — |
| Dokumen pendaftar | CRUD | CRUD | CRU | R | R (bukti bayar) | — | — |
| Kirim pengingat WA/email | ✓ | ✓ | ✓ | — | ✓ (tagihan) | — | — |
| Export Excel | ✓ | ✓ | ✓ | ✓ | ✓ | — | — |
| Transaksi | CRUD | CRUD | R | R | CRU | — | R |
| Rekonsiliasi | ✓ | ✓ | — | — | ✓ | — | — |
| Psikotes | CRUD | CRUD | CR | R | — | CRU (yang ditugaskan) | R |
| Psikolog | CRUD | CRUD | R | R | — | R (diri sendiri) | R |
| Kelas, siswa | CRUD | CRUD | R | R | R | — | R |
| Guru | CRUD | CRUD | R | R | — | — | R |
| Pengaturan sekolah (kuota, target, ambang) | CRUD | CRU | R | R | R | — | — |
| Pengguna & role | CRUD | CRU (sekolahnya) | — | — | — | — | — |
| Audit log | R | R (sekolahnya) | — | — | — | — | — |

## Batasan akses

1. Semua query dibatasi `school_id IN (sekolah yang diizinkan)`, ditegakkan oleh RLS ([13](13-ROW-LEVEL-SECURITY.md)), bukan hanya oleh UI.
2. Field sensitif dipisah ke tabel tersendiri (`applicant_family_members`, `applicant_government_data`, kolom kesehatan) agar bisa dibatasi per role. `DECISION REQUIRED` apakah keuangan boleh melihat penghasilan orang tua.
3. Psikolog eksternal hanya melihat pendaftar yang ditugaskan kepadanya (jika role ini dipakai).
4. UI menyembunyikan aksi yang tidak diizinkan, tetapi server tetap memvalidasi ulang.
5. Pengguna tanpa role di sekolah aktif melihat state **Tanpa hak akses** (sudah didesain).
