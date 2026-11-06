# 16 · File Storage (Supabase Storage)

## Temuan

| FAKTA | Sumber |
|---|---|
| 13 field file di form pendaftar (dokumen, foto, bukti bayar, surat konfirmasi, KTP orang tua/wali) | Screenshot Odoo, tab Dokumen prototipe |
| Lampiran hasil psikotes (PDF, ±97 KB atau ±845 KB) | List Psychology Odoo |
| Foto guru & psikolog | List Teacher & Psychologist Odoo |
| Logo 2 grup sekolah | Aset prototipe |

## Bucket

| Bucket | Publik | Isi | Path |
|---|---|---|---|
| `public-assets` | ✅ | Logo grup sekolah | `logos/{group_code}.webp` |
| `applicant-documents` | ❌ | 13 jenis dokumen pendaftar | `{school_id}/{applicant_id}/{doc_type}/{uuid}.{ext}` |
| `psychology-results` | ❌ | PDF hasil psikotes | `{school_id}/{applicant_id}/{test_id}.pdf` |
| `staff-photos` | ❌ | Foto guru & psikolog | `teachers/{school_id}/{teacher_id}.{ext}`, `psychologists/{id}.{ext}` |

Segmen pertama path = `school_id`, dipakai policy.

## Validasi file

| Jenis | MIME | Maks |
|---|---|---|
| Foto (student_picture, current_photo, staff) | image/jpeg, image/png, image/webp | 2 MB, sisi terpanjang di-resize ≤ 1200 px |
| Dokumen (akta, KK, rapor, surat, KTP, bukti bayar, prestasi) | application/pdf, image/jpeg, image/png | 10 MB |
| Hasil psikotes | application/pdf | 10 MB |

- Validasi di client (sebelum upload) **dan** di server (MIME dari magic bytes, bukan ekstensi).
- Nama file asli disimpan di `applicant_documents.file_name`; nama objek memakai UUID.
- Tidak ada SVG/HTML yang diterima, untuk mencegah XSS.

## Alur

| Operasi | Alur |
|---|---|
| **Upload** | Server Action `uploadApplicantDocument` menerima `FormData` → validasi → `storage.from('applicant-documents').upload(path)` → upsert `applicant_documents` (jenis yang sama menggantikan file lama, lalu file lama dihapus). Alternatif untuk file besar: `createSignedUploadUrl` lalu client upload langsung |
| **Download / lihat** | `createSignedUrl(path, 300)` (5 menit), dibuat di server setelah cek role |
| **Delete** | Server Action menghapus baris; trigger/Edge Function `cleanup-storage` menghapus objek |

## Storage policy

```sql
-- applicant-documents: baca untuk SENS + finance (bukti bayar), tulis untuk EDIT
create policy "docs read" on storage.objects for select to authenticated using (
  bucket_id = 'applicant-documents'
  and exists (select 1 from applicant_documents d
              where d.storage_path = storage.objects.name)        -- RLS applicant_documents ikut berlaku
);

create policy "docs write" on storage.objects for insert to authenticated with check (
  bucket_id = 'applicant-documents'
  and has_school_role(((storage.foldername(name))[1])::uuid,
                      array['school_admin','admission_officer']::app_role[])
);

create policy "docs delete" on storage.objects for delete to authenticated using (
  bucket_id = 'applicant-documents'
  and has_school_role(((storage.foldername(name))[1])::uuid,
                      array['school_admin','admission_officer']::app_role[])
);
```

Policy serupa berlaku untuk `psychology-results` (role SENS + psikolog yang ditugaskan) dan `staff-photos` (STAFF baca, `school_admin` tulis).

## Migrasi file dari Odoo

`ir.attachment` Odoo diunduh oleh script migrasi (service role) → diunggah ke path di atas → baris `applicant_documents` dibuat. Checksum dicatat di laporan migrasi. `OPEN QUESTION` Q-02: total volume file.
