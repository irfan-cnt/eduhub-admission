# 06 · Component Inventory

Komponen diidentifikasi dari blok markup yang berulang di prototipe dan dipetakan ke komponen React (REKOMENDASI). Nama kolom "Server/Client" mengikuti [09-TECHNICAL-ARCHITECTURE](09-TECHNICAL-ARCHITECTURE.md).

## 1. Shell

| Komponen | Fungsi | Props utama | Dependencies | S/C | Sumber prototipe |
|---|---|---|---|---|---|
| `AppShell` | Layout 2 kolom, scroll hanya di konten | `children`, `school` | Sidebar, TopBar | Server | `.shell`, `.shell-body` |
| `Sidebar` | Identitas, dropdown sekolah, menu, pintasan, profil | `school`, `navCounts`, `user` | SchoolSwitcher, NavGroup | Client (collapse) | `<aside class="shell-side">` |
| `SchoolSwitcher` | Popover daftar sekolah berkelompok | `groups: {name, brand, units[]}`, `activeCode` | Popover (shadcn) | Client | `toggleSchoolMenu`, `schoolGroups` |
| `NavGroup` / `NavItem` | Menu berkelompok + jumlah | `label`, `items: {href, label, icon, count}` | next/link | Server | `navGroups` |
| `ShortcutList` | Macet / bermasalah / Waiting/Invalid | `counts`, `stuckDays` | — | Server | `extraShortcuts` |
| `TopBar` | Toggle sidebar, breadcrumb, cari, notifikasi, tema | `schoolName`, `pageName`, `lateCount` | ThemeToggle | Client | `<header>` |
| `ThemeToggle` | Light/dark | — | localStorage | Client | `toggleDark` |
| `BrandProvider` | Set `data-brand`/`data-theme` | `brand` | — | Client | `brandClass`, `themeClass` |
| `SimulationPanel` | Demo state (dev only) | — | — | Client | `simsel` (**jangan** dibawa ke produksi) |

## 2. Data bersama

| Komponen | Fungsi | Props | S/C | Sumber |
|---|---|---|---|---|
| `PageHeader` | Judul, subjudul, slot aksi | `title`, `subtitle`, `actions` | Server | `x.title`, `listSubtitle` |
| `ViewToggle` | Tabel / Kartu / (Kanban) | `value`, `options`, `onChange` (URL) | Client | `x.vTable`, `viewTable/viewCard/viewKanban` |
| `FilterTabs` | Tab dengan badge jumlah | `tabs: {key,label,count}`, `active` | Client | `tabs`, `x.chips` |
| `FilterPills` | Filter pil tambahan | `options`, `value` | Client | `x.chips2` |
| `SearchInput` | Debounce 250 ms | `placeholder`, `value` | Client | `onSearch`, `x.onSearch` |
| `FilterSelect` | Select filter | `label`, `options`, `value` | Client | select TA/level/status/... |
| `ResetFilterButton` | Muncul bila ada filter | `visible` | Client | `hasFilters` |
| `DataTable<T>` | Tabel grid, sort, seleksi | `columns`, `rows`, `sort`, `selection` | Client | `role="table"` blok |
| `CardGrid<T>` | Grid kartu auto-fill | `items`, `renderCard`, `minWidth` | Server | `x.gitems`, `cards` |
| `GenericInfoCard` | Kartu nama + badge + key-value | `title`, `sub`, `badge`, `kv[]`, `action` | Server | `xGCards` |
| `Pager` | a–b dari n, Awal/‹/›/Akhir | `page`, `pageSize`, `total` | Client | `pageFrom…`, `x.first…` |
| `KpiRow` / `KpiCard` | Kartu KPI | `label`, `value`, `sub`, `alert`, `bar?` | Server | `kpis`, `x.kpis` |
| `StatusBadge` | Badge semantik | `tone: gray/blue/green/amber/red`, `children` | Server | `tone()` |
| `IssueFlag` | ⚠ + tooltip | `reasons: string[]` | Client (tooltip) | `r.hasFlag`, `.tipwrap` |
| `EmptyState` | Kosong / tidak ada hasil | `title`, `body`, `action` | Server | `listEmpty`, `xEmpty` |
| `ErrorState` | Gagal memuat + Coba lagi | `onRetry` | Client | `showError` |
| `NoAccessState` | Tanpa hak akses | `schoolName` | Server | `showNoAccess` |
| `Skeletons` | Pola skeleton per halaman | `variant` | Server | `.skel` |
| `Toast` | Umpan balik aksi | sonner | Client | `hasToast` |
| `ConfirmDialog` | Konfirmasi ubah tahap | `ids`, `to`, `onConfirm` | Client | `hasModal`, `m` |

