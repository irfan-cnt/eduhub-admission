# 00 · Indeks Dokumentasi

Dokumentasi ini adalah **single source of truth** untuk membangun **Edu Hub · Modul Admission** dengan Next.js (App Router), TypeScript, dan Supabase menggunakan Claude Code.

## Sumber analisis

| Sumber | Lokasi | Keterangan |
|---|---|---|
| Prototipe desktop | `design/Main.dc.html` (canvas Claude Design "Edu Hub Admission Redesign", artboard *Desktop · Admission*) | Sumber utama UI/UX. Fungsi kunci: `SCHOOLS()`, `buildData()`, `buildExtra()`, `derive()`, `extraVM()`, `appForm()`, `appView()`, `renderVals()` |
| Prototipe mobile | `design/Mobile.dc.html` (artboard *Mobile · Admission*) | 390 × 844 px |
| Spesifikasi produk awal | `docs/legacy/Edu-Hub-Admission-Dokumentasi.md` | Ditulis sebelum keputusan Supabase; asumsi Odoo di dalamnya **digantikan** dokumen ini |
| Screenshot Odoo 16 | Diberikan pemilik produk | Form pendaftar (120 field), list Class, Transaction, Psychology, Psychologist, Student, Teacher |

> **Penting:** desain tersedia sebagai prototipe Claude Design (`.dc.html`, HTML + logika JS), **bukan** file TSX. Semua referensi "file sumber" menunjuk ke prototipe tersebut. Belum ada `package.json`, sehingga versi dependency **belum dapat diverifikasi** (lihat [25-TECHNICAL-DECISIONS](25-TECHNICAL-DECISIONS.md)).

## Konvensi penandaan

| Label | Arti |
|---|---|
| **FAKTA** | Ditemukan langsung di prototipe atau screenshot Odoo |
| **ASUMSI** | Disimpulkan, belum dikonfirmasi |
| **REKOMENDASI** | Usulan teknis penulis dokumen |
| `OPEN QUESTION` | Informasi belum tersedia, lihat [24-OPEN-QUESTIONS](24-OPEN-QUESTIONS.md) |
| `DECISION REQUIRED` | Butuh keputusan pemilik produk |

## Daftar dokumen

| # | Dokumen | Fungsi | Status |
|---|---|---|---|
| 00 | [Indeks](00-DOCUMENTATION-INDEX.md) | Peta dokumentasi | ✅ |
| 01 | [Project Overview](01-PROJECT-OVERVIEW.md) | Konteks, masalah, solusi, ruang lingkup | ✅ |
| 02 | [PRD](02-PRD.md) | Requirement, user story, acceptance criteria, MVP | ✅ |
| 03 | [User Roles & Permissions](03-USER-ROLES-AND-PERMISSIONS.md) | Role & permission matrix | ⚠️ role perlu dikonfirmasi |
| 04 | [UI/UX Specification](04-UI-UX-SPECIFICATION.md) | Design system, token, layout, perilaku komponen | ✅ |
| 05 | [Page & Route Inventory](05-PAGE-AND-ROUTE-INVENTORY.md) | Semua halaman & route | ✅ |
| 06 | [Component Inventory](06-COMPONENT-INVENTORY.md) | Komponen, props, dependensi | ✅ |
| 07 | [User Flows](07-USER-FLOWS.md) | Alur pengguna (Mermaid) | ✅ |
| 08 | [Interaction Specification](08-INTERACTION-SPECIFICATION.md) | Setiap tombol, filter, tab, modal | ✅ |
| 09 | [Technical Architecture](09-TECHNICAL-ARCHITECTURE.md) | Next.js + Supabase | ✅ |
| 10 | [Database Schema](10-DATABASE-SCHEMA.md) | Tabel, kolom, SQL migration | ⚠️ draf, beberapa field `OPEN QUESTION` |
| 11 | [Database Relationships](11-DATABASE-RELATIONSHIPS.md) | ERD & kardinalitas | ✅ |
| 12 | [Supabase Authentication](12-SUPABASE-AUTHENTICATION.md) | Login, sesi, proteksi route | ⚠️ tidak ada desain login |
| 13 | [Row Level Security](13-ROW-LEVEL-SECURITY.md) | Policy per tabel | ✅ |
| 14 | [API Specification](14-API-SPECIFICATION.md) | Server Actions & Route Handlers | ✅ |
| 15 | [Business Logic](15-BUSINESS-LOGIC.md) | Aturan, perhitungan, transisi status | ✅ |
| 16 | [File Storage](16-FILE-STORAGE.md) | Bucket, upload, policy | ✅ |
| 17 | [State Management](17-STATE-MANAGEMENT.md) | URL, server, client state | ✅ |
| 18 | [Validation & Error Handling](18-VALIDATION-AND-ERROR-HANDLING.md) | Validasi & feedback | ✅ |
| 19 | [Security Requirements](19-SECURITY-REQUIREMENTS.md) | Secret, RLS, OWASP | ✅ |
| 20 | [Performance Requirements](20-PERFORMANCE-REQUIREMENTS.md) | Target & strategi | ✅ |
| 21 | [Testing & QA](21-TESTING-AND-QA.md) | Strategi & skenario uji | ✅ |
| 22 | [Implementation Roadmap](22-IMPLEMENTATION-ROADMAP.md) | Fase 0–8 | ✅ |
| 23 | [Deployment & DevOps](23-DEPLOYMENT-AND-DEVOPS.md) | Environment, CI/CD, rollback | ✅ |
| 24 | [Open Questions](24-OPEN-QUESTIONS.md) | Hal yang belum diketahui | 🔴 harus dijawab sebelum Fase 2 |
| 25 | [Technical Decisions](25-TECHNICAL-DECISIONS.md) | ADR | ✅ |
| 26 | [Claude Code Prompts](26-CLAUDE-CODE-PROMPTS.md) | Prompt step-by-step per fase untuk Claude Code | ✅ |
| — | [CHANGELOG](CHANGELOG.md) | Riwayat dokumentasi | ✅ |

File root: [`CLAUDE.md`](../CLAUDE.md) (aturan untuk Claude Code), [`AGENTS.md`](../AGENTS.md) (konvensi kode untuk semua agen & developer), [`README.md`](../README.md).

## Urutan membaca

1. **Semua orang:** 01 → 02 → 24
2. **Desainer / frontend:** 04 → 05 → 06 → 08 → 17
3. **Backend / database:** 09 → 10 → 11 → 13 → 15 → 16
4. **Auth & keamanan:** 03 → 12 → 13 → 19
5. **Sebelum mulai coding:** 22 → 25 → `CLAUDE.md` → `AGENTS.md` → 26 (prompt per fase)

## Status kelengkapan

Dokumentasi siap untuk **Fase 0–1** (setup proyek dan porting UI dengan mock data). **Fase 2 (fondasi Supabase)** baru boleh dimulai setelah pertanyaan bertanda 🔴 di [24-OPEN-QUESTIONS](24-OPEN-QUESTIONS.md) dijawab.
