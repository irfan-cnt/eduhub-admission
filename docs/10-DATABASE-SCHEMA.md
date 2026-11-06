# 10 · Database Schema (Supabase Postgres)

> **Status: DRAF.** Skema diturunkan dari 120 field form Odoo (screenshot), kolom list view Odoo (Class, Transaction, Psychology, Psychologist, Student, Teacher), dan kebutuhan UI prototipe. Kolom yang maknanya belum pasti ditandai `OPEN QUESTION`. Jangan menjalankan migration ke production sebelum [24-OPEN-QUESTIONS](24-OPEN-QUESTIONS.md) dijawab.

## Prinsip

1. PK `uuid` (`gen_random_uuid()`), kecuali tabel lookup kecil.
2. Semua tabel bisnis punya `school_id` (langsung atau lewat parent) untuk RLS.
3. `created_at`, `updated_at` (`timestamptz`, trigger `set_updated_at`), `created_by`, `updated_by` (`uuid → profiles`).
4. Data sensitif dipisah ke tabel 1:1 agar RLS bisa lebih ketat: kesehatan, keluarga, data pemerintah.
5. Status memakai Postgres `enum`. Nilai tampilan (Bahasa Indonesia) dipetakan di aplikasi.
6. Tanpa soft delete, kecuali `applicants.is_active` (field **Active** di Odoo).
7. Derivasi (lama di tahap, macet, kelengkapan) dihitung di **view/RPC**, tidak disimpan.

## Daftar tabel (30)

| # | Tabel | Purpose | Sensitif |
|---|---|---|---|
| 1 | `school_groups` | Grup brand (KGS, TJ) | |
| 2 | `schools` | 9 unit sekolah | |
| 3 | `academic_years` | Tahun ajaran (2022/2023…) | |
| 4 | `levels` | Level per sekolah | |
| 5 | `level_quotas` | Kuota & nominal tuition per level per TA | |
| 6 | `school_year_settings` | Target tuition, biaya formulir, ambang macet per TA | |
| 7 | `profiles` | Profil pengguna (1:1 `auth.users`) | |
| 8 | `user_school_roles` | Role pengguna per sekolah | |
| 9 | `lead_sources` | Sumber info (Website, Instagram, …) | |
| 10 | `applicants` | Pendaftar (inti form) | ⚠ kontak |
| 11 | `applicant_addresses` | Alamat (1:1) | ⚠ |
| 12 | `applicant_previous_schools` | Sekolah asal (1:1) | |
| 13 | `applicant_health` | Kebutuhan khusus, kondisi medis, alergi (1:1) | 🔴 |
| 14 | `applicant_family_members` | Ayah, ibu, wali (1:N) | 🔴 penghasilan |
| 15 | `applicant_emergency_contacts` | Kontak darurat (1:N) | ⚠ |
| 16 | `applicant_government_data` | NIK, KK, NISN, Dapodik (1:1) | 🔴 |
| 17 | `applicant_documents` | Metadata file (13 jenis) | 🔴 |
| 18 | `applicant_stage_history` | Riwayat perubahan tahap | |
| 19 | `psychology_organizations` | Lembaga psikologi | |
| 20 | `psychologists` | Psikolog mitra | |
| 21 | `psychology_tests` | Jadwal & hasil psikotes | 🔴 |
| 22 | `parent_interviews` | Jadwal & hasil wawancara orang tua | |
| 23 | `transactions` | Pembayaran formulir & tuition | |
| 24 | `teachers` | Guru & staf | |
| 25 | `classes` | Rombongan belajar per TA | |
| 26 | `students` | Siswa terdaftar | |
| 27 | `class_enrollments` | Siswa ↔ kelas per TA | |
| 28 | `student_level_history` | Riwayat level (`OPEN QUESTION` Q-08) | |
| 29 | `notification_logs` | Log pengingat WA/email | |
| 30 | `audit_logs` | Audit perubahan penting | |

## Enum

