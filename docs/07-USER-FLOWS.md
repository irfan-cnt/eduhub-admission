# 07 · User Flows

## 1. Autentikasi (belum didesain, REKOMENDASI)

```mermaid
flowchart TD
    A[Buka aplikasi] --> B{Sesi valid?}
    B -- Tidak --> C[/login/]
    C --> D[Isi email & password]
    D --> E{Supabase Auth OK?}
    E -- Tidak --> F[Pesan error di form] --> D
    E -- Ya --> G{Punya role di sekolah mana pun?}
    B -- Ya --> G
    G -- Tidak --> H[State Tanpa hak akses]
    G -- Ya --> I[Redirect ke /sekolah-default/dashboard]
    C --> J[Lupa password] --> K[Email reset] --> L[/reset-password/] --> C
```

## 2. Pindah sekolah

```mermaid
flowchart LR
    A[Klik dropdown sekolah] --> B[Popover: grup TJ & KGS]
    B --> C[Pilih unit]
    C --> D[Navigasi ke /kode-unit/halaman-sama]
    D --> E[Brand & logo berganti]
    D --> F[Filter halaman direset]
    D --> G[Data dimuat untuk sekolah baru]
```

## 3. Menindaklanjuti pendaftar macet

```mermaid
flowchart TD
    A[Dashboard] --> B[Panel Macet > 7 hari]
    B --> C{Pilih}
    C -- Klik item --> D[Detail pendaftar]
    C -- Lihat semua --> E[Pendaftar, filter Terlambat]
    E --> F[Centang beberapa baris]
    F --> G[Pengingat WhatsApp]
    G --> H[Server mencatat notification_logs]
    H --> I[Toast: terkirim ke N orang tua]
    D --> J[Aksi utama sesuai tahap]
```

## 4. Siklus tahap pendaftar

```mermaid
stateDiagram-v2
    [*] --> Leads
    Leads --> Register: formulir diisi
    Register --> Payment: biaya formulir dibayar
    Payment --> Tuition: psikotes + wawancara selesai, diterima
    Payment --> WaitingList: diterima, kuota penuh
    WaitingList --> Tuition: kursi tersedia
    Leads --> Invalid
    Register --> Invalid
    Payment --> Invalid
    Tuition --> [*]: tuition lunas
```

Catatan: prototipe mengizinkan pindah ke tahap **mana pun** lewat dialog (termasuk mundur). Aturan transisi yang sah adalah `DECISION REQUIRED` (lihat [15](15-BUSINESS-LOGIC.md#transisi-tahap)).

## 5. Ubah tahap (CRUD update)

```mermaid
sequenceDiagram
    actor U as Officer
    participant UI as Next.js Client
    participant SA as Server Action moveApplicantStage
    participant DB as Supabase (RLS)
    U->>UI: Drag kartu / klik Ubah tahap
    UI->>U: Dialog konfirmasi
    U->>UI: Ya, pindahkan
    UI->>UI: Optimistic update
    UI->>SA: {ids, toFlow}
    SA->>SA: Validasi Zod + cek role
    SA->>DB: rpc move_applicant_stage()
    DB-->>SA: OK (stage_history tercatat)
    SA-->>UI: success
    UI->>U: Toast + revalidate counts
    Note over UI,SA: Bila gagal: rollback + toast error
```

## 6. Melengkapi data di detail

```mermaid
flowchart TD
    A[Buka detail] --> B[Tab Ringkasan]
    B --> C[Lihat % kelengkapan]
    B --> D[Perlu dilengkapi: klik item]
    D --> E[Tab terkait terbuka]
    E --> F{Field file?}
    F -- Ya --> G[Unggah ke Storage] --> H[Simpan metadata applicant_documents]
    F -- Tidak --> I[Edit field: belum didesain]
    E --> J[Kembali ke Ringkasan]
```

## 7. Psikotes → pendaftar → kembali

```mermaid
flowchart LR
    A[Halaman Psikotes] --> B[Klik nama siswa]
    B --> C[Panel Hasil psikotes]
    C --> D[Data pendaftar →]
    D --> E[Detail pendaftar, tab Seleksi]
    E --> F[Kembali ke Hasil psikotes]
    F --> C
```

## 8. Keuangan: rekonsiliasi

```mermaid
flowchart TD
    A[Transaksi] --> B[KPI Belum terekonsiliasi]
    B --> C[Tab Lunas]
    C --> D[Baris dengan ⚠ Belum cocok]
    D --> E[Cocokkan dengan mutasi bank]
    E --> F[Tandai cocok: belum didesain]
```

## 9. Admin (belum didesain)

```mermaid
flowchart LR
    A[Super admin] --> B[/admin/users/]
    B --> C[Undang pengguna via email]
    C --> D[Tetapkan role per sekolah]
    A --> E[/sekolah/settings/]
    E --> F[Kuota per level, target tuition, ambang macet]
```
