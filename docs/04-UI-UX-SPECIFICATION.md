# 04 · UI/UX Specification

> **Prinsip mempertahankan desain:** prototipe di `design/Main.dc.html` dan `design/Mobile.dc.html` adalah acuan visual final. Implementasi **tidak boleh** mengubah layout, warna, ukuran, urutan, atau perilaku tanpa persetujuan. Setiap penyimpangan dicatat di [25-TECHNICAL-DECISIONS](25-TECHNICAL-DECISIONS.md).

## 1. Brand

| | Kreativa Global School (`kgs`) | Sekolah Talenta Juara (`tj`) |
|---|---|---|
| Unit | Kindergarten, Primary, Secondary, High School | SD Bandung, SD Cimahi, SD Pekanbaru, SMP Bandung, SMP Pekanbaru |
| Logo | `assets/kreativa-logo.webp` (perisai, 400×400) | `assets/tj-logo.webp` (wordmark, rasio ±2,66:1) |
| Sidebar | `#5001B2` | `#00A7E1` |
| Primer | `#5001B2` | `#0077A3` |
| Sekunder | `#EA3B5F` | `#00A7E1` |

Brand ditentukan dari **grup** sekolah aktif (`schools.group_id → school_groups.brand`). Logo dibungkus kotak putih supaya terbaca di mode gelap.

## 2. Design tokens

CSS variable di `:root` dengan atribut `data-theme="light|dark"` dan `data-brand="kgs|tj"`. **FAKTA**: nilai persis dari prototipe.

### 2.1 Netral

| Token | Light | Dark |
|---|---|---|
| `--bg` | `#F6F5F7` | `#151217` |
| `--surface` | `#FFFFFF` | `#1E1A21` |
| `--surface2` | `#F3F1F4` | `#29242C` |
| `--border` | `#E3DFE5` | `#3A343E` |
| `--border-strong` | `#A39BA8` | `#7A7280` |
| `--text` | `#1E1A21` | `#F2EFF3` |
| `--muted` | `#5F5866` | `#B3ABB8` |
| `--overlay` | `rgba(20,16,22,.45)` | `rgba(0,0,0,.6)` |
| `--shadow` | `0 1px 2px rgba(30,26,33,.06), 0 8px 24px rgba(30,26,33,.08)` | `0 8px 24px rgba(0,0,0,.4)` |

### 2.2 Brand

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
| `--side-muted` | `rgba(255,255,255,.82)` | `rgba(255,255,255,.8)` | `rgba(255,255,255,.9)` | `rgba(255,255,255,.82)` |
| `--side-active-bg` | `#FFFFFF` | `#B48CFF` | `#FFFFFF` | `#00A7E1` |
| `--side-active-fg` | `#5001B2` | `#1A0640` | `#005F82` | `#04222E` |
| `--side-hover` | `rgba(255,255,255,.14)` | `rgba(255,255,255,.1)` | `rgba(255,255,255,.18)` | `rgba(255,255,255,.08)` |

### 2.3 Semantik status

| Token | Light fg / bg | Dark fg / bg | Dipakai untuk |
|---|---|---|---|
| gray | `#4B5563` / `#EEF0F3` | `#C3C9D1` / `#2B2F36` | Belum, netral, kedaluwarsa |
| blue | `#1D4ED8` / `#E7EEFD` | `#9DBBFF` / `#1D2A47` | Berjalan, terjadwal |
| green | `#15803D` / `#E2F3E7` | `#86D9A6` / `#17301F` | Selesai, lunas, cocok |
| amber | `#8A4B08` / `#FCF0DA` | `#F4C97E` / `#3A2B12` | Menunggu, Waiting List |
| red | `#B42318` / `#FDE9E7` | `#FFA39B` / `#3F1D1A` | **Hanya** terlambat, bermasalah, invalid, kelebihan kapasitas |

## 3. Typography