```sql
create type school_brand        as enum ('kgs','tj');
create type app_role            as enum ('school_admin','admission_officer','principal','finance','psychologist','viewer');
create type applicant_flow      as enum ('leads','register','payment','tuition','waiting_list','invalid');
create type candidate_status    as enum ('new_student','transfer_student');          -- OPEN QUESTION: nilai lain di Odoo?
create type gender              as enum ('male','female');
create type payment_status      as enum ('incomplete','complete');                   -- Odoo: Complete/Incomplete
create type acceptance_status   as enum ('not_processed','on_progress','accepted','waiting_list','rejected','invalid'); -- OPEN QUESTION Q-03
create type tuition_status      as enum ('incomplete','complete');
create type payment_method      as enum ('qris','va_bni','va_bri');                  -- OPEN QUESTION: metode lain?
create type transaction_type    as enum ('registration_fee','tuition');
create type transaction_state   as enum ('checkout','paid','expired');               -- 'expired' ASUMSI
create type test_status         as enum ('scheduled','complete','cancelled');
create type psychology_recommendation as enum ('recommended','considered','not_recommended');
create type family_relation     as enum ('father','mother','guardian');
create type life_status         as enum ('living','deceased');
create type document_type       as enum (
  'student_picture','current_photo','birth_certificate','previous_school_report','family_card',
  'letter_of_consent','award_attachment','registration_receipt','confirmation_letter',
  'tuition_receipt','father_id_card','mother_id_card','guardian_id_card');
create type enrollment_status   as enum ('enrolled','on_leave','transferred_out');
create type teacher_position    as enum ('homeroom_teacher','subject_teacher','assistant_teacher','supporting_teacher','vice_principal','principal');
create type notification_channel as enum ('whatsapp','email');
create type notification_status  as enum ('queued','sent','failed');
```

## Tabel

### 1–6. Master sekolah

```sql
create table school_groups (
  id          uuid primary key default gen_random_uuid(),
  code        text not null unique,              -- 'kgs' | 'tj'
  name        text not null,                     -- 'Kreativa Global School'
  brand       school_brand not null,
  logo_path   text,                              -- storage: public-assets/logos/...
  created_at  timestamptz not null default now()
);

create table schools (
  id          uuid primary key default gen_random_uuid(),
  group_id    uuid not null references school_groups(id),
  code        text not null unique check (code ~ '^[a-z0-9-]+$'),  -- 'smp-tj-bdg'
  name        text not null,
  city        text,
  reg_prefix  text not null,                     -- prefiks no. registrasi, mis. '16' (dari data Odoo)
  is_active   boolean not null default true,
  created_at  timestamptz not null default now(),
  updated_at  timestamptz not null default now()
);

create table academic_years (
  id          uuid primary key default gen_random_uuid(),
  name        text not null unique check (name ~ '^\d{4}/\d{4}$'),   -- '2027/2028'
  starts_on   date not null,
  ends_on     date not null,
  check (ends_on > starts_on)
);

create table levels (
  id          uuid primary key default gen_random_uuid(),
  school_id   uuid not null references schools(id) on delete cascade,
  name        text not null,                     -- 'Grade 7', 'Kindergarten 1'
  sort_order  smallint not null,
  unique (school_id, name)
);

create table level_quotas (
  id               uuid primary key default gen_random_uuid(),
  school_id        uuid not null references schools(id) on delete cascade,
  academic_year_id uuid not null references academic_years(id),
  level_id         uuid not null references levels(id) on delete cascade,
  quota            integer not null check (quota >= 0),
  tuition_amount   numeric(14,2) check (tuition_amount >= 0),   -- OPEN QUESTION Q-07: per level atau per pendaftar?
  unique (academic_year_id, level_id)
);

create table school_year_settings (
  school_id         uuid not null references schools(id) on delete cascade,
  academic_year_id  uuid not null references academic_years(id),
  tuition_target    numeric(16,2) check (tuition_target >= 0),
  registration_fee  numeric(14,2) not null default 300000,   -- FAKTA: Rp 300.000 di data transaksi
  stuck_days        smallint not null default 7 check (stuck_days between 1 and 60),
  is_admission_open boolean not null default true,
  primary key (school_id, academic_year_id)
);
```

### 7–9. Pengguna & lookup

