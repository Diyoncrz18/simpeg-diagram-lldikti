# Komposisi Tim & Pembagian Tugas
## SIMPEG Fase 1 — LLDIKTI Wilayah XVI

> Sinkron dengan PRD-SIMPEG-Fase1-Core.md v1.1. Acuan utama teknis: Laravel 12, Blade, PostgreSQL 17, Keycloak SSO untuk autentikasi, RBAC internal SIMPEG, development DB via container, production diarahkan ke Podman, dan approval cuti berbasis approval chain yang dapat melakukan skip approver duplikat.

---

## 1. Mengapa Komposisi Perlu Disesuaikan

Proyek SIMPEG menggunakan **Laravel 12 + Blade** di mana backend dan frontend sangat terkait erat. Frontend bukan aplikasi React/Vue SPA terpisah. Ini penting karena:

- Membagi "backend murni" dan "frontend murni" di proyek Laravel justru memperlambat karena satu fitur butuh keduanya sekaligus
- Tanpa UI/UX designer, desain UI bisa diatasi dengan **Blade Components** dan design system internal yang konsisten
- QA yang hanya di akhir sprint akan memperlambat — lebih baik **QA berjalan paralel** sejak Sprint 2

---

## 2. Rekomendasi Komposisi Tim

| Nama | Role Utama | Role Sekunder | Alokasi |
|------|-----------|--------------|---------|
| **Dion Kobi** | Project Manager | System Analyst, Tech Lead, Code Reviewer | 30% coding, 70% manajemen |
| **Jordan Sutarto** | Fullstack Developer (Backend 1) | Backend leadership, business logic, queue/scheduler | 90% coding |
| **Grantly Sorongan** | Fullstack Developer (Backend 2) | Backend implementation, QA operations, documentation operations | 70% backend, 20% QA ops, 10% dokumentasi |
| **Adithian Gunawan** | Fullstack Developer (Lead Frontend) | UI/Layout, Blade Components, Dashboard | 90% coding |
| **Adriel Walintukan** | Fullstack Developer (Frontend) + QA | Testing, Bug Fixing, Export/Laporan | 70% coding, 30% QA |

### Kenapa Semua "Fullstack"?

Di Laravel, satu fitur = 1 migration + 1 model + 1 controller + 1 blade view. Kalau "backend" hanya buat API dan "frontend" hanya buat view, maka:
- Mereka saling menunggu
- Handoff antar orang memperlambat
- Bug lebih sulit dilacak (siapa yang salah?)

Dengan pendekatan **fullstack per fitur**, satu orang bisa menyelesaikan satu user story **end-to-end** dari database sampai UI.

> **Catatan:** Jordan menjadi **Backend 1** dan memimpin pembagian kerja teknis backend. Grantly menjadi **Backend 2** dengan fokus utama backend, sekaligus menjaga operasional QA dan dokumentasi agar hasil sprint rapi, bisa diuji, dan bisa dihandover.

---

## 3. Detail Role & Tanggung Jawab

### 👔 Dion Kobi — Project Manager & System Analyst

**Tanggung Jawab Utama:**
- Sprint planning, daily standup (15 menit/hari), sprint review
- Menjaga backlog dan prioritas — update Notion
- Komunikasi dengan pihak LLDIKTI (menjawab pertanyaan blocker)
- Code review untuk semua Pull Request (quality gate)
- Menulis/validasi acceptance criteria sebelum story dikerjakan
- Menyiapkan test data dan seed data reference tables

**Tanggung Jawab Teknis (30% waktu):**
- Setup awal proyek Laravel + konfigurasi environment
- Setup CI/CD pipeline (jika ada)
- Menulis migration untuk reference tables (seed data)
- Konfigurasi Keycloak client + middleware integrasi
- Menulis dokumentasi teknis (API docs, README)

**Output:**
- Notion board selalu up-to-date
- Setiap sprint punya sprint goal yang jelas
- Pull Request di-review dalam < 24 jam

---

### ⚙️ Jordan Sutarto — Backend 1 / Fullstack

**Spesialisasi:** Backend 1 Delivery, Business Logic, Queue/Scheduler, EWS, Cuti

