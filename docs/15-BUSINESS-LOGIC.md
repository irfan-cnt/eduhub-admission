# 15 · Business Logic

Semua aturan di bawah adalah **FAKTA** dari logika prototipe (`derive()`, `matchStatus()`, `renderVals()`, `extraVM()`), kecuali yang diberi label lain. Implementasi: fungsi murni di `lib/domain/*`, dengan versi SQL yang setara di `applicant_list_v` dan RPC. Keduanya wajib diuji dengan kasus yang sama.

## 1. Tahap (`flow`)

`leads → register → payment → tuition`, cabang `waiting_list` dan `invalid`.

| Flow | Arti (ASUMSI dari data Odoo) |
|---|---|
| leads | Kontak awal, formulir belum diisi |
| register | Formulir diisi, biaya formulir belum dibayar |
| payment | Biaya formulir dibayar, proses seleksi |
| tuition | Diterima, proses/selesai bayar tuition |
| waiting_list | Diterima, menunggu kuota |
| invalid | Data duplikat/tidak valid |

### Transisi tahap

| | Keterangan |
|---|---|
| **FAKTA** | Prototipe mengizinkan pindah ke tahap mana pun, termasuk mundur |
| **REKOMENDASI** | Maju bebas; mundur & ke `invalid` wajib isi catatan; ke `tuition` hanya bila `acceptance_status = accepted` |
| `DECISION REQUIRED` | Q-03: aturan transisi resmi |

Efek setiap perubahan `flow`: `stage_entered_at = now()`, baris baru di `applicant_stage_history`, dan `days_in_stage` kembali ke 0.

Efek tambahan per tahap tujuan (FAKTA prototipe `applyMove`):

| Ke | Psikotes / wawancara | Tuition |
|---|---|---|
| leads / register / payment / invalid | dianggap belum | belum |
| tuition / waiting_list | dianggap selesai | belum lunas |

`DECISION REQUIRED`: apakah pindah tahap mengubah status psikotes/wawancara di DB, atau hanya `flow`. REKOMENDASI: **hanya `flow`**, sedangkan stepper membaca status aslinya.

## 2. Progress stepper

Langkah aktif (`cur`):

| Kondisi | `cur` |
|---|---|
| flow ∈ {leads, register, invalid} | 0 (Bayar Formulir) |
| flow = payment & psikotes belum | 1 |
| flow = payment & psikotes selesai & wawancara belum | 2 |
| flow = payment & keduanya selesai | 3 (Keputusan) |
| flow = tuition & belum lunas | 4 |
| flow = tuition & lunas | 5 (semua selesai) |
| flow = waiting_list | 3 |

State per titik `i`:

| Kondisi | State |
|---|---|
| `i < cur` | done |
| `i = cur` & waiting_list | wait |
| `i = cur` & late | late |
| `i = cur` | active |
| `i > cur` | todo |
| invalid atau stale | semua non-done = todo |

## 3. Macet, ditutup, bermasalah

```text
allDone  = tuition_status = complete
stale    = TA sudah berakhir AND NOT allDone AND flow ≠ invalid
late     = NOT allDone AND flow ∉ {waiting_list, invalid} AND NOT stale AND days_in_stage > stuck_days
issues   = [ nama siswa kosong ] ∪ [ flow = invalid ] ∪ [ (flow = tuition AND NOT allDone OR flow = waiting_list) AND tuition_amount = 0 ]
```

`stuck_days` default **7**, diatur per sekolah per TA (`school_year_settings`).

Label "Lama di tahap": `Selesai` (allDone) · `Ditutup` (stale) · `Invalid` · `N hari` · `N hari · terlambat`.

## 4. Filter status (Pendaftar)

| Key | Label | Kondisi |
|---|---|---|
| late | Terlambat (macet) | `late` |
| issue | Data bermasalah | `issues ≠ ∅` |
| stale | Tidak dilanjutkan (TA lewat) | `stale` |
| form | Formulir belum dibayar | NOT stale AND flow ∈ {leads, register} |
| psy | Psikotes belum selesai | NOT stale AND flow = payment AND psikotes belum |
| itv | Wawancara belum selesai | NOT stale AND flow = payment AND psikotes selesai AND wawancara belum |
| decision | Menunggu keputusan | NOT stale AND flow = payment AND keduanya selesai |
| tuition | Tuition belum lunas | NOT stale AND flow = tuition AND NOT allDone |
| done | Tuition lunas | flow = tuition AND allDone |

## 5. Aksi utama per tahap

