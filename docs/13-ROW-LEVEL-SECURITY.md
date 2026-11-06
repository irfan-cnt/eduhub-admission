# 13 · Row Level Security

**Aturan:** RLS **aktif di semua tabel** schema `public`. Tidak boleh dimatikan untuk "mengatasi error". Service role hanya dipakai di Edge Functions & script migrasi, tidak pernah di browser.

## 1. Fungsi helper

```sql
create or replace function is_super_admin() returns boolean
language sql stable security definer set search_path = public as $$
  select coalesce((select is_super_admin from profiles where id = auth.uid()), false)
$$;

create or replace function has_school_role(p_school uuid, p_roles app_role[] default null)
returns boolean language sql stable security definer set search_path = public as $$
  select is_super_admin() or exists (
    select 1 from user_school_roles r
    where r.user_id = auth.uid() and r.school_id = p_school
      and (p_roles is null or r.role = any(p_roles)))
$$;

create or replace function applicant_school(p_applicant uuid) returns uuid
language sql stable security definer set search_path = public as $$
  select school_id from applicants where id = p_applicant
$$;

revoke all on function is_super_admin, has_school_role, applicant_school from public;
grant execute on function is_super_admin, has_school_role, applicant_school to authenticated;
```

Singkatan role di tabel:
- **STAFF** = `school_admin, admission_officer, principal, finance, viewer`
- **EDIT** = `school_admin, admission_officer`
- **SENS** (boleh membaca data sensitif) = `school_admin, admission_officer, principal`

## 2. Policy per tabel

| Tabel | SELECT | INSERT | UPDATE | DELETE |
|---|---|---|---|---|
| `school_groups` | authenticated | super admin | super admin | super admin |
| `schools` | `has_school_role(id)` | super admin | super admin / `school_admin` | super admin |
| `academic_years` | authenticated | super admin | super admin | super admin |
| `levels`, `level_quotas`, `school_year_settings` | `has_school_role(school_id)` | `school_admin` | `school_admin` | `school_admin` |
| `profiles` | diri sendiri; atau admin sekolah yang sama | trigger (service) | diri sendiri (kolom terbatas) / super admin | super admin |
| `user_school_roles` | diri sendiri; `school_admin` sekolah tsb | `school_admin` (bukan role > dirinya) | — (hapus + buat) | `school_admin` |
| `lead_sources` | authenticated | super admin | super admin | super admin |
| `applicants` | STAFF | EDIT | EDIT; `principal` hanya kolom keputusan (lewat RPC) | `school_admin` |
| `applicant_addresses`, `_previous_schools`, `_emergency_contacts` | STAFF | EDIT | EDIT | EDIT |
| `applicant_health`, `applicant_family_members`, `applicant_government_data` | **SENS** | EDIT | EDIT | EDIT |
| `applicant_documents` | SENS + `finance` hanya jenis `registration_receipt`, `tuition_receipt` | EDIT | EDIT | EDIT |
| `applicant_stage_history` | STAFF | via trigger / RPC saja | ❌ | ❌ |
| `psychology_tests` | SENS; `psychologist` yang ditugaskan | EDIT, `psychologist` | EDIT, `psychologist` miliknya | `school_admin` |
| `parent_interviews` | STAFF | EDIT | EDIT | EDIT |
| `psychologists`, `psychology_organizations` | authenticated (punya role di sekolah mana pun) | `school_admin` | `school_admin` | super admin |
| `transactions` | STAFF | `finance`, `school_admin` | `finance`, `school_admin` | ❌ (koreksi lewat transaksi baru) |
| `teachers`, `classes`, `students`, `class_enrollments`, `student_level_history` | STAFF | `school_admin` | `school_admin` | `school_admin` |
| `notification_logs` | STAFF | via Server Action/Edge Function | service | ❌ |
| `audit_logs` | `school_admin` | trigger | ❌ | ❌ |

## 3. Contoh SQL

```sql
alter table applicants enable row level security;

create policy applicants_select on applicants for select to authenticated
  using (has_school_role(school_id, array['school_admin','admission_officer','principal','finance','viewer']::app_role[]));

-- Hanya bila role psikolog diaktifkan (Q-04): psikolog melihat pendaftar yang ia tes
create policy applicants_select_psychologist on applicants for select to authenticated
  using (exists (select 1 from psychology_tests t join psychologists p on p.id = t.psychologist_id
                 where t.applicant_id = applicants.id and p.profile_id = auth.uid()));

create policy applicants_insert on applicants for insert to authenticated
  with check (has_school_role(school_id, array['school_admin','admission_officer']::app_role[]));

create policy applicants_update on applicants for update to authenticated
  using      (has_school_role(school_id, array['school_admin','admission_officer']::app_role[]))
  with check (has_school_role(school_id, array['school_admin','admission_officer']::app_role[]));

create policy applicants_delete on applicants for delete to authenticated
  using (has_school_role(school_id, array['school_admin']::app_role[]));

-- Tabel sensitif (1:1, tanpa school_id): lewat applicant_school()
alter table applicant_family_members enable row level security;
create policy fam_select on applicant_family_members for select to authenticated
  using (has_school_role(applicant_school(applicant_id),
         array['school_admin','admission_officer','principal']::app_role[]));
create policy fam_write on applicant_family_members for all to authenticated
  using      (has_school_role(applicant_school(applicant_id), array['school_admin','admission_officer']::app_role[]))
  with check (has_school_role(applicant_school(applicant_id), array['school_admin','admission_officer']::app_role[]));

-- Dokumen: keuangan hanya bukti bayar
alter table applicant_documents enable row level security;
create policy docs_select on applicant_documents for select to authenticated using (
  has_school_role(school_id, array['school_admin','admission_officer','principal']::app_role[])
  or (has_school_role(school_id, array['finance']::app_role[])
      and doc_type in ('registration_receipt','tuition_receipt')));
```

## 4. Perubahan tahap & keputusan

Update `flow` dan `acceptance_status` dilakukan lewat RPC `security invoker`. RPC memvalidasi role di dalam fungsi:

```sql
create or replace function move_applicant_stage(p_ids uuid[], p_to applicant_flow, p_note text default null)
returns int language plpgsql security invoker set search_path = public as $$
declare v_count int;
begin
  if p_to = 'tuition' and exists (select 1 from applicants where id = any(p_ids)
       and not has_school_role(school_id, array['school_admin','admission_officer','principal']::app_role[])) then
    raise exception 'forbidden' using errcode = '42501';
  end if;
  perform set_config('app.stage_note', coalesce(p_note, ''), true);   -- dibaca trigger track_stage_change
  update applicants set flow = p_to, updated_by = auth.uid()
   where id = any(p_ids) and flow is distinct from p_to;              -- RLS update tetap berlaku
  get diagnostics v_count = row_count;
  return v_count;
end $$;
```

## 5. Pengujian RLS

pgTAP di `supabase/tests/rls/*.sql`: untuk setiap role × tabel × operasi, set `request.jwt.claims` lalu verifikasi baris yang terlihat atau ditolak (lihat [21](21-TESTING-AND-QA.md)).