**Mandat Backend 1:**
- Memimpin pembagian task backend bersama Dion.
- Menjadi rujukan teknis pertama untuk business logic, queue, scheduler, notifikasi, dan modul cuti/EWS.
- Menjaga konsistensi pola model, controller, service, validation, dan migration di seluruh modul backend.
- Melakukan review teknis backend bersama Dion dan Grantly sebelum fitur kritis di-merge.

**Sprint 1 (Fondasi):**
- Implementasi user mapping & RBAC middleware (US-1.4)
- Buat migration + seeder untuk semua reference tables (16 tabel referensi)
- Setup notification system — database + mail channel (US-6.1 backend)
- CRUD Hari Libur (US-8.4)

**Sprint 2 (Data Pegawai):**
- Riwayat KGB — append-only + kalkulasi TMT KGB (US-2.6 bagian KGB)
- Hukuman disiplin — CRUD + auto-deactivate scheduler (US-2.7)
- Riwayat jabatan + jenis jabatan (US-2.6 bagian jabatan)

**Sprint 3:**
- Template import Excel/CSV generator (US-3.1)
- Profil sendiri — read-only view per pegawai (US-2.5)
- Data keluarga — CRUD (US-2.8)

**Sprint 4:**
- Form pengajuan cuti — validasi saldo, jenis cuti per PNS/PPPK (US-4.1)
- Assign atasan langsung per pegawai (US-4.11)
- Assign approver stage 2 & 3 — konfigurasi (US-4.10)
- Saldo cuti — kalkulasi + carry-over (US-4.3)

**Sprint 5:**
- EWS — halaman daftar alert aktif (US-5.2)
- Flag kinerja baik (US-5.4)
- Email notification — Laravel Mail + Queue (US-6.3)
- Session timeout (US-1.3)

**Sprint 6-7:**
- Kelola saldo cuti admin (US-4.9)
- Daftar cuti admin view (US-4.8)
- Kelola reference tables CRUD (US-8.5)
- Bug fixes

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| Backend Leadership | Standar backend, review teknis, pembagian task backend |
| RBAC & Mapping | US-1.4, US-1.3, US-1.5 |
| Riwayat & Disiplin | US-2.6 (KGB/Jabatan), US-2.7, US-2.8 |
| Pengajuan Cuti | US-4.1, US-4.3, US-4.10, US-4.11 |
| EWS Data | US-5.2, US-5.4 |
| Notifikasi | US-6.1 (backend), US-6.3 |
| Konfigurasi | US-8.4, US-8.5, US-3.1 |

---

### ⚙️ Grantly Sorongan — Backend 2 / Fullstack + QA & Documentation Ops

**Spesialisasi:** Backend Implementation, Database, API Core, Keycloak Integration, QA Operations, Documentation Operations

**Mandat Backend 2 + Operasional:**
- Fokus utama tetap backend implementation untuk modul core yang sudah ditetapkan.
- Menjadi partner Backend 1 untuk implementasi teknis, debugging, dan validasi business rule.
- Menjaga checklist QA per sprint: test scope, skenario kritis, evidence bug, dan status retest.
- Menjaga dokumentasi operasional: README, catatan setup, runbook, keputusan teknis, dan catatan handover.
- Berkoordinasi dengan Adriel untuk eksekusi QA dan dengan Dion untuk dokumentasi project-level.

**Sprint 1 (Fondasi):**
- Setup Laravel project structure + PostgreSQL connection
- Implementasi Keycloak SSO middleware (US-1.1, US-1.2)
- Buat base model dengan SoftDeletes trait
- Setup Audit Log system menggunakan `owen-it/laravel-auditing` (US-7.1)
- Buat semua migration untuk tabel utama: `employees`, `rank_histories`, `position_histories`, `kgb_histories`, `discipline_records`, `family_members`

**Sprint 2 (Data Pegawai):**
- CRUD Pegawai: Model, Controller, Form Request Validation, Blade Views (US-2.1, US-2.2)
- Halaman daftar pegawai dengan DataTables/Livewire (US-2.3)
- Halaman detail pegawai — tabbed layout (US-2.4)
- Riwayat kepangkatan — append-only logic + `is_latest` flag (US-2.6)

**Sprint 3:**
- Import Excel/CSV engine: parsing, validasi, queue job (US-3.2, US-3.3, US-3.4)
- Soft delete & restore logic (US-2.9)

