# 02 · Product Requirements Document

## 1. Tujuan produk

Membuat tim admission tahu **siapa yang harus ditindaklanjuti hari ini**, dan membuat manajemen tahu **kondisi penerimaan** per sekolah dan tahun ajaran.

**Metrik keberhasilan (REKOMENDASI, `DECISION REQUIRED` untuk target angka):**

| Metrik | Definisi |
|---|---|
| Rata-rata lama di tahap | Hari rata-rata pendaftar berada di satu tahap |
| % pendaftar macet | Pendaftar aktif dengan lama di tahap > ambang (default 7 hari) |
| Konversi Leads → Diterima | Pendaftar tahap Tuition ÷ total pendaftar TA |
| Kelengkapan data | % field terisi per pendaftar |

## 2. Functional requirements

Kode `FR-x` dipakai di roadmap dan test.

### 2.1 Shell & multi-sekolah

| ID | Requirement | Sumber |
|---|---|---|
| FR-01 | Pengguna memilih sekolah aktif dari dropdown berkelompok (2 grup, 9 unit) yang menampilkan jumlah pendaftar per unit | Prototipe: sidebar `toggleSchoolMenu`, `schoolGroups` |
| FR-02 | Logo, warna sidebar, dan warna aksen berganti sesuai brand grup sekolah aktif | `brandClass` (`b-tj` / `b-kgs`) |
| FR-03 | Hanya sekolah yang diizinkan untuk pengguna yang muncul di dropdown | REKOMENDASI (RLS) |
| FR-04 | Sidebar menampilkan menu berkelompok + jumlah data, pintasan Macet / Data bermasalah / Waiting List / Invalid | `navGroups`, `extraShortcuts` |
| FR-05 | Top bar: buka-tutup sidebar, breadcrumb, cari pendaftar, notifikasi jumlah macet, toggle tema | `toggleSide`, `goSearch`, `goLate`, `toggleDark` |

### 2.2 Dashboard

| ID | Requirement |
|---|---|
| FR-10 | Pilih tahun ajaran |
| FR-11 | 5 KPI: total pendaftar, baru 7 hari (vs 7 hari sebelumnya), konversi, tuition terkumpul vs target, sisa kuota |
| FR-12 | Funnel 5 tahap dengan drop-off dan penanda drop-off terbesar; Invalid dikecualikan |
| FR-13 | Tren mingguan (12 minggu) / harian (30 hari) dibanding TA sebelumnya |
| FR-14 | Panel Perlu tindakan: macet (5 teratas), jadwal psikotes/wawancara terdekat, pembayaran tertunda |
| FR-15 | Distribusi per level vs kuota, sumber pendaftaran + % konversi |

### 2.3 Pendaftar

| ID | Requirement |
|---|---|
| FR-20 | Tab tahapan dengan jumlah (hanya tahap yang dipakai sekolah) |
| FR-21 | Pencarian nama siswa / orang tua / no. registrasi (debounce) |
| FR-22 | Filter TA, level, status (9 opsi), rentang tanggal, petugas; reset filter |
| FR-23 | Tampilan Tabel (10/halaman), Kartu (12/halaman), Kanban |
| FR-24 | Sort berdasarkan lama di tahap dan tanggal daftar |
| FR-25 | Progress stepper 5 titik dengan tooltip |
| FR-26 | Penanda data bermasalah |
| FR-27 | Aksi utama per baris sesuai tahap |
| FR-28 | Aksi massal: pengingat WhatsApp, email, ubah tahap, export Excel |
| FR-29 | Kanban drag & drop memindahkan tahap dengan konfirmasi |
| FR-30 | Ubah tahap tercatat di riwayat beserta petugas |

### 2.4 Detail pendaftar

