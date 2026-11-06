# CLAUDE.md

> File ini berisi **aturan perilaku** untuk Claude Code: apa yang boleh dan tidak boleh dilakukan. Konvensi kode (penamaan, struktur folder, pola komponen) ada di [`AGENTS.md`](AGENTS.md). Jika ada yang terasa bertentangan, **CLAUDE.md menang untuk soal izin & keamanan**, sedangkan **AGENTS.md menang untuk soal gaya kode**.

## Project Context

**Edu Hub · Modul Admission**: aplikasi internal penerimaan siswa baru untuk 9 unit sekolah dalam 2 grup brand (Kreativa Global School, Sekolah Talenta Juara).

Stack: Next.js App Router · TypeScript strict · Tailwind CSS · shadcn/ui · Supabase (Postgres, Auth, Storage, RLS, Edge Functions) · Vitest · Playwright · pgTAP.

Sumber desain: `design/Main.dc.html` (desktop) dan `design/Mobile.dc.html` (mobile), yaitu prototipe Claude Design, **bukan** TSX.

## Documentation Rules

- **Baca `docs/00-DOCUMENTATION-INDEX.md` sebelum mengerjakan fitur apa pun.**
- Baca dokumen yang relevan sebelum coding (mis. Pendaftar → 05, 08, 14, 15).
- Dokumentasi adalah acuan implementasi. Bila kode dan dokumentasi berbeda, **berhenti dan tanyakan**.
- Bila implementasi mengubah perilaku, skema, atau keputusan: perbarui dokumen terkait + `docs/CHANGELOG.md` di commit yang sama.
- Hal yang belum jelas → tambahkan ke `docs/24-OPEN-QUESTIONS.md`; jangan memutuskan sendiri.

## UI/UX Rules

- **Jangan redesign.** Layout, warna, ukuran, urutan, dan teks harus mengikuti prototipe.
- Jangan mengubah layout tanpa persetujuan eksplisit.
- Pertahankan responsive behavior (desktop 2 kolom, < 820 px bertumpuk, mobile bottom nav).
- Pertahankan komponen yang sudah tersedia; perluas, jangan duplikasi.
- Warna hanya lewat token (`docs/04` §2). Jangan hardcode hex di komponen.
- Merah hanya untuk terlambat / bermasalah / invalid / kelebihan kapasitas.
- Jangan mengganti mock data sebelum memahami kontrak datanya (`docs/14`).
- Untuk halaman yang **belum didesain** (login, form, pengaturan): buat sketsa memakai komponen & token yang ada, lalu minta persetujuan.

## Development Rules

- TypeScript `strict`, hindari `any` (pakai `unknown` + narrowing).
- Komponen reusable; logika bisnis di `lib/domain` (fungsi murni), bukan di komponen.
- Separation of concerns: `components` ↔ `lib/data` (baca) ↔ `lib/actions` (tulis) ↔ `lib/domain`.
- Validasi dengan Zod di client **dan** server.
- Error handling konsisten dengan `ActionResult` (`docs/14`).
- Hindari duplikasi kode.

## Supabase Rules

- Gunakan pola `@supabase/ssr` yang sesuai versi dependency terpasang; cek dokumentasi versi tersebut sebelum menulis kode auth.
- **RLS wajib** di setiap tabel baru, dengan test pgTAP.
- **Jangan pernah** mengekspos service role key ke client atau environment Next.js.
- Semua perubahan skema melalui file migration di `supabase/migrations/`.
- **Jangan menonaktifkan RLS** atau membuat policy `using (true)` untuk mengatasi error.
- Jangan mengakses data tanpa otorisasi; keputusan otorisasi memakai `auth.getUser()`, bukan `getSession()`.

## Implementation Rules

- Kerjakan per fase (`docs/22`). Jangan melompat fase.
- Jangan mengerjakan semua fitur sekaligus; satu fitur per sesi/PR.
- Setelah perubahan, jalankan: `pnpm lint && pnpm typecheck && pnpm test` (dan `supabase test db` bila menyentuh DB).
- **Jangan mengklaim selesai** sebelum perintah di atas lulus dan acceptance criteria fase terpenuhi.
- Laporkan apa yang belum diverifikasi secara eksplisit.

## Safety Rules

- **Jangan menghapus data** (lokal boleh `supabase db reset`; staging/production tidak).
- **Jangan menjalankan migration ke production.**
- **Jangan deploy production.**
- Jangan mengubah environment variables production.
- Jangan menjalankan script migrasi Odoo ke environment selain lokal tanpa izin.
- Jangan melakukan tindakan destruktif (force push, drop table, hapus bucket, hapus file) tanpa persetujuan eksplisit.
- Jangan menyalin data pribadi asli (nama siswa, NIK, dll.) ke fixture, log, atau test.
