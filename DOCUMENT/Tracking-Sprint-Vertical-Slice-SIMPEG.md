# Tracking Sprint Vertical Slice
## SIMPEG Fase 1 - LLDIKTI Wilayah XVI

| Field | Detail |
|-------|--------|
| Dokumen | Tracking Sprint Vertical Slice SIMPEG Fase 1 |
| Acuan | PRD-SIMPEG-Fase1-Core.md, User-Stories-SIMPEG-Fase1.md, Issues-SIMPEG-Fase1.md |
| Model kerja | Vertical slice mulai Sprint 2 |
| Target | Sprint 1 sampai release candidate / UAT selesai pada 20 Agustus 2026 |
| Tanggal mulai project | 8 Juni 2026 |
| Tanggal selesai Sprint 1 | 20 Juni 2026 |
| Target selesai project | 20 Agustus 2026 |
| Branch integrasi | `development` |

---

Jadwal pengerjaan sprint:

- Sprint 1 dikerjakan dari tanggal 8 Juni 2026 sampai 20 Juni 2026.
- Sprint 2 dikerjakan dari tanggal 21 Juni 2026 sampai 30 Juni 2026.
- Sprint 3 dikerjakan dari tanggal 1 Juli 2026 sampai 10 Juli 2026.
- Sprint 4 dikerjakan dari tanggal 11 Juli 2026 sampai 20 Juli 2026.
- Sprint 5 dikerjakan dari tanggal 21 Juli 2026 sampai 30 Juli 2026.
- Sprint 6 dikerjakan dari tanggal 31 Juli 2026 sampai 9 Agustus 2026.
- Sprint 7 dikerjakan dari tanggal 10 Agustus 2026 sampai 20 Agustus 2026.

---

## 1. Cara Pakai Tracker

Dokumen ini dipakai sebagai pegangan kerja sprint. Setiap fitur/task dikerjakan sebagai satu vertical slice: frontend mulai dari dummy data, backend membuat real data, lalu frontend disinkronkan ke backend, PR direview, bug diperbaiki, dan Grantly melakukan QA/retest sebelum tim pindah ke fitur berikutnya.

Status yang dipakai:

| Status | Arti |
|--------|------|
| Not Started | Belum mulai |
| In Progress | Sedang dikerjakan |
| Review | Menunggu review PR oleh Adriel / reviewer pengganti |
| Returned | Ada bug/error dan dikembalikan ke owner task |
| QA | Sedang dites oleh Grantly |
| Done | Sudah merge, QA pass, dan siap lanjut slice berikutnya |

Aturan pindah ke slice berikutnya:

- Semua acceptance criteria slice aktif sudah terpenuhi.
- Frontend flow utama sudah memakai real data jika backend sudah tersedia.
- PR sudah direview dan dimerge ke `development`.
- Bug critical/major sudah diperbaiki dan diretest.
- Grantly memberi status QA `Pass` atau `Pass with Note`.

---

## 2. Role Tetap

| Nama | Role | Fokus Handle |
|------|------|--------------|
| Dion Kobi | System Analyst & Project Manager | Scope, acceptance criteria, prioritas sprint, board tracking, komunikasi stakeholder, review PR kritis |
| Adithian Gunawan | Lead Frontend | UI flow, Blade views/components, mock data, integrasi frontend ke real data |
| Adriel Walintukan | Frontend Support + GitHub Management | Frontend support, review PR, conflict resolution, merge gate, bug routing |
| Jordan Sutarto | Lead Backend | Backend architecture, migration, model, controller/service, validation, business logic, query utama |
| Grantly Sorongan | Support Backend + QA Tester | Backend support, test data, QA scenario, smoke test, bug evidence, regression, retest |

Catatan dashboard role:

- **Super Admin** dan **Admin Kepegawaian** memakai satu halaman yang sama: **Dashboard Admin**.
- Perbedaannya bukan di dashboard terpisah, tetapi di permission/menu/aksi.
- Super Admin bisa melihat semua fitur Admin Kepegawaian plus konfigurasi sistem, user/role management, reference tables, audit penuh, serta soft delete/restore sesuai permission.
- Admin Kepegawaian memakai Dashboard Admin untuk operasional kepegawaian: data pegawai, import, cuti, EWS, dashboard, dan laporan, tetapi tidak mendapat aksi khusus Super Admin.

---

## 3. Gate Wajib Setiap Vertical Slice

