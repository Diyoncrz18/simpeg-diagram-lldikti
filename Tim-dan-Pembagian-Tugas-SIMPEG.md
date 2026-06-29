# Komposisi Tim & Pembagian Tugas
## SIMPEG Fase 1 — LLDIKTI Wilayah XVI

> Sinkron dengan PRD-SIMPEG-Fase1-Core.md v1.1. Acuan utama teknis: Laravel 12, Blade, PostgreSQL 17, Keycloak SSO untuk autentikasi, RBAC internal SIMPEG, development DB via container, production diarahkan ke Podman, dan approval cuti berbasis approval chain yang dapat melakukan skip approver duplikat.

---

## 1. Mengapa Komposisi Perlu Disesuaikan

Proyek SIMPEG menggunakan **Laravel 12 + Blade** di mana backend dan frontend sangat terkait erat. Mulai **Sprint 2**, cara kerja tim dibuat sebagai **vertical slice per fitur/task**: satu fitur diselesaikan dari desain UI, backend, integrasi data, review PR, bugfix, sampai QA sebelum pindah ke fitur berikutnya.

Prinsip utamanya:

- Setiap sprint menargetkan **2-3 vertical slices**, tergantung kompleksitas fitur.
- Frontend mulai lebih dulu menggunakan **mock data / data dummy** agar desain, layout, dan flow tidak menunggu backend selesai.
- Backend mengerjakan migration, model, controller/service, validasi, dan business rule sesuai kontrak field yang disepakati.
- Setelah backend siap, frontend mengganti dummy data menjadi **real data** dari controller/query/backend.
- Adriel menjadi gate GitHub: review PR, cek conflict, merge, dan mengembalikan bug/error ke developer pemilik task.
- Grantly melakukan QA testing di akhir slice/sprint untuk memastikan fitur berjalan baik dan sesuai acceptance criteria.

---

## 2. Rekomendasi Komposisi Tim

| Nama | Role Utama | Role Sekunder | Alokasi |
|------|-----------|--------------|---------|
| **Dion Kobi** | System Analyst & Project Manager | Acceptance criteria, sprint planning, prioritas backlog, koordinasi stakeholder | 20% analisis teknis, 80% manajemen |
| **Adithian Gunawan** | Lead Frontend | UI flow, Blade views/components, mock data, integrasi real data frontend | 90% frontend, 10% support review UI |
| **Adriel Walintukan** | Frontend + GitHub Management | Frontend support, PR review, conflict resolution, merge gate, bug routing | 45% frontend, 55% review/merge/GitHub |
| **Jordan Sutarto** | Lead Backend | Arsitektur backend, business logic, migration/model/controller, integrasi data | 90% backend, 10% review teknis backend |
| **Grantly Sorongan** | Support Backend + QA Tester | Backend support, test scenario, QA execution, bug evidence, regression check | 55% backend, 35% QA, 10% dokumentasi hasil QA |

### Kenapa Vertical Slice?

Di Laravel, satu fitur biasanya menyentuh migration, model, controller/service, policy, validation, dan Blade view. Kalau backend dan frontend berjalan tanpa kontrak slice yang jelas, maka:
- Mereka saling menunggu
- Handoff antar orang memperlambat
- Bug lebih sulit dilacak (siapa yang salah?)

Dengan pendekatan **vertical slice per fitur**, semua orang membantu satu fitur yang sama sesuai role masing-masing. Frontend tidak menunggu backend karena memakai dummy data lebih dulu; backend tidak menunggu UI final karena kontrak field sudah dikunci di awal; integrasi dilakukan setelah dua sisi siap.

> **Catatan:** Jordan memimpin backend dan Grantly membantu backend sekaligus menjadi QA tester. Adithian memimpin frontend, Adriel membantu frontend secukupnya sambil memegang GitHub management: review PR, merge, conflict resolution, dan mengembalikan bug ke developer pemilik task. Dion menjaga scope, acceptance criteria, timeline, dan komunikasi project.

---

## 3. Detail Role & Tanggung Jawab

### 👔 Dion Kobi — Project Manager & System Analyst