```sql
create table profiles (
  id             uuid primary key references auth.users(id) on delete cascade,
  full_name      text not null,
  email          text not null,
  phone          text,
  is_super_admin boolean not null default false,
  default_school_id uuid references schools(id),
  last_seen_at   timestamptz,
  created_at     timestamptz not null default now(),
  updated_at     timestamptz not null default now()
);

create table user_school_roles (
  id         uuid primary key default gen_random_uuid(),
  user_id    uuid not null references profiles(id) on delete cascade,
  school_id  uuid not null references schools(id) on delete cascade,
  role       app_role not null,
  created_at timestamptz not null default now(),
  unique (user_id, school_id, role)
);
create index on user_school_roles (user_id);

create table lead_sources (
  id    smallint generated always as identity primary key,
  name  text not null unique      -- Website, Instagram, Referensi orang tua, Open House, WhatsApp (FAKTA prototipe; OPEN QUESTION nilai Odoo)
);
```

### 10. `applicants`

Inti form pendaftar. Kolom yang lain-lain ada di tabel anak.

```sql
create table applicants (
  id                     uuid primary key default gen_random_uuid(),
  school_id              uuid not null references schools(id),
  academic_year_id       uuid not null references academic_years(id),
  level_id               uuid not null references levels(id),
  registration_number    text not null,          -- Odoo "Registration Number" mis. 11.20261026.4839
  is_active              boolean not null default true,        -- Active
  -- General information
  registered_by_name     text not null,          -- Registered By
  email                  text check (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'),
  whatsapp               text check (whatsapp ~ '^62\d{8,13}$'),   -- Whatsapp, format 62…
  lead_source_id         smallint references lead_sources(id),     -- Info
  agreed_terms           boolean not null default false,           -- Agree
  message                text,                                     -- Message
  -- Student candidate
  candidate_status       candidate_status,       -- Student Status
  student_name           text,                   -- Student Name (NULLABLE: data Odoo ada yang kosong)
  gender                 gender,
  place_of_birth         text,
  birth_date             date,
  religion               text,                   -- OPEN QUESTION: daftar nilai
  student_email          text,
  freshman_year          text,                   -- OPEN QUESTION: tipe (tahun / TA?)
  has_scholarship        boolean not null default false,
  daily_language         text,
  pickup_service         boolean,
  extracurricular        text,
  -- Flow
  flow                   applicant_flow not null default 'leads',
  stage_entered_at       timestamptz not null default now(),  -- dibutuhkan untuk "lama di tahap" (tidak ada di Odoo)
  officer_id             uuid references profiles(id),         -- "Petugas" di filter (FAKTA prototipe)
  -- Registration fee
  registration_fee       numeric(14,2),           -- Admission Fee
  payment_method         payment_method,
  payment_status         payment_status not null default 'incomplete',
  payment_date           timestamptz,
  is_settled             boolean not null default false,      -- Settlement
  -- Selection result
  acceptance_status      acceptance_status not null default 'not_processed',
  acceptance_date        date,
  acceptance_note        text,                    -- Note
  reference              text,                    -- Reference
  -- Tuition fee
  confirmation_letter_confirmed boolean not null default false,  -- State Confirmation Letter
  payment_count_selected smallint check (payment_count_selected >= 0),
  tuition_amount         numeric(14,2) check (tuition_amount >= 0),   -- nominal_tuition
  tuition_date           date,
  tuition_status         tuition_status not null default 'incomplete',
  -- Notification
  notify_payment         boolean not null default false,
  notify_selection_result boolean not null default false,
  -- Bank
  bank_name              text,
  bank_account           text,
  virtual_account        text,
  -- meta
  created_at             timestamptz not null default now(),   -- Created at
  updated_at             timestamptz not null default now(),
  created_by             uuid references profiles(id),
  updated_by             uuid references profiles(id),
  unique (school_id, registration_number)
);
create index applicants_list_idx   on applicants (school_id, academic_year_id, flow, created_at desc);
create index applicants_stage_idx  on applicants (school_id, flow, stage_entered_at);
create index applicants_level_idx  on applicants (school_id, level_id);
create index applicants_officer_idx on applicants (officer_id);
create index applicants_search_idx on applicants using gin (
  (coalesce(student_name,'') || ' ' || registered_by_name || ' ' || registration_number) gin_trgm_ops);
```

