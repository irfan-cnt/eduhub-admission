# 19 · Security Requirements

Aplikasi ini mengolah **data pribadi anak dan keluarga** (NIK, KK, kesehatan, penghasilan, alamat). Standar acuan: UU No. 27/2022 tentang Pelindungan Data Pribadi (UU PDP). `DECISION REQUIRED`: tunjuk penanggung jawab data di sekolah.

## 1. Environment variables

| Variabel | Lingkup | Rahasia |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | client + server | tidak |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` (atau publishable key, sesuai versi Supabase) | client + server | tidak (dilindungi RLS) |
| `SUPABASE_SERVICE_ROLE_KEY` | **hanya** Edge Functions & script migrasi | 🔴 ya |
| `WHATSAPP_PROVIDER_API_KEY`, `EMAIL_PROVIDER_API_KEY` | Edge Functions | 🔴 ya |
| `WEBHOOK_SIGNING_SECRET` | Route handler webhook | 🔴 ya |
| `ODOO_URL`, `ODOO_DB`, `ODOO_USER`, `ODOO_PASSWORD` | script migrasi lokal saja | 🔴 ya |

Aturan:
- Service role **tidak boleh** ada di Vercel environment aplikasi Next.js. Bila terpaksa (mis. untuk invite), gunakan Edge Function.
- `.env*` masuk `.gitignore`; contoh di `.env.example` tanpa nilai.
- Secret disimpan di Vercel/Supabase dashboard, dirotasi bila ada kebocoran.

## 2. Authorization

- RLS di semua tabel ([13](13-ROW-LEVEL-SECURITY.md)) + pengecekan role di setiap Server Action.
- ID dari client tidak dipercaya: action selalu membaca ulang `school_id` dari DB.
- Data sensitif dipisah ke tabel ber-RLS ketat.
- Super admin dicatat di audit log saat mengakses sekolah yang bukan miliknya (REKOMENDASI).

## 3. Input sanitization

- Semua input lewat Zod; query memakai builder supabase-js (parameterized), tanpa SQL string manual.
- Pencarian: escape wildcard `%`, `_` sebelum `ilike`.
- Output React di-escape otomatis; **dilarang** `dangerouslySetInnerHTML` untuk data pengguna.
- Export Excel: cegah formula injection (awali sel yang dimulai `= + - @` dengan `'`).

## 4. Rate limiting

| Endpoint | Batas (REKOMENDASI) |
|---|---|
| Login, lupa password | Bawaan Supabase Auth + 5 percobaan/menit/IP |
| `sendReminders` | 200 penerima per aksi, 1.000 per sekolah per jam |
| Export | 10 per pengguna per jam |
| Webhook | Verifikasi signature |

## 5. Upload

MIME dari magic bytes, whitelist ekstensi, batas ukuran, nama objek UUID, bucket privat, signed URL 5 menit, tanpa SVG/HTML. `DECISION REQUIRED`: pemindaian malware.

## 6. OWASP Top 10

| Risiko | Mitigasi |
|---|---|
| A01 Broken Access Control | RLS + cek role di action + test pgTAP per role |
| A02 Cryptographic Failures | HTTPS, cookie Secure/httpOnly, enkripsi at-rest Supabase |
| A03 Injection | Query builder, Zod, escape ilike, CSV/Excel injection guard |
| A04 Insecure Design | Pemisahan data sensitif, prinsip least privilege |
| A05 Security Misconfiguration | Signup publik mati, CSP, header keamanan, bucket privat |
| A06 Vulnerable Components | Dependabot / `pnpm audit` di CI |
| A07 Auth Failures | Supabase Auth, password min 12, rate limit, sesi refresh |
| A08 Integrity Failures | Lockfile, CI build terverifikasi, migrasi lewat repo |
| A09 Logging Failures | Audit log perubahan penting, log error tanpa PII |
| A10 SSRF | Tidak ada fetch URL dari input pengguna; callback `next` hanya path internal |

## 7. Header keamanan

`Content-Security-Policy` (default-src 'self'; img-src 'self' data: https://*.supabase.co; connect-src 'self' https://*.supabase.co), `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` minimal, `Strict-Transport-Security`.