**Tanggung Jawab Utama:**
- Menyusun sprint goal, urutan vertical slice, acceptance criteria, dan batas scope setiap fitur.
- Menjaga backlog/Notion board, timeline, dependensi, dan komunikasi dengan pihak LLDIKTI.
- Mengunci kontrak awal setiap slice: field, flow, role access, validasi, dan data dummy yang boleh dipakai frontend.
- Memimpin daily standup, sprint review, dan keputusan prioritas jika ada task yang harus dipotong atau dipindah sprint.
- Melakukan review teknis untuk PR kritis bersama Jordan/Adriel, terutama auth, cuti, EWS, export, dan release candidate.

**Output:**
- Notion board selalu up-to-date
- Setiap sprint punya sprint goal yang jelas
- Setiap vertical slice punya acceptance criteria, owner, due date, dan status review/QA

---

### ⚙️ Jordan Sutarto — Lead Backend

**Spesialisasi:** backend architecture, business logic, queue/scheduler, cuti, EWS, data integration.

**Mandat Lead Backend:**
- Memimpin pembagian task backend bersama Dion dan memastikan Grantly mendapat task support yang jelas.
- Mengunci pola migration, model, controller, service, validation, policy/middleware, dan query agar konsisten.
- Menjadi owner backend utama untuk fitur kompleks: data pegawai, cuti, EWS, notifikasi, dashboard query, dan export.
- Menyediakan kontrak data untuk frontend sebelum backend final: field, route, status, error message, dan sample payload.
- Melakukan review teknis backend untuk PR kritis sebelum Adriel merge.

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

### ⚙️ Grantly Sorongan — Support Backend + QA Tester

**Spesialisasi:** backend support, database/query, import/export support, QA scenario, smoke test, regression, evidence bug.

**Mandat Support Backend + QA:**
- Membantu Jordan mengerjakan backend, terutama task yang bisa diparalelkan: migration pendukung, query, import/export, scheduler, seed, dan test data.
- Menyiapkan checklist QA untuk setiap vertical slice sejak awal, berdasarkan acceptance criteria dari Dion.
- Melakukan QA di akhir slice/sprint: happy path, negative case, role access, validasi form, audit log, dan data real.
- Mencatat bug dengan evidence jelas: langkah reproduksi, screenshot/log jika ada, severity, expected result, actual result, dan owner perbaikan.
- Melakukan retest setelah bug diperbaiki dan memberi status final: Pass, Pass with Note, atau Failed.

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| Keycloak SSO | US-1.1, US-1.2, US-1.3, US-1.4 |
| Core Data Pegawai | US-2.1, US-2.2, US-2.3, US-2.4, US-2.6 |
| Import Excel/CSV | US-3.2, US-3.3, US-3.4 |
| Approval Engine | US-4.4, US-4.5, US-4.6, US-4.12 |
| EWS Engine | US-5.1, US-5.5 |
| Export Engine | US-9.1, US-9.3 |
| QA & Documentation | Checklist QA, smoke/regression test, retest tracking, evidence bug, runbook, handover sprint |
| Documentation Operations | README, runbook setup, catatan teknis, handover sprint |

---

### 🎨 Adithian Gunawan — Lead Frontend

**Spesialisasi:** UI flow, Blade views/components, responsive layout, dashboard visualization, mock data, frontend integration.

**Mandat Lead Frontend:**
- Memimpin desain UI setiap vertical slice menggunakan dummy data yang disepakati bersama Dion/Jordan.
- Menentukan struktur Blade view/component, state kosong/error/loading, validasi client-side, dan konsistensi visual.
- Membagi task frontend kecil ke Adriel jika diperlukan, terutama komponen pendukung, polish, atau halaman turunan.
- Mengganti dummy data menjadi real data saat backend siap, lalu memastikan UI tidak berubah perilaku setelah integrasi.
- Menyediakan screenshot/evidence UI di PR agar Adriel bisa melakukan review/merge gate dengan jelas.

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

### 🔎 Adriel Walintukan — Frontend + GitHub Management

**Spesialisasi:** frontend support, component polish, PR review, conflict resolution, merge gate, release branch hygiene.

