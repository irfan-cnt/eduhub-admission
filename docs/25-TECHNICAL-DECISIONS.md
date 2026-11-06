# 25 · Technical Decisions (ADR)

Format: Konteks · Keputusan · Alasan · Alternatif · Konsekuensi · Status

## ADR-001 · Supabase penuh menggantikan Odoo

- **Konteks:** brief awal menyimpan data di Odoo 16. Pemilik produk memilih Supabase penuh (5 Nov 2026).
- **Keputusan:** Supabase menjadi satu-satunya backend (Postgres, Auth, Storage). Odoo hanya sumber migrasi satu kali.
- **Alasan:** kontrol skema, RLS per sekolah, integrasi Next.js, tidak bergantung pada JSON-RPC Odoo.
- **Alternatif:** (a) Odoo saja; (b) hybrid Odoo + Supabase.
- **Konsekuensi:** perlu script migrasi & periode transisi; modul Odoo lain yang membaca data PSB (mis. akuntansi) harus dievaluasi (`OPEN QUESTION` Q-02).
- **Status:** Diterima.

## ADR-002 · Data sensitif di tabel 1:1 terpisah

- **Keputusan:** kesehatan, keluarga (penghasilan), data pemerintah (NIK, KK, NISN) di tabel sendiri.
- **Alasan:** RLS bekerja per baris, bukan per kolom; pemisahan memungkinkan akses lebih ketat (finance tidak bisa membaca).
- **Alternatif:** column-level privileges / view terfilter (lebih rumit dipelihara).
- **Konsekuensi:** detail memakai beberapa query; insert pendaftar memakai RPC transaksional.
- **Status:** Diusulkan.

## ADR-003 · Sekolah aktif sebagai segmen route

- **Keputusan:** `/[schoolCode]/<halaman>`.
- **Alasan:** sekolah memengaruhi semua data & brand; URL dapat dibagikan; layout bisa melakukan guard sekali.
- **Alternatif:** query `?school=` (prototipe) atau cookie.
- **Status:** Diusulkan.

## ADR-004 · Server Components sebagai sumber data, TanStack Query terbatas

- **Keputusan:** list & detail di-fetch di Server Components; TanStack Query hanya untuk infinite list mobile (dan bila nanti dibutuhkan polling).
- **Alasan:** mengikuti App Router, mengurangi JS di client, RLS tetap di server.
- **Alternatif:** TanStack Query di semua halaman (lebih banyak client JS).
- **Status:** Diusulkan.

## ADR-005 · URL state dengan nuqs

- **Keputusan:** filter/tab/view/page/detail di search params via `nuqs`.
- **Alasan:** NFR-09 (URL dapat dibagikan), type-safe parser.
- **Alternatif:** `useSearchParams` manual.
- **Konsekuensi:** dependency tambahan (kecil).
- **Status:** Diusulkan.

## ADR-006 · Derivasi status di SQL **dan** domain TS

- **Keputusan:** `late/stale/issues/steps` dihitung di view SQL (untuk filter & count server-side) dan di `lib/domain` (untuk tampilan/tooltip); keduanya diuji dengan dataset yang sama.
- **Alasan:** filter "Terlambat" harus akurat dengan paging server-side.
- **Alternatif:** kolom tersimpan yang diperbarui cron (bisa basi).
- **Status:** Diusulkan.

## ADR-007 · `stage_entered_at` ditambahkan

- **Konteks:** Odoo tidak punya tanggal masuk tahap; "lama di tahap" butuh itu.
- **Keputusan:** kolom baru + trigger; untuk data migrasi diisi dari tracking Odoo (`mail.tracking.value`) bila ada, selain itu `write_date`.
- **Status:** Diusulkan.

## ADR-008 · Status psikotes & wawancara dari tabel sendiri

- **Keputusan:** sumber tunggal `psychology_tests` & `parent_interviews` (K-04).
- **Status:** Diusulkan.

## ADR-009 · Tidak ada REST API publik

- **Keputusan:** Server Actions + Route Handlers khusus (export, callback, webhook).
- **Alasan:** aplikasi internal, permukaan serangan lebih kecil.
- **Status:** Diusulkan.

## ADR-010 · Library tambahan

| Library | Alasan | Status |
|---|---|---|
| shadcn/ui | Komponen aksesibel yang bisa diubah, cocok dengan token CSS var | Diusulkan |
| TanStack Table | Sort/seleksi/kolom | Diusulkan |
| Recharts | Line chart tren (hanya 1 chart) | Diusulkan; alternatif SVG manual |
| dnd-kit | Kanban + keyboard sensor | Diusulkan |
| Zod | Validasi bersama client/server | Diusulkan |
| nuqs | ADR-005 | Diusulkan |
| exceljs (server) | Export .xlsx | Diusulkan |
| sonner | Toast | Diusulkan |

Versi **belum diverifikasi** (tidak ada `package.json`); dicatat di sini saat Fase 1 (Q-12).