> Ekstensi `pg_trgm` dibutuhkan untuk pencarian.

### 11–17. Tabel anak pendaftar

```sql
create table applicant_addresses (
  applicant_id uuid primary key references applicants(id) on delete cascade,
  address text, rt text, rw text, zip_code text check (zip_code ~ '^\d{5}$'),
  province text, city text, district text, subdistrict text,
  updated_at timestamptz not null default now()
);

create table applicant_previous_schools (
  applicant_id uuid primary key references applicants(id) on delete cascade,
  school_name text, level_name text, npsn text check (npsn ~ '^\d{8}$'),
  address text, top_awards text,
  updated_at timestamptz not null default now()
);

create table applicant_health (
  applicant_id uuid primary key references applicants(id) on delete cascade,
  special_needs text, medical_condition text, allergy_details text,
  updated_at timestamptz not null default now()
);

create table applicant_family_members (
  id           uuid primary key default gen_random_uuid(),
  applicant_id uuid not null references applicants(id) on delete cascade,
  relation     family_relation not null,
  full_name    text,
  life_status  life_status,          -- hanya ayah/ibu
  occupation   text,                 -- OPEN QUESTION: daftar nilai Odoo (PNS/BUMN/BUMD, Housewife, ...)
  education    text,
  income_range text,                 -- OPEN QUESTION: daftar rentang ("2.100.000 - 5.000.000", "Without Income", ...)
  guardian_relationship text,        -- hanya wali
  updated_at   timestamptz not null default now(),
  unique (applicant_id, relation)
);

create table applicant_emergency_contacts (
  id           uuid primary key default gen_random_uuid(),
  applicant_id uuid not null references applicants(id) on delete cascade,
  full_name    text, phone text, relationship text,
  updated_at   timestamptz not null default now()
);

create table applicant_government_data (
  applicant_id   uuid primary key references applicants(id) on delete cascade,
  nisn text check (nisn ~ '^\d{10}$'), nis text, nipd text,
  nik text check (nik ~ '^\d{16}$'),
  family_card_number text check (family_card_number ~ '^\d{16}$'),   -- Family ID
  birth_certificate_number text,
  kps_number text, kip_number text, un_number text, ijazah_number text,
  kks_number text, pip_number text, skhun text,
  housing_type text, transport_vehicle text,
  home_school_distance_km numeric(6,2) check (home_school_distance_km >= 0),
  child_order smallint check (child_order >= 1),
  sibling_count smallint check (sibling_count >= 0),
  latitude numeric(9,6) check (latitude between -90 and 90),
  longitude numeric(9,6) check (longitude between -180 and 180),
  weight_kg numeric(5,1), height_cm numeric(5,1), head_circumference_cm numeric(5,1),
  updated_at timestamptz not null default now()
);

create table applicant_documents (
  id           uuid primary key default gen_random_uuid(),
  applicant_id uuid not null references applicants(id) on delete cascade,
  school_id    uuid not null references schools(id),          -- denormalisasi untuk RLS storage
  doc_type     document_type not null,
  storage_path text not null unique,                          -- applicant-documents/{school}/{applicant}/{type}/{uuid}.{ext}
  file_name    text not null,
  mime_type    text not null,
  size_bytes   integer not null check (size_bytes > 0 and size_bytes <= 10485760),
  uploaded_by  uuid references profiles(id),
  uploaded_at  timestamptz not null default now(),
  unique (applicant_id, doc_type)                             -- satu file aktif per jenis
);
```

### 18. `applicant_stage_history`

```sql
create table applicant_stage_history (
  id           bigint generated always as identity primary key,
  applicant_id uuid not null references applicants(id) on delete cascade,
  from_flow    applicant_flow,
  to_flow      applicant_flow not null,
  changed_by   uuid references profiles(id),
  changed_at   timestamptz not null default now(),
  note         text
);
create index on applicant_stage_history (applicant_id, changed_at desc);
```

### 19–22. Psikotes & wawancara