**Mandat Frontend + GitHub Gatekeeper:**
- Membantu frontend saat Adithian membutuhkan support, tetapi porsi frontend Adriel lebih kecil daripada Adithian.
- Menjadi reviewer/merge gate GitHub untuk PR harian ke `develop`.
- Mengecek deskripsi PR, checklist acceptance criteria, screenshot/evidence UI, hasil test/checks, dan conflict sebelum merge.
- Jika ada bug/error dari review atau QA, mengembalikan PR/task ke developer pemilik task dengan catatan perbaikan yang jelas.
- Menyelesaikan conflict GitHub jika conflict bersifat integrasi; jika conflict mengubah business logic, Adriel meminta owner fitur ikut memperbaiki.
- Tidak melakukan self-review: PR milik Adriel harus di-review dan di-merge oleh Dion atau Adithian.

**Ringkasan Ownership:**
| Area | Stories |
|------|---------|
| GitHub Review/Merge | PR checklist, review, conflict check, merge gate ke `develop` |
| Reusable Components | Modal, alert, toast, form elements, empty state, breadcrumb |
| Frontend Pegawai | US-2.5, US-2.8, US-2.9, US-2.10 views |
| Frontend Cuti | US-4.3, US-4.7 partial, US-4.11 views |
| Frontend EWS/Notif | US-5.4 views, email templates, US-6.2/US-6.4 support |
| Export PDF Views | US-9.2, US-9.4 |
| Audit Log Views | US-7.2, US-7.3 |

---

## 4. Peta Tugas per Sprint Berbasis Vertical Slice

Mulai Sprint 2, tim tidak membagi sprint hanya berdasarkan "frontend dulu" atau "backend dulu". Setiap fitur dikerjakan sebagai vertical slice dengan urutan kerja yang sama.

### Alur Baku Setiap Vertical Slice

| Tahap | Durasi Normal | Owner Utama | Output |
|-------|---------------|-------------|--------|
| 1. Kickoff slice | 0.5 hari | Dion | Acceptance criteria, scope, field, role access, dummy data, dan kontrak awal disepakati |
| 2. UI mock/dummy | 1-2 hari | Adithian, dibantu Adriel jika perlu | Blade view/component berjalan memakai dummy data |
| 3. Backend real data | 1-3 hari | Jordan, dibantu Grantly | Migration/model/controller/service/validation/query siap |
| 4. Sinkronisasi | 0.5-1 hari | Adithian + Jordan | Dummy data diganti real data, error/empty/loading state tetap aman |
| 5. PR review & merge gate | 0.5 hari | Adriel | PR direview, conflict diselesaikan, bug dikembalikan ke owner task |
| 6. QA & retest | 0.5-1 hari | Grantly | QA evidence, bug list, retest status, dan sign-off slice |

> Untuk fitur kompleks, satu slice boleh memakai 5-8 hari. Untuk fitur kecil, satu sprint bisa menyelesaikan 3 slice.

### Sprint 1 — Fondasi (Minggu 1-2)

Sprint 1 tetap fokus pada fondasi karena vertical slice baru efektif jika layout, auth, database awal, branch workflow, dan komponen dasar sudah siap.

| Owner | Tugas Utama | Output |
|-------|-------------|--------|
| Dion | Project baseline, acceptance criteria awal, seed/reference plan, unblock Keycloak/env | Backlog dan scope Sprint 1-2 jelas |
| Jordan | Mapping user/RBAC, reference tables, notification backend, hari libur | Backend foundation siap dipakai Sprint 2 |
| Grantly | Keycloak SSO/logout, audit log base, migration utama, QA checklist template | Auth/audit dasar dan format QA siap |
| Adithian | Design system, layout, sidebar, navbar, halaman auth/unregistered | Frontend foundation siap mock data |
| Adriel | PR template, branch naming, checklist review, komponen kecil, merge gate awal | GitHub workflow siap dipakai harian |

