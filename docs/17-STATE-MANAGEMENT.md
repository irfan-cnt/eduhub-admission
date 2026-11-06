# 17 · State Management

Di prototipe, semua state ada di satu objek komponen (`S()` di `Main.dc.html`). Di implementasi, state itu dipecah menurut jenisnya:

| Jenis | Contoh (dari prototipe) | Tempat |
|---|---|---|
| **Route** | sekolah aktif (`school`), halaman (`page`) | Segmen path `/[school]/<halaman>` |
| **URL search params** | `tab`, `q`, `year`, `level`, `status`, `range`, `officer`, `view`, `sortKey/sortDir`, `pageNo`, `detailId`, `detailTab`, `dashYear`, `trendMode`, `xf`, `xf2`, `xq`, `xp`, `xView`, `xDetail` | `nuqs` (ADR-005) |
| **Server state** | daftar pendaftar, jumlah tab, KPI, detail | Server Components (fetch per request) |
| **Client server-state** | infinite list mobile, optimistic move | TanStack Query (hanya di bagian ini, ADR-004) |
| **Local UI state** | `selected` (seleksi baris), `modal`, `dragOver`, `schoolMenu`, `dFilter`, `navHist` | `useState` / `useReducer` di komponen |
| **Preferensi** | `dark`, `sideCollapsed` | `localStorage` (try/catch) + cookie `theme` agar SSR tidak berkedip |
| **Global** | user, roles sekolah aktif, brand | React Context dari `[school]/layout.tsx` (read-only) |
| **Dev only** | `sim` (simulasi state) | Dihapus; diganti Storybook / query `?sim=` khusus development |

## Nama param URL

| Param | Nilai | Default | Halaman |
|---|---|---|---|
| `tab` | `all, leads, register, payment, tuition, waiting_list, invalid` | `all` | Pendaftar |
| `q` | string ≤ 100 | — | semua list |
| `year` | `all` \| nama TA | TA admission aktif | Pendaftar, Dashboard |
| `level`, `status`, `range`, `officer` | lihat [14](14-API-SPECIFICATION.md) | `all` | Pendaftar |
| `view` | `table, card, kanban` | per halaman (Guru/Psikolog: `card`) | semua list |
| `sort` | `created.desc`… | `created.desc` | Pendaftar |
| `page` | ≥ 1 | 1 | semua list |
| `f` / `f2` | filter tab / pil modul pendukung | `all` | Transaksi … Guru |
| `detail` | uuid | — | Pendaftar, Psikotes |
| `dtab` | `overview, profil, seleksi, dokumen, keluarga, dapodik, history` | `overview` | Detail |
| `trend` | `week, day` | `week` | Dashboard |

Aturan:
- Mengubah filter apa pun → `page` = 1.
- Pindah sekolah → hapus semua param kecuali `view`.
- `navHist` (riwayat "Kembali ke …") **tidak** masuk URL; dikosongkan saat detail ditutup.

## Cache & sinkronisasi

| Data | Strategi |
|---|---|
| Halaman list & detail | Dinamis per request (data per pengguna/RLS). Tidak pakai `force-cache` |
| Setelah mutasi | `revalidatePath` halaman terkait + jumlah sidebar |
| Optimistic | Ubah tahap (tabel, kanban, massal): `useOptimistic`, rollback bila action gagal |
| Daftar sekolah & role | Dibaca di layout per request; aman di-cache per request dengan `cache()` React |
| TanStack Query (mobile infinite) | `staleTime` 30 detik, query key menyertakan `schoolId` + filter |
| Multi-tab | Tidak ada realtime di MVP; data segar saat navigasi (`router.refresh()` setelah aksi) |