```sql
create table psychology_organizations (
  id uuid primary key default gen_random_uuid(),
  name text not null unique
);

create table psychologists (
  id              uuid primary key default gen_random_uuid(),
  full_name       text not null check (length(full_name) >= 3),   -- data Odoo "za" ditolak/ditandai saat migrasi
  organization_id uuid references psychology_organizations(id),
  phone text, email text,
  profile_id      uuid unique references profiles(id),             -- bila psikolog punya akun (Q-04)
  photo_path      text,
  is_active       boolean not null default true,
  created_at timestamptz not null default now(), updated_at timestamptz not null default now()
);
-- Psikolog bersifat lintas sekolah (FAKTA: daftar sama di semua sekolah di Odoo)

create table psychology_tests (
  id               uuid primary key default gen_random_uuid(),
  applicant_id     uuid not null references applicants(id) on delete cascade,
  school_id        uuid not null references schools(id),
  psychologist_id  uuid references psychologists(id),
  status           test_status not null default 'scheduled',
  scheduled_at     timestamptz,
  tested_at        timestamptz,
  iq_score         smallint check (iq_score between 40 and 200),
  recommendation   psychology_recommendation,
  description      text,                 -- Odoo "Description" (catatan psikolog)
  result_file_path text,                 -- Odoo "Attach File"
  result_file_size integer,
  verified_by      uuid references psychologists(id),   -- Odoo "Verified By"
  created_at timestamptz not null default now(), updated_at timestamptz not null default now(),
  check (status <> 'complete' or tested_at is not null)
);
create index on psychology_tests (school_id, tested_at desc);
create index on psychology_tests (applicant_id);

create table parent_interviews (
  id             uuid primary key default gen_random_uuid(),
  applicant_id   uuid not null references applicants(id) on delete cascade,
  school_id      uuid not null references schools(id),
  status         test_status not null default 'scheduled',
  scheduled_at   timestamptz,
  completed_at   timestamptz,
  interviewer_id uuid references profiles(id),
  notes          text,
  created_at timestamptz not null default now(), updated_at timestamptz not null default now()
);
create index on parent_interviews (school_id, scheduled_at);
```

> Aspek psikotes (verbal, numerik, dll.) di prototipe adalah **angka contoh** dan tidak punya field di Odoo, sehingga **tidak** dibuat kolom. `OPEN QUESTION` Q-09.

### 23. `transactions`

```sql
create table transactions (
  id               uuid primary key default gen_random_uuid(),
  school_id        uuid not null references schools(id),            -- Company
  applicant_id     uuid references applicants(id) on delete set null,
  type             transaction_type not null,                       -- ASUMSI: diturunkan dari Product
  product_name     text,                                            -- Product (kosong di data Odoo)
  external_id      text,                                            -- Id Transaction (OPEN QUESTION: berisi nama siswa di screenshot)
  method           payment_method not null,                         -- Payment
  amount           numeric(14,2) not null check (amount >= 0),
  state            transaction_state not null default 'checkout',
  is_reconciled    boolean not null default false,                  -- Counted (ASUMSI makna)
  reconciled_at    timestamptz, reconciled_by uuid references profiles(id),
  proof_number     text,                                            -- No Bukti
  va_number        text,                                            -- No VA
  unique_code      text,                                            -- Unique Code
  paid_by_name     text,                                            -- On Behalf Of
  payment_channel  text,                                            -- Id Via Bayar ('bank')
  paid_at          timestamptz,
  created_at timestamptz not null default now(),                    -- Created at
  updated_at timestamptz not null default now()                     -- Last Updated on
);
create index on transactions (school_id, created_at desc);
create index on transactions (school_id, state, method);
create index on transactions (applicant_id);
```

### 24–28. Akademik