| Urutan | Stage | Owner Utama | Yang Dikerjakan | Output Wajib |
|--------|-------|-------------|-----------------|--------------|
| 1 | Kickoff slice | Dion | Kunci scope, AC, role access, field, dummy data, dan prioritas | Kontrak slice disepakati |
| 2 | Frontend dummy | Adithian | Buat UI dengan mock/dummy data, empty state, error state, responsive view | UI bisa diklik/didemo tanpa backend final |
| 3 | Backend real data | Jordan | Buat migration/model/controller/service/validation/query | Real data siap dipakai frontend |
| 4 | Backend support | Grantly | Bantu query, seed, import/export, scheduler, test data sesuai slice | Backend lebih cepat stabil |
| 5 | Integrasi data | Adithian + Jordan | Ganti dummy data ke real data, cek field dan error response | UI berjalan dengan data backend |
| 6 | PR review | Adriel | Review PR, cek checklist, screenshot/evidence, conflict, dan checks | PR di-merge atau dikembalikan |
| 7 | Bugfix | Owner task | Perbaiki bug/error dari review atau QA | PR/update siap retest |
| 8 | QA/retest | Grantly | Smoke test, negative case, role access, data real, audit/log jika relevan | QA evidence dan status final |

---

## 4. Sprint 1 - Fondasi (8 Juni 2026 - 20 Juni 2026)

Sprint 1 belum memakai vertical slice penuh. Fokusnya menyiapkan fondasi agar Sprint 2 bisa berjalan dengan UI dummy dan backend real data secara paralel.

| No | Area | Owner | Yang Dikerjakan | Output | Status |
|----|------|-------|-----------------|--------|--------|
| 1.1 | Project baseline | Dion | Setup scope, backlog, board, sprint goal, branch workflow | Backlog dan workflow siap | Not Started |
| 1.2 | Keycloak SSO | Grantly | Login SSO, logout, middleware auth, fallback akun belum terdaftar, bootstrap Super Admin dari login SSO pertama | Auth dasar berjalan | Not Started |
| 1.3 | User mapping & RBAC | Jordan | Mapping user Keycloak ke pegawai, role/permission dasar, user tanpa role mendapat 403 hubungi admin | Akses per role siap | Not Started |
| 1.4 | Audit log base | Grantly | Audit log otomatis untuk auth dan CRUD awal | Audit log siap dipanggil fitur | Not Started |
| 1.5 | Notification base | Jordan | In-app notification base dan mail channel awal | Notifikasi dasar siap | Not Started |
| 1.6 | Reference tables | Jordan | Migration/seeder reference tables dan hari libur | Master data awal siap | Not Started |
| 1.7 | Design system | Adithian | Layout master, sidebar, navbar, table, form, badge, modal | Komponen UI siap dummy data | Not Started |
| 1.8 | GitHub workflow | Adriel | PR template, branch naming, checklist review, label, merge rule | Review/merge gate siap | Not Started |
| 1.9 | QA template | Grantly | Template checklist QA, bug report, retest evidence | Format QA siap dipakai | Not Started |

DoD Sprint 1:

- [x] `development` menjadi branch integrasi.
- [ ] Layout utama dan komponen dasar tersedia.
- [ ] Auth, mapping role, audit log, notification base, dan reference table minimal tersedia.
- [ ] PR template dan aturan review sudah dipakai.
- [ ] QA checklist template tersedia.

---

## 5. Sprint 2 - Data Pegawai Core (21 Juni 2026 - 30 Juni 2026)

> **Catatan penyelarasan 11 Agustus 2026:** PR #180 (`71d2dae`, exact head `9c1c1e9`) telah merge ke `development` dan menutup BUG-01 pada US-2.4 AC-2 serta melengkapi US-2.6 AC-1 sampai AC-5. Halaman detail kini memiliki kontrol tambah Kepangkatan/Jabatan/KGB yang digate permission, Upload SK, dan multipart submission. Tabel stage di bawah adalah rencana awal dan status `Not Started` tidak boleh dibaca sebagai status source terkini. Bukti source dan review menyatakan kriteria terkait selesai, sedangkan full browser/E2E submit multipart serta regression/UAT formal tetap menjadi tindak lanjut QA.

### Slice 2.1 - CRUD Pegawai Core

| Field | Detail |
|-------|--------|
| User stories | US-2.1, US-2.2, US-2.3, US-2.4 |
| Durasi target | Hari 1-5 |
| Goal | Admin bisa tambah, edit, lihat daftar, dan lihat detail pegawai memakai real data |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci field wajib, validasi, role admin, AC CRUD | Kontrak field pegawai | Not Started |
| Frontend dummy | Adithian | Form tambah/edit, tabel daftar, detail pegawai dengan dummy data | UI CRUD bisa didemo | Not Started |
| Frontend support | Adriel | Komponen form, upload preview, empty state, pagination | Komponen siap dipakai | Not Started |
| Backend | Jordan | Model, migration, controller, request validation, policy | CRUD real data siap | Not Started |
| Backend support | Grantly | Seeder/test data, query list/detail, skenario QA CRUD | Data uji tersedia | Not Started |
| Integrasi | Adithian + Jordan | Ganti dummy ke real data, cek validation error | UI CRUD real data | Not Started |
| Review | Adriel | Review PR CRUD, cek conflict/checklist/evidence | Merge atau return bug | Not Started |
| QA | Grantly | Test tambah, edit, list, detail, validasi NIP/NIK, audit log | QA pass/retest note | Not Started |