**Sprint 4:**
- Approval chain engine untuk cuti — state machine logic (US-4.4, US-4.5, US-4.6)
- Kalkulasi hari kerja — business days calculation (US-4.12)

**Sprint 5:**
- EWS Scheduler — Laravel Task Scheduler + command (US-5.1)
- Kalkulasi TMT otomatis — event listener saat riwayat ditambahkan (US-5.5)

**Sprint 6-7:**
- Dashboard data queries (US-8.1)
- Export Excel/PDF engine menggunakan `maatwebsite/excel` + `barryvdh/laravel-dompdf` (US-9.1, US-9.3)
- Bug fixes & optimisasi query

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| Keycloak SSO | US-1.1, US-1.2, US-1.3, US-1.4 |
| Core Data Pegawai | US-2.1, US-2.2, US-2.3, US-2.4, US-2.6 |
| Import Excel/CSV | US-3.2, US-3.3, US-3.4 |
| Approval Engine | US-4.4, US-4.5, US-4.6, US-4.12 |
| EWS Engine | US-5.1, US-5.5 |
| Export Engine | US-9.1, US-9.3 |
| QA Operations | Checklist QA, retest tracking, evidence bug, koordinasi dengan Adriel |
| Documentation Operations | README, runbook setup, catatan teknis, handover sprint |

---

### 🎨 Adithian Gunawan — Lead Frontend / Fullstack

**Spesialisasi:** UI/Layout, Blade Components, Dashboard, Visualisasi Data

**Sprint 1 (Fondasi):**
- Setup UI framework: pilih dan konfigurasi component library (Filament / Blade UI Kit / custom)
- Buat **Design System** Laravel: layout master, sidebar, navbar, card, table, form components
- Buat halaman login (redirect ke Keycloak) + halaman "Akun belum terdaftar"
- Notifikasi in-app — bell icon + dropdown di navbar (US-6.1 frontend)

**Sprint 2 (Data Pegawai):**
- UI Form tambah/edit pegawai — multi-tab, upload foto, validasi client-side (US-2.1, US-2.2 views)
- UI Daftar pegawai — responsive table + search + filter (US-2.3 views)
- UI Detail pegawai — tabbed/accordion layout, badge, timeline riwayat (US-2.4 views)

**Sprint 3:**
- UI Import Excel/CSV — upload, preview table, mapping kolom, progress bar (US-3.2, US-3.3, US-3.4 views)
- UI Profil sendiri — read-only version dari detail pegawai (US-2.5 views)

**Sprint 4:**
- UI Form pengajuan cuti — date picker, kalkulasi hari realtime, validasi (US-4.1 views)
- UI Daftar cuti pegawai — tabel + badge status warna (US-4.2)
- UI Approval cuti — daftar pending, tombol approve/tunda, form alasan (US-4.4, US-4.5, US-4.6 views)

**Sprint 5:**
- UI Daftar EWS — tabel warna merah/kuning/hijau (US-5.2 views)
- UI EWS pribadi di profil pegawai (US-5.3)

**Sprint 6 (Dashboard):**
- **Dashboard Admin** — 7 widget: KPI cards, pie chart, bar chart, line chart, tabel mini (US-8.1)
- **Dashboard Pegawai** — profil ringkas, saldo cuti, EWS pribadi (US-8.2)
- Menggunakan Chart.js atau ApexCharts untuk visualisasi

**Sprint 7:**
- Dashboard Atasan Langsung (US-8.3)
- Timeline approval cuti — visual vertikal (US-4.7)
- Halaman semua notifikasi (US-6.2)
- Polish: responsive, micro-animations, loading states

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| Design System | Layout, components, navbar, sidebar |
| UI Forms | US-2.1, US-2.2, US-4.1 (views) |
| UI Tables | US-2.3, US-4.2 (views) |
| Dashboard | US-8.1, US-8.2, US-8.3 |
| Import Excel/CSV Views | US-3.2, US-3.3 (views) |
| EWS Views | US-5.2, US-5.3 (views) |
| Notifikasi UI | US-6.1 (frontend), US-6.2 |

---

### 🧪 Adriel Walintukan — Frontend + QA Tester