```sql
create table teachers (
  id         uuid primary key default gen_random_uuid(),
  school_id  uuid not null references schools(id),
  full_name  text not null,
  position   teacher_position not null,
  email      text, phone text,
  photo_path text,
  profile_id uuid unique references profiles(id),     -- presence "Online" dari profiles.last_seen_at (Q-11)
  is_active  boolean not null default true,
  created_at timestamptz not null default now(), updated_at timestamptz not null default now()
);
create index on teachers (school_id, position);

create table classes (
  id                  uuid primary key default gen_random_uuid(),
  school_id           uuid not null references schools(id),
  academic_year_id    uuid not null references academic_years(id),
  level_id            uuid not null references levels(id),
  name                text not null,                      -- Level Name, mis. '7 Al Fida', 'K1 Blossom'
  capacity            smallint check (capacity > 0),      -- OPEN QUESTION Q-07: tidak ada di Odoo
  homeroom_teacher_id uuid references teachers(id) on delete set null,
  is_active           boolean not null default true,
  created_at timestamptz not null default now(), updated_at timestamptz not null default now(),
  unique (school_id, academic_year_id, name)
);

create table students (
  id           uuid primary key default gen_random_uuid(),
  school_id    uuid not null references schools(id),
  applicant_id uuid unique references applicants(id) on delete set null,   -- asal pendaftar (bila ada)
  full_name    text not null,
  nis          text,
  nisn         text,
  gender       gender,
  birth_date   date,
  created_at timestamptz not null default now(), updated_at timestamptz not null default now(),
  unique (school_id, nis)
);

create table class_enrollments (
  id               uuid primary key default gen_random_uuid(),
  student_id       uuid not null references students(id) on delete cascade,
  class_id         uuid not null references classes(id) on delete cascade,
  academic_year_id uuid not null references academic_years(id),
  status           enrollment_status not null default 'enrolled',     -- State
  enrolled_at      date not null default current_date,
  unique (student_id, academic_year_id)
);
create index on class_enrollments (class_id, status);

-- ASUMSI, menunggu Q-08 (isi tab Level History Odoo)
create table student_level_history (
  id               uuid primary key default gen_random_uuid(),
  student_id       uuid not null references students(id) on delete cascade,
  academic_year_id uuid not null references academic_years(id),
  level_id         uuid not null references levels(id),
  class_id         uuid references classes(id),
  recorded_at      timestamptz not null default now()
);
```

`Headcount` Odoo **tidak** disimpan, tetapi dihitung dari `class_enrollments` (view `class_overview_v`). Saat migrasi, bila headcount Odoo tidak cocok dengan jumlah siswa yang dimigrasi, catat di laporan migrasi.

### 29–30. Log

```sql
create table notification_logs (
  id           bigint generated always as identity primary key,
  school_id    uuid not null references schools(id),
  applicant_id uuid references applicants(id) on delete set null,
  channel      notification_channel not null,
  template_key text not null,                 -- 'payment_reminder', 'form_link', 'tuition_invoice', 'psychology_result', ...
  recipient    text not null,
  status       notification_status not null default 'queued',
  error        text,
  sent_by      uuid references profiles(id),
  created_at   timestamptz not null default now(),
  sent_at      timestamptz
);

create table audit_logs (
  id          bigint generated always as identity primary key,
  school_id   uuid references schools(id),
  table_name  text not null,
  record_id   uuid not null,
  action      text not null check (action in ('insert','update','delete')),
  changed_by  uuid references profiles(id),
  changed_at  timestamptz not null default now(),
  changed_fields text[] not null default '{}'   -- nama kolom saja; NILAI data sensitif tidak disimpan
);
create index on audit_logs (table_name, record_id, changed_at desc);
```

## View & fungsi turunan

| Objek | Isi | Dipakai |
|---|---|---|
| `applicant_list_v` | `applicants` + nama level, `days_in_stage`, `is_late`, `is_stale`, `issue_flags text[]`, status psikotes & wawancara terakhir, `next_schedule_at` | Tabel/kartu/kanban, panel tindakan |
| `class_overview_v` | `classes` + `headcount`, nama wali kelas | Kelas |
| `teacher_overview_v` | `teachers` + kelas yang diampu (array), `is_online` | Guru |
| `rpc dashboard_kpis(school, year)` | 5 KPI | Dashboard |
| `rpc applicant_funnel(school, year)` | 5 tahap kumulatif | Dashboard |
| `rpc applicant_trend(school, year, granularity)` | seri tahun ini & tahun sebelumnya | Dashboard |
| `rpc move_applicant_stage(ids uuid[], to applicant_flow, note text)` | Update `flow`, reset `stage_entered_at`, tulis history | Ubah tahap |

Semua view dibuat dengan `security_invoker = true` agar RLS tabel dasar berlaku.

Contoh:

```sql
create extension if not exists pg_trgm;

create or replace view applicant_list_v with (security_invoker = true) as
select a.*,
       l.name as level_name,
       ay.name as academic_year_name,
       floor(extract(epoch from (now() - a.stage_entered_at)) / 86400)::int as days_in_stage,
       (a.tuition_status = 'complete') as all_done,
       (ay.ends_on < current_date and a.tuition_status <> 'complete' and a.flow <> 'invalid') as is_stale,
       (a.flow not in ('waiting_list','invalid')
         and a.tuition_status <> 'complete'
         and ay.ends_on >= current_date
         and now() - a.stage_entered_at > make_interval(days => coalesce(s.stuck_days, 7))) as is_late,
       array_remove(array[
         case when a.student_name is null or btrim(a.student_name) = '' then 'missing_student_name' end,
         case when a.flow = 'invalid' then 'invalid' end,
         case when a.flow in ('tuition','waiting_list') and a.tuition_status <> 'complete'
                   and coalesce(a.tuition_amount, 0) = 0 then 'zero_tuition' end
       ], null) as issue_flags
from applicants a
join levels l on l.id = a.level_id
join academic_years ay on ay.id = a.academic_year_id
left join school_year_settings s on s.school_id = a.school_id and s.academic_year_id = a.academic_year_id;
```

## Trigger standar

```sql
create or replace function set_updated_at() returns trigger language plpgsql as $$
begin new.updated_at = now(); return new; end $$;
-- pasang ke setiap tabel yang punya updated_at:
-- create trigger t_upd before update on <table> for each row execute function set_updated_at();

-- security definer: tabel history tidak punya policy INSERT untuk pengguna
create or replace function track_stage_change() returns trigger
language plpgsql security definer set search_path = public as $$
begin
  if new.flow is distinct from old.flow then
    new.stage_entered_at = now();
    insert into applicant_stage_history (applicant_id, from_flow, to_flow, changed_by, note)
    values (new.id, old.flow, new.flow, auth.uid(),
            nullif(current_setting('app.stage_note', true), ''));
  end if;
  return new;
end $$;
create trigger t_stage before update of flow on applicants
for each row execute function track_stage_change();
```

## Pemetaan 120 field

Setiap field form Odoo dipetakan ke satu kolom (atau satu baris `applicant_documents`). Tab tampilan baru sesuai [04](04-UI-UX-SPECIFICATION.md) dan prototipe `appView()`.

### General (46)

| Section Odoo | Field Odoo | Kolom Supabase |
|---|---|---|
| General Information | Active | `applicants.is_active` |
| | Registered By | `applicants.registered_by_name` |
| | Email | `applicants.email` |
| | Whatsapp | `applicants.whatsapp` |
| | Info | `applicants.lead_source_id` |
| | Agree | `applicants.agreed_terms` |
| | Created at | `applicants.created_at` |
| | Company | `applicants.school_id → schools` |
| | School | `applicants.school_id → schools` (`OPEN QUESTION` Q-13: bisa berbeda dari Company?) |
| | Level | `applicants.level_id` |
| | Academic Year | `applicants.academic_year_id` |
| | Message | `applicants.message` |
| Student Candidate | Student Status | `applicants.candidate_status` |
| | NISN | `applicant_government_data.nisn` |
| | Student Name | `applicants.student_name` |
| | Student Gender | `applicants.gender` |
| | Place of Birth | `applicants.place_of_birth` |
| | Birth Date | `applicants.birth_date` |
| Previous School | School Name | `applicant_previous_schools.school_name` |
| | Level | `applicant_previous_schools.level_name` |
| | School NPSN | `applicant_previous_schools.npsn` |
| | School Address | `applicant_previous_schools.address` |
| | Top Awards | `applicant_previous_schools.top_awards` |
| | Award Attachment | `applicant_documents(doc_type='award_attachment')` |
| Address | Address, RT, RW, Zip Code | `applicant_addresses.address, rt, rw, zip_code` |
| | Province, City, District, Subdistrict | `applicant_addresses.province, city, district, subdistrict` |
| Document | Family ID | `applicant_government_data.family_card_number` |
| | NIK | `applicant_government_data.nik` |
| | Student Picture | `applicant_documents('student_picture')` |
| | Birth Certificate | `applicant_documents('birth_certificate')` |
| | Previous School Report | `applicant_documents('previous_school_report')` |
| | Family Card | `applicant_documents('family_card')` |
| | Letter of Consent | `applicant_documents('letter_of_consent')` |
| Other | Flow | `applicants.flow` |
| | Special Needs | `applicant_health.special_needs` |
| | Daily Language | `applicants.daily_language` |
| | Medical Condition | `applicant_health.medical_condition` |
| | Allergy Details | `applicant_health.allergy_details` |
| | Pickup Service | `applicants.pickup_service` |
| | Extracurricular | `applicants.extracurricular` |