### Sprint 2 — Data Pegawai Core (Minggu 3-4)

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 2.1 | Hari 1-5 | CRUD pegawai core (US-2.1, US-2.2, US-2.3, US-2.4) | Tambah, edit, daftar, dan detail pegawai selesai end-to-end | Dion kunci field wajib dan AC; Adithian buat form/tabel/detail dengan dummy; Jordan bangun model-controller-validation; Grantly bantu query/test data dan QA CRUD; Adriel review PR dan merge | Admin bisa tambah/edit/lihat daftar/detail pegawai memakai real data |
| 2.2 | Hari 6-8 | Riwayat pegawai (US-2.6) | Riwayat pangkat, jabatan, dan KGB tampil di detail pegawai | Adithian buat tab/timeline riwayat dummy; Jordan buat append-only logic dan latest flag; Grantly bantu seed dan skenario validasi; Adriel review integrasi UI-backend | Riwayat pegawai bisa ditambah dan tampil di detail |
| 2.3 | Hari 9-10 | Hukuman disiplin + hardening Sprint 2 (US-2.7) | Data disiplin masuk ke profil pegawai dan statusnya valid | Adithian buat UI status/riwayat disiplin; Jordan buat CRUD dan auto-deactivate rule; Grantly QA regresi data pegawai; Adriel pastikan semua PR Sprint 2 bersih conflict | Sprint 2 siap demo: CRUD pegawai + riwayat + disiplin |

### Sprint 3 — Import & Pelengkap Data Pegawai (Minggu 5-6)

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 3.1 | Hari 1-5 | Import Excel/CSV (US-3.1, US-3.2, US-3.3, US-3.4) | Template, upload, preview, validasi, dan eksekusi import | Dion kunci format kolom; Adithian buat UI upload/preview dummy; Jordan buat template dan kontrak import; Grantly bangun parser/queue/validasi dan QA file sample; Adriel review PR import | Admin bisa import data pegawai dari file sample dan melihat hasil validasi |
| 3.2 | Hari 6-8 | Profil sendiri + data keluarga (US-2.5, US-2.8) | Pegawai bisa melihat data sendiri dan admin bisa kelola keluarga | Adithian buat profil read-only dan tab keluarga; Jordan buat policy/query data sendiri; Grantly bantu CRUD keluarga dan QA role access; Adriel review UI dan conflict | Profil sendiri dan data keluarga berjalan dengan real data |
| 3.3 | Hari 9-10 | Soft delete/restore + hard delete terbatas (US-2.9, US-2.10) | Penghapusan aman sesuai role | Adithian buat dialog konfirmasi; Jordan buat soft delete/restore/hard delete policy; Grantly QA negative case; Adriel review flow destruktif sebelum merge | Data pegawai bisa dinonaktifkan, dipulihkan, dan hard delete hanya untuk role yang berhak |

### Sprint 4 — Cuti Core (Minggu 7-9)

Sprint 4 dibuat 3 minggu karena cuti punya dependensi business rule, role approval, saldo, dan timeline.

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 4.1 | Hari 1-5 | Setup aturan cuti (US-4.10, US-4.11, US-4.12, US-4.3) | Atasan, approver, hari kerja, jenis cuti, dan saldo siap | Dion validasi aturan LLDIKTI; Adithian buat UI assign/saldo dummy; Jordan buat tabel dan service saldo/approver; Grantly bantu kalkulasi hari kerja dan QA aturan; Adriel review PR kritis | Admin bisa assign atasan/approver dan saldo cuti terhitung |
| 4.2 | Hari 6-10 | Pengajuan & daftar cuti pegawai (US-4.1, US-4.2) | Pegawai bisa ajukan cuti dan melihat status | Adithian buat form cuti dan daftar status dummy; Jordan buat controller validasi saldo dan submit; Grantly QA pengajuan valid/invalid; Adriel review PR flow pegawai | Pengajuan cuti masuk ke sistem dan status awal tampil |
| 4.3 | Hari 11-15 | Approval stage + timeline (US-4.4, US-4.5, US-4.6, US-4.7) | Approval berjenjang dan timeline terlihat | Adithian buat daftar approval dan timeline; Jordan buat state machine approval; Grantly QA E2E approve/tunda/tolak dan skip approver duplikat; Adriel review/merge bertahap | Cuti end-to-end siap demo dari pengajuan sampai approval final |

