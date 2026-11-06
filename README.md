# Edu Hub · Modul Admission

Aplikasi internal penerimaan siswa baru (PSB) untuk **Kreativa Global School** dan **Sekolah Talenta Juara**: dashboard, pengelolaan pendaftar dari Leads sampai Tuition, detail 120 field, transaksi, psikotes, psikolog, kelas, siswa, dan guru, dengan tema yang mengikuti brand tiap sekolah.

> Status: **dokumentasi & desain selesai, implementasi belum dimulai.** Lihat [`docs/22-IMPLEMENTATION-ROADMAP.md`](docs/22-IMPLEMENTATION-ROADMAP.md).

## Fitur utama

- Multi-sekolah (9 unit, 2 brand) dengan pergantian logo & warna otomatis
- Dashboard: KPI, funnel konversi, tren, daftar "Perlu tindakan", kuota per level
- Pendaftar: tabel / kartu / kanban, filter & pencarian, progress stepper 5 tahap, penanda data bermasalah, aksi massal, drag & drop pindah tahap
- Detail pendaftar 7 tab dengan penghitung kelengkapan data & upload dokumen
- Transaksi + rekonsiliasi, hasil psikotes, psikolog, kelas, siswa, guru
- Light/dark mode, desktop & mobile, WCAG 2.1 AA

## Tech stack

Next.js (App Router) · TypeScript · Tailwind CSS · shadcn/ui · Supabase (Postgres, Auth, Storage, Edge Functions) · Vitest · Playwright · pgTAP. Versi akan dikunci di `package.json` pada Fase 1.

## Prerequisites

- Node.js LTS + pnpm
- Docker (untuk Supabase lokal)
- Supabase CLI

## Instalasi

```bash
git clone <repo-url> edu-hub-admission
cd edu-hub-admission
pnpm install
cp .env.example .env.local
```

## Environment variables

| Variabel | Keterangan |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | URL proyek Supabase |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Anon/publishable key |
| `SUPABASE_SERVICE_ROLE_KEY` | **Hanya** untuk Edge Functions & script migrasi; jangan diisi di Vercel |

Lihat [`docs/19-SECURITY-REQUIREMENTS.md`](docs/19-SECURITY-REQUIREMENTS.md). Jangan commit nilai asli.

## Development

```bash
supabase start
supabase db reset          # migration + seed contoh
supabase gen types typescript --local > lib/supabase/database.types.ts
pnpm dev                   # http://localhost:3000
```

## Build production

```bash
pnpm build && pnpm start
```

## Testing

```bash
pnpm lint
pnpm typecheck
pnpm test          # unit & komponen
supabase test db   # pgTAP (skema & RLS)
pnpm test:e2e      # Playwright + axe
```

## Supabase setup & migrations

- Skema: [`docs/10-DATABASE-SCHEMA.md`](docs/10-DATABASE-SCHEMA.md); RLS: [`docs/13-ROW-LEVEL-SECURITY.md`](docs/13-ROW-LEVEL-SECURITY.md)
- Buat migration: `supabase migration new <nama>`; terapkan lokal: `supabase db reset`
- Staging dideploy via CI; **production dijalankan manual setelah review**

## Dokumentasi

Mulai dari [`docs/00-DOCUMENTATION-INDEX.md`](docs/00-DOCUMENTATION-INDEX.md). Aturan untuk AI coding agent: [`CLAUDE.md`](CLAUDE.md) dan [`AGENTS.md`](AGENTS.md).

## Deployment

Vercel (Next.js) + Supabase Cloud, region Singapore. Detail: [`docs/23-DEPLOYMENT-AND-DEVOPS.md`](docs/23-DEPLOYMENT-AND-DEVOPS.md).
