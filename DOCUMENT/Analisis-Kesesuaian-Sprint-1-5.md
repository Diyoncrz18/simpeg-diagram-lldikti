# Analisis Kesesuaian Sprint 1 sampai Sprint 5 — SIMPEG

| Field | Nilai |
|---|---|
| Tanggal analisis | 17 Juli 2026 |
| Ruang lingkup | Sprint 1 — Fondasi; Sprint 2 — Data Pegawai Core; Sprint 3 — Import & Pelengkap; Sprint 4 — Cuti Core; Sprint 5 — EWS & Notifikasi |
| Kondisi repository | `SIMPEG/` pada branch `development`, commit `4ea27e8` (`feat: tambah autocomplete pegawai pada halaman cuti (#110)`) |
| Metode | Inspeksi source, migration, seeder, route, FormRequest, Action/Service, view Blade, konfigurasi scheduler, dan source test |
| Sumber acuan | `AGENTS.md`, PRD v1.2, panduan kode, user stories, issues, tracker vertical slice, dan pembagian tugas |
| Perubahan dalam analisis | Hanya laporan ini. Source aplikasi tidak diubah. |
| Analisis sebelumnya | Commit `50b1e77` — 14 Juli 2026 |
| Commit baru sejak analisis sebelumnya | 13 commit (`50b1e77`..`4ea27e8`): export hardening, dashboard Kepala Bagian & Pimpinan, alur cuti multi-role, dokumen Kepala Lembaga, ledger fix, refactor controller, perluas cuti mandiri, mapping user (Issue #4 #109), dokumen Feat (#108), autocomplete pegawai cuti (#110) |

## Legend Status

| Icon | Status | Arti |
|---|---|---|
| ✅ | Selesai pada source | Implementasi dan test/source relevan tersedia. Belum setara status tracker `Done`, karena QA/runtime belum dapat dieksekusi ulang. |
| ⚠️ | Sebagian | Fondasi tersedia, tetapi masih ada acceptance criteria, kesesuaian PRD, arsitektur, atau evidence QA yang belum terpenuhi. |
| ❌ | Belum selesai / tidak sesuai | Implementasi belum tersedia atau ada pelanggaran terhadap PRD, `AGENTS.md`, atau panduan kode. |

## Kesimpulan Eksekutif

Dari 38 issue Sprint 1–5, setelah 13 commit terbaru status adalah: 15 berstatus ✅ pada level source, 19 ⚠️ sebagian, dan 4 ❌ tidak sesuai. Terdapat kemajuan nyata dibanding analisis sebelumnya (commit `50b1e77`).

**Diperbaiki / ditutup sejak analisis sebelumnya:**

1. **Issue #4 (Mapping User) — naik ke ✅**: `UserMappingController` kini menggunakan `UpdateUserMappingRequest`, menyimpan `employee_id` ke tabel `users` berdasarkan pencocokan email pegawai, memvalidasi uniqueness `keycloak_id`, dan menulis audit log ke `audit_logs` via `AuditService`. `UserMappingControllerTest` tersedia dengan skenario guest, non-super-admin, update role, dan pencocokan `employee_id`.
2. **Dokumen eksternal Kepala Lembaga — ditutup sebagian (Issue #31)**: Fitur `KepalaLembagaSupportingDocument` tersedia dengan model, migration, controller, action (store/delete/list/prepare), FormRequest, test, dan route. Admin Kepegawaian dapat mengunggah scan dokumen untuk cuti Kepala Lembaga yang di-approve di luar SIMPEG, dengan audit dan perlindungan `is_kepala_lembaga`.
3. **Penghapusan Fitur Hard Delete Pegawai & Pemulihan Halaman Data Backup — Selesai**: Seluruh mekanisme *hard delete* (force delete, action, command console, dan jadwal scheduler harian pukul 02:00) telah dihapus sepenuhnya dari aplikasi. Halaman *Data Backup* telah dikembalikan sebagai wadah permanen data pegawai nonaktif tanpa waktu kedaluwarsa atau penghapusan otomatis (sesuai PRD).
4. **Autocomplete pegawai pada form cuti (#110)**: Komponen `employee-combobox` Blade tersedia dengan `EmployeeLookupRequest`, `CutiEmployeeLookupController`, `LookupEmployeesAction`, test `CutiEmployeeLookupTest`, dan route API.
5. **Email label lengkap**: `cuti.perlu_perubahan` dan `cuti.tidak_disetujui` kini terdaftar dalam `NotificationRecipientResolver::emailEnabled()` (sebelumnya absen).

**Masalah lama yang BELUM diperbaiki:**

1. Beberapa sub-fitur riwayat hukuman disiplin masih dapat dihapus (DELETE route/action aktif), bertentangan dengan prinsip append-only.
2. Profil pegawai masih mengekspos route mutasi keluarga (`profil-saya/keluarga`) dan pendidikan (`profil-saya/pendidikan`) yang melampaui batas Fase 1.
3. Import belum memiliki template lanjutan, manual column mapping, snapshot riwayat, dan kalkulasi TMT pasca-import.
4. Skema cuti berbeda dengan PRD canonical (`leave_request_steps` vs `leave_approval_steps`, dll). Keputusan belum didokumentasikan.
5. `RejectLeaveAction`, `LeaveApprovalService::reject()`, internal status `'rejected'`/`'request_rejected'`, label `'Ditolak'` di `LeaveProofService`, dan route `cuti.reject` masih aktif. Melanggar nomenklatur resmi Fase 1.
6. `app:run-ews` terdaftar ganda di `bootstrap/app.php` (baris 22–24) DAN `routes/console.php` (baris 27–29).
7. `NotificationRecipientResolver::emailEnabled()` masih hardcoded; tidak membaca `ref_notification_channels`; `ews.satyalancana` absen.
8. `SESSION_LIFETIME=120` di `.env.example` tidak konsisten dengan middleware idle 30 menit.

Batasan runtime tetap berlaku: PHP host 8.3.30 tidak kompatibel; Podman tidak tersedia.

## Bukti Validasi yang Tersedia

| Area | Bukti |
|---|---|
| Import Sprint 3 | `EmployeeImportController`, `UploadImportBatchAction`, `ValidateImportBatchAction`, `ExecuteImportBatchAction`, `ImportEmployeeBatchJob`, view wizard, serta test import/template tersedia. |
| Profil, keluarga, soft delete | `ShowProfilePageAction`, `ShowMyProfileAction`, Action keluarga, `DeactivateEmployeeAction`, `RestoreEmployeeAction`, test profile/family/deactivate-restore tersedia. |
| Fondasi cuti | Migration, model request/chain/step/balance/ledger/proof, schema test, assign supervisor, dan konfigurasi approval tersedia. |
| Pengajuan dan approval cuti | `StoreLeaveRequestRequest`, `SubmitLeaveRequestAction`, `LeaveApprovalService`, Action keputusan, notifikasi in-app/email queue, QR proof, serta test approval/proof tersedia. |
| Saldo dan hari kerja | `WorkdayCalculator`, endpoint API v1, `LeaveBalanceService`, rollover scheduler, seeder saldo 2026, feature/unit tests tersedia. |
| EWS Sprint 5 | Engine lima trigger, migration alert/log scheduler, command, konfigurasi threshold, halaman EWS, follow-up ber-audit, dan feature test tersedia. |
| Flag, email, dan timeout Sprint 5 | Action/FormRequest/test flag kinerja dan Satyalancana, job email ber-queue dengan tiga retry, template email, serta middleware timeout dan feature test tersedia. |
| **[BARU] Mapping user** | `UpdateUserMappingRequest`, penyimpanan `employee_id`, validasi uniqueness `keycloak_id`, audit via `AuditService`, `UserMappingControllerTest` tersedia. |
| **[BARU] Dokumen Kepala Lembaga** | Model `KepalaLembagaSupportingDocument`, migration, controller, 4 action, FormRequest, `KepalaLembagaSupportingDocumentTest`, route tersedia. |
| **[BARU] Autocomplete cuti** | `LookupEmployeesAction`, `EmployeeLookupRequest`, `CutiEmployeeLookupController`, komponen `employee-combobox`, `CutiEmployeeLookupTest`, route API tersedia. |
| PostgreSQL dan Podman | `compose.yml` dan konfigurasi PostgreSQL 17 tersedia; verifikasi runtime tidak bisa dilakukan karena Podman tidak ditemukan. |
| Test | Banyak Feature/Unit test tersedia. `phpunit.xml` telah diperbarui. Namun test masih memakai SQLite in-memory sebagai basis utama; tidak ada Laravel Dusk. |

## Status Issue Sprint 1

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #1 | Setup Laravel dan environment | ⚠️ | Laravel 12, PostgreSQL 17, queue, compose, README, Pint, PHPStan, Vite, dan helper Podman tersedia. Runtime terblokir oleh PHP host/PODMAN. |
| #2 | Keycloak SSO dan middleware | ⚠️ | Controller, Action callback/logout/redirect, middleware, config Socialite, dan test tersedia. Login IdP nyata belum diuji; bootstrap first mapped employee masih membutuhkan keputusan terdokumentasi. |
| #3 | Logout dan session management | ⚠️ | Logout POST, invalidasi session, dan audit tersedia. Route GET `/logout` juga memutasi session tanpa CSRF. |
| #4 | Mapping user Keycloak dan RBAC | ✅ | **[DIPERBAIKI commit #109]** `UpdateUserMappingRequest` terpisah, `employee_id` disimpan otomatis saat mapping berdasarkan pencocokan email pegawai, validasi uniqueness `keycloak_id`, audit log ditulis ke `audit_logs` via `AuditService`, dan `UserMappingControllerTest` tersedia (guest/role/update/employee_id). |
| #5 | Audit log | ✅ | Migration `audit_logs`, model immutable, `AuditService`, audit auth dan mutasi tersedia. |
| #6 | Notifikasi in-app backend | ✅ | Migration/model, service, action inbox/read/unread, controller API, dan test tersedia. |
| #7 | Bell icon notifikasi | ✅ | Komponen notification bell, endpoint, dan JavaScript aplikasi tersedia. |
| #8 | Design system dan layout master | ✅ | Layout, komponen UI/form reusable, Tailwind/Vite, dan UI Bahasa Indonesia tersedia. |
| #9 | Reusable Blade components | ⚠️ | Komponen utama tersedia, tetapi `resources/views/components/README.md` atau dokumentasi setara belum ada. |
| #10 | CRUD hari libur dan cuti bersama | ✅ | Model, API v1, FormRequest, Action, controller, view, audit, seeder, dan test tersedia. |
| #11 | Migration/seeder reference tables | ⚠️ | Mayoritas tersedia. `ref_notification_channels` tidak ada; unit kerja belum hierarkis; jabatan belum memuat BUP override/status aktif; seed status pegawai belum lengkap. |
| #12 | Setup testing framework | ⚠️ | PHPUnit dan test tersedia. Tidak ada Dusk; test memakai SQLite sehingga bukan bukti PostgreSQL 17. |

## Status Issue Sprint 2

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #13 | Migration tabel utama pegawai | ✅ | Model/relasi, UUID, index, cast enkripsi, dan migration sinkronisasi PRD tersedia. |
| #14 | Form tambah pegawai multi-tab | ✅ | FormRequest, Action, view multi-tab, validasi upload, storage, audit, dan test tersedia. |
| #15 | Form edit pegawai | ✅ | FormRequest, Action, view, penggantian berkas/foto, audit, dan test tersedia. |
| #16 | Halaman daftar pegawai | ✅ | Search, filter, sort, pagination, eager loading, default aktif, RBAC, dan test tersedia. |
| #17 | Halaman detail pegawai bertab | ✅ | Data relasi, tab, informasi EWS, view, dan test detail tersedia. |
| #18 | Riwayat pangkat, jabatan, KGB append-only | ✅ | `EmployeeHistoryService`, transaksi, `is_latest`, audit, pembaruan tanggal terhitung, dan test tersedia. |
| #19 | Riwayat hukuman disiplin | ❌ | Pembuatan/scheduler tersedia, tetapi route DELETE, action delete, dan tombol hapus menghapus riwayat append-only. |

## Status Issue Sprint 3

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #20 | Download template import | ⚠️ | Tombol dan download CSV UTF-8/XLSX tersedia melalui `GenerateImportTemplateAction` dan `ImportTemplateWriter`. Hanya template `utama` tersedia, hanya satu baris contoh, dan template Data Pelengkap/Riwayat Kepangkatan/Riwayat Jabatan/Riwayat KGB belum ada. |
| #21 | Upload, preview, validasi Excel/CSV | ⚠️ | Upload maksimal 10 MB, parser XLS/XLSX/CSV, preview, validasi per baris, duplicate NIP skip, error detail, wizard, dan test tersedia. Auto-match hanya menerima header canonical; dropdown/manual column mapping dan peringatan mapping kolom tak cocok belum ditemukan. |
| #22 | Eksekusi import queue dan laporan | ⚠️ | `ImportEmployeeBatchJob` memproses batch valid, progress polling, hasil inserted/skipped/failed, audit, notifikasi, dan unduh CSV error dari UI tersedia. Belum ada snapshot riwayat pangkat/jabatan/KGB, kalkulasi TMT setelah import, atau perhitungan pensiun ketika kolom kosong. |
| #23 | Profil sendiri read-only | ⚠️ | Profil memakai data pegawai login, saldo, tanggal pangkat/KGB, serta endpoint profile sendiri tersedia. Namun halaman memuat perubahan password dan API `/profil-saya/keluarga` serta `/profil-saya/pendidikan` untuk create/update/delete. Ini bukan read-only dan melampaui batas Fase 1. |
| #24 | CRUD data keluarga | ✅ | CRUD keluarga admin memakai FormRequest, Action, soft delete, audit masking NIK, data scope, API v1, dan test create/update/delete/role tersedia. |
| #25 | Soft delete dan restore pegawai | ✅ | Soft delete, restore, audit, daftar nonaktif, serta EWS exclusion tersedia. Seluruh mekanisme *hard delete* (force delete, action, command, scheduler harian) telah dihapus dari backend. Halaman Data Backup dikembalikan sebagai filter terpisah untuk restorasi data nonaktif secara aman tanpa batas waktu penyimpanan. |

## Status Issue Sprint 4

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #26 | Migration tabel cuti | ⚠️ | Tabel/model chain, snapshot step, balance, ledger, dan QR proof tersedia serta diuji. Schema runtime memakai `leave_request_steps`, `leave_balance_ledger`, `leave_proofs`; PRD mensyaratkan `leave_approval_steps`, `leave_balance_adjustments`, `leave_documents`, serta field canonical request. Konflik schema perlu keputusan terdokumentasi. |
| #27 | Assign Kepala Bagian per pegawai | ⚠️ | `kepala_bagian_id`, `SupervisorAssignment`, riwayat perubahan, larangan self-assign, audit, dan test tersedia. Route dibatasi Super Admin, padahal PRD/US menunjuk Admin Kepegawaian; tanggal mulai tidak dapat ditentukan pengguna; tidak ada enforcement bahwa semua pegawai memiliki tepat satu kepala bagian aktif. |
| #28 | Konfigurasi approval chain | ⚠️ | Chain per pegawai, step, PYBMC global, backfill, audit, FormRequest, dan skip duplicate tersedia. Konfigurasi per unit belum ada; form utama masih legacy stage 2/3; UI untuk menyusun chain per pegawai secara penuh tidak terbukti; step type dibatasi `kepala_bagian`/ `verifier`. |
| #29 | Kalkulasi hari kerja otomatis | ⚠️ | `WorkdayCalculator`, endpoint `/api/v1/cuti/calculate-workdays`, warning weekend/libur, validasi request, dan test unit/feature tersedia. Tetapi Blade menetapkan `workDays = result.data`, padahal kontrak API mengembalikan `data.jumlah_hari_kerja`; nilai/warning real-time tidak dirender sesuai kontrak. |
| #30 | Form pengajuan cuti | ⚠️ | Form, upload lampiran tervalidasi, hitung server-side, saldo, snapshot chain, status menunggu, audit, in-app/email queue, dan test tersedia. Dropdown tidak terbukti menyaring jenis Cuti Besar/CLTN untuk PPPK; validasi lintas tahun hanya diterapkan pada cuti tahunan, bukan semua pengajuan. |
| #31 | Approval engine cuti dinamis | ⚠️ | **[SEBAGIAN DIPERBAIKI]** Snapshot per request, data-scope approver, notifikasi, audit, final deduction, QR proof, timeline, dan test E2E tersedia. Dokumen eksternal Kepala Lembaga kini tersedia via `KepalaLembagaSupportingDocument` dengan upload, audit, dan akses `is_kepala_lembaga`. Namun source masih memakai `RejectLeaveAction` (class name), audit payload `'REJECT'`, `LeaveApprovalService::reject()`, internal status `'rejected'`/`'request_rejected'`, label `'Ditolak'` di `LeaveProofService`, dan route/name `cuti.reject`. Ini masih melanggar nomenklatur resmi Fase 1. |
| #32 | Saldo cuti dan daftar cuti pegawai | ✅ | Balance ledger, bucket N-2/N-1/tahun berjalan, seeder 2026, koreksi awal/manual ber-audit, rollover scheduler, halaman/API saldo, riwayat, dan test tersedia. Commit `5442035` mengunci jenis mutasi ledger saldo cuti. |

## Status Issue Sprint 5

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #33 | Kalkulasi TMT otomatis | ⚠️ | EmployeeHistoryService menghitung kenaikan pangkat dari TMT pangkat +4 tahun, KGB +2 tahun, serta pensiun dari tanggal lahir dan BUP jenis jabatan ketika riwayat terbaru dibuat. Namun tidak ada TmtCalculatorService/observer/listener, milestone Satyalancana tidak disimpan, tidak ada rekalkulasi saat import, dan perubahan referensi BUP tidak memicu hitung ulang. |
| #34 | EWS scheduler harian | ⚠️ | `EwsEngineService` memeriksa pegawai aktif untuk lima trigger, threshold configurable, unique alert, log run, error notification, command, dan test tersedia. **Double registrasi masih ada**: `app:run-ews` terdaftar di `bootstrap/app.php` baris 22–24 DAN `routes/console.php` baris 27–29, berisiko dieksekusi dua kali per hari. Alert tidak menyimpan `is_eligible`/`trigger_days`/`tahun` seperti task issue; in-app notification hanya dibuat untuk pegawai, bukan Admin Kepegawaian. |
| #35 | Halaman daftar EWS aktif | ✅ | Controller/action/view menampilkan nama, NIP, event, target, sisa hari, eligibility, dan follow-up; urutan sisa hari, warna urgensi, filter event/status, detail pegawai, akses Admin/Super Admin/Pimpinan, aksi ditangani/tidak perlu ber-catatan, audit, dan test role tersedia. |
| #36 | Flag kinerja baik dan kelayakan Satyalancana | ⚠️ | Toggle AJAX, endpoint JSON, FormRequest, Action, RBAC route, audit, flag/catatan Satyalancana, dan test tersedia. Deskripsi UI bukan tooltip dengan teks yang diminta. Saat flag kinerja false, source tetap membuat dan menampilkan alert pangkat sebagai Tidak Eligible, bertentangan dengan PRD yang menyatakan pegawai tidak muncul pada EWS kenaikan pangkat. |
| #37 | Notifikasi email | ⚠️ | `NotificationService`, job queue, tiga retry, log kegagalan final, template responsive Bahasa Indonesia, CTA aman, Mailpit development, dan test tersedia. **Kemajuan**: `cuti.perlu_perubahan` dan `cuti.tidak_disetujui` kini ditambahkan ke `emailEnabled()`. Namun pemilihan event email masih hardcoded via array di `NotificationRecipientResolver::emailEnabled()`, tidak membaca `ref_notification_channels`; `ews.satyalancana` masih absen dari daftar email. Admin EWS hanya menerima email tambahan, bukan notifikasi in-app. |
| #38 | Session timeout | ⚠️ | Middleware menegakkan idle timeout 30 menit, logout/invalidate session, flash, redirect route login Keycloak, JSON 401, audit SESSION_TIMEOUT, dan test polling tersedia. Tetapi .env.example dan default Laravel masih menetapkan SESSION_LIFETIME=120, bukan 30 sebagaimana Issue #38. |

## Task yang Sudah Selesai pada Source

Status berikut tetap memerlukan QA/retest pada PHP kompatibel dan PostgreSQL sebelum tracker dapat diubah menjadi `Done`.

### Sprint 1

- ✅ Audit log database, service, immutable model, dan halaman audit tersedia.
- ✅ Notifikasi in-app, inbox/read/unread, notification bell, serta queue email pendukung tersedia.
- ✅ Layout master, design system, dan komponen UI/form dasar tersedia.
- ✅ CRUD hari libur/cuti bersama dengan RBAC, audit, seeder, dan test tersedia.
- ✅ **[BARU]** Mapping user Keycloak: `UpdateUserMappingRequest`, simpan `employee_id`, validasi uniqueness `keycloak_id`, audit log ke `audit_logs`, dan test lengkap tersedia (Issue #4, commit #109).

### Sprint 2

- ✅ Migration/model data pegawai dan relasi riwayat tersedia.
- ✅ Tambah/edit/daftar/detail pegawai memakai data nyata, validasi, upload, audit, dan test tersedia. `EmployeeIndexTest` diperluas di commit terbaru.
- ✅ Riwayat pangkat, jabatan, dan KGB bersifat append-only dengan `is_latest`, transaksi, dan audit.

### Sprint 3

- ✅ CRUD keluarga admin memakai API v1, FormRequest, Action, data scope, soft delete, audit masking NIK, dan test.

### Sprint 4

- ✅ Saldo cuti berbasis ledger, initial balance, koreksi beralasan/audit, rollover tahunan, seed 2026, dan halaman/API saldo tersedia.
- ✅ **[BARU]** Dokumen eksternal Kepala Lembaga: `KepalaLembagaSupportingDocument` model/migration/controller/action/FormRequest/test tersedia; Admin dapat mengunggah dokumen persetujuan eksternal untuk cuti Kepala Lembaga (Issue #31 sebagian).

### Sprint 5

- ✅ Halaman daftar EWS aktif: data nyata, sort urgensi, filter event/status, warna urgensi, detail pegawai, akses role, follow-up ber-catatan, audit, dan test tersedia.
- ✅ Perhitungan TMT pangkat/KGB/pensiun saat riwayat terbaru ditambahkan tersedia pada `EmployeeHistoryService`.
- ✅ Engine EWS memiliki lima trigger, threshold configurable, unique index alert, log eksekusi, command, dan test source.
- ✅ Toggle kinerja dan kelayakan Satyalancana menggunakan AJAX, validasi, Action, audit, serta test role/invalid request.
- ✅ Email memakai job queue dengan tiga retry, log kegagalan, template Bahasa Indonesia, dan CTA internal aman.
- ✅ Idle timeout 30 menit memiliki invalidasi sesi, redirect Keycloak, audit, respons JSON, dan test source.

## Task yang Masih Belum Selesai atau Harus Dikoreksi

### Prioritas P0 — wajib sebelum Sprint 1–5 dinyatakan selesai

- ✅ **[SELESAI]** Hapus endpoint/API force delete, `DeleteEmployeeAction::forceDelete()`, `PurgeDeletedEmployeesAction`, command `employees:purge-deleted`, dan jadwal purge harian pukul 02:00 di `bootstrap/app.php`. Halaman Data Backup dikembalikan sebagai filter nonaktif permanen tanpa purge otomatis.
- ❌ Hapus route DELETE (`disiplin.destroy` di API v1 dan web), `DeleteDisciplineRecordAction`, dan endpoint penghapusan `DisciplineRecord`; riwayat disiplin wajib append-only.
- ✅ Perbaiki mapping Keycloak: simpan `users.employee_id`, validasi uniqueness `keycloak_id`, audit persisten di `audit_logs`, dan test tersedia — **SELESAI commit #109**.
- ❌ Ubah profil pegawai menjadi read-only. Hapus route mutasi `profil-saya/keluarga` (store/update/delete) dan `profil-saya/pendidikan` dari scope Fase 1; pisahkan perubahan password jika diperlukan secara administratif.
- ❌ Ganti semua istilah/action `REJECT` pada domain cuti: ganti nama class `RejectLeaveAction`, audit payload `'REJECT'`, `LeaveApprovalService::reject()`, internal status `'rejected'`/`'request_rejected'`, label `'Ditolak'` di `LeaveProofService`, dan route/name `cuti.reject` dengan nomenklatur `Tidak Disetujui`/`NOT_APPROVED`; pertahankan keterangan wajib dan audit.
- ❌ Putuskan lalu dokumentasikan schema canonical cuti sebelum membuat migration baru: apakah PRD memakai `leave_approval_steps`, `leave_balance_adjustments`, `leave_documents`, atau proyek mempertahankan nama semantic alternatif saat ini (`leave_request_steps`, `leave_balance_ledger`, `leave_proofs`). Konflik ini tidak boleh diselesaikan dengan asumsi.

### Prioritas P0 — tutup acceptance criteria Sprint 5

- ❌ Hapus salah satu registrasi jadwal `app:run-ews`. Saat ini terdaftar ganda: `bootstrap/app.php` (baris 22–24) DAN `routes/console.php` (baris 27–29), berisiko dieksekusi dua kali per hari. Pertahankan satu saja di `routes/console.php` yang configurable via `EwsConfig`, dan hapus dari `bootstrap/app.php`.
- ❌ Buat satu mekanisme kalkulasi TMT yang dipanggil saat riwayat dibuat/diubah, import selesai, dan referensi BUP berubah. Simpan atau modelkan milestone Satyalancana secara konsisten tanpa menduplikasi aturan di engine.
- ❌ Lengkapi data alert EWS yang diperlukan requirement (`is_eligible`, `trigger_days`, `tahun`), lalu pastikan semua penerima yang diwajibkan mendapat notifikasi in-app dan email untuk kelima event EWS termasuk Admin Kepegawaian.
- ❌ Terapkan source of truth PRD untuk flag kinerja false atau dokumentasikan keputusan: PRD menyebut alert pangkat tidak tampil, sedangkan source saat ini menampilkan sebagai Tidak Eligible.
- ❌ Buat notification dispatcher yang membaca konfigurasi `ref_notification_channels`. `NotificationRecipientResolver::emailEnabled()` saat ini hardcoded. Daftarkan `ews.satyalancana` yang masih absen, dan daftarkan seluruh keputusan cuti dan kelima event EWS.

### Prioritas P1 — tutup acceptance criteria Sprint 3

- ❌ Tambahkan template Data Pelengkap, Riwayat Kepangkatan, Riwayat Jabatan, dan Riwayat KGB.
- ❌ Tambahkan dua contoh data dummy pada setiap template yang relevan.
- ❌ Tambahkan manual column mapping dropdown, warning kolom tidak cocok, kontrak API, dan test-nya.
- ❌ Saat import sukses, buat snapshot riwayat yang tersedia, hitung TMT pangkat/KGB, dan hitung pensiun jika kolom kosong serta reference BUP tersedia.
- ❌ Sediakan laporan hasil import yang persisten/downloadable untuk error eksekusi, bukan hanya CSV yang dibangun dari state browser.

### Prioritas P1 — tutup acceptance criteria Sprint 4

- ❌ Beri Admin Kepegawaian hak assign Kepala Bagian sesuai PRD/US (bukan hanya Super Admin), dukung input tanggal mulai, dan tetapkan proses untuk memastikan satu Kepala Bagian aktif per pegawai.
- ❌ Tambahkan konfigurasi approval chain per unit dan UI penyusunan chain per pegawai yang lengkap. Ketua Tim Kerja tetap dipilih sebagai verifier tanpa role baru.
- ❌ Saring dropdown jenis cuti berdasarkan PNS/PPPK pada form. Pertahankan validasi server sebagai sumber kebenaran.
- ❌ Tolak semua pengajuan yang melintasi tahun kalender, bukan hanya Cuti Tahunan.
- ❌ Perbaiki integrasi form kalkulasi hari kerja agar membaca `result.data.jumlah_hari_kerja` dan menampilkan `result.data.warnings`; tambahkan test browser/feature kontrak respons.
- ❌ Lengkapi tampilan status awal dengan label `Menunggu [nama step pertama]` yang berasal dari snapshot chain.

### Prioritas P1 — pulihkan verifikasi dan delivery gate

- ❌ Jalankan project dengan PHP `>= 8.4.1` atau image/container yang sesuai `composer.lock`.
- ❌ Pulihkan Podman, jalankan PostgreSQL 17, migration, seeder, test fokus Sprint 1–5, `composer qa`, dan `php artisan route:list --path=api/v1 --json`.
- ❌ Sediakan dan jalankan suite PostgreSQL untuk UUID, JSON, FK, index, transaction, pagination, dan route binding. SQLite in-memory bukan bukti tunggal yang cukup.
- ❌ Kumpulkan evidence PR review/merge dan QA/retest, lalu perbarui tracker dari `Not Started` hanya setelah Definition of Done terpenuhi.
- ❌ Samakan `SESSION_LIFETIME` di `.env.example` dengan 30 menit atau dokumentasikan alasan penggunaan lifetime Laravel 120 menit bersama idle timeout custom 30 menit.

## Konflik Dokumen dan Keputusan yang Diperlukan

| Topik | Dokumen/implementasi | Dampak | Keputusan yang diperlukan |
|---|---|---|---|
| Nama dan bentuk tabel cuti | PRD §15.2 memakai `leave_approval_steps`, `leave_balance_adjustments`, `leave_documents`; source memakai `leave_request_steps`, `leave_balance_ledger`, `leave_proofs`. | Migration, model, API, audit, dan laporan dapat drift. | Tetapkan schema canonical dan strategi migrasi tanpa merusak data/foreign key. |
| Kepala Bagian | PRD/US-4.11 menugaskan Admin Kepegawaian; source route hanya Super Admin dan start date otomatis hari ini. | Proses operasional tidak sesuai role dan histori tidak lengkap. | Sahkan role yang berwenang serta field effective date. |
| Chain per unit | PRD mengizinkan chain per pegawai atau unit; source hanya per pegawai + global legacy. | Unit kerja tidak dapat menjadi scope konfigurasi. | Tetapkan model/unit hierarchy dan precedence global–unit–pegawai sebelum implementasi. |
| Status keputusan cuti | PRD/AGENTS melarang `REJECT`; source memakainya secara internal dan pada route/action. | Kontrak domain dan audit tidak konsisten. | Gunakan satu vocabulary resmi untuk UI, audit, kode, dan test. |
| Flag kinerja EWS | PRD §10.3 menyatakan pegawai dengan flag Tidak tidak muncul pada EWS kenaikan pangkat; Issue #36 menyatakan Tidak Eligible; source membuat alert dan menampilkannya sebagai Tidak Eligible. | Perilaku dashboard, notifikasi, dan pekerjaan admin berbeda menurut dokumen. | Karena PRD adalah sumber utama, selaraskan source agar alert tidak tampil atau rekam keputusan perubahan PRD sebelum mempertahankan perilaku sekarang. |

## Verifikasi yang Gagal Dijalankan

| Perintah/cek | Hasil | Penyebab |
|---|---|---|
| `php artisan route:list --path=api/v1 --json` | Gagal sebelum Laravel bootstrap | Composer platform check menuntut PHP `>= 8.4.1`; host memakai PHP `8.3.30`. |
| Laravel focused test Sprint 1–5 dan `composer qa` | Tidak dijalankan | Penyebab PHP yang sama. |
| Podman compose/PostgreSQL integration test | Tidak dapat dijalankan | Binary `podman` tidak ditemukan pada workstation. |
| PostgreSQL-sensitive test | Tidak dapat diverifikasi | Runtime container tidak tersedia dan `phpunit.xml` memakai SQLite in-memory. |

## Perbandingan Status Antar Versi Analisis

| Area | Status 14 Juli (`50b1e77`) | Status 17 Juli (`4ea27e8`) | Perubahan |
|---|:---:|:---:|---|
| Issue #4 Mapping User | ✅ (partial) | ✅ | FormRequest, `employee_id`, uniqueness, audit, test lengkap (commit #109) |
| Dokumen eksternal Kepala Lembaga | ❌ | ⚠️ (Issue #31 naik) | Model/controller/action/test tersedia; REJECT masih ada |
| Tombol backup hard delete di UI | ❌ | ✅ dihapus | commit `690d4d3`; backend tetap ada |
| Autocomplete pegawai cuti | — | ✅ baru | commit #110 |
| Email `cuti.perlu_perubahan`/`cuti.tidak_disetujui` | ❌ absen | ⚠️ ditambahkan | Kini ada di `emailEnabled()`, tapi masih hardcoded |
| Force delete pegawai backend | ❌ | ✅ | Mekanisme hard delete, action, command, scheduler harian dihapus; halaman Data Backup kembali aman tanpa purge otomatis |
| REJECT nomenclature | ❌ | ❌ | Belum diperbaiki |
| Double EWS scheduler | ❌ | ❌ | Belum diperbaiki |
| SESSION_LIFETIME `.env.example` | ❌ | ❌ | Belum diperbaiki |
| Profil self-service routes | ❌ | ❌ | Belum diperbaiki |
| Notification dispatcher hardcoded | ❌ | ❌ | Belum diperbaiki (`emailEnabled` masih array) |

## Batasan Analisis

- Analisis menilai source yang ada pada commit `4ea27e8`; tidak menggantikan QA manual, UAT, atau sign-off tim.
- Credential Keycloak, SMTP/email operasional, dan deployment Podman produksi tidak diuji.
- Ikon ✅ hanya menyatakan bukti implementasi source/test tersedia. Tracker tetap tidak dapat menyatakan `Done` tanpa review, QA/retest, dan evidence sesuai dokumen proses.