### Sprint 5 — EWS & Notifikasi (Minggu 10-11)

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 5.1 | Hari 1-4 | Kalkulasi TMT & scheduler EWS (US-5.1, US-5.5) | Alert kepegawaian bisa dihitung otomatis | Dion validasi rule; Jordan buat command/scheduler; Grantly bantu query dan dry-run QA; Adithian siapkan placeholder hasil dummy; Adriel review PR backend kritis | Scheduler EWS menghasilkan data alert yang bisa diuji |
| 5.2 | Hari 5-7 | Daftar EWS + flag kinerja + EWS pribadi (US-5.2, US-5.3, US-5.4) | Alert bisa dilihat admin/pegawai sesuai role | Adithian buat tabel/warna/status dummy; Jordan buat filter dan update flag; Grantly QA role access dan edge case; Adriel review UI/merge | EWS tampil dengan real data dan flag bisa diperbarui |
| 5.3 | Hari 8-10 | Notifikasi & session timeout (US-6.2, US-6.3, US-6.4, US-1.3) | Notifikasi email/in-app dan keamanan session siap | Adithian buat halaman notifikasi; Jordan buat mail/queue/session timeout; Grantly QA email via Mailpit dan tandai dibaca; Adriel review template/merge | Notifikasi dan session timeout siap diuji regression |

### Sprint 6 — Dashboard & Laporan (Minggu 12-13)

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 6.1 | Hari 1-4 | Dashboard admin & pegawai (US-8.1, US-8.2) | Ringkasan data utama tampil sebagai KPI/chart | Dion validasi metrik; Adithian buat dashboard dummy; Jordan buat query agregasi; Grantly QA angka dashboard; Adriel review UI/chart | Dashboard admin dan pegawai memakai real data |
| 6.2 | Hari 5-7 | Dashboard atasan + reference tables (US-8.3, US-8.5) | Atasan melihat bawahan dan admin kelola master data | Adithian buat dashboard atasan dan CRUD reference UI; Jordan buat policy/query; Grantly QA role access/reference update; Adriel review PR | Dashboard atasan dan reference tables siap |
| 6.3 | Hari 8-10 | Laporan & export (US-9.1, US-9.2, US-9.3, US-9.4) | Export pegawai dan cuti ke Excel/PDF | Adithian siapkan halaman filter/export; Jordan buat query laporan; Grantly bantu export Excel dan QA file output; Adriel bantu layout PDF/review merge | Export Excel/PDF berjalan dan hasil file diverifikasi |

### Sprint 7 — Stabilization, Regression, UAT, Go-Live Prep (Minggu 14-16)

| Urutan | Durasi | Vertical Slice | Fokus Bersama | Pembagian Kerja | Output Selesai |
|--------|--------|----------------|---------------|-----------------|----------------|
| 7.1 | Hari 1-4 | Audit view + redirect role + polish P1 (US-7.2, US-7.3, US-1.5) | Fitur P1 yang mendukung operasional ditutup | Adithian polish UI; Jordan selesaikan redirect/policy; Grantly QA audit dan role redirect; Adriel review/merge | Audit log bisa dibaca, redirect role benar, polish P1 masuk |
| 7.2 | Hari 5-8 | Bugfix hasil review & QA | Semua bug mayor dari Sprint 2-6 ditutup | Dion prioritisasi bug; owner task memperbaiki; Adriel mengembalikan PR ke owner jika bug belum fix; Grantly retest dan beri status | Tidak ada bug kritis/major terbuka sebelum UAT |
| 7.3 | Hari 9-15 | Full regression, UAT, release candidate | Sistem dicek dari login sampai laporan | Dion koordinasi UAT dan dokumentasi final; Adriel tahan merge yang belum punya evidence; Grantly jalankan QA akhir/regression; Jordan/Adithian fix bug sesuai area | Release candidate siap, QA sign-off, dan dokumen handover siap |

### Dependensi yang Harus Didahulukan

