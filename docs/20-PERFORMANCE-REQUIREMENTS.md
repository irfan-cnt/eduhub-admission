# 20 · Performance Requirements

## Target

| Metrik | Target (p75, 4G) |
|---|---|
| LCP | ≤ 2,5 s |
| INP | ≤ 200 ms |
| CLS | ≤ 0,1 |
| TTFB halaman list | ≤ 600 ms |
| Query list pendaftar (10 baris + total) | ≤ 150 ms di DB |
| RPC dashboard | ≤ 300 ms |
| JS first load per route | ≤ 200 KB gzip (Recharts & dnd-kit di-lazy load) |

Volume acuan (**FAKTA** data Odoo): ±650 pendaftar per sekolah terbesar (SMP TJ Bandung) dan ±1.600 transaksi lintas sekolah. REKOMENDASI desain untuk 10× volume tersebut.

## Strategi

| Area | Strategi |
|---|---|
| Database | Index di [10](10-DATABASE-SCHEMA.md) (komposit `school_id, academic_year_id, flow, created_at`; trigram pencarian). Paging server-side `range()`. Agregasi dashboard di RPC. `count: 'exact'` hanya pada query total (atau `estimated` bila > 10.000) |
| Query | Select kolom yang dibutuhkan saja; detail memuat tabel anak paralel (`Promise.all`) |
| Rendering | Server Components; widget dashboard di `<Suspense>` terpisah agar muncul bertahap |
| Bundle | `next/dynamic` untuk TrendChart, KanbanBoard, drawer; ikon lucide per-import |
| Gambar | Logo via `next/image` (ukuran tetap); foto staf & dokumen lewat signed URL dengan thumbnail ≤ 200 px |
| Font | `next/font` (self-host, `display: swap`) |
| Interaksi | Debounce pencarian 250 ms; optimistic update untuk ubah tahap |
| Prefetch | `<Link prefetch>` untuk menu; halaman berikutnya list di-prefetch saat hover pager |
| Export | > 2.000 baris diproses di Edge Function lalu dikirim sebagai link unduhan |