### Slice 2.2 - Riwayat Pegawai

| Field | Detail |
|-------|--------|
| User stories | US-2.6 |
| Durasi target | Hari 6-8 |
| Goal | Riwayat pangkat, jabatan, dan KGB bisa ditambah dan tampil di detail pegawai |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci jenis riwayat, field, aturan append-only, AC | Kontrak riwayat | Not Started |
| Frontend dummy | Adithian | Tab/timeline riwayat dengan dummy data | UI riwayat siap | Not Started |
| Backend | Jordan | Append-only logic, latest flag, relation ke pegawai | Riwayat real data siap | Not Started |
| Backend support | Grantly | Seeder riwayat, validasi edge case, test data | Data riwayat uji | Not Started |
| Integrasi | Adithian + Jordan | Tampilkan riwayat real di detail pegawai | Tab riwayat real data | Not Started |
| Review | Adriel | Review PR riwayat dan conflict | Merge atau return bug | Not Started |
| QA | Grantly | Test tambah riwayat, latest flag, audit log, detail pegawai | QA pass/retest note | Not Started |

### Slice 2.3 - Hukuman Disiplin + Hardening Sprint 2

| Field | Detail |
|-------|--------|
| User stories | US-2.7 |
| Durasi target | Hari 9-10 |
| Goal | Data disiplin masuk ke profil pegawai dan statusnya valid |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci field disiplin, status, aturan aktif/nonaktif | Kontrak disiplin | Not Started |
| Frontend dummy | Adithian | UI status/riwayat disiplin di detail pegawai | UI disiplin siap | Not Started |
| Backend | Jordan | CRUD disiplin dan auto-deactivate rule | Backend disiplin siap | Not Started |
| Backend support | Grantly | Test data disiplin dan QA regression data pegawai | Data QA siap | Not Started |
| Integrasi | Adithian + Jordan | Tampilkan disiplin real di detail pegawai | Data disiplin real | Not Started |
| Review | Adriel | Review PR dan pastikan semua PR Sprint 2 bersih conflict | Merge Sprint 2 | Not Started |
| QA | Grantly | Regression CRUD pegawai, riwayat, disiplin | Sprint 2 QA pass | Not Started |

---

## 6. Sprint 3 - Import & Pelengkap Data Pegawai (1 Juli 2026 - 10 Juli 2026)

> **Catatan penyelarasan 11 Agustus 2026:** PR #183 (`4f3f2c3`, exact head `cbf907b`) telah merge ke `development` dan menyelesaikan US-3.2 AC-4/AC-5 pada level source. Mapping header dapat diubah melalui dropdown, disimpan pada batch, dipakai kembali oleh preview/validasi/eksekusi, serta menolak target ganda dan field wajib yang belum dipetakan. Warning header asing, klasifikasi canonical/ignored, normalisasi UI/backend, dan enforcement fail-closed source `Role` juga tersedia. Tabel stage di bawah adalah rencana awal dan status `Not Started` tidak boleh dibaca sebagai status source terkini. Laravel Dusk belum menjadi quality gate CI; browser regression/UAT formal tetap menjadi tindak lanjut QA.

### Slice 3.1 - Import Excel/CSV

| Field | Detail |
|-------|--------|
| User stories | US-3.1, US-3.2, US-3.3, US-3.4 |
| Durasi target | Hari 1-5 |
| Goal | Admin bisa download template, upload file, preview, validasi, dan eksekusi import |