| Sprint | Harus selesai dulu | Baru bisa dilanjutkan ke | Cara agar tetap jalan |
|--------|--------------------|--------------------------|-----------------------|
| Sprint 1 | Project Laravel, `.env.example`, koneksi DB, branch `develop`, PR template | Semua slice Sprint 2-7 | Frontend mulai dari Blade static dan dummy data; backend menyiapkan kontrak field |
| Sprint 1 | Layout master dan komponen dasar form/table/badge/modal | Semua halaman fitur | Adithian pegang komponen utama, Adriel bantu komponen kecil dan review |
| Sprint 1 | Employee model, auth mapping, role/permission dasar | Data pegawai, cuti, dashboard, EWS | Jordan/Grantly sepakati nama tabel/field sebelum UI real data |
| Sprint 2 | Data pegawai core | Import, cuti, EWS, dashboard, laporan | Jika backend belum final, frontend memakai fixture yang sama dengan kontrak field |
| Sprint 3 | Import dan profil/keluarga stabil | Data awal untuk demo/UAT | Grantly pakai file sample untuk QA dan Dion validasi mapping |
| Sprint 4 | Cuti end-to-end stabil | Dashboard cuti dan export cuti | Timeline/approval diuji E2E sebelum masuk laporan |
| Sprint 5 | EWS dan notifikasi stabil | Dashboard alert dan regression | Scheduler diuji dry-run sebelum aktif otomatis |
| Sprint 6 | Dashboard/export stabil | UAT dan release candidate | File export dan angka dashboard wajib punya evidence QA |

### Prinsip Supaya Tidak Saling Menunggu

- Setiap slice dimulai dengan kontrak singkat: route, controller/method, field request, field view/response, dummy data, role access, dan acceptance criteria.
- Frontend boleh mulai dari fixture/mock data selama kontrak field sudah disepakati.
- Backend boleh merge service/controller dasar dulu walau UI belum final, selama route/field tidak berubah sembarangan.
- Setelah backend selesai, tugas integrasi wajib menghapus dummy data dari flow utama dan menggantinya dengan real data.
- Adriel melakukan review PR harian; bug/error dikembalikan ke developer pemilik task, bukan dibiarkan menumpuk di akhir sprint.
- Grantly melakukan QA akhir slice/sprint, mencatat evidence, dan retest setelah bug diperbaiki.

---

## 5. Workflow Harian

```
09:00   Daily Standup (15 menit, semua anggota)
        - Apa yang dikerjakan kemarin?
        - Apa yang akan dikerjakan hari ini?
        - Ada blocker?

09:15   Vertical slice kickoff / lanjut kerja
        - Dion cek AC dan scope slice aktif
        - Frontend lanjut UI dummy atau integrasi real data
        - Backend lanjut real data/logic sesuai kontrak

14:00   Sync frontend-backend
        - Kunci/cek route, field, validation error, empty state, dan role access
        - Jika backend sudah siap, frontend mulai ganti dummy data ke real data

16:00   Adriel: Review PR
        - Bug dicatat dengan evidence dan dikembalikan ke owner task
        - Owner task fix bug di hari yang sama jika memungkinkan

16:30   Adriel: review PR, cek conflict/checklist/evidence, merge PR yang layak
        Catatan: PR milik Adriel di-review dan di-merge oleh Dion atau Adithian

17:00   Gratnly QA Testing, melakukan penecekal secara keseluruhan dan memasikan tidak ada masalah pada fitur


Dion: update board, status slice, blocker, dan prioritas bug besok
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
       └── feature/US-2.8-family-ui          (Adriel)
```

### Code Review Rules
- Setiap PR **wajib di-review minimal 1 orang** sebelum merge.
- Adriel menjadi **GitHub review/merge gate**: cek deskripsi PR, checklist AC, screenshot/evidence, conflict, dan hasil test/checks sebelum merge ke `develop`.
- PR backend direview teknis oleh Jordan, Grantly, atau Dion; setelah approve, Adriel melakukan merge.
- PR frontend direview teknis oleh Adithian atau Adriel; setelah approve/checks aman, Adriel melakukan merge.
- Jika review menemukan bug/error, Adriel mengembalikan PR ke developer pemilik task dengan catatan perbaikan.
- PR critical (auth, approval engine, EWS, audit log, export final) wajib mendapat review Dion atau reviewer domain senior sebelum Adriel merge.
- PR milik Adriel tidak boleh self-review/self-merge; review dan merge dilakukan Dion atau Adithian.
- PR milik Grantly yang terkait QA/docs tetap perlu review domain jika mengubah logic aplikasi; perubahan dokumentasi murni cukup review Adriel/Dion.

