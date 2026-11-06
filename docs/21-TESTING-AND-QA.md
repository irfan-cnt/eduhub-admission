# 21 · Testing & QA

## 1. Lapisan pengujian

| Lapisan | Alat | Cakupan | Target |
|---|---|---|---|
| Unit | Vitest | `lib/domain/*` (stepper, late, stale, issues, filter status, aksi utama, kelengkapan, format, parser IQ) | ≥ 95% line coverage domain |
| Komponen | Vitest + Testing Library | Render & interaksi komponen data, stepper, drawer | semua komponen bersama |
| Database | pgTAP (`supabase test db`) | Constraint, trigger, view turunan, RPC, **RLS per role** | setiap tabel × role × operasi |
| Integrasi | Vitest + Supabase lokal | Server Actions dengan DB lokal | semua action |
| E2E | Playwright | Alur kritis desktop & mobile (390×844) | 10 skenario di bawah |
| Aksesibilitas | @axe-core/playwright | Semua route, light/dark, kedua brand | 0 serious/critical |
| Visual | Playwright screenshot | Bandingkan dengan prototipe | per halaman utama |
| Keamanan | `pnpm audit`, test RLS, ZAP baseline (opsional) | | |

## 2. Skenario unit (domain), wajib

| # | Kasus | Hasil |
|---|---|---|
| U1 | flow=payment, psikotes belum, 3 hari | cur=1, titik 2 active, bukan late |
| U2 | flow=payment, psikotes belum, 10 hari, ambang 7 | titik 2 late, filter `late` cocok |
| U3 | flow=waiting_list, 30 hari | wait, **bukan** late |
| U4 | flow=invalid | semua todo, flag "Ditandai Invalid", bukan late |
| U5 | TA berakhir, flow=register | stale, label "Ditutup", bukan late, filter `stale` |
| U6 | flow=tuition, tuition lunas | allDone, label "Selesai", filter `done` |
| U7 | flow=tuition, belum lunas, amount 0 | flag "Nominal tuition Rp 0" |
| U8 | student_name kosong | flag "Nama siswa kosong" |
| U9 | Tabel aksi utama (10 baris di [15 §5](15-BUSINESS-LOGIC.md#5-aksi-utama-per-tahap)) | label sesuai |
| U10 | Funnel: invalid dikecualikan, drop-off terbesar benar | |
| U11 | Parser IQ: "Lulus dengan skor IQ 92", "lulus dengan skor IQ113", "Lulus skor IQ 94", "Lulus dengan skor 106" | 92, 113, 94, 106 |
| U12 | Format: `Rp 12.500.000`, `Rp 1,50 M`, `26 Okt 2026, 15:47` | |

## 3. Skenario RLS (pgTAP), contoh

| # | Sebagai | Aksi | Harapan |
|---|---|---|---|
| R1 | officer SMP TJ Bandung | select applicants SD TJ Cimahi | 0 baris |
| R2 | finance | select applicant_family_members | 0 baris |
| R3 | finance | select applicant_documents type `birth_certificate` | 0 baris; `registration_receipt` terlihat |
| R4 | viewer | update applicants | ditolak |
| R5 | officer | delete applicants | ditolak |
| R6 | officer | insert applicant_stage_history langsung | ditolak |
| R7 | principal | `move_applicant_stage` ke tuition | sukses |
| R8 | anon | select apa pun | 0 baris / ditolak |
| R9 | officer | upload storage ke folder sekolah lain | ditolak |
| R10 | school_admin | assign role ke sekolah lain | ditolak |

## 4. Skenario E2E

| # | Alur |
|---|---|
| E1 | Login → redirect ke sekolah default → dashboard tampil |
| E2 | Ganti sekolah TJ ↔ KGS: logo, warna sidebar, primer berubah; filter direset |
| E3 | Pendaftar: filter status Terlambat + cari nama → URL disalin → dibuka di tab baru menghasilkan tampilan sama |
| E4 | Pilih 3 baris → Ubah tahap → konfirmasi → jumlah tab berubah, toast muncul |
| E5 | Kanban: drag kartu dengan mouse **dan** keyboard → dialog → pindah |
| E6 | Detail: 7 tab, total field 119, "Belum diisi saja", Perlu dilengkapi → loncat tab → Kembali |
| E7 | Upload dokumen PDF → status Diunggah → buka signed URL |
| E8 | Psikotes → nama → Data pendaftar → Kembali ke Hasil psikotes |
| E9 | Semua halaman data: toggle Tabel/Kartu, pagination 10/12 |
| E10 | Mobile 390×844: cari → detail → tab sticky tidak tertimpa saat scroll → kembali |

## 5. Acceptance testing

Gunakan acceptance criteria di [02-PRD §4](02-PRD.md#4-user-stories--acceptance-criteria). Pemilik produk menguji di staging dengan data hasil migrasi Odoo minimal untuk 2 sekolah (1 per brand).

## 6. Definition of Done per fitur

- [ ] Sesuai prototipe (screenshot dibandingkan)
- [ ] `pnpm lint`, `pnpm typecheck`, `pnpm test` lulus
- [ ] Test RLS untuk tabel yang disentuh
- [ ] Axe bersih untuk halaman yang disentuh
- [ ] Dokumentasi diperbarui (dan CHANGELOG)