| Kondisi | Label pendek | Label panjang | Action |
|---|---|---|---|
| invalid | Verifikasi data | Verifikasi Data Pendaftar | buka detail |
| stale | Lihat riwayat | Lihat Riwayat | buka detail |
| leads | Kirim link formulir | Kirim Link Formulir | `sendReminders('form_link')` |
| register | Ingatkan bayar | Kirim Pengingat Pembayaran | `sendReminders('payment_reminder')` |
| payment, psikotes belum | Jadwalkan psikotes | Jadwalkan Psikotes | `scheduleTest` |
| payment, wawancara belum | Jadwalkan wawancara | Jadwalkan Wawancara | `scheduleInterview` |
| payment, tes selesai | Input keputusan | Input Keputusan | `setAcceptanceDecision` |
| tuition, belum lunas | Kirim tagihan | Kirim Tagihan Tuition | `sendReminders('tuition_invoice')` |
| tuition, lunas | Lihat | Lihat Bukti Pembayaran | signed URL bukti |
| waiting_list | Tawarkan kursi | Tawarkan Kursi | `DECISION REQUIRED` alur |

## 6. Dashboard

| Angka | Rumus |
|---|---|
| Total pendaftar | `count(*)` di TA terpilih |
| Baru 7 hari | `created_at ≥ now() − 7d`; pembanding: 7–14 hari lalu |
| Diterima | `flow = tuition` (**FAKTA** prototipe; `OPEN QUESTION` Q-03: hitung juga waiting_list / acceptance_status = accepted?) |
| Konversi | diterima ÷ total × 100, 1 desimal |
| Tuition terkumpul | Σ `tuition_amount` di mana `tuition_status = complete` (REKOMENDASI: idealnya Σ `transactions.amount` type tuition state paid, Q-07) |
| Target | `school_year_settings.tuition_target` |
| Sisa kuota | Σ quota − diterima; per level `max(0, quota_level − diterima_level)`; tampilkan 3 sisa terbesar |
| Funnel | Leads masuk = total − invalid; Register = flow ≠ leads & ≠ invalid; Bayar formulir = flow ∈ {payment, tuition, waiting_list}; Tes & wawancara selesai = flow ∈ {tuition, waiting_list} ∪ (payment & keduanya selesai); Diterima = tuition |
| Drop-off | `(prev − curr) / prev`, terbesar diberi label |
| Tren | Bucket 12 minggu / 30 hari; TA berjalan berakhir hari ini, TA lampau berakhir di pendaftaran terakhir + 1 hari; pembanding = TA sebelumnya pada posisi relatif yang sama (**prototipe memakai angka acak; implementasi wajib data asli**) |
| Sumber | count per `lead_source` + % diterima |

## 7. Modul pendukung

| Modul | Aturan |
|---|---|
| Transaksi | No. registrasi `{reg_prefix}.{YYYYMMDD}.{seq}`. Belum terekonsiliasi = `state = paid AND NOT is_reconciled`. Rekonsiliasi tampil "—" bila state ≠ paid |
| Psikotes | Rekomendasi: `iq ≥ 95 → recommended`, selain itu `considered` (**ASUMSI prototipe**; sebaiknya input psikolog, Q-09). Skor IQ dari data Odoo di-parse dari teks: regex `/IQ\s*(\d{2,3})/i` atau `/skor\s*(\d{2,3})/i` |
| Psikolog | Data belum lengkap = nama tidak valid (< 3 huruf) atau email kosong |
| Kelas | headcount = enrollment `enrolled`; kelebihan = headcount > capacity; tanpa wali = homeroom null. Tab level hanya untuk level yang punya kelas |
| Guru | Satu guru tampil sekali dengan semua kelas yang diampu: `"Kelas A (2025/2026), B (2024/2025)"` |
| Siswa | Status: enrolled = Aktif, on_leave = Cuti, transferred_out = Mutasi keluar |

## 8. Kelengkapan data (detail)

Per tab baru: `terisi / total`. Field "terisi" = teks tidak kosong, boolean selalu terisi, dokumen ada file. Daftar "Perlu dilengkapi" (FAKTA prototipe) adalah field penting yang kosong:

Nama siswa, Foto saat ini, Alamat, Tanggal lahir, Nama sekolah asal, Akta kelahiran, Kartu keluarga, Foto siswa, NIK, No. KK, Rapor sekolah asal, Bukti bayar, Surat konfirmasi, KTP ayah, KTP ibu (maks 8 ditampilkan).

`DECISION REQUIRED`: daftar field wajib per tahap.

## 9. Edge cases

| Kasus | Perilaku |
|---|---|
| Nama siswa kosong | Tampil "(Nama siswa kosong)" merah miring + flag; tetap bisa diproses |
| Nominal tuition Rp 0 di tahap tuition | Flag "Nominal tuition Rp 0", badge "Nominal perlu dicek" |
| Headcount 0 (mis. 9 Ibnu Thufail) | Tampil 0/kapasitas, tidak dianggap error |
| Pendaftar TA lama yang tidak selesai | stale, **tidak** merah, tidak masuk "macet" |
| Pindah ke tahap yang sama | Diabaikan |
| Guru wali di beberapa kelas | Satu baris guru, banyak kelas |
| Sekolah tanpa tahap Waiting List / Invalid | Tab & pintasan tidak muncul |
| Ejaan kelas berbeda antar-TA ("Ibu Majid" vs "Ibnu Majid") | Disimpan apa adanya; laporkan saat migrasi (Q-15) |