**Spesialisasi:** Frontend Support, Testing, Export/Laporan, Bug Fixing

**Sprint 1 (Fondasi):**
- Bantu Adithian setup design system — membuat reusable Blade components (alert, modal, toast, pagination)
- Setup testing framework: PHPUnit (unit test) + Laravel Dusk (browser test)
- Tulis test cases dasar untuk auth flow

**Sprint 2 (Data Pegawai):**
- **QA:** Test semua CRUD pegawai — happy path + edge cases (NIP duplikat, NIK invalid, file upload >10MB)
- Buat Blade components: form elements (date picker, dropdown, file upload with preview)
- UI Data keluarga — form + tabel di tab detail pegawai (US-2.8 views)

**Sprint 3:**
- **QA:** Test import CSV — file corrupt, encoding, baris error, NIP duplikat
- UI Soft delete — konfirmasi dialog, daftar non-aktif, restore (US-2.9 views)
- UI Hard delete — double confirm + ketik nama (US-2.10 views)

**Sprint 4:**
- **QA:** Test flow cuti end-to-end: submit → approval chain → skip approver duplikat jika ada → saldo berkurang saat final approve
- UI Saldo cuti — card view + riwayat (US-4.3 views)
- Assign atasan langsung — UI dropdown + riwayat (US-4.11 views)

**Sprint 5:**
- **QA:** Test EWS scheduler — trigger notifikasi, eligibility, no-duplicate alerts
- UI Flag kinerja — toggle dengan tooltip (US-5.4 views)
- Email template — HTML responsive template untuk semua jenis notifikasi

**Sprint 6:**
- Export PDF — layout laporan dengan header/footer/tanda tangan (US-9.2, US-9.4)
- **QA:** Test dashboard — widget data accuracy, responsive

**Sprint 7:**
- **QA FULL REGRESSION**: Test seluruh sistem end-to-end
- Audit log views — halaman daftar + diff view (US-7.2, US-7.3 views)
- Tandai notifikasi dibaca (US-6.4)
- Redirect per role (US-1.5)
- Bug fixing dari hasil QA

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| Reusable Components | Modal, alert, toast, form elements |
| Soft/Hard Delete | US-2.9, US-2.10 (views) |
| Saldo Cuti Views | US-4.3, US-4.9 (views) |
| Export PDF | US-9.2, US-9.4 |
| Audit Log Views | US-7.2, US-7.3 |
| Notifikasi | US-6.4, US-1.5 |
| **QA Testing** | **Semua sprint** |

---

## 4. Peta Tugas per Sprint

### Sprint 1 — Fondasi (Minggu 1–2)

```
Dion      │ Setup project + environment + Keycloak config + seed data
Jordan    │ Backend 1: US-1.4 Mapping User + US-6.1 Notifikasi + US-8.4 Hari Libur
Grantly   │ Backend 2 + Ops: US-1.1 Login SSO + US-1.2 Logout + US-7.1 Audit Log + QA/docs checklist
Adithian  │ Design System: layout, sidebar, navbar, components
Adriel    │ Reusable components + setup testing framework
```

### Sprint 2 — Data Pegawai (Minggu 3–4)

```
Dion      │ Code review + validasi acceptance criteria + sprint planning
Jordan    │ Backend 1: US-2.6 Riwayat (Pangkat/Jabatan/KGB) + US-2.7 Disiplin
Grantly   │ Backend 2 + Ops: US-2.1 Tambah + US-2.2 Edit + US-2.3 Daftar + US-2.4 Detail + QA/docs checklist
Adithian  │ UI: form pegawai, tabel daftar, halaman detail (tabbed)
Adriel    │ QA Sprint 1 + UI data keluarga + blade components
```

### Sprint 3 — Import Excel/CSV & Pelengkap (Minggu 5–6)

```
Dion      │ Code review + siapkan template CSV + sample data
Jordan    │ Backend 1: US-3.1 Template + US-2.5 Profil + US-2.8 Keluarga
Grantly   │ Backend 2 + Ops: US-3.2, US-3.3, US-3.4 Import engine + QA/docs checklist
Adithian  │ UI import CSV: upload, preview, progress bar
Adriel    │ QA Sprint 2 + US-2.9 Soft delete + US-2.10 Hard delete views
```