### Admission (24)

| Section | Field Odoo | Kolom |
|---|---|---|
| Registration Fee | Registration Number | `applicants.registration_number` |
| | Admission Fee | `applicants.registration_fee` |
| | Payment Receipt | `applicant_documents('registration_receipt')` |
| | Payment Method | `applicants.payment_method` |
| | Payment Status | `applicants.payment_status` |
| | Payment Date | `applicants.payment_date` |
| | Settlement | `applicants.is_settled` |
| Selection Result | Psychology Test | `psychology_tests.status` (terbaru) |
| | Psychology Data | relasi `psychology_tests.id` |
| | Psychology Date | `psychology_tests.tested_at` / `scheduled_at` |
| | Parents Interview | `parent_interviews.status` |
| | Interview Date | `parent_interviews.completed_at` / `scheduled_at` |
| | Acceptance Status | `applicants.acceptance_status` |
| | Acceptance Date | `applicants.acceptance_date` |
| | Note | `applicants.acceptance_note` |
| | Reference | `applicants.reference` |
| Tuition Fee | Confirmation Letter | `applicant_documents('confirmation_letter')` |
| | State Confirmation Letter | `applicants.confirmation_letter_confirmed` |
| | Tuition Receipt | `applicant_documents('tuition_receipt')` |
| | Payment Count Selected | `applicants.payment_count_selected` |
| | Tuition Date | `applicants.tuition_date` |
| | Tuition Status | `applicants.tuition_status` |
| Notification | Send Notification Payment | `applicants.notify_payment` |
| | Send Notification Selection Result | `applicants.notify_selection_result` |

### Student (29)

| Section | Field Odoo | Kolom |
|---|---|---|
| (atas) | Current Photo | `applicant_documents('current_photo')` |
| | Student Email | `applicants.student_email` |
| | Freshman Year | `applicants.freshman_year` |
| | Religion | `applicants.religion` |
| | Scholarship | `applicants.has_scholarship` |
| Document | NISN | `applicant_government_data.nisn` (sama dengan General, **satu kolom**) |
| | NIS, NIPD | `applicant_government_data.nis, nipd` |
| | No KPS, No KIP, No UN, No Ijazah, No KKS, No PIP, SKHUN | `kps_number, kip_number, un_number, ijazah_number, kks_number, pip_number, skhun` |
| | Birth Certificate Number | `applicant_government_data.birth_certificate_number` |
| Bank Information | Bank, Bank Account, Virtual Account | `applicants.bank_name, bank_account, virtual_account` |
| Information for Government | Housing Type, Transport Vehicle, Home-School Distance, Child Order, Number of Siblings, Latitude, Longitude, Weight, Height, Head Circumference | `applicant_government_data.housing_type … head_circumference_cm` |

### Parents (21)

| Section | Field Odoo | Kolom |
|---|---|---|
| Father Info | Father Name, Status, Job, Education, Income | `applicant_family_members(relation='father')` `full_name, life_status, occupation, education, income_range` |
| | Father ID Card | `applicant_documents('father_id_card')` |
| Mother Info | (sama, 6 field) | `relation='mother'` + `mother_id_card` |
| Guardian Info | Guardian Name, Job, Education, Income, Relationship | `relation='guardian'` + `guardian_relationship` |
| | Guardian ID Card | `applicant_documents('guardian_id_card')` |
| Emergency Contact | Name, Phone, Relationship | `applicant_emergency_contacts` |

**Total:** 120 field Odoo → 119 kolom/jenis dokumen yang berbeda (NISN digabung).