| ID | Requirement |
|---|---|
| FR-40 | Panel slide-over dengan header, aksi utama, WhatsApp, ubah tahap, stepper besar |
| FR-41 | 7 tab: Ringkasan, Profil siswa, Seleksi & pembayaran, Dokumen, Keluarga, Dapodik, Riwayat level |
| FR-42 | Menampilkan seluruh 120 field form tanpa menambah/mengurangi ([10-DATABASE-SCHEMA](10-DATABASE-SCHEMA.md#pemetaan-120-field)) |
| FR-43 | Penghitung "x/y terisi" per tab, ringkasan kelengkapan, daftar "Perlu dilengkapi" |
| FR-44 | Filter "Belum diisi saja" |
| FR-45 | Unggah/ganti 9 jenis dokumen |
| FR-46 | Navigasi "Kembali ke …" berbasis riwayat |
| FR-47 | **Edit field** (`DECISION REQUIRED`: prototipe hanya menampilkan data; edit belum didesain) |

### 2.5 Modul pendukung

| ID | Requirement |
|---|---|
| FR-50 | Transaksi: KPI, tab status, filter metode, kolom rekonsiliasi |
| FR-51 | Psikotes: KPI, tab rekomendasi, skor IQ, panel detail, link ke detail pendaftar |
| FR-52 | Psikolog: KPI, tab, flag data tidak lengkap |
| FR-53 | Kelas: KPI, tab level, filter TA, kapasitas, wali kelas, "Lihat siswa" |
| FR-54 | Siswa: KPI, tab level, filter status |
| FR-55 | Guru: KPI, tab jabatan, kelas yang diampu, status online |
| FR-56 | Semua halaman data punya pilihan tampilan Tabel / Kartu |

### 2.6 Dibutuhkan agar aplikasi berfungsi (belum didesain)

| ID | Requirement | Status |
|---|---|---|
| FR-90 | Login email + password, logout, lupa & reset password | `DECISION REQUIRED` desain |
| FR-91 | Manajemen pengguna & penugasan role per sekolah | `DECISION REQUIRED` |
| FR-92 | Form pendaftar baru & edit | `DECISION REQUIRED` |
| FR-93 | Input hasil psikotes & catatan psikolog | `DECISION REQUIRED` |
| FR-94 | CRUD kelas, guru, psikolog | `DECISION REQUIRED` |
| FR-95 | Pengaturan sekolah: kuota per level, target tuition, ambang macet | `DECISION REQUIRED` |
| FR-96 | Integrasi kirim WhatsApp & email | `OPEN QUESTION` Q-05 |
| FR-97 | Migrasi data dari Odoo | `OPEN QUESTION` Q-02 |

## 3. Non-functional requirements

| ID | Kategori | Requirement |
|---|---|---|
| NFR-01 | Bahasa | UI Bahasa Indonesia, tanggal `26 Okt 2026, 15:47` (Asia/Jakarta), Rupiah `Rp 12.500.000` |
| NFR-02 | Aksesibilitas | WCAG 2.1 AA; status tidak hanya warna; navigasi keyboard penuh |
| NFR-03 | Responsive | Desktop 1440, tablet 1024, mobile 375–390 px tanpa scroll horizontal |
| NFR-04 | Tema | Light & dark untuk kedua brand |
| NFR-05 | Keamanan | RLS di semua tabel; data hanya untuk sekolah yang diizinkan |
| NFR-06 | Performa | Lihat [20-PERFORMANCE-REQUIREMENTS](20-PERFORMANCE-REQUIREMENTS.md) |
| NFR-07 | Privasi | Data anak & keluarga (NIK, KK, kesehatan, penghasilan) dilindungi, tidak masuk log |
| NFR-08 | Audit | Perubahan tahap & data penting tercatat (siapa, kapan, apa) |
| NFR-09 | URL | Filter/tab/view/halaman/detail bisa dibagikan lewat URL |

## 4. User stories & acceptance criteria

### US-01 · Menindaklanjuti pendaftar macet
**Sebagai** officer admission, **saya ingin** melihat pendaftar yang tertahan lebih dari 7 hari **agar** bisa menghubungi orang tuanya.

- Diberikan ada pendaftar aktif dengan lama di tahap > 7 hari, ketika saya membuka dashboard, maka panel "Macet" menampilkan 5 pendaftar terlama dengan badge merah "N hari".
- Ketika saya klik "Lihat semua yang macet", maka halaman Pendaftar terbuka dengan filter status = Terlambat.
- Pendaftar Waiting List, Invalid, TA lewat, dan yang tuition-nya lunas **tidak** pernah muncul sebagai macet.

### US-02 · Memindahkan tahap
**Sebagai** officer, **saya ingin** memindahkan pendaftar ke tahap berikutnya **agar** status selalu akurat.

- Ketika saya drag kartu kanban ke kolom lain atau memilih "Ubah tahap", maka dialog konfirmasi muncul dengan tahap tujuan.
- Setelah konfirmasi, `flow` berubah, "lama di tahap" kembali ke 0, riwayat tercatat dengan nama saya, dan toast konfirmasi muncul.
- Bila gagal, tampilan kembali ke kondisi semula dan pesan error muncul.

### US-03 · Melengkapi data pendaftar
**Sebagai** officer, **saya ingin** tahu field mana yang belum diisi **agar** data siap untuk Dapodik.

- Tab Ringkasan menampilkan % kelengkapan dan daftar "Perlu dilengkapi"; klik item membuka tab terkait.
- Toggle "Belum diisi saja" hanya menampilkan field kosong.
- Total field di semua tab = 119 tampilan (120 field; NISN sekali).

### US-04 · Berpindah sekolah
**Sebagai** pengguna dengan akses ke beberapa sekolah, **saya ingin** berpindah sekolah **tanpa** login ulang.

- Dropdown hanya menampilkan sekolah yang diizinkan.
- Memilih sekolah mengganti brand, mereset filter, dan memuat data sekolah tersebut.

### US-05 · Memantau pembayaran
**Sebagai** staf keuangan, **saya ingin** melihat transaksi yang belum terekonsiliasi.

- KPI "Belum terekonsiliasi" menghitung transaksi lunas yang belum dicocokkan.
- Kolom Rekonsiliasi menampilkan ✓ Cocok / ⚠ Belum cocok / —.

### US-06 · Meninjau hasil psikotes
**Sebagai** kepala sekolah, **saya ingin** melihat hasil psikotes calon siswa.

- Klik nama di halaman Psikotes membuka panel detail; "Data pendaftar →" membuka detail pendaftar dengan tombol "Kembali ke Hasil psikotes".

## 5. MVP & future

| Prioritas | Fitur |
|---|---|
| **MVP** | Auth + role per sekolah, dashboard, pendaftar (tabel/kartu/kanban, filter, ubah tahap), detail 7 tab (baca + edit dasar), upload dokumen, transaksi (baca), psikotes (baca + input), kelas/siswa/guru/psikolog (baca), export CSV/Excel, migrasi data Odoo |
| **Fase 2** | Pengiriman WhatsApp/email, CRUD penuh modul pendukung, pengaturan sekolah, notifikasi in-app |
| **Future** | Portal orang tua, payment gateway, riwayat level, integrasi Dapodik |