### Sprint 4 — Cuti (Minggu 7–9)

```
Dion      │ Code review + koordinasi data approval chain dengan LLDIKTI
Jordan    │ Backend 1: US-4.1 Ajukan cuti + US-4.10, US-4.11 Assign atasan/approver + US-4.3 Saldo
Grantly   │ Backend 2 + Ops: US-4.4, US-4.5, US-4.6 Approval engine + US-4.12 Kalkulasi + QA/docs checklist
Adithian  │ UI: form cuti, daftar cuti, approval views, badge status
Adriel    │ QA Sprint 3 + US-4.3 Saldo views + US-4.11 views
```

### Sprint 5 — EWS & Notifikasi (Minggu 10–11)

```
Dion      │ Code review + verifikasi business rules EWS
Jordan    │ Backend 1: US-5.2 Daftar EWS + US-5.4 Flag kinerja + US-6.3 Email + US-1.3 Timeout
Grantly   │ Backend 2 + Ops: US-5.1 Scheduler EWS + US-5.5 Kalkulasi TMT + QA/docs checklist
Adithian  │ UI: daftar EWS (warna), EWS pribadi (US-5.3)
Adriel    │ QA Sprint 4 (flow cuti E2E) + US-5.4 views + email templates
```

### Sprint 6 — Dashboard & Laporan (Minggu 12–13)

```
Dion      │ Code review + UAT preparation
Jordan    │ Backend 1: US-4.9 Kelola saldo + US-4.8 Daftar cuti admin
Grantly   │ Backend 2 + Ops: US-8.1 Dashboard queries + US-9.1 Export Excel pegawai + US-9.3 Export Excel cuti + QA/docs checklist
Adithian  │ US-8.1 Dashboard Admin (charts) + US-8.2 Dashboard Pegawai
Adriel    │ QA Sprint 5 + US-9.2 Export PDF pegawai + US-9.4 Export PDF cuti
```

### Sprint 7 — P1 Stories & UAT (Minggu 14–16)

```
Dion      │ UAT coordination + deployment preparation + dokumentasi
Jordan    │ Backend 1: Bug fixes + US-4.8 + US-4.9 + US-1.5 redirect
Grantly   │ Backend 2 + Ops: Bug fixes + performance optimization + US-8.5 Reference tables + QA/docs finalization
Adithian  │ US-8.3 Dashboard atasan + US-4.7 Timeline + US-6.2 Semua notifikasi
Adriel    │ FULL REGRESSION TEST + US-7.2, US-7.3 Audit views + US-6.4 + bug fixes
```

---

## 5. Workflow Harian

```
09:00   Daily Standup (15 menit, semua anggota)
        - Apa yang dikerjakan kemarin?
        - Apa yang akan dikerjakan hari ini?
        - Ada blocker?

09:15   Mulai coding
        - Setiap story: buat branch → code → PR → review → merge

14:00   Mid-day sync (opsional, jika ada blocker)

16:30   Adriel: mulai QA testing untuk PR yang sudah di-merge hari ini
        Grantly: update QA checklist, catatan retest, dan dokumentasi teknis terkait PR

17:00   Dion: Review PR yang masuk + update Notion board
```

---

## 6. Aturan Kerja Tim

### Git Workflow
```
main (production)
  └── develop (integration)
       ├── feature/US-1.1-login-sso         (Grantly)
       ├── feature/US-1.4-mapping-user       (Jordan)
       ├── feature/design-system             (Adithian)
       └── fix/US-2.1-nip-validation         (Adriel - bug fix)
```

### Code Review Rules
- Setiap PR **wajib di-review minimal 1 orang** sebelum merge
- PR backend → di-review Jordan (Backend 1), Grantly (Backend 2), atau Dion
- PR frontend → di-review Adithian atau Adriel
- PR critical (auth, approval engine, EWS) → **wajib di-review Dion**

### Definition of Done (DoD)
Sebuah user story dianggap "Done" jika:
- [ ] Code sudah di-merge ke branch `develop`
- [ ] Semua acceptance criteria terpenuhi
- [ ] Unit test untuk business logic kritis sudah ada
- [ ] Adriel sudah melakukan QA dan tidak ada bug kritis
- [ ] Grantly sudah memperbarui checklist QA, catatan retest, dan dokumentasi operasional jika story berdampak ke setup/flow teknis
- [ ] UI sesuai dengan design (responsive di desktop & tablet)
- [ ] Audit log berfungsi untuk operasi CRUD terkait

