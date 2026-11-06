# CHANGELOG · Dokumentasi

## 2026-11-06 · v1.1.0

**Perubahan:** tambah [26-CLAUDE-CODE-PROMPTS](26-CLAUDE-CODE-PROMPTS.md), berisi prompt step-by-step Fase 0–8 dengan gerbang, kriteria selesai, prompt perbaikan, review, dan sinkronisasi dokumentasi.
**Alasan:** panduan eksekusi roadmap dengan Claude Code.

## 2026-11-05 · v1.0.0

**Perubahan:** dokumentasi lengkap dibuat (docs 00–25, `CLAUDE.md`, `AGENTS.md`, `README.md`).
**Alasan:** menjadi single source of truth sebelum implementasi Next.js + Supabase.
**Keputusan terkait:** ADR-001 (Supabase penuh).

### Riwayat desain yang menjadi dasar (prototipe Claude Design)

| Tanggal | Iterasi |
|---|---|
| 27 Okt 2026 | Prototipe awal (dashboard, pendaftar, drawer, mobile, state, dark mode) |
| 27 Okt | Brand Kreativa (`#5001B2`, `#EA3B5F`, Source Sans, logo) |
| 27 Okt | 10 baris/halaman, header & menu tetap, 9 sekolah, tahap Invalid & "Tidak dilanjutkan" |
| 27 Okt | Brand Talenta Juara (`#00A7E1`), tema mengikuti sekolah |
| 27 Okt | Teks menu putih, sidebar KGS `#5001B2` |
| 27 Okt | Layout 2 kolom ala shadcn, dropdown pindah sekolah |
| 27 Okt | Modul Transaksi, Psikotes, Psikolog, Kelas, Siswa, Guru |
| 27 Okt | Spasi & ukuran font; panel detail psikotes |
| 28 Okt | Detail pendaftar 120 field → 7 tab berbasis alur kerja |
| 28 Okt | Navigasi "Kembali ke …", data kelas asli 6 sekolah, level Kindergarten 1–2 |
| 29 Okt | Perbaikan tab mobile tertimpa; pilihan tampilan Tabel/Kartu di semua halaman data |

### Checklist validasi dokumentasi (v1.0.0)

- [x] Semua halaman UI tercatat (P1–P10 + 4 layar mobile) di [05](05-PAGE-AND-ROUTE-INVENTORY.md)
- [x] Halaman yang dibutuhkan tetapi belum didesain tercatat (N1–N15)
- [x] Semua fitur UI tercatat di [02](02-PRD.md) (FR-01…FR-56) + kebutuhan tambahan (FR-90…FR-97)
- [x] Interaksi penting tercatat di [08](08-INTERACTION-SPECIFICATION.md)
- [x] 120 field form dipetakan ke skema ([10](10-DATABASE-SCHEMA.md#pemetaan-120-field))
- [x] Relasi & ERD konsisten dengan skema ([11](11-DATABASE-RELATIONSHIPS.md))
- [x] RLS dibahas untuk semua tabel ([13](13-ROW-LEVEL-SECURITY.md))
- [x] API sesuai kebutuhan halaman ([14](14-API-SPECIFICATION.md))
- [x] Roadmap berurutan sesuai dependency ([22](22-IMPLEMENTATION-ROADMAP.md))
- [x] Asumsi ditandai & dikumpulkan di [24](24-OPEN-QUESTIONS.md)
- [x] Diagram Mermaid memakai sintaks standar (flowchart, sequenceDiagram, stateDiagram-v2, erDiagram)
- [x] Tautan internal relatif
- [ ] Disetujui pemilik produk