> **Batasan ruang lingkup import (keputusan pengguna 22 Juli 2026):** hanya template Data Utama yang aktif. Import membuat record pegawai beserta field snapshot awal, tidak membuat riwayat kepangkatan/jabatan/KGB, dan tidak memanggil kalkulasi TMT. Tanggal pensiun hasil import dipertahankan apa adanya. Riwayat resmi diinput per pegawai melalui CRUD append-only. Template lanjutan multi-jenis tidak dipulihkan tanpa keputusan eksplisit baru. Slice ini tidak bergantung pada Slice hari libur (Issue #10).

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci format kolom, sample file, rules validasi | Kontrak import | Not Started |
| Frontend dummy | Adithian | UI upload, preview, mapping, progress, error table dummy | UI import siap | Not Started |
| Backend | Jordan | Template generator dan kontrak endpoint import | Template/endpoint siap | Not Started |
| Backend support | Grantly | Parser, validasi row, queue job, report import | Engine import siap | Not Started |
| Integrasi | Adithian + Jordan + Grantly | Hubungkan UI upload/preview ke backend import | Import real berjalan | Not Started |
| Review | Adriel | Review PR import dan evidence file sample | Merge atau return bug | Not Started |
| QA | Grantly | Test file valid, invalid, duplikat, required field, hasil import | QA pass/retest note | Not Started |

### Slice 3.2 - Profil Sendiri + Data Keluarga

| Field | Detail |
|-------|--------|
| User stories | US-2.5, US-2.8 |
| Durasi target | Hari 6-8 |
| Goal | Pegawai bisa melihat profil sendiri dan admin bisa kelola data keluarga |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci role access profil sendiri dan keluarga | Kontrak akses | Not Started |
| Frontend dummy | Adithian | Profil read-only dan tab keluarga dummy | UI profil/keluarga siap | Not Started |
| Frontend support | Adriel | Komponen tabel/form keluarga jika dibutuhkan | UI support siap | Not Started |
| Backend | Jordan | Policy data sendiri dan query profil | Profil real data siap | Not Started |
| Backend support | Grantly | CRUD keluarga dan test data keluarga | Backend keluarga siap | Not Started |
| Integrasi | Adithian + Jordan | Ganti dummy profil/keluarga ke real data | Flow real data | Not Started |
| Review | Adriel | Review PR profil dan keluarga | Merge atau return bug | Not Started |
| QA | Grantly | Test role pegawai/admin, CRUD keluarga, audit log | QA pass/retest note | Not Started |

### Slice 3.3 - Soft Delete dan Restore Pegawai

| Field | Detail |
|-------|--------|
| User stories | US-2.9, US-2.10 |
| Durasi target | Hari 9-10 |
| Goal | Data pegawai bisa dinonaktifkan dan dipulihkan tanpa penghapusan permanen |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci aturan soft delete, restore, role access, audit log | Kontrak delete | Not Started |
| Frontend dummy | Adithian | Dialog soft delete/restore dan state nonaktif | UI delete siap | Not Started |
| Frontend support | Adriel | Review dialog konfirmasi soft delete dan flow restore | UI delete aman | Not Started |
| Backend | Jordan | Soft delete, restore, policy akses, filter data non-aktif | Backend delete siap | Not Started |
| Backend support | Grantly | Negative case, data uji, audit evidence | QA data siap | Not Started |
| Integrasi | Adithian + Jordan | Hubungkan UI delete ke backend | Flow delete real data | Not Started |
| Review | Adriel | Review PR soft delete/restore sebelum merge | Merge atau return bug | Not Started |
| QA | Grantly | Test permission, soft delete, restore, audit log | Sprint 3 QA pass | Not Started |

---

## 7. Sprint 4 - Cuti Core (11 Juli 2026 - 20 Juli 2026)

### Slice 4.1 - Setup Aturan Cuti

| Field | Detail |
|-------|--------|
| User stories | US-4.10, US-4.11, US-4.12, US-4.3 |
| Durasi target | Hari 1-5 |
| Goal | Kepala Bagian, approver, hari kerja, jenis cuti, dan saldo siap dipakai pengajuan |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Validasi aturan LLDIKTI, approval chain, carry-over | Kontrak aturan cuti | Not Started |
| Frontend dummy | Adithian | UI assign kepala bagian/approver dan saldo dummy | UI setup cuti siap | Not Started |
| Frontend support | Adriel | Komponen dropdown, card saldo, history saldo | UI support siap | Not Started |
| Backend | Jordan | Tabel cuti, saldo, approver, supervisor, service saldo | Backend aturan siap | Not Started |
| Backend support | Grantly | Kalkulasi hari kerja dan seed hari libur | Kalkulasi siap | Not Started |
| Integrasi | Adithian + Jordan | Hubungkan assign/saldo ke real data | Setup cuti real data | Not Started |
| Review | Adriel | Review PR kritis setup cuti | Merge atau return bug | Not Started |
| QA | Grantly | Test assign, saldo, hari kerja, role access | QA pass/retest note | Not Started |

### Slice 4.2 - Pengajuan & Daftar Cuti Pegawai

| Field | Detail |
|-------|--------|
| User stories | US-4.1, US-4.2 |
| Durasi target | Hari 6-10 |
| Goal | Pegawai bisa ajukan cuti dan melihat status pengajuan |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci jenis cuti, validasi saldo, status awal | Kontrak pengajuan | Selesai (Keputusan-Skema-Cuti-Canonical, Keputusan-Cuti-Saldo-Tahap-0) |
| Frontend dummy | Adithian | Form pengajuan, daftar cuti, badge status dummy | UI cuti pegawai siap | Selesai (PR #116) |
| Backend | Jordan | Controller submit, validation, status awal, saldo check | Backend pengajuan siap | Selesai (PR #104, #107) |
| Backend support | Grantly | Test data saldo, negative case, weekend/holiday | Data QA siap | Belum terverifikasi |
| Integrasi | Adithian + Jordan | Form dan daftar memakai real data | Pengajuan real data | Selesai, tanpa dummy data tersisa pada flow utama |
| Review | Adriel | Review PR flow pegawai dan screenshot | Merge atau return bug | Selesai untuk PR yang sudah masuk `development` |
| QA | Grantly | Test submit valid/invalid, saldo habis, hari libur, audit log | QA pass/retest note | Belum terverifikasi |

> **Catatan penyelarasan 7 Agustus 2026:** status di atas sebelumnya tercatat `Not Started` untuk seluruh stage, padahal pekerjaan slice ini sudah masuk `development` melalui beberapa PR. Penyelarasan dilakukan hanya untuk stage yang memiliki bukti PR atau dokumen; stage milik Grantly dibiarkan terbuka karena buktinya tidak diperiksa. Seluruh acceptance criteria US-4.2 kini selesai setelah PR #171 masuk. Mohon koreksi bila ada bukti tambahan.

### Slice 4.3 - Approval Stage + Timeline

| Field | Detail |
|-------|--------|
| User stories | US-4.4, US-4.5, US-4.6, US-4.7 |
| Durasi target | Hari 11-15 |
| Goal | Cuti berjalan end-to-end dari pengajuan sampai approval final dan timeline terlihat |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci approval stage, tunda/tolak, skip approver duplikat | Kontrak approval | Not Started |
| Frontend dummy | Adithian | Daftar approval, tombol aksi, timeline dummy | UI approval siap | Not Started |
| Frontend support | Adriel | Timeline partial dan state status approval | UI support siap | Not Started |
| Backend | Jordan | State machine approval dan final approve | Backend approval siap | Not Started |
| Backend support | Grantly | QA scenario approval E2E dan skip approver | Scenario siap | Not Started |
| Integrasi | Adithian + Jordan | Timeline dan action memakai real data | Approval real data | Not Started |
| Review | Adriel | Review/merge bertahap approval critical | Merge atau return bug | Not Started |
| QA | Grantly | Test E2E approve/tunda/tolak, saldo, notification, audit | Sprint 4 QA pass | Not Started |

---

## 8. Sprint 5 - EWS & Notifikasi (21 Juli 2026 - 30 Juli 2026)

### Slice 5.1 - Kalkulasi TMT & Scheduler EWS

| Field | Detail |
|-------|--------|
| User stories | US-5.1, US-5.5 |
| Durasi target | Hari 1-4 |
| Goal | Alert kepegawaian bisa dihitung otomatis dan diuji dry-run |

> **Batasan kalkulasi TMT (keputusan pengguna 22 Juli 2026):** kalkulasi TMT hanya dipicu saat riwayat/sumber resmi disimpan per pegawai, bukan saat import massal selesai. Import Data Utama tidak memanggil kalkulasi ini dan tanggal pensiun hasil import dipertahankan apa adanya.

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Validasi rule KGB, pangkat, pensiun, kontrak PPPK | Kontrak EWS | Not Started |
| Frontend dummy | Adithian | Placeholder hasil EWS dummy untuk dashboard/list | UI placeholder siap | Not Started |
| Backend | Jordan | Command/scheduler, query alert, dry-run | Scheduler siap | Not Started |
| Backend support | Grantly | Query support, seed tanggal target, dry-run QA | Data EWS siap | Not Started |
| Integrasi | Adithian + Jordan | Siapkan output EWS untuk UI list | Data alert tersedia | Not Started |
| Review | Adriel | Review PR backend critical | Merge atau return bug | Not Started |
| QA | Grantly | Test dry-run, no duplicate, tanggal target, log scheduler | QA pass/retest note | Not Started |

### Slice 5.2 - Daftar EWS, Flag Kinerja, EWS Pribadi

| Field | Detail |
|-------|--------|
| User stories | US-5.2, US-5.3, US-5.4 |
| Durasi target | Hari 5-7 |
| Goal | Alert bisa dilihat admin/pegawai sesuai role dan flag bisa diperbarui |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci role access dan status urgency | Kontrak EWS UI | Selesai, kontrak terbaca pada kriteria US-5.2 dan US-5.3 yang sudah selesai |
| Frontend dummy | Adithian | Tabel EWS, warna urgency, EWS pribadi dummy | UI EWS siap | Selesai |
| Frontend support | Adriel | Toggle flag kinerja dan empty/error state | UI support siap | Selesai |
| Backend | Jordan | Filter EWS, role access, update flag | Backend EWS siap | Selesai (PR #173 melengkapi penahanan pengingat dan teks bantuan flag kinerja) |
| Backend support | Grantly | Edge case alert dan role access test data | QA data siap | Belum terverifikasi |
| Integrasi | Adithian + Jordan | Tabel EWS memakai real data | EWS real data | Selesai, daftar EWS aktif dan EWS pribadi membaca data nyata |
| Review | Adriel | Review PR EWS dan UI evidence | Merge atau return bug | Selesai untuk PR yang sudah masuk `development` |
| QA | Grantly | Test role, urgency, flag, EWS pribadi | QA pass/retest note | Belum terverifikasi |

> **Catatan penyelarasan 8 Agustus 2026:** status di atas sebelumnya tercatat `Not Started` untuk seluruh stage, padahal seluruh acceptance criteria US-5.2, US-5.3, dan US-5.4 sudah berstatus selesai pada branch `development` @ `7e831c1`. Stage milik Grantly dibiarkan belum terverifikasi karena tidak ada bukti pelaksanaan yang dapat ditelusuri.

### Slice 5.3 - Notifikasi & Session Timeout

| Field | Detail |
|-------|--------|
| User stories | US-6.2, US-6.3, US-6.4, US-1.3 |
| Durasi target | Hari 8-10 |
| Goal | Notifikasi email/in-app dan keamanan session siap diuji regression |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci trigger notifikasi dan timeout | Kontrak notifikasi | Not Started |
| Frontend dummy | Adithian | Halaman notifikasi dan badge state dummy | UI notifikasi siap | Not Started |
| Frontend support | Adriel | Template email dan aksi tandai dibaca | UI/template siap | Not Started |
| Backend | Jordan | Mail queue, notification read/unread, session timeout | Backend notifikasi siap | Not Started |
| Backend support | Grantly | Mailpit test, queue test, timeout scenario | QA data siap | Not Started |
| Integrasi | Adithian + Jordan | Badge dan halaman memakai real notifications | Notifikasi real data | Not Started |
| Review | Adriel | Review template, PR, conflict | Merge atau return bug | Not Started |
| QA | Grantly | Test Mailpit, read/unread, timeout, role redirect | Sprint 5 QA pass | Not Started |

---

## 9. Sprint 6 - Dashboard & Laporan (31 Juli 2026 - 9 Agustus 2026)

> **Rekonsiliasi 10 Agustus 2026:** seluruh issue Sprint 6 (#39, #40, #41, #42, #43, #46) sudah selesai pada source. Tabel stage di bawah adalah rencana/pembagian kerja awal dan tidak boleh dipakai sebagai status implementasi terkini; bukti PR per issue berada di `Tracking-Sprint-1-7/Sprint-6-Dashboard-dan-Laporan.md`. Formal QA/UAT tetap mengikuti Sprint 7.

### Slice 6.1 - Dashboard Admin & Pegawai

| Field | Detail |
|-------|--------|
| User stories | US-8.1, US-8.2 |
| Durasi target | Hari 1-4 |
| Goal | Dashboard admin untuk Super Admin/Admin Kepegawaian dan dashboard pegawai menampilkan KPI/chart dari real data |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Validasi metrik dashboard, sumber data, dan perbedaan permission Super Admin/Admin Kepegawaian | Kontrak metrik | Not Started |
| Frontend dummy | Adithian | KPI card, chart, mini table memakai dummy data | UI dashboard siap | Not Started |
| Backend | Jordan | Query agregasi dashboard admin/pegawai | Query dashboard siap | Not Started |
| Backend support | Grantly | Validasi angka dan sample data dashboard | QA data siap | Not Started |
| Integrasi | Adithian + Jordan | Dashboard memakai query real data | Dashboard real data | Not Started |
| Review | Adriel | Review UI/chart/evidence | Merge atau return bug | Not Started |
| QA | Grantly | Cek akurasi angka, role access, empty state | QA pass/retest note | Not Started |

### Slice 6.2 - Dashboard Kepala Bagian + Reference Tables

| Field | Detail |
|-------|--------|
| User stories | US-8.3, US-8.5 |
| Durasi target | Hari 5-7 |
| Goal | Kepala Bagian melihat bawahan dan admin bisa kelola reference tables |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci role kepala bagian dan daftar reference table | Kontrak dashboard kepala bagian | Not Started |
| Frontend dummy | Adithian | Dashboard Kepala Bagian dan CRUD reference dummy | UI siap | Not Started |
| Frontend support | Adriel | Komponen filter/table jika dibutuhkan | UI support siap | Not Started |
| Backend | Jordan | Policy/query bawahan dan CRUD reference table | Backend siap | Not Started |
| Backend support | Grantly | Test data bawahan dan reference table edge case | QA data siap | Not Started |
| Integrasi | Adithian + Jordan | UI memakai real data | Flow real data | Not Started |
| Review | Adriel | Review PR dashboard/reference | Merge atau return bug | Not Started |
| QA | Grantly | Test role kepala bagian, CRUD reference, delete protection | QA pass/retest note | Not Started |

### Slice 6.3 - Laporan & Export

| Field | Detail |
|-------|--------|
| User stories | US-9.1, US-9.2, US-9.3, US-9.4 |
| Durasi target | Hari 8-10 |
| Goal | Export data pegawai dan cuti ke Excel/PDF berjalan dan file output diverifikasi |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci format laporan, filter, header/footer, tanda tangan | Kontrak export | Not Started |
| Frontend dummy | Adithian | Halaman filter/export dan state download | UI export siap | Not Started |
| Frontend support | Adriel | Layout PDF, header/footer, signature area | PDF view siap | Not Started |
| Backend | Jordan | Query laporan pegawai/cuti dan endpoint export | Backend export siap | Not Started |
| Backend support | Grantly | Export Excel dan validasi file output | Excel/QA siap | Not Started |
| Integrasi | Adithian + Jordan + Grantly | Tombol export menghasilkan file real | Export berjalan | Not Started |
| Review | Adriel | Review file evidence, layout PDF, conflict | Merge atau return bug | Not Started |
| QA | Grantly | Test filter, Excel, PDF, data kosong, data besar sample | Sprint 6 QA pass | Not Started |

---

## 10. Sprint 7 - Stabilization, Regression, UAT, Go-Live Prep (10 Agustus 2026 - 20 Agustus 2026)

> **Rekonsiliasi 10 Agustus 2026:** audit server-side/masking/immutability, EWS flag kinerja, Hari Libur database-backed, validasi lintas tahun, carry-over source Sprint 6, dan template rantai approval unit sudah selesai melalui PR #140/#166/#170/#173/#174/#176/#177. Audit fail-closed menyeluruh serta QA/UAT/release gate masih terbuka. Issue #178 merupakan follow-up hardening terpisah. Rincian status aktif berada di `Tracking-Sprint-1-7/Sprint-7-Stabilization-Regression-UAT.md`.

### Slice 7.1 - Audit View + Redirect Role + Polish P1

| Field | Detail |
|-------|--------|
| User stories | US-7.2, US-7.3, US-1.5 |
| Durasi target | Hari 1-4 |
| Goal | Audit log bisa dibaca, redirect role benar, dan polish P1 selesai |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Kickoff | Dion | Kunci P1 yang wajib masuk sebelum UAT | Prioritas P1 final | Not Started |
| Frontend dummy | Adithian | Polish UI dan audit table/detail dummy jika perlu | UI polish siap | Not Started |
| Frontend support | Adriel | Audit view/detail diff dan review redirect PR | UI audit siap | Not Started |
| Backend | Jordan | Redirect per role, query audit, policy access | Backend audit/redirect siap | Not Started |
| Backend support | Grantly | QA audit log dan role redirect scenario | Scenario siap | Not Started |
| Integrasi | Adithian + Jordan | Audit/redirect memakai real data | Flow real data | Not Started |
| Review | Adriel | Review PR P1/release candidate awal | Merge atau return bug | Not Started |
| QA | Grantly | Test audit list/detail, diff, redirect semua role | QA pass/retest note | Not Started |

### Slice 7.2 - Bugfix Hasil Review & QA

| Field | Detail |
|-------|--------|
| User stories | Semua bug dari Sprint 2-6 |
| Durasi target | Hari 5-8 |
| Goal | Tidak ada bug critical/major terbuka sebelum UAT |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| Prioritization | Dion | Kelompokkan bug critical/major/minor dan tentukan owner | Bug board rapi | Not Started |
| Frontend fix | Adithian + Adriel | Fix bug UI, responsive, state, data binding | Bug frontend turun | Not Started |
| Backend fix | Jordan + Grantly | Fix bug validation, query, policy, service, export | Bug backend turun | Not Started |
| Review | Adriel | Review PR bugfix dan conflict | Merge atau return bug | Not Started |
| Retest | Grantly | Retest semua bug fixed dan update status | Bug critical/major closed | Not Started |
| Sign-off | Dion + Grantly | Putuskan bug minor yang boleh masuk known issue | UAT readiness | Not Started |

### Slice 7.3 - Full Regression, UAT, Release Candidate

| Field | Detail |
|-------|--------|
| User stories | Regression/UAT/release candidate |
| Durasi target | Hari 9-15 |
| Goal | Sistem dicek dari login sampai laporan dan siap handover/release candidate |

| Stage | Owner | Yang Dihandle | Output | Status |
|-------|-------|---------------|--------|--------|
| UAT prep | Dion | Jadwal UAT, skenario demo, data demo, daftar fitur | UAT plan siap | Not Started |
| Regression run | Grantly | Full regression auth, pegawai, import, cuti, EWS, dashboard, export, audit, notif | Regression evidence | Not Started |
| Release gate | Adriel | Tahan merge tanpa evidence, cek branch `development`, conflict, PR final | Branch siap RC | Not Started |
| Backend fix | Jordan | Fix bug backend dari regression/UAT | Backend stabil | Not Started |
| Frontend fix | Adithian | Fix bug UI dari regression/UAT | Frontend stabil | Not Started |
| Retest | Grantly | Retest bug UAT dan update status | UAT bug status jelas | Not Started |
| Handover | Dion + Grantly | Catatan known issue, runbook, QA summary, release note | Dokumen final siap | Not Started |

DoD Sprint 7:

- [ ] Semua P0 dan P1 kritis sudah merge ke `development`.
- [ ] Tidak ada bug critical/major terbuka.
- [ ] Full regression selesai dengan evidence.
- [ ] UAT selesai atau daftar issue UAT sudah diprioritaskan.
- [ ] Release candidate siap untuk deployment preparation.

---

## 11. Checklist Review PR Adriel

Gunakan checklist ini untuk semua PR sebelum merge.

- [ ] Branch mengikuti format yang disepakati.
- [ ] Deskripsi PR menjelaskan fitur, scope, dan issue/story terkait.
- [ ] Acceptance criteria dicentang.
- [ ] Screenshot/evidence UI tersedia untuk perubahan frontend.
- [ ] Test/checks lokal atau CI sudah dijalankan sesuai kebutuhan.
- [ ] Tidak ada conflict dengan `development`.
- [ ] Tidak ada dummy data tersisa di flow utama setelah backend real data tersedia.
- [ ] Jika ada bug/error, PR diberi komentar dan dikembalikan ke owner task.
- [ ] PR milik Adriel direview dan dimerge oleh Dion atau Adithian.

---

## 12. Checklist QA Grantly

Gunakan checklist ini di akhir setiap slice.

- [ ] Happy path berjalan.
- [ ] Negative case divalidasi.
- [ ] Role access sesuai permission.
- [ ] Empty state dan error state aman.
- [ ] Data yang tampil sudah real data jika backend tersedia.
- [ ] Operasi penting tercatat di audit log.
- [ ] Notifikasi/email diuji jika slice memicu notifikasi.
- [ ] Bug dicatat dengan langkah reproduksi, expected result, actual result, severity, dan owner.
- [ ] Retest dilakukan setelah bug diperbaiki.
- [ ] Status akhir slice dicatat: Pass, Pass with Note, atau Failed.

---

## 13. Template Bug Report

```text
Judul:
Sprint/Slice:
Severity: Critical / Major / Minor
Owner perbaikan:
Environment:

Langkah reproduksi:
1.
2.
3.

Expected result:
Actual result:
Evidence:
Catatan tambahan:

Status retest:
```

---

## 14. Ringkasan Timeline

| Sprint | Periode Pengerjaan | Fokus | Jumlah Slice | Gate Akhir |
|--------|--------|-------|--------------|------------|
| Sprint 1 | 8 Juni 2026 - 20 Juni 2026 | Fondasi project, auth, layout, GitHub, QA template | Foundation | Fondasi siap Sprint 2 |
| Sprint 2 | 21 Juni 2026 - 30 Juni 2026 | Data pegawai core | 3 | CRUD, riwayat, disiplin QA pass |
| Sprint 3 | 1 Juli 2026 - 10 Juli 2026 | Import dan pelengkap data pegawai | 3 | Import, profil/keluarga, delete policy QA pass |
| Sprint 4 | 11 Juli 2026 - 20 Juli 2026 | Cuti core | 3 | Cuti E2E QA pass |
| Sprint 5 | 21 Juli 2026 - 30 Juli 2026 | EWS dan notifikasi | 3 | EWS/notifikasi QA pass |
| Sprint 6 | 31 Juli 2026 - 9 Agustus 2026 | Dashboard dan laporan | 3 | Dashboard/export QA pass |
| Sprint 7 | 10 Agustus 2026 - 20 Agustus 2026 | Stabilization, regression, UAT, release candidate | 3 | RC siap dan tidak ada bug critical/major |