### Definition of Done (DoD)
Sebuah user story dianggap "Done" jika:
- [ ] Code sudah di-merge ke branch `develop`
- [ ] Semua acceptance criteria terpenuhi
- [ ] Frontend sudah tidak memakai dummy data untuk flow utama setelah backend real data tersedia
- [ ] Unit test untuk business logic kritis sudah ada
- [ ] Adriel sudah melakukan review/merge gate GitHub atau PR milik Adriel sudah di-review/merge oleh Dion/Adithian
- [ ] Grantly sudah melakukan QA/retest dan tidak ada bug kritis terbuka
- [ ] Grantly sudah memperbarui checklist QA, catatan retest, dan dokumentasi operasional jika story berdampak ke setup/flow teknis
- [ ] UI sesuai dengan design (responsive di desktop & tablet)
- [ ] Audit log berfungsi untuk operasi CRUD terkait

---

## 7. Distribusi Beban Kerja

| Nama | Total Stories (Owner) | Total SP | Rata-rata SP/Sprint |
|------|-----------------------|----------|---------------------|
| Dion | PM/System Analyst semua slice | ~15 koordinasi teknis | ~2 + planning/review |
| Jordan | Lead backend mayoritas slice | ~70 backend | ~10 |
| Grantly | Support backend + QA semua slice | ~45 backend + QA | ~6 backend + QA |
| Adithian | Lead frontend mayoritas slice | ~60 frontend | ~8-9 |
| Adriel | Frontend support + GitHub gate | ~30 frontend + review/merge | ~4 frontend + review harian |

> **Catatan:** Angka di atas bersifat indikatif karena semua slice dikerjakan kolaboratif. Jordan tetap memegang backend lebih besar daripada Grantly, Adithian memegang frontend lebih besar daripada Adriel, Adriel fokus pada GitHub gate, dan Grantly menjadi QA tester akhir.

---

## 8. Skill Gap & Rekomendasi Belajar

| Anggota | Perlu Dipelajari | Resource |
|---------|-----------------|----------|
| **Jordan (Lead Backend)** | Laravel Queue + Scheduler | [Laravel Queues](https://laravel.com/docs/queues) |
| **Jordan (Lead Backend)** | Laravel Notification | [Laravel Notifications](https://laravel.com/docs/notifications) |
| **Grantly (Support Backend + QA)** | Keycloak + Socialite/OpenID | [Laravel Socialite](https://laravel.com/docs/socialite) |
| **Grantly (Support Backend + QA)** | Laravel Auditing + dokumentasi teknis | [Laravel Auditing](https://laravel-auditing.com) |
| **Grantly (Support Backend + QA)** | QA checklist, smoke test, regression evidence | Laravel/PHPUnit/Dusk docs internal |
| **Adithian** | Filament (jika dipakai) | [filamentphp.com](https://filamentphp.com) |
| **Adithian** | Chart.js / ApexCharts | [apexcharts.com](https://apexcharts.com) |
| **Adriel** | GitHub PR review, branch protection, merge workflow | GitHub Pull Requests docs |
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
| Frontend menunggu backend | Tinggi | Wajib mulai dari dummy data dan kontrak field setiap slice |
| Backend/UI tidak sinkron saat integrasi | Tinggi | Sync harian jam 14:00 dan PR kecil per slice |
| Bug menumpuk di akhir sprint | Tinggi | Adriel return bug saat review; Grantly QA/retest per slice |
| Tidak ada UI/UX designer | Sedang | Gunakan Filament/component library dan design system internal |
| Satu orang sakit/tidak hadir | Sedang | Setiap area punya backup (lihat tabel di bawah) |
| Sprint 7 terlalu penuh | Tinggi | Sprint 7 dipakai untuk stabilization, regression, UAT, dan hanya P1 kritis |

### Backup Matrix
| Primary | Backup | Area |
|---------|--------|------|
| Jordan | Grantly | Lead backend / business logic |
| Grantly | Jordan | Support backend / query, import, export, scheduler |
| Grantly | Dion + owner fitur | QA operations, bug evidence, retest |
| Adithian | Adriel | Frontend/UI |
| Adriel | Dion + Adithian | GitHub review/merge |
| Dion | Grantly | Tech decisions |
