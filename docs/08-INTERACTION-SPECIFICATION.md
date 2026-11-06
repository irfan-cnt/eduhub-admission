# 08 · Interaction Specification

Kolom **Backend** menyebut operasi yang **dibutuhkan** (REKOMENDASI). Di prototipe semua aksi bersifat simulasi: data in-memory, dan aksi kirim/unggah/export hanya menampilkan toast. **Tidak ada backend yang sudah tersedia.**

Nama operasi merujuk ke [14-API-SPECIFICATION](14-API-SPECIFICATION.md).

## 1. Shell

| Elemen | Trigger | Perilaku | State berubah | Backend | Prototipe |
|---|---|---|---|---|---|
| Dropdown sekolah | Click | Buka popover berkelompok; klik luar menutup | `schoolMenu` | `getAccessibleSchools` | `toggleSchoolMenu` |
| Item sekolah | Click | Navigasi ke `/[code]/<halaman sama>`, reset filter & detail, toast "Beralih ke …" | route | — | `u.pick` |
| Item menu | Click | Navigasi halaman, reset filter halaman, scroll ke atas | route | — | `ni.go` |
| Pintasan Macet | Click | Pendaftar `?status=late` | route | — | `goLate` |
| Pintasan Data bermasalah | Click | Pendaftar `?status=issue&year=all` | route | — | `goIssue` |
| Pintasan Waiting/Invalid | Click | Pendaftar `?tab=…` | route | — | `sc.go` |
| Toggle sidebar | Click | Sembunyikan/tampilkan sidebar | localStorage | — | `toggleSide` |
| Cari pendaftar (top bar) | Click / `/` | Buka Pendaftar, fokus ke input cari | route | — | `goSearch` |
| Notifikasi | Click | = Pintasan Macet | route | `countLate` | `goLate` |
| Toggle tema | Click | Light ↔ dark | localStorage | — | `toggleDark` |

## 2. Dashboard

| Elemen | Trigger | Perilaku | Backend |
|---|---|---|---|
| Select tahun ajaran | Change | Muat ulang semua widget untuk TA terpilih | `getDashboard(schoolId, yearId)` |
| Mingguan / Harian | Click | Ganti granularitas tren | `getTrend(…, granularity)` |
| "Lihat pendaftar per tahap →" | Click | Ke Pendaftar | — |
| Item Macet / Jadwal / Pembayaran | Click | Buka detail pendaftar | `getApplicantDetail` |
| "Lihat semua yang macet →" | Click | Pendaftar `?status=late` | — |
| Bar Kelengkapan (detail) | Click | Pindah tab | — |

## 3. Pendaftar

| Elemen | Trigger | Input | Perilaku | Validasi / error | Backend |
|---|---|---|---|---|---|
| Tab tahap | Click | flow | Filter tahap, reset halaman ke 1 | — | `listApplicants`, `countApplicantsByFlow` |
| Cari | Input (debounce 250 ms) | teks | Cari nama siswa / orang tua / no. registrasi | Trim; maks 100 karakter | `listApplicants(q)` |
| Filter TA / level / status / rentang / petugas | Change | nilai | Filter + reset halaman | Nilai harus dari enum | `listApplicants` |
| Reset filter | Click | — | Kembalikan default (TA 2027/2028) | — | — |
| Lama di tahap / Tanggal daftar (header) | Click | — | Toggle sort desc ↔ asc | — | `listApplicants(sort)` |
| ViewToggle | Click | table/card/kanban | Ganti tampilan, halaman ke 1 | — | — |
| Checkbox baris / semua | Change | id | Pilih/batal pilih; "semua" = halaman ini | — | — |
| Pengingat WhatsApp (massal) | Click | ids | Kirim, toast jumlah | Hanya pendaftar dengan nomor WA valid | `sendReminders(channel='whatsapp')` |
| Email (massal) | Click | ids | Kirim, toast | Email valid | `sendReminders(channel='email')` |
| Ubah tahap (massal) | Click | ids | Dialog konfirmasi | — | `moveApplicantStage` |
| Export Excel | Click | filter aktif / ids | Unduh .xlsx | Batas baris (REKOMENDASI 10.000) | `GET /api/export/applicants` |
| Pendaftar baru | Click | — | **Belum didesain** (toast) | — | `createApplicant` |
| Nama / tombol aksi | Click | id | Buka detail `?detail=id&dtab=overview` | 404 → toast | `getApplicantDetail` |
| Ikon ⚠ | Hover / focus | — | Tooltip alasan | — | — |
| Titik stepper | Hover / focus | — | Tooltip tahap · status · tanggal | — | — |
| Pager | Click | page | Pindah halaman, scroll ke atas | Clamp 1..max | `listApplicants(page)` |
| Kartu kanban | Drag → drop | id, flow | Dialog "Pindahkan tahap?" | Drop di kolom sama diabaikan | `moveApplicantStage` |
| "+N lainnya — buka di tabel" | Click | flow | Tabel dengan tab tahap | — | — |
| Dialog: select tahap tujuan | Change | flow | — | Wajib | — |
| Dialog: Ya, pindahkan | Click | — | Optimistic update, toast, tutup | Rollback + toast bila gagal | `moveApplicantStage` |
| Dialog: Batal / overlay / Esc | Click | — | Tutup | — | — |