---

## 7. Distribusi Beban Kerja

| Nama | Total Stories (Owner) | Total SP | Rata-rata SP/Sprint |
|------|-----------------------|----------|---------------------|
| Dion | 5 (setup + review) | ~15 | ~2 |
| Jordan | 16 (Backend 1) | ~60 | ~9 |
| Grantly | 16 (Backend 2) + QA/dokumentasi ops | ~65 + ops | ~9 + ops |
| Adithian | 12 | ~55 | ~8 |
| Adriel | 8 + QA semua sprint | ~40 + QA | ~6 + QA |

> **Catatan:** Banyak stories dikerjakan kolaboratif (backend + frontend), jadi angka di atas bersifat indikatif. Yang penting adalah setiap story punya **satu owner** yang bertanggung jawab atas delivery-nya.

---

## 8. Skill Gap & Rekomendasi Belajar

| Anggota | Perlu Dipelajari | Resource |
|---------|-----------------|----------|
| **Jordan (Backend 1)** | Laravel Queue + Scheduler | [Laravel Queues](https://laravel.com/docs/queues) |
| **Jordan (Backend 1)** | Laravel Notification | [Laravel Notifications](https://laravel.com/docs/notifications) |
| **Grantly (Backend 2 + Ops)** | Keycloak + Socialite/OpenID | [Laravel Socialite](https://laravel.com/docs/socialite) |
| **Grantly (Backend 2 + Ops)** | Laravel Auditing + dokumentasi teknis | [owen-it/laravel-auditing](https://laravel-auditing.com) |
| **Adithian** | Filament (jika dipakai) | [filamentphp.com](https://filamentphp.com) |
| **Adithian** | Chart.js / ApexCharts | [apexcharts.com](https://apexcharts.com) |
| **Adriel** | Laravel Dusk (browser testing) | [Laravel Dusk](https://laravel.com/docs/dusk) |
| **Adriel** | DomPDF / Snappy | [barryvdh/laravel-dompdf](https://github.com/barryvdh/laravel-dompdf) |
| **Semua** | Laravel Livewire (jika dipakai) | [livewire.laravel.com](https://livewire.laravel.com) |

---

## 9. Rekomendasi Tools

| Kebutuhan | Tool | Keterangan |
|-----------|------|-----------|
| Project Management | **Notion** (sudah setup) | Board + docs |
| Version Control | **GitHub** | Repository + PR + CI |
| Communication | **Discord / WhatsApp Group** | Daily sync |
| Code Editor | **VS Code** + PHP extensions | Standard |
| API Testing | **Postman / Insomnia** | Test endpoints |
| Database GUI | **DBeaver / pgAdmin** | PostgreSQL management |
| Design Reference | **Filament Demo** | UI component reference |

---

## 10. Risiko & Mitigasi

| Risiko | Impact | Mitigasi |
|--------|--------|---------|
| Tim magang kurang pengalaman Laravel 12 + Blade | Tinggi | Dion aktif code review + pair programming awal |
| Credential Keycloak belum diterima | Blocker | Siapkan adapter Socialite/trait dan mock auth sementara sampai Client ID, Client Secret, URL, dan akun testing diterima |
| Tidak ada UI/UX designer | Sedang | Gunakan Filament/component library, hindari custom design |
| Satu orang sakit/tidak hadir | Sedang | Setiap area punya backup (lihat tabel di bawah) |
| Sprint 7 terlalu besar (43 SP) | Tinggi | Prioritaskan P1 yang kritis, sisanya bisa post go-live |

### Backup Matrix
| Primary | Backup | Area |
|---------|--------|------|
| Jordan | Grantly | Backend 1 / business logic |
| Grantly | Jordan | Backend 2 / core implementation |
| Grantly | Adriel + Dion | QA operations & dokumentasi teknis |
| Adithian | Adriel | Frontend/UI |
| Adriel | Adithian | QA/Testing |
| Dion | Grantly | Tech decisions |