## 3. Admission

| Komponen | Fungsi | Props | S/C | Sumber |
|---|---|---|---|---|
| `ProgressStepper` | 5 titik + garis + tooltip; varian `sm`/`lg`/`vertical` | `steps: Step[]` | Client (tooltip) | `r.steps`, `d.steps` |
| `StageBadge` | Badge `flow` (merah Invalid) | `flow` | Server | `flowStyle` |
| `DaysInStageBadge` | N hari / Selesai / Ditutup / Invalid | `applicant` | Server | `daysText`, `daysStyle` |
| `ApplicantTable` | Kolom §7.2 PRD | `rows`, `selection`, `sort` | Client | tabel pendaftar |
| `ApplicantCard` | Kartu pendaftar | `applicant`, `selected` | Client | `showCards` |
| `BulkActionBar` | Aksi massal | `selectedIds` | Client | `hasSel` |
| `KanbanBoard` / `KanbanColumn` / `KanbanCard` | dnd-kit | `columns`, `onMove` | Client | `kcols` |
| `ApplicantDrawer` | Slide-over detail | `applicantId`, `tab` | Client shell + Server data | `hasDetail` |
| `ApplicantHeader` | Header detail | `applicant` | Server | header `d.*` |
| `BackNav` | "← Kembali ke …" | `history` | Client | `nav.back` |
| `CompletenessCard` | % + bar per tab | `tabs` | Server | `v.complete` |
| `TodoList` | Perlu dilengkapi | `items` | Server | `v.todo` |
| `FieldSection` | Kartu field + ikon + x/y | `title`, `icon`, `fields` | Server | `v.profil`, `v.keluarga` |
| `FieldView` | Tampilan 1 field (teks, kosong, badge, cek, file, foto) | `field` | Server | `f.isText…isPhoto` |
| `SelectionTimeline` | 5 tahap seleksi | `steps` | Server | `v.steps` |
| `DocumentTile` | Kotak dokumen + Unggah/Ganti | `doc`, `onUpload` | Client | `v.docFiles` |
| `EmptyFieldFilter` | Semua / Belum diisi saja | `value` | Client | `v.filterAll/Empty` |

## 4. Dashboard

| Komponen | Props | Sumber |
|---|---|---|
| `AcademicYearSelect` | `years`, `value` | `onDashYear` |
| `FunnelChart` | `steps: {label,count,pct,drop,worst}` | `funnel` |
| `TrendChart` (Recharts) | `current[]`, `previous[]`, `mode` | `trend` |
| `ActionPanel` (3 kolom) | `late[]`, `schedule[]`, `payments[]` | `act` |
| `LevelQuotaChart` | `levels: {name, apps, accepted, quota}` | `levels` |
| `SourceChart` | `sources: {name, count, conversion}` | `sources` |

## 5. Modul pendukung

| Komponen | Sumber |
|---|---|
| `TransactionTable` / `TransactionCard` | `isTrx` |
| `ReconciliationCell` | `t.recOk/recNo/recNa` |
| `PsychologyTable` / `PsychologyCard` | `isPsy` |
| `IqScoreCell` (angka + bar) | `p.iqPct` |
| `PsychologyDrawer` (+ `IqScale`, `AspectBars`) | `xd` |
| `PsychologistCard` / table | `isPsyc` |
| `ClassTable` / `ClassCard` + `CapacityBar` | `isClass` |
| `StudentTable` / `StudentCard` | `isStudent` |
| `TeacherCard` / table + `PresenceBadge` | `isTeacher` |

## 6. Mobile

| Komponen | Sumber |
|---|---|
| `MobileAppBar` (logo, judul, sekolah → sheet) | `Mobile.dc.html` header |
| `BottomNav` (3 item) | `nav aria-label="Navigasi utama"` |
| `FilterSheet`, `SchoolSheet`, `MoveStageSheet` | `hasSheet` |
| `PillTabs` (sticky, tinggi 56 px) | tablist detail |
| `LoadMoreButton` (15 per batch) | `loadMore` |
| `SelectModeBar` | `hasSel` |

## Reusability

- **Generik, dipakai lintas modul:** DataTable, CardGrid, GenericInfoCard, Pager, FilterTabs, ViewToggle, KpiCard, StatusBadge, IssueFlag, state components, FieldView, FieldSection.
- **Khusus admission:** ProgressStepper, KanbanBoard, ApplicantDrawer, SelectionTimeline.
- Semua komponen menerima data siap tampil (view model); logika status ada di `lib/domain/` ([15](15-BUSINESS-LOGIC.md)).