## 4. Detail pendaftar

| Elemen | Trigger | Perilaku | Backend |
|---|---|---|---|
| Tutup (X / overlay / Esc) | Click / key | Tutup panel, kosongkan riwayat navigasi | — |
| Aksi utama (sesuai tahap) | Click | Lihat tabel aksi di [15](15-BUSINESS-LOGIC.md#5-aksi-utama-per-tahap) | bervariasi |
| WhatsApp | Click | Buka `wa.me/<nomor>` di tab baru (REKOMENDASI) | — |
| Kirim email | Click | `mailto:` atau kirim template | `sendReminders` |
| Ubah tahap | Click | Dialog konfirmasi | `moveApplicantStage` |
| Tab (7) | Click | Pindah tab, catat riwayat untuk "Kembali" | — |
| Kembali ke … | Click | Kembali ke tab/panel sebelumnya | — |
| Semua field / Belum diisi saja | Click | Filter tampilan field | — |
| Item "Perlu dilengkapi" | Click | Pindah ke tab terkait | — |
| Unggah / Ganti (dokumen) | Click → pilih file | Upload, progress, simpan metadata | `uploadApplicantDocument` |
| Edit field | — | **Belum didesain** | `updateApplicantSection` |

## 5. Transaksi, Psikotes, Psikolog, Kelas, Siswa, Guru

| Elemen | Halaman | Perilaku | Backend |
|---|---|---|---|
| Tab status / rekomendasi / level / jabatan | semua | Filter + badge jumlah | `list<Entity>` |
| Pil metode (QRIS/VA BNI/VA BRI) | Transaksi | Filter metode | `listTransactions(method)` |
| Pil TA | Kelas | Filter tahun ajaran | `listClasses(yearId)` |
| Pil status (Aktif/Cuti/Mutasi) | Siswa | Filter status | `listStudents(status)` |
| Cari | semua | Debounce 250 ms | `list<Entity>(q)` |
| ViewToggle Tabel/Kartu | semua | Ganti tampilan; 10/12 per halaman | — |
| Export Excel | semua | Unduh .xlsx | `GET /api/export/<entity>` |
| Tambah … / Input hasil tes | Psikotes, Psikolog, Kelas, Guru | **Belum didesain** | `create<Entity>` |
| Nama siswa | Psikotes | Panel detail psikotes | `getPsychologyTest` |
| Lampiran PDF | Psikotes | Buka signed URL | `getSignedUrl` |
| Unduh hasil (PDF) | Panel psikotes | Unduh lampiran | `getSignedUrl` |
| Kirim ke orang tua | Panel psikotes | Kirim WA/email | `sendPsychologyResult` |
| Minta catatan | Panel psikotes | Notifikasi ke psikolog | `requestPsychologistNote` |
| Data pendaftar → | Panel psikotes | Detail pendaftar tab Seleksi + riwayat kembali | — |
| Lihat siswa | Kelas | Siswa `?q=<kelas>` | — |

## 6. Mobile

| Elemen | Perilaku |
|---|---|
| Bottom nav | Ringkasan / Pendaftar / Tindakan |
| Nama sekolah di app bar | Bottom sheet pilih sekolah |
| Filter | Bottom sheet TA / level / status; "Tampilkan N hasil" |
| Pilih / Selesai | Mode seleksi; bar aksi Ingatkan WA / Ubah tahap |
| Muat lebih banyak | +15 kartu (infinite query) |
| Segmented Tindakan | Macet / Jadwal / Pembayaran |
| Detail: Kembali | Tutup layar detail |
| Tab pil | Sticky, tinggi tetap 56 px |

## 7. Loading, empty, error

| Kondisi | Tampilan | Pemicu |
|---|---|---|
| Loading | Skeleton sesuai pola halaman | Query pending (Suspense / `loading.tsx`) |
| Kosong | EmptyState + CTA | Data 0 tanpa filter |
| Tidak ada hasil | "Tidak ada data yang cocok" + Reset filter | Data 0 dengan filter |
| Error | ErrorState "Gagal memuat data" + Coba lagi | Error jaringan / Supabase (`error.tsx`) |
| Tanpa akses | NoAccessState + "Minta akses ke admin" | Tidak ada role di sekolah (401/403) |
| Sukses aksi | Toast | Server Action sukses |
