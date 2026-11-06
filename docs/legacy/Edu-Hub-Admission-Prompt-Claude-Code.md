# Edu Hub Admission: Prompt Step-by-Step untuk Claude Code

Panduan ini berisi urutan prompt untuk membangun aplikasi **Edu Hub · Modul Admission** (Next.js + Odoo 16) dengan Claude Code. Spesifikasinya mengacu ke file **`Edu-Hub-Admission-Dokumentasi.md`**, disingkat **DOK** di bawah. Nomor § merujuk ke bagian di DOK.

---

## Cara pakai

1. **Satu fase = satu sesi.** Jalankan `/clear` sebelum mulai fase baru supaya konteks bersih. Claude Code tetap tahu aturan proyek dari `CLAUDE.md`.
2. **Mulai dengan plan mode** (tekan `Shift+Tab` dua kali) untuk fase besar. Setujui rencananya dulu, baru biarkan Claude Code menulis kode.
3. **Commit setiap fase selesai**, setelah semua kriteria "Selesai bila" terpenuhi.
4. **Jangan lompat fase.** Fase berikutnya bergantung pada komponen dari fase sebelumnya.
5. Kalau hasilnya meleset, perbaiki dengan prompt pendek yang spesifik (contoh di [Prompt perbaikan](#prompt-perbaikan)), jangan ulang prompt fase dari awal.

### Peta fase

| Fase | Hasil | Perkiraan sesi |
|---|---|---|
| 0 | Repo, DOK, `CLAUDE.md` | 1 |
| 1 | Scaffold Next.js + tooling | 1 |
| 2 | Design tokens & tema per brand | 1 |
| 3 | Tipe data, mock data, lapisan API | 1–2 |
| 4 | Logika turunan (stepper, macet, flag) + unit test | 1 |
| 5 | App shell: sidebar, pindah sekolah, top bar | 1 |
| 6 | Komponen data bersama (toolbar, tabel, kartu, pager, state) | 1–2 |
| 7 | Halaman Pendaftar (tabel, kartu, filter, aksi massal) | 2 |
| 8 | Kanban + drag & drop | 1 |
| 9 | Detail pendaftar (7 tab) | 2 |
| 10 | Dashboard Ringkasan | 1–2 |
| 11 | Transaksi & Psikotes (+ panel detail) | 1–2 |
| 12 | Psikolog, Kelas, Siswa, Guru | 1–2 |
| 13 | Mobile | 1–2 |
| 14 | Aksesibilitas, state layar, E2E test | 1 |
| 15 | Koneksi ke Odoo sungguhan | 2 |
| 16 | Hardening & deploy | 1 |

---

## Fase 0: Persiapan (manual, tanpa prompt)

```bash
mkdir edu-hub-admission && cd edu-hub-admission
git init
mkdir -p docs/assets
# salin file dokumentasi & logo
cp ~/Downloads/Edu-Hub-Admission-Dokumentasi.md docs/
cp ~/Downloads/kreativa-logo.webp docs/assets/
cp ~/Downloads/talenta-juara-logo.webp docs/assets/
claude
```

Lalu kirim prompt ini untuk membuat `CLAUDE.md`:

```text
Buat file CLAUDE.md di root proyek berisi aturan kerja berikut. Jangan menulis kode lain dulu.

# Proyek
Edu Hub · Modul Admission. Frontend Next.js untuk data penerimaan siswa baru yang disimpan di Odoo 16.
Spesifikasi lengkap: docs/Edu-Hub-Admission-Dokumentasi.md (selanjutnya "DOK"). Selalu baca bagian DOK yang relevan sebelum mengerjakan fitur.

# Stack
Next.js 15 App Router, TypeScript strict, Tailwind CSS, shadcn/ui, TanStack Table, TanStack Query, Recharts, dnd-kit, Zod, Vitest + Testing Library, Playwright.

# Aturan
- Bahasa UI: Bahasa Indonesia. Nama kode (variabel, file, komponen): bahasa Inggris.
- Format tanggal "26 Okt 2026, 15:47" (Asia/Jakarta), Rupiah "Rp 12.500.000".
- Semua data lewat lib/api (jangan fetch langsung di komponen). Data contoh hanya di lib/mock.
- Warna hanya lewat CSS variable/token Tailwind dari DOK §20. Jangan hardcode hex di komponen.
- Merah hanya untuk terlambat, data bermasalah, invalid, kelebihan kapasitas (DOK §16.4).
- Status tidak boleh dibedakan oleh warna saja: selalu ada ikon atau teks.
- Aksesibilitas WCAG 2.1 AA: elemen interaktif asli (button/a/input), aria-label untuk tombol ikon, fokus terlihat, target sentuh ≥ 44px di mobile.
- State filter/tab/view/halaman disimpan di URL search params (DOK §23.3).
- Setiap fitur selesai harus lolos: pnpm lint, pnpm typecheck, pnpm test.
- Jangan menambah atau mengurangi field form pendaftar dari DOK §21.

# Perintah
pnpm dev | pnpm lint | pnpm typecheck | pnpm test | pnpm test:e2e
```

---

## Fase 1: Scaffold proyek

```text
Baca CLAUDE.md dan DOK §23.1–23.2.

Siapkan proyek:
1. Next.js 15 (App Router, TypeScript strict, src/ tidak dipakai, alias "@/").
2. Tailwind CSS + shadcn/ui (init, lalu tambahkan: button, input, select, checkbox, dialog, sheet, dropdown-menu, tabs, tooltip, badge, skeleton, toast/sonner, popover, separator, scroll-area).
3. Install: @tanstack/react-table @tanstack/react-query recharts @dnd-kit/core @dnd-kit/sortable zod nuqs date-fns lucide-react.
4. Dev: vitest @testing-library/react @testing-library/jest-dom jsdom @playwright/test prettier eslint-config-prettier.
5. Script package.json: dev, build, lint, typecheck (tsc --noEmit), test (vitest run), test:e2e (playwright test).
6. Buat struktur folder kosong persis seperti DOK §23.2 (file berisi placeholder komponen minimal yang me-render judul halaman).
7. Font "Source Sans 3" via next/font/google, berat 400/500/600/700.
8. Halaman root "/" redirect ke "/dashboard".

Selesai bila: pnpm dev jalan, setiap route di §5.2 bisa dibuka dan menampilkan judulnya, lint + typecheck + test lulus.
```

---

## Fase 2: Design tokens & tema per brand

```text
Baca DOK §4 dan §20.

1. Di app/globals.css, definisikan CSS variable untuk 4 kombinasi:
   [data-theme="light"|"dark"] × [data-brand="kgs"|"tj"], dengan nilai persis dari tabel §20.2–20.4
   (termasuk --side-*, warna semantik gray/blue/green/amber/red, --overlay, --shadow).
2. tailwind.config.ts mengikuti contoh §20.5.
3. lib/brand.ts:
   - type BrandId = "kgs" | "tj"
   - daftar 9 sekolah dari §4.1 (id, name, group, brand, levels, logo)
   - helper getBrand(schoolId)
4. components/providers/BrandProvider.tsx: set data-brand & data-theme di <html> sesuai sekolah aktif (dari URL ?school=) dan preferensi tema (localStorage, dibungkus try/catch, fallback prefers-color-scheme).
5. Halaman /dev/tokens (hanya development): swatch semua token untuk kedua brand dan kedua tema, termasuk badge status dan tombol primer.

Selesai bila: /dev/tokens menampilkan 4 kombinasi dengan benar dan mengganti ?school= mengubah brand.
Cek kontras: teks --on-primary di atas --primary ≥ 4.5:1 untuk keempat kombinasi (tulis unit test kecil yang menghitung rasio kontras).
```

---

## Fase 3: Tipe data, mock data, lapisan API

```text
Baca DOK §16, §21, §22, §23.4.

1. lib/types.ts (dengan skema Zod di lib/schemas.ts):
   - Applicant: field list §21.6 + id, regNo, parentName, phone, email, source, officer, stageDate (tanggal masuk tahap), scheduleDate, nominal.
   - ApplicantDetail: SEMUA 120 field form Odoo dari §21.1–21.4 (tipe sesuai: string | boolean | date | number | attachment {name,url}|null). Beri komentar label Odoo asli di setiap field.
   - Transaction, PsychologyResult, Psychologist, ClassRoom, Student, Teacher (kolom sesuai §9–§14).
   - School, Level, Flow ("Leads"|"Register"|"Payment"|"Tuition"|"Waiting List"|"Invalid").
2. lib/mock/: generator data contoh deterministik (seed per sekolah) untuk 9 sekolah, mengikuti konfigurasi di DOK:
   - jumlah per tahap & per tahun ajaran, level & kuota, target tuition, nominal tuition
   - data kelas ASLI dari Odoo untuk 6 sekolah di §22.1. Tanyakan ke saya isi tabel kelasnya sebelum menulis; jangan mengarang data kelas untuk sekolah-sekolah itu.
   - nama orang semuanya karangan, beberapa nama siswa sengaja dikosongkan dan beberapa tuition Rp 0 (untuk menguji flag).
3. lib/api/: interface AdmissionApi dengan method per widget dari §23.4 (listApplicants({filters, sort, page, pageSize}), countByFlow, getApplicant, moveStage, dashboardKpis, funnel, trend, actionItems, levelDistribution, sourceDistribution, listTransactions, listPsychology, getPsychology, listPsychologists, listClasses, listStudents, listTeachers, sendReminder).
   - Implementasi pertama: mockApi (baca lib/mock, simulasi latency 300 ms, dukung paging/sort/filter di sisi "server").
   - Pilih implementasi lewat env NEXT_PUBLIC_DATA_SOURCE=mock|odoo.
4. lib/queries/: hook TanStack Query per method, query key menyertakan schoolId.

Selesai bila: unit test memverifikasi jumlah per tahap mock = angka di DOK untuk KG Secondary (238) dan SMP TJ Bandung (646), serta paging server-side 10/12 per halaman.
```

> Siapkan tabel kelas dari screenshot Odoo untuk ditempel saat Claude Code menanyakannya (Fase 3, poin 2).

---

## Fase 4: Logika turunan + unit test

```text
Baca DOK §7.6 dan §16 dengan teliti.

Buat lib/derive.ts (fungsi murni, tanpa React):
1. deriveSteps(applicant, now, stuckDays=7) → 5 langkah {key, label, state: "done"|"active"|"wait"|"late"|"todo", date?, tip}
2. isLate, isStale (TA lewat & tidak sampai Tuition), isInvalid, allDone
3. dataIssues(applicant) → daftar alasan ("Nama siswa kosong", "Ditandai Invalid", "Nominal tuition Rp 0")
4. daysInStage, daysLabel ("N hari", "Selesai", "Ditutup", "Invalid")
5. primaryAction(applicant) → {short, long} sesuai tabel §7.6
6. statusFilterMatch(applicant, filterKey) untuk semua opsi filter status di §7.1
7. completeness(detail) → per tab baru (§21.5) jumlah terisi/total

lib/format.ts: formatDate, formatDateTime, formatRupiah, formatRupiahShort ("Rp 1,50 M", "Rp 352 jt").

Tulis unit test Vitest yang menutup setiap baris tabel §7.6 dan §16.2–16.3, termasuk kasus tepi: Waiting List tidak pernah "late", pendaftar TA lama tidak ditandai merah, Invalid tidak masuk funnel.

Selesai bila: coverage lib/derive.ts ≥ 95% dan semua test lulus.
```

---

## Fase 5: App shell

```text
Baca DOK §5.1 dan lihat diagram layout-nya.

Bangun app/(admission)/layout.tsx:
1. Sidebar (components/shell/Sidebar.tsx), lebar 272px, bg var(--side-bg), teks putih:
   - Logo sekolah aktif + "Edu Hub · Admission PSB"
   - SchoolSwitcher: tombol (inisial grup, nama grup, nama unit, ikon ⇅) membuka popover daftar berkelompok "Sekolah Talenta Juara" (5) dan "Kreativa Global School" (4), tiap item dengan jumlah pendaftar & centang aktif. Teks item 13px weight normal. Memilih sekolah mengganti ?school=, mereset filter halaman, dan mengganti brand.
   - Menu berkelompok Admission / Akademik / SDM & Mitra dengan jumlah data, item aktif berlatar putih + teks brand. Padding item 10px 14px, min-height 44px, radius 10px, font 14px.
   - Pintasan: Macet > N hari, Data bermasalah, Waiting List/Invalid (hanya bila tahap dipakai sekolah)
   - Profil pengguna di bawah
2. TopBar: tombol collapse sidebar, breadcrumb "Sekolah / Halaman", tombol "Cari pendaftar" (membuka /applicants dan fokus ke input cari; shortcut "/"), notifikasi jumlah macet, toggle tema.
3. Hanya area konten yang scroll; sidebar & top bar tetap. Di < 820px layout bertumpuk.
4. Sidebar collapse disimpan di localStorage (try/catch).

Selesai bila: ganti sekolah antara SMP TJ Bandung dan KG Secondary mengubah logo, warna sidebar, dan warna primer; navigasi keyboard Tab melewati semua item menu dengan fokus terlihat.
```

---

## Fase 6: Komponen data bersama

```text
Baca DOK §15 dan §17.

Buat di components/data/ dan components/states/:
1. DataToolbar: judul, subjudul, slot tombol kanan (ViewToggle, Export, Tambah), tab filter dengan badge jumlah (garis bawah primer), input cari (debounce 250ms), slot filter tambahan, tombol Reset filter (muncul bila ada filter).
2. ViewToggle: segmented control ikon Tabel / Kartu (opsional Kanban), aria-pressed, aria-label; ikon aktif berlatar putih dengan warna primer. Nilai di URL ?view=.
3. DataTable (TanStack Table): header sticky, sort server-side dengan aria-sort, row selection, scroll horizontal di dalam kotak, baris hover.
4. CardGrid: grid auto-fill minmax(290px,1fr).
5. Pager: "Menampilkan a–b dari n · X per halaman", Awal/‹/›/Akhir, scroll konten ke atas saat ganti halaman. Page size 10 (tabel) / 12 (kartu).
6. KpiRow + KpiCard (angka merah bila alert).
7. Badge status (gray/blue/green/amber/red), IssueFlag (ikon segitiga + tooltip, bisa difokus keyboard).
8. EmptyState, ErrorState ("Gagal memuat data dari Odoo" + Coba lagi), NoAccess, skeleton per pola.
9. Hook useDataPage(): menyatukan URL params (tab, q, filters, sort, page, view) dengan nuqs.

Buat halaman /dev/components (development) yang memamerkan semua komponen di keempat kombinasi tema/brand.

Selesai bila: semua komponen punya test render dasar dan bisa dioperasikan dengan keyboard.
```

---

## Fase 7: Halaman Pendaftar

```text
Baca DOK §7 dan §16.

Bangun app/(admission)/applicants/page.tsx memakai komponen Fase 6 dan logika Fase 4:
1. Tab tahapan + jumlah (dari countByFlow dengan filter aktif); tab hanya untuk tahap yang dipakai sekolah.
2. Filter: tahun ajaran (default 2027/2028), level, status (semua opsi §7.1), rentang tanggal, petugas; pencarian nama/orang tua/no. registrasi.
3. Tampilan Tabel: kolom persis §7.2. Komponen ProgressStepper (5 titik, garis antar titik, tooltip per titik dari deriveSteps), kolom "Lama di tahap" berwarna sesuai aturan, sort pada Lama di tahap & Tanggal daftar.
4. Tampilan Kartu: isi persis §7.3.
5. Aksi massal saat ada pilihan: Pengingat WhatsApp, Email, Ubah tahap (dialog konfirmasi), Export Excel, Batal pilih.
6. Tombol aksi utama per baris sesuai primaryAction.
7. Klik nama / tombol aksi membuka detail via ?detail=<id> (drawer dibuat di Fase 9; sementara cukup placeholder Sheet).
8. Semua state (loading/kosong/tidak ada hasil/error) memakai komponen Fase 6.

Selesai bila: dengan data mock KG Secondary, tab menampilkan Leads 11, Register 55, Payment 74, Tuition 87, Waiting List 11 (filter TA "Semua"); URL bisa disalin dan membuka keadaan yang sama; E2E Playwright: cari nama → buka detail → tutup.
```

---

## Fase 8: Kanban + drag & drop

```text
Baca DOK §7.4.

Tambahkan view "kanban" di Pendaftar dengan dnd-kit:
- Satu kolom per tahap, 8 kartu per kolom (urut dari terlama di tahap), link "+N lainnya — buka di tabel" yang pindah ke tabel dengan tab tahap itu.
- Drag kartu ke kolom lain → dialog "Pindahkan tahap?" dengan select tahap tujuan dan catatan bahwa perubahan dicatat di timeline. Konfirmasi memanggil api.moveStage dengan optimistic update + rollback bila gagal, lalu toast.
- Dukung keyboard drag (KeyboardSensor) dan announcement screen reader.

Selesai bila: drag & drop bekerja dengan mouse dan keyboard, dan jumlah di tab ter-update setelah pindah.
```

---

## Fase 9: Detail pendaftar (7 tab)

```text
Baca DOK §8 dan §21 SECARA LENGKAP. Jangan menambah atau mengurangi field.

Bangun components/admission/ApplicantDrawer.tsx (Sheet dari kanan, lebar 780px, tutup dengan X / klik luar / Esc; versi halaman penuh di /applicants/[id]):
1. Header: kode registrasi, avatar inisial, nama (merah miring bila kosong), level · sekolah · TA, tanggal daftar, badge tahap/lama/flag, tombol aksi utama, WhatsApp, Ubah tahap, stepper besar dengan tanggal.
2. Tab: Ringkasan, Profil siswa, Seleksi & pembayaran, Dokumen, Keluarga, Dapodik, Riwayat level. Pengelompokan kartu dan urutan field PERSIS tabel §21.1–21.4 kolom "Tampil di". Tiap tab menampilkan badge x/y terisi.
3. Ringkasan: Kelengkapan data (% + bar per tab, klik pindah tab), Perlu dilengkapi (field penting kosong → klik pindah ke tab terkait), Informasi pendaftaran (12 field) + Chat WhatsApp / Kirim email.
4. Seleksi & pembayaran: timeline 5 tahap (kartu per tahap berisi status, tanggal, field; tahap aktif disorot), kartu Rekening & VA, kartu Notifikasi.
5. Dokumen: progress x/9, No. KK & NIK, 9 kotak dokumen dengan status + tombol Unggah/Ganti.
6. Komponen FieldView yang menampilkan: teks, "Belum diisi" (label putus-putus), badge status, checkbox Ya/Tidak, file (nama file + Unggah/Ganti), foto.
7. Toggle "Semua field / Belum diisi saja" di tab data.
8. Navigasi "← Kembali ke …" berbasis riwayat (antar-tab, dan dari panel Psikotes nanti); riwayat dikosongkan saat drawer ditutup. Tab aktif di URL ?dtab=.
9. Tab Riwayat level: empty state (field belum diketahui).

Tulis test yang memastikan total field yang dirender = 119 dan setiap label Odoo dari §21 muncul tepat satu kali.

Selesai bila: test jumlah field lulus, dan untuk pendaftar tahap Leads/Register/Tuition penghitung kelengkapan masuk akal.
```

---

## Fase 10: Dashboard Ringkasan

```text
Baca DOK §6.

Bangun app/(admission)/dashboard/page.tsx:
1. Pemilih tahun ajaran (2022/2023–2028/2029) di header.
2. 5 KPI (§6.1) dengan rumus persis; panah naik/turun untuk pendaftar baru.
3. Funnel (§6.2): bar horizontal, % kumulatif, teks drop-off, label "drop-off terbesar", Invalid dikecualikan dengan catatan.
4. Tren (Recharts LineChart): toggle Mingguan 12 minggu / Harian 30 hari, garis TA sebelumnya putus-putus, ringkasan teks, aria-label ringkasan data. Untuk TA lampau, jendela waktu berakhir di tanggal pendaftaran terakhir.
5. Perlu tindakan: 3 kolom (§6.4), item klik membuka drawer detail.
6. Distribusi level & kuota (bar ganda) dan sumber pendaftaran (+ % konversi).
7. Skeleton per kartu saat loading; empty state bila TA tanpa pendaftar.

Gunakan warna dari token (primer untuk data utama, sekunder untuk progress/aksen).

Selesai bila: angka KPI dan funnel cocok dengan perhitungan manual dari data mock (tulis test untuk fungsi agregasi).
```

---

## Fase 11: Transaksi & Psikotes

```text
Baca DOK §9 dan §10.

1. /transactions: KPI, tab status (Lunas/Menunggu bayar/Kedaluwarsa), filter metode (pil QRIS/VA BNI/VA BRI), kolom tabel §9 (No. VA monospace, Rekonsiliasi ✓/⚠/—), tampilan Kartu. Jangan tampilkan Product, Unique Code, No Bukti.
2. /psychology: KPI, tab rekomendasi, kolom §10 (skor IQ + bar mini, catatan kosong merah + ⚠), tampilan Kartu.
   - Parser skor IQ dari teks description (contoh: "Lulus dengan skor IQ 92", "lulus dengan skor IQ113", "Lulus skor IQ 94") + unit test.
3. PsychologyDrawer (lebar 600px), dibuka dari nama siswa: header, tombol Unduh PDF / Kirim ke orang tua / "Data pendaftar →", skala IQ 70–140, aspek yang diukur, catatan (blok merah + "Minta catatan" bila kosong), informasi tes.
   - "Data pendaftar →" membuka ApplicantDrawer di tab Seleksi & pembayaran dengan riwayat "Kembali ke Hasil psikotes".

Selesai bila: alur Psikotes → Data pendaftar → Kembali berjalan, dan parser IQ lulus semua contoh.
```

---

## Fase 12: Psikolog, Kelas, Siswa, Guru

```text
Baca DOK §11–§14.

Bangun 4 halaman dengan komponen bersama (semua punya ViewToggle Tabel/Kartu):
1. /psychologists (default Kartu): KPI, tab Semua/Menangani tes/Data belum lengkap, kartu dengan lembaga/telepon/email/jumlah tes, flag data rusak.
2. /classes (default Tabel): KPI (Total kelas per TA, total siswa, tanpa wali, melebihi kapasitas), tab level (hanya level yang punya kelas), filter TA, kolom §12, tombol "Lihat siswa" → /students?class=<nama>&year=<TA>.
3. /students (default Tabel): KPI, tab level, filter status Aktif/Cuti/Mutasi keluar, kolom §13.
4. /teachers (default Kartu): KPI, tab jabatan, kartu dengan jabatan · semua kelas yang diampu (gabungkan guru yang sama di beberapa kelas/TA), email, telepon, badge Online/Offline.

Selesai bila: data kelas 6 sekolah di §22.1 tampil persis seperti data Odoo yang saya berikan di Fase 3 (tulis snapshot test per sekolah).
```

---

## Fase 13: Mobile

```text
Baca DOK §18.

Buat pengalaman mobile (< 640px) tanpa duplikasi logika:
1. App bar: logo sekolah, judul halaman, nama sekolah (tap → bottom sheet pilih sekolah), toggle tema, notifikasi.
2. Bottom nav (bg var(--side-bg)): Ringkasan · Pendaftar · Tindakan. Halaman lain diakses lewat menu "Lainnya" di app bar (sheet berisi menu lengkap).
3. Ringkasan mobile: 4 KPI 2×2, kartu tuition + sisa kuota, funnel, 3 macet teratas.
4. Pendaftar mobile: pencarian, tab pil yang bisa digeser, tombol Filter (bottom sheet + badge jumlah filter), mode Pilih untuk aksi massal (bar aksi di atas bottom nav), kartu dengan stepper mini, "Muat lebih banyak" (15 per batch, infinite query).
5. Tindakan: segmented Macet / Jadwal / Pembayaran.
6. Detail mobile: layar penuh, tab pil sticky dengan tinggi tetap 56px (tidak boleh tertimpa konten), kartu satu kolom, tombol bawah WhatsApp + aksi utama.
7. Halaman data lain di mobile otomatis memakai tampilan Kartu.

Selesai bila: Playwright dengan viewport iPhone 13 (390×844) lulus untuk alur cari → buka detail → pindah tab → kembali; tidak ada scroll horizontal di level halaman.
```

---

## Fase 14: Aksesibilitas, state layar, E2E

```text
Baca DOK §17 dan §19.

1. Tambahkan @axe-core/playwright; jalankan audit di semua route (desktop & mobile, light & dark, kedua brand). Perbaiki semua pelanggaran serious/critical.
2. Pastikan: aria-current pada menu, aria-pressed pada toggle, aria-sort pada kolom, role tab/tablist di detail, dialog modal dengan focus trap & Esc, toast role=status.
3. Tambahkan mode simulasi state via ?sim=loading|empty|error|noaccess (hanya di development) agar setiap state bisa diuji.
4. E2E: ganti sekolah (brand berubah), filter + URL share, bulk action, kanban drag dengan keyboard, detail 7 tab, psikotes → pendaftar → kembali.
5. Laporkan kontras teks putih di sidebar biru Talenta Juara (#00A7E1); jangan ubah warnanya, cukup catat di README sebagai keputusan yang perlu disetujui.

Selesai bila: axe bersih (serious/critical = 0) kecuali catatan kontras sidebar TJ, dan semua E2E lulus.
```

---

## Fase 15: Koneksi ke Odoo

```text
Baca DOK §23.4. Implementasikan lib/api/odooApi.ts yang memenuhi interface AdmissionApi yang sama dengan mockApi.

1. lib/odoo-client.ts: wrapper JSON-RPC (/web/session/authenticate, /web/dataset/call_kw) dengan method searchRead, searchCount, readGroup, read, write, callKw. Timeout 30 detik, retry 1x untuk error jaringan, error ditipe (OdooAuthError, OdooAccessError, OdooTimeoutError).
2. Autentikasi via session Odoo di route handler Next.js (app/api/odoo/[...path]/route.ts) sebagai proxy, sehingga cookie session tidak terekspos ke klien. Kredensial hanya dari env server.
3. Sebelum menulis mapping, TANYAKAN ke saya nama model & field teknis Odoo yang sebenarnya; nama di DOK hanya placeholder. Simpan mapping di lib/odoo/mapping.ts (satu tempat).
4. Map OdooAccessError → state NoAccess, timeout → ErrorState.
5. Semua filter/sort/paging dijalankan di Odoo (domain, order, limit, offset); jumlah tab memakai read_group dengan domain filter aktif.
6. "Lama di tahap" memakai field stage_date. Bila belum ada di Odoo, beri tahu saya dan buat fallback dari mail.tracking.value.
7. Tambahkan test kontrak: mockApi dan odooApi (dengan fixture respons Odoo) menghasilkan bentuk data yang sama (validasi Zod).

Selesai bila: NEXT_PUBLIC_DATA_SOURCE=odoo menampilkan data asli staging untuk satu sekolah, dan semua test lama tetap lulus.
```

> Siapkan sebelum fase ini: URL staging Odoo, database, user API dengan hak baca (dan tulis untuk ubah tahap), serta daftar nama model & field teknis.

---

## Fase 16: Hardening & deploy

```text
1. Optimasi: prefetch data halaman berikutnya, staleTime per query (dashboard 60s, list 30s), virtualisasi baris bila > 100 per halaman (tidak dipakai di default 10/12, siapkan saja).
2. Keamanan: CSP header, rate limit di route proxy Odoo, tidak ada data pribadi siswa di log, sanitasi input pencarian ke domain Odoo.
3. Observability: error boundary per halaman dengan ErrorState, logging error proxy (tanpa PII).
4. README: setup lokal, env vars, cara ganti mock/odoo, keputusan desain terbuka (kontras sidebar TJ, kapasitas kelas, stage_date).
5. Dockerfile multi-stage + GitHub Actions: lint, typecheck, test, e2e, build.

Selesai bila: pipeline CI hijau dan build produksi jalan di container.
```

---

## Prompt perbaikan

Gunakan prompt pendek seperti ini saat ada yang meleset. Selalu sebut lokasi dan hasil yang diharapkan:

```text
Di sidebar, teks menu kurangi 1px dan beri padding 10px 14px. Jangan ubah komponen lain.
```

```text
Di detail pendaftar mobile, deretan tab tertimpa tombol saat scroll. Buat tab sticky dengan tinggi tetap 56px dan z-index di atas konten. Tambahkan test Playwright untuk kasus ini.
```

```text
Bandingkan tab Dokumen dengan DOK §21. Ada field yang hilang atau dobel? Laporkan dulu, jangan langsung ubah.
```

```text
Data kelas SD Talenta Juara Cimahi harus persis tabel ini: [tempel tabel]. Perbarui lib/mock dan snapshot test-nya.
```

---

## Checklist akhir

- [ ] Ganti sekolah mengubah logo, sidebar, dan semua warna aksen (kedua arah)
- [ ] Merah hanya muncul untuk terlambat / bermasalah / invalid / kelebihan kapasitas
- [ ] Semua halaman data punya ViewToggle Tabel/Kartu; pagination 10/12
- [ ] Detail pendaftar menampilkan 119 field (120 field Odoo, NISN sekali)
- [ ] Navigasi "Kembali ke …" bekerja antar-tab dan dari Psikotes
- [ ] Data kelas 6 sekolah sama dengan Odoo
- [ ] URL bisa dibagikan untuk semua filter/tab/view/detail
- [ ] Axe bersih, keyboard-only bisa menyelesaikan semua alur
- [ ] Mobile 390px tanpa scroll horizontal, tab detail tidak tertimpa
- [ ] Mode mock dan Odoo lolos test kontrak yang sama
