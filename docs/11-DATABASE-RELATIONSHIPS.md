# 11 · Database Relationships

## ERD: master & akses

```mermaid
erDiagram
    SCHOOL_GROUPS ||--o{ SCHOOLS : memiliki
    SCHOOLS ||--o{ LEVELS : memiliki
    SCHOOLS ||--o{ LEVEL_QUOTAS : menetapkan
    LEVELS ||--o{ LEVEL_QUOTAS : "kuota per TA"
    ACADEMIC_YEARS ||--o{ LEVEL_QUOTAS : berlaku
    SCHOOLS ||--o{ SCHOOL_YEAR_SETTINGS : mengatur
    ACADEMIC_YEARS ||--o{ SCHOOL_YEAR_SETTINGS : berlaku
    PROFILES ||--o{ USER_SCHOOL_ROLES : punya
    SCHOOLS ||--o{ USER_SCHOOL_ROLES : memberi
```

## ERD: pendaftar

```mermaid
erDiagram
    SCHOOLS ||--o{ APPLICANTS : menerima
    ACADEMIC_YEARS ||--o{ APPLICANTS : untuk
    LEVELS ||--o{ APPLICANTS : mendaftar_ke
    LEAD_SOURCES ||--o{ APPLICANTS : sumber
    PROFILES ||--o{ APPLICANTS : petugas
    APPLICANTS ||--|| APPLICANT_ADDRESSES : alamat
    APPLICANTS ||--|| APPLICANT_PREVIOUS_SCHOOLS : sekolah_asal
    APPLICANTS ||--|| APPLICANT_HEALTH : kesehatan
    APPLICANTS ||--|| APPLICANT_GOVERNMENT_DATA : dapodik
    APPLICANTS ||--o{ APPLICANT_FAMILY_MEMBERS : keluarga
    APPLICANTS ||--o{ APPLICANT_EMERGENCY_CONTACTS : kontak_darurat
    APPLICANTS ||--o{ APPLICANT_DOCUMENTS : dokumen
    APPLICANTS ||--o{ APPLICANT_STAGE_HISTORY : riwayat_tahap
    APPLICANTS ||--o{ PSYCHOLOGY_TESTS : psikotes
    APPLICANTS ||--o{ PARENT_INTERVIEWS : wawancara
    APPLICANTS ||--o{ TRANSACTIONS : pembayaran
    APPLICANTS ||--o{ NOTIFICATION_LOGS : pengingat
    PSYCHOLOGISTS ||--o{ PSYCHOLOGY_TESTS : memeriksa
    PSYCHOLOGY_ORGANIZATIONS ||--o{ PSYCHOLOGISTS : menaungi
```

## ERD: akademik

```mermaid
erDiagram
    SCHOOLS ||--o{ TEACHERS : mempekerjakan
    SCHOOLS ||--o{ CLASSES : membuka
    ACADEMIC_YEARS ||--o{ CLASSES : per_TA
    LEVELS ||--o{ CLASSES : tingkat
    TEACHERS |o--o{ CLASSES : wali_kelas
    SCHOOLS ||--o{ STUDENTS : mendidik
    APPLICANTS |o--o| STUDENTS : menjadi
    STUDENTS ||--o{ CLASS_ENROLLMENTS : terdaftar
    CLASSES ||--o{ CLASS_ENROLLMENTS : berisi
    STUDENTS ||--o{ STUDENT_LEVEL_HISTORY : riwayat
    PROFILES |o--o| TEACHERS : akun
    PROFILES |o--o| PSYCHOLOGISTS : akun
```

## Kardinalitas & integritas

| Relasi | Kardinalitas | FK on delete | Catatan |
|---|---|---|---|
| school_groups → schools | 1:N | restrict | Grup menentukan brand |
| schools → levels | 1:N | cascade | |
| levels + academic_years → level_quotas | 1:1 per (TA, level) | cascade | `unique(academic_year_id, level_id)` |
| profiles → user_school_roles | 1:N | cascade | Banyak role per sekolah diizinkan |
| schools → applicants | 1:N | restrict | Tidak boleh hapus sekolah yang punya data |
| applicants → tabel 1:1 (alamat, sekolah asal, kesehatan, pemerintah) | 1:1 | cascade | PK = `applicant_id` |
| applicants → family_members | 1:N, maks 1 per relasi | cascade | `unique(applicant_id, relation)` |
| applicants → documents | 1:N, maks 1 per jenis | cascade | Hapus baris wajib diikuti hapus objek Storage (trigger/Edge Function) |
| applicants → stage_history | 1:N | cascade | Append-only |
| applicants → psychology_tests | 1:N | cascade | Hasil terbaru dipakai di detail |
| applicants → transactions | 1:N | set null | Transaksi keuangan tetap disimpan |
| applicants → students | 1:0..1 | set null | Siswa bisa berasal dari luar sistem PSB |
| teachers → classes (wali) | 1:N | set null | Satu guru boleh wali beberapa kelas lintas TA (FAKTA data Odoo) |
| students + academic_years → class_enrollments | 1 kelas per siswa per TA | cascade | `unique(student_id, academic_year_id)` |
| psychologists → psychology_tests | 1:N | restrict | Psikolog lintas sekolah |

## Konsistensi lintas tabel (wajib ditegakkan)

| Aturan | Mekanisme |
|---|---|
| `applicants.level_id` harus milik `applicants.school_id` | Trigger `check_level_school` |
| `classes.level_id` & `homeroom_teacher_id` harus milik `classes.school_id` | Trigger |
| `psychology_tests.school_id` = `applicants.school_id` | Trigger mengisi otomatis dari applicant |
| `applicant_documents.school_id` = `applicants.school_id` | Trigger mengisi otomatis |
| `class_enrollments.academic_year_id` = `classes.academic_year_id` | Trigger |