| Elemen | Ukuran / berat |
|---|---|
| Font family | `'Source Sans 3', 'Source Sans Pro', system-ui, sans-serif` |
| Teks dasar konten | 14 px / 400, line-height 1.5 |
| Menu sidebar | 14 px; label grup 11 px / 700 uppercase, letter-spacing .07em |
| Item dropdown sekolah | 13 px / 400 |
| Judul halaman | 25 px / 700, letter-spacing −0.01em |
| Judul kartu | 15–16 px / 700 |
| Angka KPI | 27–29 px / 700 |
| Isi tabel | 13–14 px; header 12–13 px / 600 muted |
| Badge | 11–12 px / 600 |

## 4. Spacing, radius, elevasi

| Token | Nilai |
|---|---|
| Grid | 4/8 px |
| Padding konten | 24 px 32 px 48 px (desktop) |
| Gap antar kartu | 16 px |
| Radius | 6 (input kecil), 8 (tombol), 10 (item menu), 12 (kartu/panel), 999 (badge/pil) |
| Tinggi kontrol | 40 px (input/tombol), 36 px (pager), 44 px (item menu, target sentuh mobile) |

## 5. Layout

### Desktop (≥ 820 px)
- Dua kolom: **sidebar 272 px** (warna brand) + area kanan
- Area kanan: **top bar 64 px** + konten. Hanya konten yang scroll (`.shell { height:100vh; overflow:hidden }`)
- Sidebar bisa disembunyikan dari tombol di top bar

### < 820 px
Layout bertumpuk (sidebar di atas, konten di bawah, scroll normal).

### Mobile (390 px), artboard terpisah
App bar 64 px, konten scroll, bottom nav 72 px. Detail berupa layar penuh. Filter, pilih sekolah, dan ubah tahap berupa bottom sheet.

## 6. Perilaku komponen UI

| Komponen | Perilaku |
|---|---|
| Item menu aktif | Bg `--side-active-bg`, teks `--side-active-fg`, weight 700 |
| Tab filter | Garis bawah 2 px primer, label + badge jumlah (aktif: bg primer) |
| Pilihan tampilan | Segmented control ikon; aktif bg surface + warna primer + shadow kecil |
| Tombol primer | Bg primer, hover `--primary-hover` |
| Tombol sekunder | Border, bg surface, hover surface2 |
| Baris tabel | Hover surface2; terpilih primary-soft |
| Badge status | Bg + fg semantik; selalu berisi teks |
| Stepper | Titik 20 px (tabel), 28 px (detail), garis 12–14 px, hijau bila selesai |
| Tooltip | Muncul saat hover **dan** focus |
| Slide-over | Overlay, panel kanan 780 px (pendaftar) / 600 px (psikotes), sticky tab bar |
| Toast | Bawah tengah, ±2,8 detik |
| Skeleton | Shimmer gradient surface2 → border |
| Field kosong (detail) | Label putus-putus "Belum diisi" |

## 7. Responsive behavior

| Breakpoint | Perilaku |
|---|---|
| ≥ 1280 px | Grid KPI 5 kolom, dashboard 2 kolom |
| 820–1279 px | Grid `auto-fit` menyesuaikan; tabel scroll horizontal di dalam kotak |
| < 820 px | Sidebar bertumpuk |
| ≤ 640 px | Pengalaman mobile (bottom nav, kartu, bottom sheet). `DECISION REQUIRED`: breakpoint pasti antara desktop dan artboard mobile |

## 8. Aksesibilitas

- Kontras ≥ 4,5:1, **kecuali** teks putih di sidebar TJ `#00A7E1` (±2,7:1). `DECISION REQUIRED` (lihat Q-10)
- Status selalu punya ikon/teks; bentuk titik stepper berbeda per state
- Elemen asli: `button`, `a`, `input`, `select`, `label`; `aria-label` untuk tombol ikon; `aria-pressed`, `aria-current`, `aria-sort`, `role=tab`, `role=dialog` + `aria-modal`, `role=status` untuk toast
- Fokus terlihat: `outline: 2px solid var(--primary); outline-offset: 2px`
- `Esc` menutup panel dan dialog

## 9. Format konten

| Jenis | Format |
|---|---|
| Tanggal + jam | `26 Okt 2026, 15:47` |
| Tanggal | `26 Okt 2026` |
| Rupiah | `Rp 12.500.000` |
| Rupiah ringkas | `Rp 1,50 M`, `Rp 352 jt` |
| Persen | `29,9%` (koma desimal) |
