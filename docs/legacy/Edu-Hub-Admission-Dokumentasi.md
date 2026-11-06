# Edu Hub · Modul Admission: Dokumentasi Aplikasi

> Redesign modul Penerimaan Siswa Baru (PSB) Edu Hub, dari list view Odoo 16 menjadi frontend modern untuk dua grup sekolah: **Kreativa Global School** dan **Sekolah Talenta Juara**.
>
> Status dokumen: **prototipe interaktif (desain)**, versi 29, 29 Oktober 2026. Prototipe ada di canvas Claude Design *"Edu Hub Admission Redesign"*, dengan dua artboard: **Desktop** (1440 px, fluid) dan **Mobile** (390 × 844 px).

---

## Daftar isi

1. [Ringkasan](#1-ringkasan)
2. [Pengguna & tujuan](#2-pengguna--tujuan)
3. [Masalah versi Odoo & keputusan UX](#3-masalah-versi-odoo--keputusan-ux)
4. [Multi-sekolah & identitas brand](#4-multi-sekolah--identitas-brand)
5. [Struktur aplikasi & navigasi](#5-struktur-aplikasi--navigasi)
6. [Halaman: Ringkasan (dashboard)](#6-halaman-ringkasan-dashboard)
7. [Halaman: Pendaftar](#7-halaman-pendaftar)
8. [Detail pendaftar](#8-detail-pendaftar)
9. [Halaman: Transaksi](#9-halaman-transaksi)
10. [Halaman: Psikotes](#10-halaman-psikotes)
11. [Halaman: Psikolog](#11-halaman-psikolog)
12. [Halaman: Kelas](#12-halaman-kelas)
13. [Halaman: Siswa](#13-halaman-siswa)
14. [Halaman: Guru](#14-halaman-guru)
15. [Pola UI bersama](#15-pola-ui-bersama)
16. [Status, tahapan & aturan warna](#16-status-tahapan--aturan-warna)
17. [State layar (loading, kosong, error, tanpa akses)](#17-state-layar)
18. [Versi mobile](#18-versi-mobile)
19. [Aksesibilitas](#19-aksesibilitas)
20. [Design tokens](#20-design-tokens)
21. [Pemetaan field Odoo → tampilan baru](#21-pemetaan-field-odoo--tampilan-baru)
22. [Data contoh & asumsi](#22-data-contoh--asumsi)
23. [Panduan implementasi (Next.js + Odoo)](#23-panduan-implementasi-nextjs--odoo)
24. [Yang belum dikerjakan](#24-yang-belum-dikerjakan)
25. [Riwayat perubahan](#25-riwayat-perubahan)

---

## 1. Ringkasan

Edu Hub Admission adalah aplikasi untuk tim penerimaan siswa baru: mengelola pendaftar dari tahap **Leads** sampai **Tuition lunas**, ditambah data pendukungnya (transaksi, psikotes, psikolog, kelas, siswa, guru). Data tetap disimpan di Odoo 16. Aplikasi ini hanya lapisan tampilan dan alur kerja di atasnya.

Prinsip utamanya:

- **Satu status per baris, bukan lima badge merah.** Lima kolom status Odoo digabung menjadi satu *progress stepper* 5 titik.
- **Merah hanya untuk masalah**: terlambat, data bermasalah, invalid. Status "belum" berwarna abu-abu.
- **Insight di depan**: KPI, funnel konversi, dan daftar "Perlu tindakan" tampil di halaman pertama.
- **Ikut brand sekolah**: logo dan warna berganti otomatis sesuai sekolah yang dipilih.

---

## 2. Pengguna & tujuan

| Pengguna | Kebutuhan utama |
|---|---|
| Tim admission (officer) | Menindaklanjuti pendaftar per tahap, mengirim pengingat, melengkapi data & dokumen |
| Kepala sekolah | Memantau jumlah pendaftar, konversi, kuota per level |
| Tim keuangan | Memantau pembayaran formulir & tuition, rekonsiliasi transaksi |

Tujuan bisnis: mengurangi beban chat WhatsApp dari orang tua dengan membuat status setiap pendaftar jelas dan mudah ditindaklanjuti.

---

## 3. Masalah versi Odoo & keputusan UX

| Masalah di Odoo | Keputusan di redesign |
|---|---|
| 5 kolom status (Payment, Psychology, Interview, Acceptance, Tuition) dalam satu baris, semuanya merah "Incomplete" | Diganti **progress stepper 5 titik**: Bayar Formulir → Psikotes → Wawancara → Keputusan → Tuition. Hover/fokus menampilkan detail dan tanggal |
| Kolom *Flow* dan *Acceptance Status* maknanya tumpang tindih | **Tahap** hanya memakai `flow`. `acceptance_status` menjadi titik "Keputusan" di stepper |
| Kolom *School* berulang di setiap baris | Dihapus. Sekolah dipilih sekali lewat dropdown di sidebar |
| Tidak ada ringkasan atau insight | Halaman **Ringkasan**: KPI, funnel, tren, perlu tindakan, distribusi level & sumber |
| Data tidak valid (nama kosong, tuition 0) tidak ditandai | Ikon **⚠ merah** + tooltip di tabel, badge merah di detail, filter "Data bermasalah" |
| Sidebar FLOW di kiri tabel | Diganti **tab tahapan** di atas tabel, lengkap dengan jumlah |
| Kolom kosong di Transaksi (Product, Unique Code, No Bukti) | Dihapus dari tampilan. *Counted* diganti status **Rekonsiliasi** |
| Form detail berupa 5 tab field mentah | Disusun ulang menjadi **7 tab berbasis alur kerja** dengan penghitung kelengkapan. Jumlah field tetap sama |

---

## 4. Multi-sekolah & identitas brand

### 4.1 Daftar sekolah (9 unit, 2 grup)

| Grup | Unit | Kode internal | Level |
|---|---|---|---|
| **Sekolah Talenta Juara** | SD Talenta Juara Bandung | `sd-tj-bdg` | Grade 1–6 |
| | SD Talenta Juara Cimahi | `sd-tj-cmh` | Grade 1–6 |
| | SD Talenta Juara Pekanbaru | `sd-tj-pku` | Grade 1–6 |
| | SMP Talenta Juara Bandung | `smp-tj-bdg` | Grade 7–9 |
| | SMP Talenta Juara Pekanbaru | `smp-tj-pku` | Grade 7–9 |
| **Kreativa Global School** | Kreativa Global Kindergarten | `kg-kinder` | Kindergarten 1–2 |
| | Kreativa Global Primary | `kg-primary` | Grade 1–6 |
| | Kreativa Global Secondary | `kg-sec` | Grade 7–9 |
| | Kreativa Global High School | `kg-hs` | Grade 10–12 |

### 4.2 Brand per grup

| | Kreativa Global School | Sekolah Talenta Juara |
|---|---|---|
| Logo | Perisai Kreativa (ungu–merah muda) | Logo "TJ" biru + wordmark |
| Warna sidebar | `#5001B2` | `#00A7E1` |
| Primer (tombol, link, tab aktif) | `#5001B2` | `#0077A3`* |
| Sekunder (aksen, progress bar) | `#EA3B5F` | `#00A7E1` |
| Teks menu | Putih | Putih |
| Font | Source Sans 3 (pengganti resmi Source Sans Pro) | Source Sans 3 |

\* Untuk Talenta Juara, tombol dan link memakai biru yang lebih tua (`#0077A3`) karena teks putih di atas `#00A7E1` kontrasnya hanya ±2,7:1. Biru brand `#00A7E1` tetap dipakai untuk sidebar, garis aksen, dan bar grafik.

Saat pengguna memilih unit dari grup lain, logo, warna sidebar, dan semua warna aksen langsung berganti.

---

## 5. Struktur aplikasi & navigasi

### 5.1 Layout desktop: dua kolom (pola admin shadcn)

```
┌──────────────────┬──────────────────────────────────────────────┐
│ [Logo] Edu Hub   │ ☰ │ Sekolah / Halaman      [Cari] [🔔] [☾]   │  ← top bar (diam)
│ Admission · PSB  ├──────────────────────────────────────────────┤
│ ┌──────────────┐ │                                              │
│ │ TJ  Grup     │ │                                              │
│ │     Unit   ⇅ │ │            KONTEN HALAMAN                    │
│ └──────────────┘ │            (hanya area ini yang scroll)      │
│ ADMISSION        │                                              │
│  Ringkasan       │                                              │
│  Pendaftar  324  │                                              │
│  Transaksi  453  │                                              │
│  Psikotes   215  │                                              │
│ AKADEMIK         │                                              │
│  Kelas       32  │                                              │
│  Siswa      833  │                                              │
│ SDM & MITRA      │                                              │
│  Guru        40  │                                              │
│  Psikolog    13  │                                              │
│ PINTASAN         │                                              │
│  Macet > 7 hari  │                                              │
│  Data bermasalah │                                              │
│  Waiting/Invalid │                                              │
│ [Simulasi state] │                                              │
├──────────────────┤                                              │
│ (A) Admin        │                                              │
└──────────────────┴──────────────────────────────────────────────┘
```

**Kolom kiri (sidebar, 272 px, warna brand):**
1. Identitas: logo sekolah + "Edu Hub · Admission PSB"
2. **Dropdown pindah sekolah**: tombol menampilkan grup dan unit aktif. Panelnya berisi daftar berkelompok (Talenta Juara 5 unit, Kreativa 4 unit) dengan jumlah pendaftar per unit dan tanda centang pada unit aktif
3. Menu berkelompok dengan jumlah data per menu
4. Pintasan: *Macet > N hari*, *Data bermasalah*, dan *Waiting List* atau *Invalid* (muncul jika tahap itu dipakai sekolah)
5. Panel **Simulasi state** (khusus demo): Normal / Loading / Kosong / Error / Tanpa hak akses
6. Profil pengguna di bagian bawah

**Kolom kanan:**
- **Top bar**: tombol buka-tutup sidebar, breadcrumb (*Nama sekolah / Halaman*), tombol **Cari pendaftar** (langsung membuka Pendaftar dengan fokus di kolom cari), notifikasi jumlah pendaftar macet, toggle mode gelap/terang
- **Konten**: satu-satunya area yang ter-scroll. Sidebar dan top bar tetap di tempat

Di bawah 820 px layout kembali bertumpuk satu kolom.

### 5.2 Peta halaman

| Grup menu | Halaman | Tampilan default |
|---|---|---|
| Admission | Ringkasan | Dashboard |
| | Pendaftar | Tabel (opsi Kartu, Kanban) |
| | Transaksi | Tabel (opsi Kartu) |
| | Psikotes | Tabel (opsi Kartu) |
| Akademik | Kelas | Tabel (opsi Kartu) |
| | Siswa | Tabel (opsi Kartu) |
| SDM & Mitra | Guru | Kartu (opsi Tabel) |
| | Psikolog | Kartu (opsi Tabel) |

---

## 6. Halaman: Ringkasan (dashboard)

Header: judul, nama sekolah, waktu pembaruan, dan pemilih **tahun ajaran** (2022/2023 s.d. 2028/2029).

### 6.1 KPI (5 kartu)

| KPI | Rumus | Keterangan |
|---|---|---|
| Total pendaftar | Jumlah pendaftar di TA terpilih | Semua tahap |
| Pendaftar baru (7 hari) | `created_at` ≤ 7 hari | Dibanding 7 hari sebelumnya (panah naik/turun) |
| Konversi Leads → Diterima | Pendaftar tahap Tuition ÷ total | Dalam % |
| Tuition terkumpul | Σ nominal tuition yang lunas | Dengan progress bar vs target sekolah |
| Sisa kuota | Σ kuota level − jumlah diterima | Rincian 3 level dengan sisa terbanyak + progress bar |

### 6.2 Pipeline per tahap (funnel)

Leads masuk → Register → Bayar formulir → Tes & wawancara selesai → Diterima. Setiap tahap menampilkan jumlah, persentase kumulatif, dan **drop-off** dari tahap sebelumnya. Tahap dengan drop-off terbesar diberi label *"drop-off terbesar"*. Pendaftar berstatus Invalid tidak ikut dihitung, dan jumlahnya disebutkan di bawah funnel.

### 6.3 Tren pendaftaran

Grafik garis **Mingguan (12 minggu)** atau **Harian (30 hari)**. Garis tahun ini dibandingkan dengan garis putus-putus *TA sebelumnya, periode sama*, disertai ringkasan teks (naik/turun %).

### 6.4 Perlu tindakan (3 kolom)

1. **Macet > N hari**: 5 pendaftar paling lama di tahapnya
2. **Jadwal psikotes & wawancara terdekat**: kalender mini
3. **Pembayaran tertunda**: formulir belum dibayar dan tuition belum lunas

Setiap item bisa diklik untuk membuka detail pendaftar.

### 6.5 Distribusi

- **Per level & kuota**: bar ganda (ungu = diterima, merah muda = total pendaftar) relatif terhadap kuota
- **Sumber pendaftaran**: Website, Instagram, Referensi orang tua, Open House, WhatsApp, masing-masing dengan % konversi ke diterima

---

## 7. Halaman: Pendaftar

### 7.1 Kontrol

| Elemen | Detail |
|---|---|
| Tab tahapan | Semua · Leads · Register · Payment · Tuition · Waiting List · Invalid, masing-masing dengan jumlah. Tab hanya muncul bila tahap itu dipakai sekolah |
| Pencarian | Nama siswa, orang tua, atau no. pendaftaran (debounce 250 ms) |
| Filter tahun ajaran | Semua / per TA (default 2027/2028) |
| Filter level | Sesuai level sekolah |
| Filter status | Terlambat (macet) · Data bermasalah · Formulir belum dibayar · Psikotes belum selesai · Wawancara belum selesai · Menunggu keputusan · Tuition belum lunas · Tuition lunas · Tidak dilanjutkan (TA lewat) |
| Filter tanggal daftar | 7 / 30 / 90 hari terakhir |
| Filter petugas | Per officer PIC |
| Reset filter | Muncul bila ada filter aktif |
| Pilihan tampilan | **Tabel · Kartu · Kanban** |
| Aksi | Export Excel, Pendaftar baru |

### 7.2 Tampilan Tabel (10 baris per halaman)

| Kolom | Isi |
|---|---|
| ☐ | Pilih (aksi massal) |
| Siswa & orang tua | Nama (link ke detail) + ⚠ bila bermasalah; baris kedua: orang tua · no. registrasi |
| Level | Grade / Kindergarten |
| Tahap | Badge `flow` (merah untuk Invalid) |
| Progress | Stepper 5 titik + ringkasan teks |
| Lama di tahap | `N hari`, merah + ikon jam bila terlambat, hijau "Selesai", "Ditutup", "Invalid". Bisa diurutkan |
| Tanggal daftar | `26 Okt 2026, 15:47`. Bisa diurutkan |
| Aksi | Tombol aksi utama sesuai tahap |

Pagination: **Awal · ‹ · Hal. X dari Y · › · Akhir**. Pindah halaman otomatis kembali ke atas.

### 7.3 Tampilan Kartu (12 per halaman)

Kartu berisi checkbox, nama, badge tahap, orang tua · no. registrasi, stepper, lama di tahap, ringkasan, level · tanggal daftar, dan tombol aksi.

### 7.4 Tampilan Kanban

Satu kolom per tahap (8 kartu per kolom, diurutkan dari yang paling lama di tahapnya, dengan link "+N lainnya"). **Drag & drop** kartu ke kolom lain membuka dialog konfirmasi *"Pindahkan tahap?"*.

### 7.5 Aksi massal

Muncul setelah ada baris yang dipilih: **Pengingat WhatsApp · Email · Ubah tahap · Export Excel · Batal pilih**.

### 7.6 Aksi utama per tahap

| Kondisi | Tombol |
|---|---|
| Leads | Kirim link formulir |
| Register | Ingatkan bayar |
| Payment, psikotes belum | Jadwalkan psikotes |
| Payment, wawancara belum | Jadwalkan wawancara |
| Payment, tes selesai | Input keputusan |
| Tuition belum lunas | Kirim tagihan |
| Tuition lunas | Lihat (bukti bayar) |
| Waiting List | Tawarkan kursi |
| Invalid | Verifikasi data |
| TA lewat & tidak dilanjutkan | Lihat riwayat |

---

## 8. Detail pendaftar

Dibuka sebagai **slide-over dari kanan** (lebar 780 px) dengan klik nama, tombol aksi, atau item di dashboard. Bisa ditutup dengan tombol X, klik di luar panel, atau `Esc`.

### 8.1 Header

- Kode registrasi, avatar inisial, nama (merah miring bila kosong), level · sekolah · TA, tanggal daftar
- Badge: tahap, lama di tahap, flag masalah
- Tombol: **aksi utama** (sesuai tahap), **WhatsApp**, **Ubah tahap**
- **Stepper besar** 5 tahap dengan tanggal atau status per tahap

### 8.2 Tab (berbasis alur kerja)

Setiap tab menampilkan penghitung **x/y terisi** (hijau bila lengkap).

| Tab | Isi |
|---|---|
| **Ringkasan** | *Kelengkapan data* (% total + bar per tab, bisa diklik), *Perlu dilengkapi* (field penting yang kosong, klik untuk loncat ke tab terkait), *Informasi pendaftaran* (12 field) + tombol Chat WhatsApp / Kirim email |
| **Profil siswa** | Kartu *Identitas calon siswa*, *Sekolah asal*, *Alamat tinggal*, *Kesehatan, bahasa & minat* |
| **Seleksi & pembayaran** | **Timeline 5 tahap** (Pembayaran formulir → Psikotes → Wawancara orang tua → Keputusan penerimaan → Pembayaran tuition). Tiap tahap berupa kartu berisi status, tanggal, dan field-nya, dengan tahap aktif disorot. Ditambah kartu *Rekening & virtual account* dan *Notifikasi otomatis ke orang tua* |
| **Dokumen** | Progress "x dari 9 dokumen diunggah", No. KK & NIK, lalu **9 kotak dokumen** dengan status dan tombol Unggah/Ganti |
| **Keluarga** | Kartu *Ayah*, *Ibu*, *Wali*, *Kontak darurat* |
| **Dapodik** | Catatan pelaporan Dapodik, kartu *Nomor induk & dokumen resmi*, kartu *Data pendukung* |
| **Riwayat level** | Masih kosong (lihat §24) |

**Fitur tambahan detail:**
- Toggle **Semua field / Belum diisi saja** di setiap tab data
- Field kosong ditampilkan sebagai label **"Belum diisi"**, tidak disembunyikan
- Checkbox tampil sebagai *Ya / Tidak*, field status sebagai badge berwarna
- **Navigasi kembali**: tombol *"← Kembali ke …"* mengikuti jalur yang dilalui (antar-tab, atau kembali ke panel Hasil psikotes bila datang dari sana), bisa beberapa langkah. Riwayatnya dikosongkan saat panel ditutup

Pemetaan lengkap field ada di §21.

---

## 9. Halaman: Transaksi

**KPI:** Total diterima · Menunggu pembayaran · Metode terbanyak · Belum terekonsiliasi (merah bila > 0).

**Tab:** Semua · Lunas · Menunggu bayar · Kedaluwarsa. **Filter metode:** QRIS · VA BNI · VA BRI.

| Kolom tabel | Isi |
|---|---|
| Dibuat | Tanggal & jam |
| Siswa & no. registrasi | Format `KODE.YYYYMMDD.NNNN` |
| Jenis | Formulir / Tuition |
| Metode | QRIS / VA BNI / VA BRI |
| Nominal | Rupiah, merah bila Rp 0 |
| Status | Lunas (hijau) · Menunggu bayar (kuning) · Kedaluwarsa (abu) |
| No. VA · dibayar oleh | Font monospace + nama pembayar |
| Rekonsiliasi | ✓ Cocok · ⚠ Belum cocok · — |

Kolom Odoo yang dihapus dari tampilan karena selalu kosong: *Product, Unique Code, No Bukti*. *Counted* diganti kolom Rekonsiliasi.

---

## 10. Halaman: Psikotes

**KPI:** Total psikotes · Rata-rata skor IQ (+ rentang) · Disarankan diterima (% IQ ≥ 95) · Tanpa catatan psikolog.

**Tab:** Semua · Disarankan · Dipertimbangkan · Tanpa catatan.

| Kolom | Isi |
|---|---|
| Tanggal tes | |
| Siswa | **Link ke panel detail**; baris kedua: tanggal lahir · umur · level |
| Psikolog | |
| Skor IQ | Angka + bar mini |
| Rekomendasi | Disarankan (hijau) / Dipertimbangkan (kuning) |
| Catatan psikolog | Merah miring + ⚠ bila kosong |
| Lampiran | Tombol PDF · ukuran file |

**Panel detail hasil psikotes (slide-over):**
- Header: nama, lahir/umur/level, badge rekomendasi, skor IQ + kategori
- Tombol: Unduh hasil (PDF), Kirim ke orang tua, **Data pendaftar →** (membuka detail pendaftar di tab Seleksi & pembayaran, dengan tombol kembali ke panel ini)
- Skala IQ 70–140 (di bawah rata-rata / rata-rata / di atas rata-rata)
- Aspek yang diukur: verbal, numerik, penalaran logis, konsentrasi, sosial-emosional *(angka contoh)*
- Catatan psikolog; bila kosong ditampilkan blok merah + tombol "Minta catatan"
- Informasi tes: tanggal, psikolog, lembaga, sekolah tujuan, lampiran, status verifikasi

Kolom Odoo *Fullname* (duplikat Student Name) dihapus dari tampilan. Skor IQ diambil dari teks *Description*.

---

## 11. Halaman: Psikolog

**KPI:** Psikolog mitra · Lembaga mitra · Tes ditangani · Data belum lengkap.

**Tab:** Semua · Menangani tes · Data belum lengkap.

Kartu (default) berisi avatar inisial, nama + gelar, lembaga, telepon, email, badge jumlah tes. Data rusak (nama tidak valid seperti "za", email kosong) ditandai ⚠. Tampilan Tabel: Nama · Lembaga · Telepon · Email · Status.

---

## 12. Halaman: Kelas

**KPI:** Total kelas (rincian per TA) · Total siswa (rata-rata per kelas) · Tanpa wali kelas · Melebihi kapasitas.

**Tab level** (hanya level yang punya kelas) + **filter tahun ajaran** (Semua TA / per TA).

| Kolom | Isi |
|---|---|
| Kelas | Nama rombel |
| Level | |
| Wali kelas | Merah miring + ⚠ "Belum ada wali kelas" bila kosong |
| Jumlah siswa / kapasitas | Bar + `head/cap`, merah bila kelebihan |
| TA | |
| Aksi | **Lihat siswa** (membuka halaman Siswa dengan filter kelas tersebut) |

---

## 13. Halaman: Siswa

**KPI:** Siswa aktif · Jumlah kelas · Cuti · Mutasi keluar.

**Tab level** + **filter status**: Semua · Aktif · Cuti · Mutasi keluar.

Kolom: Siswa (nama + NIS) · Level · Kelas · Wali kelas · TA · Status.

---

## 14. Halaman: Guru

**KPI:** Total guru & staf · Wali kelas · Online sekarang · Kelas tanpa wali.

**Tab jabatan:** Wali Kelas · Guru Mata Pelajaran · Guru Pendamping · Guru Penunjang · Wakil Kepala Sekolah · Kepala Sekolah.

Kartu (default): avatar inisial, nama, jabatan · kelas yang diampu (semua kelas di semua TA, tanpa duplikasi guru), email, telepon, badge **Online/Offline** (berupa teks, bukan titik warna saja). Tampilan Tabel: Nama · Email · Telepon · Status.

---

## 15. Pola UI bersama

| Pola | Spesifikasi |
|---|---|
| **Pilihan tampilan** | Segmented control ikon (Tabel ⊞ / Kartu ▦) di kanan atas setiap halaman data. Ikon aktif berlatar putih dengan warna brand. Pilihan diingat per halaman |
| **Pagination** | 10 baris (tabel) / 12 kartu per halaman |
| **Tab filter** | Garis bawah warna primer + badge jumlah |
| **KPI card** | Label, angka besar, sub-teks. Angka merah bila menunjukkan masalah |
| **Slide-over detail** | Overlay gelap, panel kanan, tombol X, klik luar & `Esc` untuk menutup |
| **Dialog konfirmasi** | Untuk ubah tahap (tunggal / massal / drag kanban), dengan catatan bahwa perubahan dicatat di timeline |
| **Toast** | Umpan balik aksi simulasi di bawah layar (±2,8 detik) |
| **Tooltip** | Hover dan fokus keyboard pada titik stepper & ikon ⚠ |
| **Tombol "Data bermasalah"** | Ikon segitiga merah + tooltip alasan |
| **Format** | Tanggal `26 Okt 2026, 15:47`; mata uang `Rp 12.500.000`; ringkas `Rp 1,50 M` / `Rp 352 jt` |

---

## 16. Status, tahapan & aturan warna

### 16.1 Tahap (`flow`)

`Leads → Register → Payment → Tuition`, dengan cabang `Waiting List` dan `Invalid`.

### 16.2 Progress stepper (5 titik)

| Titik | Diambil dari |
|---|---|
| 1. Bayar Formulir | `payment_status` |
| 2. Psikotes | `psychology_test` |
| 3. Wawancara | `parents_interview` |
| 4. Keputusan | `acceptance_status` |
| 5. Tuition | `tuition_status` |

| State titik | Tampilan | Arti |
|---|---|---|
| Selesai | Hijau penuh + ✓ | Tahap selesai |
| Berjalan | Lingkaran biru + titik | Tahap aktif |
| Menunggu | Lingkaran kuning + jam | Waiting List (menunggu kuota) |
| Terlambat | Merah penuh + ! | Aktif lebih dari N hari |
| Belum | Lingkaran putus-putus abu | Belum dimulai |

Bentuk ikon berbeda untuk tiap state, jadi status tidak dibedakan oleh warna saja.

### 16.3 Aturan turunan

| Aturan | Logika |
|---|---|
| **Terlambat / macet** | Belum selesai, bukan Waiting List, bukan Invalid, TA masih berjalan, dan `lama di tahap > N hari` (N default **7**, bisa diatur) |
| **Tidak dilanjutkan** | TA sudah lewat dan pendaftar tidak sampai Tuition. Status "Ditutup", **tidak** ditandai merah |
| **Data bermasalah** | Nama siswa kosong · status Invalid · nominal tuition Rp 0 di tahap Tuition (belum lunas) atau Waiting List |

### 16.4 Semantik warna

| Warna | Dipakai untuk |
|---|---|
| Abu-abu | Belum dimulai / netral |
| Biru | Sedang berjalan / terjadwal |
| Hijau | Selesai / lunas / cocok |
| Kuning | Menunggu (Waiting List, menunggu bayar) |
| **Merah** | **Hanya** terlambat, data bermasalah, invalid, kapasitas lebih |

Merah status (`#B42318`) sengaja berbeda dari merah brand Kreativa (`#EA3B5F`) supaya tidak tertukar.

---

## 17. State layar

| State | Tampilan |
|---|---|
| **Loading** | Skeleton shimmer (KPI, chart, baris tabel) |
| **Kosong** | Ikon, judul, penjelasan, tombol aksi (Tambah pendaftar / Reset filter) |
| **Tidak ada hasil filter** | "Tidak ada data yang cocok" + Reset filter |
| **Error** | "Gagal memuat data dari Odoo" (timeout 30 detik), data lama tidak ditampilkan, tombol **Coba lagi** |
| **Tanpa hak akses** | Ikon gembok, penjelasan grup akses Odoo yang dibutuhkan, tombol *Minta akses ke admin* |

Di prototipe semua state bisa dicoba lewat panel **Simulasi state** di sidebar (desktop) atau properti Tweaks (mobile).

---

## 18. Versi mobile

Artboard 390 × 844 px.

| Bagian | Desain |
|---|---|
| App bar | Logo sekolah, judul halaman, **nama sekolah (tap untuk ganti)**, toggle tema, notifikasi |
| Bottom nav | **Ringkasan · Pendaftar · Tindakan**, berlatar warna brand |
| Ringkasan | 4 KPI (2×2), kartu tuition + sisa kuota, funnel, 3 pendaftar macet teratas |
| Pendaftar | Pencarian, tab tahap berbentuk pil (bisa digeser), tombol Filter (bottom sheet) dengan badge jumlah filter, mode **Pilih** untuk aksi massal, daftar kartu dengan stepper mini, "Muat lebih banyak" (15 per batch) |
| Tindakan | Segmented: Macet · Jadwal · Pembayaran |
| Detail | Layar penuh, tab pil sticky (tinggi tetap 56 px), struktur tab sama dengan desktop, tombol bawah: WhatsApp + aksi utama |
| Pilih sekolah | Bottom sheet daftar 9 unit |
| Ubah tahap | Bottom sheet konfirmasi |

Target sentuh minimal 44 px.

---

## 19. Aksesibilitas

- Kontras teks memenuhi WCAG 2.1 AA (≥ 4,5:1), **kecuali** teks putih di sidebar biru Talenta Juara `#00A7E1` (±2,7:1). Rekomendasi: gelapkan ke `#0077A3` bila keterbacaan diutamakan
- Status tidak dibedakan oleh warna saja (ikon, teks, bentuk)
- Semua kontrol berupa `<button>`, `<input>`, `<select>`, `<label>` asli dan bisa dinavigasi dengan keyboard, dengan `:focus-visible` berwarna primer
- `aria-label` pada tombol ikon, `aria-pressed` pada toggle, `aria-current` pada menu, `aria-sort` pada kolom yang bisa diurutkan, `role="tab"`/`tablist` pada tab detail
- Dialog memakai `role="dialog"` + `aria-modal` dan bisa ditutup dengan `Esc`
- Grafik memiliki `role="img"` + `aria-label` ringkasan data
- Toast memakai `role="status"` + `aria-live="polite"`

---

## 20. Design tokens

### 20.1 Tipografi & spasi

| Token | Nilai |
|---|---|
| Font | `'Source Sans 3', 'Source Sans Pro', system-ui, sans-serif` |
| Ukuran dasar konten | 14 px (sidebar 14 px, label grup 11 px) |
| Judul halaman | 25 px / 700 |
| Angka KPI | 27–29 px / 700 |
| Isi tabel | 13–14 px |
| Grid spasi | Kelipatan 4/8 px |
| Radius | 6 px (input kecil), 8 px (tombol), 10–12 px (kartu, panel) |
| Lebar sidebar | 272 px |
| Lebar slide-over | 600 px (psikotes), 780 px (pendaftar) |

### 20.2 Warna dasar (light / dark)

| Token | Light | Dark |
|---|---|---|
| `--bg` | `#F6F5F7` | `#151217` |
| `--surface` | `#FFFFFF` | `#1E1A21` |
| `--surface2` | `#F3F1F4` | `#29242C` |
| `--border` | `#E3DFE5` | `#3A343E` |
| `--border-strong` | `#A39BA8` | `#7A7280` |
| `--text` | `#1E1A21` | `#F2EFF3` |
| `--muted` | `#5F5866` | `#B3ABB8` |

### 20.3 Warna brand

| Token | KGS light | KGS dark | TJ light | TJ dark |
|---|---|---|---|---|
| `--primary` | `#5001B2` | `#B48CFF` | `#0077A3` | `#5CCBF2` |
| `--primary-hover` | `#3E008C` | `#C9ABFF` | `#00628A` | `#86D8F5` |
| `--on-primary` | `#FFFFFF` | `#1A0640` | `#FFFFFF` | `#04222E` |
| `--primary-soft` | `#EFE6FB` | `#2B1C4D` | `#E0F5FC` | `#10313F` |
| `--secondary` | `#EA3B5F` | `#FF7A95` | `#00A7E1` | `#33BDEB` |
| `--secondary-soft` | `#FDE8ED` | `#45202B` | `#D6F2FB` | `#0E3442` |
| `--side-bg` | `#5001B2` | `#2E0A6B` | `#00A7E1` | `#0A3A4D` |
| `--side-fg` | `#FFFFFF` | `#FFFFFF` | `#FFFFFF` | `#FFFFFF` |
| `--side-active-bg` | `#FFFFFF` | `#B48CFF` | `#FFFFFF` | `#00A7E1` |
| `--side-active-fg` | `#5001B2` | `#1A0640` | `#005F82` | `#04222E` |

### 20.4 Warna semantik

| Token | Light fg / bg | Dark fg / bg |
|---|---|---|
| gray | `#4B5563` / `#EEF0F3` | `#C3C9D1` / `#2B2F36` |
| blue | `#1D4ED8` / `#E7EEFD` | `#9DBBFF` / `#1D2A47` |
| green | `#15803D` / `#E2F3E7` | `#86D9A6` / `#17301F` |
| amber | `#8A4B08` / `#FCF0DA` | `#F4C97E` / `#3A2B12` |
| red | `#B42318` / `#FDE9E7` | `#FFA39B` / `#3F1D1A` |

### 20.5 Contoh `tailwind.config.ts`

```ts
import type { Config } from "tailwindcss";

export default {
  darkMode: ["class"],
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  theme: {
    extend: {
      fontFamily: { sans: ['"Source Sans 3"', '"Source Sans Pro"', "system-ui", "sans-serif"] },
      colors: {
        bg: "var(--bg)", surface: "var(--surface)", surface2: "var(--surface2)",
        border: "var(--border)", "border-strong": "var(--border-strong)",
        text: "var(--text)", muted: "var(--muted)",
        primary: { DEFAULT: "var(--primary)", hover: "var(--primary-hover)", soft: "var(--primary-soft)", fg: "var(--on-primary)" },
        secondary: { DEFAULT: "var(--secondary)", soft: "var(--secondary-soft)" },
        side: { bg: "var(--side-bg)", fg: "var(--side-fg)", muted: "var(--side-muted)", active: "var(--side-active-bg)", "active-fg": "var(--side-active-fg)" },
        status: {
          gray: { fg: "var(--gray-fg)", bg: "var(--gray-bg)" },
          blue: { fg: "var(--blue-fg)", bg: "var(--blue-bg)" },
          green: { fg: "var(--green-fg)", bg: "var(--green-bg)" },
          amber: { fg: "var(--amber-fg)", bg: "var(--amber-bg)" },
          red: { fg: "var(--red-fg)", bg: "var(--red-bg)" },
        },
      },
      borderRadius: { sm: "6px", md: "8px", lg: "10px", xl: "12px" },
    },
  },
} satisfies Config;
```

Variabel CSS ditentukan di root dengan kombinasi kelas `theme-light|theme-dark` × `brand-kgs|brand-tj`, sesuai nilai §20.2–20.4.

---

## 21. Pemetaan field Odoo → tampilan baru

Form pendaftar Odoo memiliki **120 field** di 4 tab (General 46, Admission 24, Student 29, Parents 21) + tab Level History. Tampilan baru menampilkan semua field itu **tanpa menambah atau mengurangi**. Satu-satunya pengecualian: *NISN* muncul di dua tab Odoo dan ditampilkan sekali saja, sehingga ada **119 tampilan field**.

### 21.1 Tab General (46)

| Section Odoo | Field Odoo | Tampil di (tab › kartu) |
|---|---|---|
| General Information | Active, Registered By, Email, Whatsapp, Info, Agree, Created at, Company, School, Level, Academic Year, Message | Ringkasan › Informasi pendaftaran |
| Student Candidate | Student Status, Student Name, Student Gender, Place of Birth, Birth Date | Profil siswa › Identitas calon siswa |
| | NISN | Dapodik › Nomor induk (digabung dengan NISN tab Student) |
| Previous School Information | School Name, Level, School NPSN, School Address, Top Awards | Profil siswa › Sekolah asal |
| | Award Attachment | Dokumen › kotak dokumen |
| Address | Address, RT, RW, Zip Code, Province, City, District, Subdistrict | Profil siswa › Alamat tinggal |
| Document | Family ID, NIK | Dokumen › Kelengkapan dokumen |
| | Student Picture, Birth Certificate, Previous School Report, Family Card, Letter of Consent | Dokumen › kotak dokumen |
| Other | Flow | Seleksi & pembayaran › Keputusan penerimaan |
| | Special Needs, Daily Language, Medical Condition, Allergy Details, Pickup Service, Extracurricular | Profil siswa › Kesehatan, bahasa & minat |

### 21.2 Tab Admission (24)

| Section Odoo | Field Odoo | Tampil di |
|---|---|---|
| Registration Fee | Registration Number, Admission Fee, Payment Receipt, Payment Method, Payment Status, Payment Date, Settlement | Seleksi › 1. Pembayaran formulir |
| Selection Result | Psychology Test, Psychology Data, Psychology Date | Seleksi › 2. Psikotes |
| | Parents Interview, Interview Date | Seleksi › 3. Wawancara orang tua |
| | Acceptance Status, Acceptance Date, Note, Reference | Seleksi › 4. Keputusan penerimaan |
| Tuition Fee | Confirmation Letter, State Confirmation Letter, Tuition Receipt, Payment Count Selected, Tuition Date, Tuition Status | Seleksi › 5. Pembayaran tuition |
| Notification | Send Notification Payment, Send Notification Selection Result | Seleksi › Notifikasi otomatis ke orang tua |

### 21.3 Tab Student (29)

| Section Odoo | Field Odoo | Tampil di |
|---|---|---|
| (atas) | Current Photo, Student Email, Freshman Year, Religion, Scholarship | Profil siswa › Identitas calon siswa |
| Document | NISN, NIS, NIPD, No KPS, No KIP, No UN, No Ijazah, No KKS, No PIP, SKHUN, Birth Certificate Number | Dapodik › Nomor induk & dokumen resmi |
| Bank Information | Bank, Bank Account, Virtual Account | Seleksi › Rekening & virtual account |
| Information for Government | Housing Type, Transport Vehicle, Home-School Distance, Child Order, Number of Siblings, Latitude, Longitude, Weight, Height, Head Circumference | Dapodik › Data pendukung |

### 21.4 Tab Parents (21)

| Section Odoo | Field Odoo | Tampil di |
|---|---|---|
| Father Info | Father Name, Status, Job, Education, Income | Keluarga › Ayah |
| | Father ID Card | Dokumen › kotak dokumen |
| Mother Info | Mother Name, Status, Job, Education, Income | Keluarga › Ibu |
| | Mother ID Card | Dokumen › kotak dokumen |
| Guardian Info | Guardian Name, Job, Education, Income, Relationship | Keluarga › Wali |
| | Guardian ID Card | Dokumen › kotak dokumen |
| Emergency Contact | Name, Phone, Relationship | Keluarga › Kontak darurat |

### 21.5 Rekap per tab baru

| Tab baru | Jumlah field |
|---|---|
| Ringkasan | 12 |
| Profil siswa | 29 |
| Seleksi & pembayaran | 28 |
| Dokumen | 11 |
| Keluarga | 18 |
| Dapodik | 21 |
| **Total** | **119** (120 field Odoo, NISN tampil sekali) |

### 21.6 Field model list pendaftar (dipakai di tabel & dashboard)

`created_at`, `student_name`, `registered_by`, `school`, `level`, `academic_year`, `active`, `nominal_tuition`, `payment_status`, `psychology_test`, `parents_interview`, `acceptance_status`, `tuition_status`, `flow`.

---

## 22. Data contoh & asumsi

### 22.1 Data dari Odoo (sesuai screenshot)

| Data | Sumber |
|---|---|
| Distribusi tahap & tahun ajaran pendaftar Kreativa Global Secondary (238) dan SMP Talenta Juara Bandung (646) | List view Odoo |
| **Data kelas** SD TJ Bandung (32), SD TJ Cimahi (23), SD TJ Pekanbaru (12), SMP TJ Bandung (20), KG Kindergarten (8), KG Primary (4): nama kelas, level, TA, jumlah siswa, wali kelas | List view *Class* Odoo |
| Biaya formulir Rp 300.000, metode QRIS / VA BNI / VA BRI | List view *Transaction* Odoo |

### 22.2 Data karangan (bukan data asli)

- **Semua nama** siswa, orang tua, guru (kecuali wali kelas dari data kelas Odoo), dan psikolog
- Kelas & siswa untuk KG Secondary, KG High School, SMP TJ Pekanbaru
- Transaksi, hasil psikotes, skor aspek psikotes, isi field detail (alamat, NIK, agama, data Dapodik, dll.)
- Foto diganti avatar inisial berwarna

### 22.3 Asumsi yang perlu dikonfirmasi

| Asumsi | Nilai di prototipe |
|---|---|
| Kapasitas kelas (tidak ada di Odoo) | K1: 16 · K2: 18 · KG Primary: 20 · Talenta Juara: 32 |
| Kuota penerimaan per level | Contoh per sekolah (mis. KG Secondary: G7 120, G8 20, G9 10) |
| Target tuition per sekolah | Contoh, Rp 0,9 M – 2,8 M |
| Ambang "macet" | 7 hari |
| Skor IQ ≥ 95 = "Disarankan" | Aturan prototipe |
| Ejaan kelas yang mungkin salah ketik di Odoo | "9 Ibu Majid" vs "9 Ibnu Majid", "Ibnu Batuta" vs "Ibnu Battuta", "Wahyudin" vs "Wahyudi" |

---

## 23. Panduan implementasi (Next.js + Odoo)

### 23.1 Stack

Next.js 15 (App Router) · TypeScript · Tailwind CSS · shadcn/ui · TanStack Table · TanStack Query · Recharts · dnd-kit.

### 23.2 Struktur folder

```
app/
  (admission)/
    layout.tsx                 # sidebar + top bar, brand provider
    dashboard/page.tsx         # Ringkasan
    applicants/page.tsx        # Pendaftar (tabel/kartu/kanban)
    applicants/[id]/page.tsx   # fallback halaman penuh detail
    transactions/page.tsx
    psychology/page.tsx
    psychologists/page.tsx
    classes/page.tsx
    students/page.tsx
    teachers/page.tsx
components/
  shell/ Sidebar.tsx SchoolSwitcher.tsx TopBar.tsx
  data/ DataToolbar.tsx ViewToggle.tsx DataTable.tsx CardGrid.tsx Pager.tsx KpiRow.tsx
  admission/ ProgressStepper.tsx StageBadge.tsx IssueFlag.tsx ApplicantDrawer.tsx KanbanBoard.tsx
  states/ Skeletons.tsx EmptyState.tsx ErrorState.tsx NoAccess.tsx
lib/
  odoo-client.ts               # JSON-RPC wrapper (session Odoo)
  queries/*.ts                 # hook TanStack Query per model
  derive.ts                    # stepper, late, stale, flags (logika §16)
  format.ts                    # tanggal id-ID, Rupiah
  brand.ts                     # mapping company → brand (kgs/tj)
```

### 23.3 URL search params (bisa dibagikan)

`?school=smp-tj-bdg&tab=Payment&year=2027/2028&level=Grade 7&status=late&range=30&officer=…&q=…&view=card&sort=days.desc&page=2&detail=<id>&dtab=dokumen`

### 23.4 Endpoint / method Odoo per widget

> Nama model di bawah adalah **placeholder**. Sesuaikan dengan nama model custom di Edu Hub.

| Widget | Method JSON-RPC | Model (placeholder) | Catatan |
|---|---|---|---|
| Dropdown sekolah | `search_read` | `res.company` | Fields: `name`, logo, grup brand |
| KPI total / baru 7 hari | `search_count` | `edu.admission.form` | Domain `academic_year`, `create_date` |
| Funnel per tahap | `read_group` | `edu.admission.form` | `groupby=['flow']` |
| Tren harian/mingguan | `read_group` | `edu.admission.form` | `groupby=['create_date:week']` / `:day` |
| Tuition terkumpul | `read_group` | `edu.transaction` | `sum(amount)`, `state='paid'`, jenis tuition |
| Sisa kuota | `read_group` + `search_read` | `edu.admission.form`, `edu.level.quota`* | *Model kuota mungkin perlu dibuat |
| Perlu tindakan: macet | `search_read` | `edu.admission.form` | Butuh field "tanggal masuk tahap" (lihat catatan) |
| Jadwal psikotes/wawancara | `search_read` | `edu.admission.form` | `psychology_date`, `interview_date` ≥ hari ini |
| Distribusi level / sumber | `read_group` | `edu.admission.form` | `groupby=['level']` / `['info']` |
| Tabel pendaftar | `search_read` + `search_count` | `edu.admission.form` | Server-side paging, sort, filter |
| Tab jumlah per tahap | `read_group` | `edu.admission.form` | `groupby=['flow']` dengan domain filter aktif |
| Detail pendaftar | `read` | `edu.admission.form` | Semua 120 field + attachment |
| Ubah tahap | `write` | `edu.admission.form` | `flow`, dicatat di chatter |
| Pengingat WhatsApp/email | `call_kw` (method custom) | `edu.admission.form` | Mis. `action_send_reminder` |
| Transaksi | `search_read` | `edu.transaction` | |
| Psikotes | `search_read` | `edu.psychology` | Parsing skor IQ dari `description` |
| Psikolog | `search_read` | `edu.psychologist` | |
| Kelas | `search_read` | `edu.class` | `headcount`, `homeroom_id` |
| Siswa | `search_read` | `edu.student` | `state` |
| Guru | `search_read` | `hr.employee` | `job_id`, status presence |

**Catatan data:** perhitungan "lama di tahap" butuh tanggal kapan pendaftar masuk tahap saat ini. Bila Odoo belum menyimpannya, tambahkan field `stage_date` yang diperbarui saat `flow` berubah, atau baca dari `mail.tracking.value`.

---

## 24. Yang belum dikerjakan

| Item | Status |
|---|---|
| Tab **Riwayat level** di detail pendaftar | Field belum diketahui. Butuh screenshot tab *Level History* Odoo |
| Panel detail untuk Transaksi, Kelas, Siswa, Guru, Psikolog | Belum ada. Pola sama dengan panel Psikotes bisa diterapkan |
| 6 halaman (Transaksi … Guru) di **mobile** | Baru tersedia di desktop |
| Data kelas KG Secondary, KG High School, SMP TJ Pekanbaru | Masih data contoh |
| Form tambah/edit (pendaftar baru, input hasil tes, tambah kelas/guru) | Di luar cakupan prototipe |
| Aksi kirim WhatsApp/email, unggah file, export | Simulasi (toast) |

---

## 25. Riwayat perubahan

| # | Perubahan |
|---|---|
| 1 | Prototipe awal: Ringkasan, Pendaftar (tabel + kanban), drawer detail, desktop & mobile, state lengkap, mode gelap |
| 2 | Brand Kreativa: primer `#5001B2`, sekunder `#EA3B5F`, font Source Sans, logo |
| 3 | Tabel 10 baris per halaman; header & menu tetap, hanya konten yang scroll; 9 sekolah dengan data contoh masing-masing; tahap Invalid & "Tidak dilanjutkan" |
| 4 | Brand Talenta Juara: logo, sidebar `#00A7E1`, warna mengikuti sekolah terpilih |
| 5 | Teks menu putih; sidebar Kreativa `#5001B2` |
| 6 | Layout dua kolom ala shadcn, dropdown pindah sekolah di atas menu, breadcrumb & top bar |
| 7 | Halaman baru: Transaksi, Psikotes, Psikolog, Kelas, Siswa, Guru + menu berkelompok |
| 8 | Penyesuaian spasi & ukuran font sidebar dan konten; perbaikan warna dropdown simulasi |
| 9 | Panel detail hasil psikotes dari nama siswa |
| 10 | Detail pendaftar mengikuti semua field form Odoo (120 field) |
| 11 | Detail pendaftar disusun ulang menjadi 7 tab berbasis alur kerja + kelengkapan data + filter "Belum diisi saja" |
| 12 | Navigasi "Kembali ke …" di detail |
| 13 | Daftar sekolah di dropdown: font 13 px, weight normal |
| 14 | Data kelas asli Odoo: KG Kindergarten, KG Primary, SMP TJ Bandung, SD TJ Pekanbaru, SD TJ Cimahi, SD TJ Bandung; level Kindergarten 1–2 |
| 15 | Perbaikan tab detail mobile yang tertimpa tombol |
| 16 | Pilihan tampilan **Tabel / Kartu** di semua halaman data |

---

*Dokumen ini disusun dari prototipe interaktif di canvas "Edu Hub Admission Redesign". Semua angka dan nama di data contoh bukan data produksi, kecuali yang disebut di §22.1.*
