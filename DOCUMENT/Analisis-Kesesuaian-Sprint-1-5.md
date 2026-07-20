# Analisis Kesesuaian Sprint 1 sampai Sprint 6 — SIMPEG

| Field | Nilai |
|---|---|
| Tanggal analisis | 21 Juli 2026 |
| Ruang lingkup | Sprint 1 — Fondasi; Sprint 2 — Data Pegawai Core; Sprint 3 — Import & Pelengkap; Sprint 4 — Cuti Core; Sprint 5 — EWS & Notifikasi; Sprint 6 — Dashboard & Laporan |
| Kondisi repository | Baseline `development` commit `32ada6b` (20 Juli 2026), ditambah implementasi pada branch `fix/reference-tables-and-testing` melalui PR [#118](https://github.com/LLDIKTI-XVI-TEAM/SIMPEG/pull/118), commit `1ae6106` dan `7612c8e`; status PR `OPEN`, merge state `CLEAN`, dan CI `Pint + PHPStan + Test` lulus. |
| Metode | Inspeksi source, migration, seeder, route, FormRequest, Action/Service, view Blade, konfigurasi scheduler, dan source test |
| Sumber acuan | `AGENTS.md`, PRD v1.2, panduan kode, user stories, issues, tracker vertical slice, dan pembagian tugas |
| Perubahan dalam analisis | Laporan diperbarui untuk mencatat hasil implementasi dan verifikasi PR #118. Tidak ada perubahan keputusan produk pada PRD. |
| Analisis sebelumnya | Commit `32ada6b` — 20 Juli 2026 |
| Commit baru sejak analisis sebelumnya | PR #118 menambahkan dua commit source (`1ae6106` dan `7612c8e`): reference table Fase 1 dilengkapi, pemilihan kanal notifikasi memakai konfigurasi, fondasi Dusk/test ditambahkan, dan fixture export diselaraskan dengan hierarchy unit kanonis. CI PR lulus setelah perbaikan format. |

## Legend Status

| Icon | Status | Arti |
|---|---|---|
| ✅ | Selesai pada source | Implementasi dan test/source relevan tersedia. Belum setara status tracker `Done`, yang tetap memerlukan review, QA/retest, dan evidence delivery. |
| ⚠️ | Sebagian | Fondasi tersedia, tetapi masih ada acceptance criteria, kesesuaian PRD, arsitektur, atau evidence QA yang belum terpenuhi. |
| ❌ | Belum selesai / tidak sesuai | Implementasi belum tersedia atau ada pelanggaran terhadap PRD, `AGENTS.md`, atau panduan kode. |

## Kesimpulan Eksekutif

Dari 38 issue Sprint 1–5, penghitungan ulang tabel status menghasilkan **20 berstatus ✅** pada level source, **18 ⚠️** sebagian, dan **0 ❌**. Dua temuan P0—riwayat disiplin append-only dan profil mandiri read-only—telah ditutup pada commit `32ada6b`; Issue #11 dan #12 ditutup pada source di PR #118 yang telah lulus CI. Status implementasi ini belum mengubah tracker delivery menjadi `Done` sebelum PR digabung, review, dan QA/retest terdokumentasi.

Untuk Sprint 6 terdapat **6 issue (#39–#43 dan #46)**. Pada level issue, **5 issue berstatus ⚠️ sebagian** dan **1 issue berstatus ❌ belum selesai**; belum ada issue Sprint 6 yang layak dinyatakan ✅ secara utuh. Setelah semua bullet issue dipecah—termasuk sembilan reference table pada Issue #46—terdapat **61 task audit**: **15 selesai pada source, 22 sebagian, dan 24 belum selesai/tidak sesuai**. Foundation hierarchy unit kerja dan notification channel pada Issue #46 tersedia di PR #118, tetapi CRUD Admin tetap belum ada. Tracker delivery masih menandai seluruh stage Sprint 6 sebagai `Not Started`; status source di bagian Sprint 6 tidak boleh dipakai untuk mengubah tracker menjadi `Done` sebelum review, QA/retest, dan evidence terpenuhi.

**Diperbaiki / ditutup sejak analisis sebelumnya (commit `32ada6b` dan PR #118):**

1. **Issue #19 — riwayat hukuman disiplin menjadi append-only:** `DeleteDisciplineRecordAction`, method `destroy`, route API DELETE, permission `discipline_records.delete`, dan tombol hapus terkait dihilangkan. `DeleteDocumentAction` kini menolak penghapusan dokumen yang dipakai riwayat disiplin, termasuk bila file fisiknya terhubung. Penegakan berada di server/transaction, bukan hanya Blade. `DisciplineRecordTest`, `EmployeeDocumentTest`, dan `RbacPermissionMiddlewareTest` mengunci route lama tidak tersedia, record tetap ada, permission tidak disemai, serta dokumen/SK disiplin tidak dapat dihapus.
2. **Issue #23 — profil mandiri kembali read-only:** hanya route GET `profil-saya/keluarga` dan `profil-saya/pendidikan` yang tersisa; controller self-service hanya membaca `employee_id` dari sesi. Endpoint POST/PUT/PATCH/DELETE, FormRequest self-service, serta tombol/modal tambah keluarga dihapus. RBAC role `pegawai` kini hanya memiliki permission baca keluarga dan riwayat. `MyFamilyTest`, `MyEducationHistoryTest`, dan `ProfileTest` mengunci data scope pegawai, ketiadaan route mutasi, serta tampilan tanpa tombol tambah/edit; `EducationHistoryTest` menjaga CRUD Admin Kepegawaian tetap tersedia.
3. **Issue #11 — migration/seeder reference tables:** migration baru melengkapi hierarchy `ref_unit_kerja` (`parent_id`, `level`, `jenis_unit`, `is_active`), lifecycle jabatan (`default_bup`, `is_active`), katalog dan kode status pegawai, serta `ref_notification_channels`. Seeder kini memuat hierarchy kanonis PRD, sepuluh status pegawai minimum, kategori Jabatan Akademik / Dosen, dan tiga kanal notifikasi tanpa menimpa pilihan enable/disable operator. `ReferenceSeederTest` dan test PostgreSQL memverifikasi hasilnya.
4. **Issue #12 — testing framework:** Dusk dan smoke test browser `/up`, helper TestCase, test autentikasi Keycloak/unregistered identity, konfigurasi `phpunit.dusk.xml`, serta dokumentasi perintah test ditambahkan. Verifikasi mencakup 59 test fokus SQLite, 71 test fokus PostgreSQL 17, Dusk, dan CI PR `Pint + PHPStan + Test` yang lulus. Test constraint ledger yang di-skip pada SQLite sengaja dijalankan penuh pada PostgreSQL 17.

**Diperbaiki / ditutup sejak analisis 17 Juli (commit `4ea27e8` sampai `869b842`):**

1. **Export nominatif custom Pimpinan (US-9.1B) — tersedia baru**: `PimpinanCustomEmployeeExportAction` tersedia dengan `ALLOWED_COLUMNS` whitelist (9 kolom), urutan kolom sesuai pilihan pengguna, output streaming XLSX, proteksi formula injection via `TYPE_STRING`, dan styling tabel profesional. `PimpinanEmployeeController::reportCustom()`, `CustomEmployeeExportRequest`, route `pimpinan.laporan.pegawai.custom`, dan view kolom-selector tersedia.
2. **Status kelengkapan dokumen pegawai ditingkatkan ke 4-nilai**: `ListEmployeesAction` kini mengembalikan `'kosong'`, `'tersedia'`, `'tidak_lengkap'`, atau `'lengkap'` (menggantikan boolean). `ShowEmployeeDocumentStatusAction` diperluas menggabungkan arsip `Document` dan riwayat SK ke dalam satu daftar terpadu dengan format seragam.
3. **`EmployeeIndexTest` diperluas secara komprehensif**: 8 test baru mencakup keempat kondisi kelengkapan berkas, endpoint `/api/v1/pegawai/{id}/status-dokumen`, verifikasi storage file fisik, dan skenario `tersedia` (berkas lain ada tanpa riwayat SK).
4. **GlobalSearch mendukung role Pimpinan**: `GlobalSearchController` mendeteksi role `pimpinan` dan mengarahkan URL hasil pencarian pegawai/cuti ke route pimpinan. Pencarian Unit Kerja dan Dokumen dibatasi hanya untuk non-Pimpinan. `GlobalSearchAuthorizationTest` ditambahkan.
5. **`Gate::before` di `AppServiceProvider`**: Mendaftarkan gate listener agar directive `@can()` Blade membaca `hasPermission()` dari model User secara konsisten — memperkuat pagar RBAC sisi view di seluruh aplikasi.
6. **Konsolidasi halaman pegawai Pimpinan**: `pimpinan/pegawai/index.blade.php` dihapus; Pimpinan kini menggunakan `admin.pegawai.index` melalui `PimpinanEmployeeController::index()`, mengurangi duplikasi kode.
7. **Daftar cuti Pimpinan diperluas**: `ListPimpinanLeavesAction` mengembalikan counter statistik (menunggu tindakan saya, total menunggu, disetujui, ditangguhkan). `pimpinan/cuti/index.blade.php` diperluas dengan filter jenis cuti, unit kerja, dan status.
8. **Dashboard Pimpinan diperluas signifikan**: `pimpinan/dashboard.blade.php` mendapat penambahan 744 baris widget data real-time.

**Diperbaiki / ditutup sejak analisis 14 Juli (commit `50b1e77`):**

1. **Issue #4 (Mapping User) — naik ke ✅**: `UpdateUserMappingRequest`, simpan `employee_id`, uniqueness `keycloak_id`, audit via `AuditService`, dan `UserMappingControllerTest` tersedia (commit #109).
2. **Dokumen eksternal Kepala Lembaga — ditutup sebagian (Issue #31)**: `KepalaLembagaSupportingDocument` dengan model, migration, controller, 4 action, FormRequest, test, dan route tersedia.
3. **Hard Delete Pegawai — Selesai**: Seluruh mekanisme force delete (action, command, scheduler harian pukul 02:00) dihapus sepenuhnya. Halaman Data Backup dikembalikan sebagai filter permanen tanpa purge otomatis.
4. **Autocomplete pegawai pada form cuti (#110)**: Komponen `employee-combobox`, `CutiEmployeeLookupController`, `LookupEmployeesAction`, test, dan route API tersedia.
5. **Email label lengkap**: `cuti.perlu_perubahan` dan `cuti.tidak_disetujui` kini terdaftar dalam `NotificationRecipientResolver::emailEnabled()`.

**Masalah lama yang BELUM diperbaiki:**

1. Import belum memiliki template lanjutan, manual column mapping, snapshot riwayat, dan kalkulasi TMT pasca-import.
2. Skema cuti berbeda dengan PRD canonical (`leave_request_steps` vs `leave_approval_steps`, dll). Keputusan belum didokumentasikan.
3. `RejectLeaveAction`, `LeaveApprovalService::reject()`, internal status `'rejected'`/`'request_rejected'`, label `'Ditolak'` di `LeaveProofService`, dan route `cuti.reject` masih aktif. Melanggar nomenklatur resmi Fase 1.
4. `app:run-ews` terdaftar ganda di `bootstrap/app.php` (baris 22–24) DAN `routes/console.php` (baris 27–29).
5. Kelayakan event pada `NotificationRecipientResolver::emailEnabled()` masih hardcoded meskipun enable/disable kanal sudah membaca `ref_notification_channels`; `ews.satyalancana` absen.
6. `SESSION_LIFETIME=120` di `.env.example` tidak konsisten dengan middleware idle 30 menit.

Verifikasi Laravel kini dapat dijalankan pada host ini. Podman dan PostgreSQL 17 juga telah dipakai untuk test fokus; versi PHP lokal 8.3 masih berbeda dari PHP 8.4 pada CI.

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
| **[BARU] Riwayat disiplin append-only** | Route DELETE/action/permission/tombol hapus dihilangkan; `DeleteDocumentAction` menolak dokumen yang masih dipakai riwayat disiplin; `DisciplineRecordTest`, `EmployeeDocumentTest`, dan `RbacPermissionMiddlewareTest` tersedia. |
| **[BARU] Profil mandiri read-only** | Route keluarga dan pendidikan hanya GET; controller membaca scope dari sesi; FormRequest serta UI mutasi dihapus. `MyFamilyTest`, `MyEducationHistoryTest`, `ProfileTest`, dan `EducationHistoryTest` tersedia. |
| PostgreSQL dan Podman | Container PostgreSQL 17 sementara berhasil digunakan untuk 71 test fokus (246 assertion), termasuk `CutiFoundationSchemaTest` tanpa skip. Container dihentikan kembali setelah verifikasi. |
| Verifikasi 20 Juli 2026 | `php artisan route:list --path=api/v1 --json` berhasil; hanya GET/POST disiplin dan GET profil mandiri yang terdaftar. Tujuh test fokus perubahan `32ada6b` lulus: 64 test, 211 assertion. |
| Test dan CI PR #118 | 59 test fokus SQLite (215 assertion), 71 test fokus PostgreSQL 17 (246 assertion), browser smoke test Dusk, dan suite penuh 848 lulus/1 skip (4.047 assertion) tersedia sebagai evidence. Workflow GitHub `Pint + PHPStan + Test` lulus pada PostgreSQL 17; skip constraint ledger hanya berlaku pada SQLite lokal. |

## Status Issue Sprint 1

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #1 | Setup Laravel dan environment | ⚠️ | Laravel 12, PostgreSQL 17, queue, compose, README, Pint, PHPStan, Vite, dan helper Podman tersedia. Test Laravel dan PostgreSQL 17 telah diverifikasi; perbedaan versi PHP lokal 8.3 terhadap CI PHP 8.4 serta evidence deployment Podman production masih perlu ditutup. |
| #2 | Keycloak SSO dan middleware | ⚠️ | Controller, Action callback/logout/redirect, middleware, config Socialite, dan test tersedia. Login IdP nyata belum diuji; bootstrap first mapped employee masih membutuhkan keputusan terdokumentasi. |
| #3 | Logout dan session management | ⚠️ | Logout POST, invalidasi session, dan audit tersedia. Route GET `/logout` juga memutasi session tanpa CSRF. |
| #4 | Mapping user Keycloak dan RBAC | ✅ | **[DIPERBAIKI commit #109]** `UpdateUserMappingRequest` terpisah, `employee_id` disimpan otomatis saat mapping berdasarkan pencocokan email pegawai, validasi uniqueness `keycloak_id`, audit log ditulis ke `audit_logs` via `AuditService`, dan `UserMappingControllerTest` tersedia (guest/role/update/employee_id). |
| #5 | Audit log | ✅ | Migration `audit_logs`, model immutable, `AuditService`, audit auth dan mutasi tersedia. |
| #6 | Notifikasi in-app backend | ✅ | Migration/model, service, action inbox/read/unread, controller API, dan test tersedia. |
| #7 | Bell icon notifikasi | ✅ | Komponen notification bell, endpoint, dan JavaScript aplikasi tersedia. |
| #8 | Design system dan layout master | ✅ | Layout, komponen UI/form reusable, Tailwind/Vite, dan UI Bahasa Indonesia tersedia. |
| #9 | Reusable Blade components | ⚠️ | Komponen utama tersedia, tetapi `resources/views/components/README.md` atau dokumentasi setara belum ada. |
| #10 | CRUD hari libur dan cuti bersama | ✅ | Model, API v1, FormRequest, Action, controller, view, audit, seeder, dan test tersedia. |
| #11 | Migration/seeder reference tables | ✅ | PR #118 melengkapi `ref_notification_channels`; hierarchy unit kerja (`parent_id`, `level`, `jenis_unit`, `is_active`); `default_bup` dan `is_active` pada jabatan; serta kode/kelompok dan katalog sepuluh status pegawai. Seeder hierarchy dan idempotensi kanal diuji. |
| #12 | Setup testing framework | ✅ | PHPUnit tetap tersedia dan kini dilengkapi Laravel Dusk, `DuskTestCase`, smoke test browser, helper test, test auth, `phpunit.dusk.xml`, dan panduan README. Bukti PostgreSQL 17 tersedia dari test fokus serta CI PR yang menjalankan `composer test` dengan driver `pgsql`. |

## Status Issue Sprint 2

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #13 | Migration tabel utama pegawai | ✅ | Model/relasi, UUID, index, cast enkripsi, dan migration sinkronisasi PRD tersedia. |
| #14 | Form tambah pegawai multi-tab | ✅ | FormRequest, Action, view multi-tab, validasi upload, storage, audit, dan test tersedia. |
| #15 | Form edit pegawai | ✅ | FormRequest, Action, view, penggantian berkas/foto, audit, dan test tersedia. |
| #16 | Halaman daftar pegawai | ✅ | Search, filter, sort, pagination, eager loading, default aktif, RBAC, dan test tersedia. |
| #17 | Halaman detail pegawai bertab | ✅ | Data relasi, tab, informasi EWS, view, dan test detail tersedia. |
| #18 | Riwayat pangkat, jabatan, KGB append-only | ✅ | `EmployeeHistoryService`, transaksi, `is_latest`, audit, pembaruan tanggal terhitung, dan test tersedia. |
| #19 | Riwayat hukuman disiplin | ✅ | **[DIPERBAIKI `32ada6b`]** Pembuatan, status aktif, scheduler, upload SK, audit, dan list tersedia. Jalur DELETE/action/permission/tombol hapus telah dihapus. `DeleteDocumentAction` memblokir penghapusan dokumen yang terhubung ke riwayat disiplin pada server-side transaction sehingga SK/file fisik tidak dapat menjadi bypass append-only. Test mengunci route lama tidak ada/URI lama 404 atau 405 dengan record tetap tersimpan. |

## Status Issue Sprint 3

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #20 | Download template import | ⚠️ | Tombol dan download CSV UTF-8/XLSX tersedia melalui `GenerateImportTemplateAction` dan `ImportTemplateWriter`. Hanya template `utama` tersedia, hanya satu baris contoh, dan template Data Pelengkap/Riwayat Kepangkatan/Riwayat Jabatan/Riwayat KGB belum ada. |
| #21 | Upload, preview, validasi Excel/CSV | ⚠️ | Upload maksimal 10 MB, parser XLS/XLSX/CSV, preview, validasi per baris, duplicate NIP skip, error detail, wizard, dan test tersedia. Auto-match hanya menerima header canonical; dropdown/manual column mapping dan peringatan mapping kolom tak cocok belum ditemukan. |
| #22 | Eksekusi import queue dan laporan | ⚠️ | `ImportEmployeeBatchJob` memproses batch valid, progress polling, hasil inserted/skipped/failed, audit, notifikasi, dan unduh CSV error dari UI tersedia. Belum ada snapshot riwayat pangkat/jabatan/KGB, kalkulasi TMT setelah import, atau perhitungan pensiun ketika kolom kosong. |
| #23 | Profil sendiri read-only | ✅ | **[DIPERBAIKI `32ada6b`]** Profil memakai data pegawai login, saldo, tanggal pangkat/KGB, serta endpoint profile sendiri. Keluarga dan pendidikan mandiri kini hanya dapat dibaca melalui route GET dengan `employee_id` dari sesi; endpoint mutasi, FormRequest, dan tombol/modal tambah dihapus. Test mengunci data scope, route mutasi tidak tersedia, dan UI read-only, sementara CRUD pendidikan Admin Kepegawaian tetap diuji terpisah. |
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
| #33 | Kalkulasi TMT otomatis | ⚠️ | EmployeeHistoryService menghitung kenaikan pangkat dari TMT pangkat +4 tahun, KGB +2 tahun, serta pensiun dari tanggal lahir dengan prioritas `default_bup` jabatan lalu BUP jenis jabatan ketika riwayat terbaru dibuat. Namun tidak ada TmtCalculatorService/observer/listener, milestone Satyalancana tidak disimpan, tidak ada rekalkulasi saat import, dan perubahan referensi BUP tidak memicu hitung ulang. |
| #34 | EWS scheduler harian | ⚠️ | `EwsEngineService` memeriksa pegawai aktif untuk lima trigger, threshold configurable, unique alert, log run, error notification, command, dan test tersedia. **Double registrasi masih ada**: `app:run-ews` terdaftar di `bootstrap/app.php` baris 22–24 DAN `routes/console.php` baris 27–29, berisiko dieksekusi dua kali per hari. Alert tidak menyimpan `is_eligible`/`trigger_days`/`tahun` seperti task issue; in-app notification hanya dibuat untuk pegawai, bukan Admin Kepegawaian. |
| #35 | Halaman daftar EWS aktif | ✅ | Controller/action/view menampilkan nama, NIP, event, target, sisa hari, eligibility, dan follow-up; urutan sisa hari, warna urgensi, filter event/status, detail pegawai, akses Admin/Super Admin/Pimpinan, aksi ditangani/tidak perlu ber-catatan, audit, dan test role tersedia. |
| #36 | Flag kinerja baik dan kelayakan Satyalancana | ⚠️ | Toggle AJAX, endpoint JSON, FormRequest, Action, RBAC route, audit, flag/catatan Satyalancana, dan test tersedia. Deskripsi UI bukan tooltip dengan teks yang diminta. Saat flag kinerja false, source tetap membuat dan menampilkan alert pangkat sebagai Tidak Eligible, bertentangan dengan PRD yang menyatakan pegawai tidak muncul pada EWS kenaikan pangkat. |
| #37 | Notifikasi email | ⚠️ | `NotificationService`, job queue, tiga retry, log kegagalan final, template responsive Bahasa Indonesia, CTA aman, Mailpit development, dan test tersedia. Kanal `email` dan `in_app` kini dibaca dari `ref_notification_channels`; test mengunci perilaku saat kanal dinonaktifkan. Namun kelayakan event email masih hardcoded via array di `NotificationRecipientResolver::emailEnabled()`, `ews.satyalancana` masih absen dari daftar email, dan Admin EWS hanya menerima email tambahan, bukan notifikasi in-app. |
| #38 | Session timeout | ⚠️ | Middleware menegakkan idle timeout 30 menit, logout/invalidate session, flash, redirect route login Keycloak, JSON 401, audit SESSION_TIMEOUT, dan test polling tersedia. Tetapi .env.example dan default Laravel masih menetapkan SESSION_LIFETIME=120, bukan 30 sebagaimana Issue #38. |

## Status Task Sprint 6 — Dashboard & Laporan

Sprint 6 dijadwalkan pada **31 Juli 2026 sampai 9 Agustus 2026**. Sumber daftar task adalah `Issues-SIMPEG-Fase1.md` (#39–#43 dan #46). Tracker `Tracking-Sprint-Vertical-Slice-SIMPEG.md` masih menandai seluruh stage Slice 6.1, 6.2, dan 6.3 sebagai `Not Started`. Tabel berikut menilai **source yang sudah ada**, sehingga statusnya dapat berbeda dari tracker formal.

| Issue | Deliverable | Status issue | Ringkasan implementasi source |
|---:|---|:---:|---|
| #39 | Dashboard Admin & Pimpinan (7 widget) | ⚠️ | Dashboard Pimpinan memakai data database dan mencakup tujuh widget. Dashboard Admin hanya memakai data nyata untuk EWS; KPI pegawai/pangkat/cuti, daftar pangkat/cuti, distribusi golongan, tren, dan audit masih hardcoded/dummy. |
| #40 | Dashboard Pegawai pribadi | ⚠️ | EWS pribadi real dan terscope sudah tersedia. Profil ringkas lengkap, saldo cuti, daftar cuti aktif, serta lima notifikasi terbaru belum tersedia pada dashboard. |
| #41 | Dashboard Kepala Bagian | ⚠️ | Daftar bawahan, EWS bawahan, dan detail read-only terscope tersedia. Daftar cuti pending tersedia, tetapi quick action keputusan tidak dirender langsung di dashboard; tindakan dilakukan setelah membuka detail. |
| #42 | Export daftar pegawai Excel + PDF | ⚠️ | Export Excel standar dan custom tersedia melalui PhpSpreadsheet. Export custom khusus Pimpinan mengikuti urutan input, tetapi export custom umum/Admin memaksa urutan canonical. Seluruh export PDF daftar pegawai belum tersedia. |
| #43 | Export rekap cuti Excel + PDF | ⚠️ | Excel dua sheet dan PDF tersedia dengan filter. Nama file Excel tidak sesuai requirement; sheet kedua adalah saldo cuti, bukan ringkasan cuti per pegawai; tabel PDF berisi detail pengajuan, bukan rekap per pegawai; footer belum memberi nomor halaman. |
| #46 | Kelola reference tables | ❌ | Foundation hierarchy `ref_unit_kerja` dan `ref_notification_channels` kini tersedia pada PR #118, tetapi belum ada CRUD admin, FormRequest/Action, route/view, delete protection, soft delete, audit mutation, atau test CRUD. |

### Rekap Jumlah Task Sprint 6

Jumlah berikut menghitung setiap bullet task pada issue. Daftar sembilan reference table pada Issue #46 dipecah menjadi sembilan task agar tidak menyembunyikan pekerjaan yang belum ada.

| Issue | ✅ Selesai source | ⚠️ Sebagian | ❌ Belum/tidak sesuai | Total task audit |
|---:|---:|---:|---:|---:|
| #39 | 0 | 16 | 0 | 16 |
| #40 | 1 | 1 | 4 | 6 |
| #41 | 3 | 1 | 0 | 4 |
| #42 | 7 | 1 | 6 | 14 |
| #43 | 4 | 1 | 3 | 8 |
| #46 | 0 | 2 | 11 | 13 |
| **Total** | **15** | **22** | **24** | **61** |

### Issue #39 — Dashboard Admin & Pimpinan (7 Widget)

Status semua baris di issue ini ⚠️ karena implementasi Pimpinan sudah memakai data real, sedangkan permukaan Admin untuk widget yang sama masih dummy atau hardcoded. `BuildPimpinanDashboardAction`, `PimpinanDashboardController`, `pimpinan/dashboard.blade.php`, dan `PimpinanDashboardDataTest` adalah evidence sisi Pimpinan. `DashboardController@index` hanya memasok EWS ke `admin/dashboard.blade.php`; angka `228`, data `Ahmad Fauzi`, `$cutiList`, `$distribusiGolongan`, SVG tren statis, dan `$auditLog` di view Admin adalah evidence gap integrasi.

| No | Task issue | Status | Sudah tersedia | Masih harus dikerjakan |
|---:|---|:---:|---|---|
| 39.1 | Buat `DashboardController@admin` untuk menghitung semua data widget | ⚠️ | Pimpinan memiliki controller tipis + `BuildPimpinanDashboardAction`. | Buat Action/query dashboard Admin; jangan menaruh agregasi domain di route, controller, atau Blade. |
| 39.2 | W1 backend — total pegawai aktif dan komposisi PNS/PPPK | ⚠️ | Query real tersedia untuk Pimpinan. | Hubungkan Dashboard Admin ke query real; angka Admin masih hardcoded. |
| 39.3 | W2 backend — kenaikan pangkat bulan/tahun ini | ⚠️ | Query real dan lima baris pangkat tersedia untuk Pimpinan. | Hubungkan Dashboard Admin; KPI dan daftar Admin masih dummy. |
| 39.4 | W3 backend — cuti pending, disetujui bulan ini, ditunda | ⚠️ | Query real tersedia untuk Pimpinan. | Hubungkan Dashboard Admin; KPI dan `$cutiList` Admin masih dummy serta tombolnya hanya memanipulasi HTML lokal. |
| 39.5 | W4 backend — top 5 EWS paling urgent | ⚠️ | Pimpinan dan Admin menerima maksimum lima alert dari `ListActiveEwsAlertsAction`. | Tambahkan test akurasi dan urutan urgensi khusus Dashboard Admin; QA runtime belum tersedia. |
| 39.6 | W5 backend — distribusi golongan | ⚠️ | Agregasi real tersedia untuk Pimpinan. | Hubungkan Dashboard Admin; distribusi Admin masih array hardcoded. |
| 39.7 | W6 backend — lima audit log terbaru | ⚠️ | Query `AuditLog::latest()->limit(5)` tersedia untuk Pimpinan. | Hubungkan Dashboard Admin; `$auditLog` Admin masih hardcoded. |
| 39.8 | W7 backend — tren pegawai 12 bulan terakhir | ⚠️ | Query 12 bulan tersedia untuk Pimpinan. | Hubungkan Dashboard Admin; grafik Admin memakai path SVG dan label statis. |
| 39.9 | Layout dashboard responsive berbasis grid | ⚠️ | View Admin dan Pimpinan memakai kelas grid/responsive Tailwind. | Verifikasi browser desktop/tablet/mobile dan hilangkan dummy data sebelum flow dianggap selesai. |
| 39.10 | W1 frontend — KPI card + pie chart | ⚠️ | Komponen UI tersedia pada Admin dan Pimpinan; Pimpinan memakai data real. | Bind UI Admin ke data real. |
| 39.11 | W2 frontend — KPI card + mini list | ⚠️ | UI real tersedia pada Pimpinan. | Ganti angka dan daftar pegawai dummy di Admin. |
| 39.12 | W3 frontend — KPI card | ⚠️ | UI real tersedia pada Pimpinan. | Ganti angka dan daftar cuti dummy Admin; keputusan harus melalui endpoint terotorisasi, bukan JavaScript lokal. |
| 39.13 | W4 frontend — tabel mini dengan warna indikator | ⚠️ | Tabel, badge urgensi, empty state, dan data real tersedia. | Lakukan QA tampilan/urutan dan test Dashboard Admin. |
| 39.14 | W5 frontend — bar chart horizontal | ⚠️ | Bar chart dinamis tersedia pada Pimpinan. | Ganti array hardcoded Admin dengan agregasi real. |
| 39.15 | W6 frontend — list audit item | ⚠️ | List berbasis data real tersedia pada Pimpinan. | Ganti list dummy Admin dengan lima audit terbaru. |
| 39.16 | W7 frontend — line chart | ⚠️ | Grafik Pimpinan membangun titik dari `$trenPegawai`. | Ganti path/label statis Admin dengan data 12 bulan real dan verifikasi responsif. |

### Issue #40 — Dashboard Pegawai (Pribadi)

`DashboardController@index` hanya membangun data EWS untuk role `pegawai`, lalu `pegawai/dashboard.blade.php` merender welcome banner dan tabel EWS. `DashboardEwsTest` membuktikan alert dibatasi ke pegawai login. Tidak ada builder dashboard pegawai yang mengambil profil, saldo, cuti aktif, atau notifikasi.

| No | Task issue | Status | Bukti dan pekerjaan tersisa |
|---:|---|:---:|---|
| 40.1 | Profil ringkas: foto, nama, NIP, golongan, jabatan, unit kerja | ❌ | Dashboard hanya memakai `auth()->user()->name`; foto dan field kepegawaian lain tidak dirender. |
| 40.2 | Card saldo cuti | ❌ | Tidak ada query/variabel/card saldo pada dashboard pegawai. |
| 40.3 | Daftar pengajuan cuti aktif + status | ❌ | Tidak ada query/daftar cuti pada dashboard pegawai. |
| 40.4 | EWS pribadi | ✅ | Data real, maksimal lima alert, link `ews.saya`, data scope pegawai login, empty state, badge urgensi, dan `DashboardEwsTest` tersedia. |
| 40.5 | Lima notifikasi terbaru | ❌ | Notification bell/halaman notifikasi tersedia secara global, tetapi lima notifikasi tidak diambil atau dirender pada dashboard pegawai. |
| 40.6 | Responsive | ⚠️ | View memakai kelas responsive Tailwind dan tabel overflow, tetapi flow dashboard yang diwajibkan belum lengkap dan belum ada evidence browser QA. |

### Issue #41 — Dashboard Kepala Bagian

Implementasi memakai `KepalaBagianDashboardController`, `BuildKepalaBagianDashboardAction`, `KepalaBagianScopeService`, view `kabag/dashboard.blade.php`, serta `KepalaBagianFrontendTest` dan `KepalaBagianRouteGateTest`. Data dibatasi ke bawahan langsung dan akses role lain ditolak.

| No | Task issue | Status | Bukti dan pekerjaan tersisa |
|---:|---|:---:|---|
| 41.1 | Daftar bawahan langsung | ✅ | Query scope bawahan langsung, jumlah aktif, lima bawahan ringkas, halaman daftar, dan test isolasi bawahan tersedia. |
| 41.2 | Pengajuan cuti pending + quick action dengan label keputusan resmi | ⚠️ | Antrean pending/ditangguhkan dan flow detail keputusan `Disetujui`, `Perubahan`, `Ditangguhkan`, `Tidak Disetujui` tersedia. Dashboard hanya memberi link ke detail; tidak ada quick action keputusan langsung pada baris dashboard. |
| 41.3 | EWS bawahan | ✅ | Hanya EWS bawahan langsung yang diambil/ditampilkan; halaman penuh, link detail, dan test data scope tersedia. |
| 41.4 | Klik nama ke detail ringkas read-only | ✅ | Nama/foto mengarah ke `kepala-bagian.bawahan.show`; pegawai di luar scope menghasilkan 403 dan view tidak menyediakan mutasi data. |

### Issue #42 — Export Daftar Pegawai (Excel + PDF)

Implementasi Excel menggunakan dependency yang sudah terpasang, `phpoffice/phpspreadsheet`, melalui Action dan Service. Ini memenuhi kebutuhan perilaku tanpa menambah `maatwebsite/excel`, selaras dengan aturan memakai Laravel/dependency yang sudah ada. Evidence utama: `ExportPegawaiExcelAction`, `ExportCustomPegawaiExcelAction`, `EmployeeExportDataService`, `EmployeeExportSpreadsheetService`, `ExportPegawaiRequest`, `CustomEmployeeExportRequest`, dan `EmployeeReportExportTest`.

| No | Task issue | Status | Bukti dan pekerjaan tersisa |
|---:|---|:---:|---|
| 42.1 | Buat class export pegawai | ✅ | Dipenuhi secara ekuivalen oleh `ExportPegawaiExcelAction` + service PhpSpreadsheet; tidak memakai nama/package opsional persis dari issue. |
| 42.2 | Kolom tetap: No, NIP, Nama, Golongan, Jabatan, Unit Kerja, Jenis, Status | ✅ | Delapan kolom dan urutannya dikunci oleh `COLUMNS`; test membaca workbook aktual. |
| 42.3 | Export mengikuti filter aktif | ✅ | Filter status, unit, jenis, golongan, jabatan, pencarian, periode pensiun, dan sort diterapkan oleh `EmployeeExportDataService`; preview dan file memakai filter yang sama. |
| 42.4 | Nama file `Daftar_Pegawai_LLDIKTI_XVI_{tanggal}.xlsx` | ✅ | Nama file memakai format `Daftar_Pegawai_LLDIKTI_XVI_YYYYMMDD.xlsx` dan dikunci test. |
| 42.5 | Custom — pilihan kolom dari whitelist aman | ✅ | `CustomEmployeeExportRequest::ALLOWED_COLUMNS` menolak field sensitif; test menolak kolom `nik`. |
| 42.6 | Custom — filter status, unit/tim, jenis, golongan, jabatan, periode pensiun | ✅ | Seluruh filter diteruskan ke `EmployeeExportDataService`. |
| 42.7 | Custom — output Excel saja | ✅ | Endpoint custom menghasilkan streaming XLSX; tidak ada output custom PDF. |
| 42.8 | Custom — urutan kolom mengikuti pilihan user | ⚠️ | `PimpinanCustomEmployeeExportAction` mempertahankan urutan input, tetapi endpoint custom umum/Admin mengurutkan ulang pilihan memakai `COLUMN_ORDER`; test-nya justru mengunci urutan canonical walaupun input pengguna berbeda. Samakan kontrak semua role dengan urutan input tervalidasi. |
| 42.9 | PDF — Blade `exports/employees-pdf.blade.php` | ❌ | File/view PDF daftar pegawai tidak ditemukan. |
| 42.10 | PDF — header nama instansi, judul, tanggal cetak | ❌ | Belum ada template PDF daftar pegawai. |
| 42.11 | PDF — tabel data | ❌ | Belum ada query/action/view PDF daftar pegawai. |
| 42.12 | PDF — footer halaman X dari Y | ❌ | Belum ada template/footer PDF daftar pegawai. |
| 42.13 | PDF — orientasi landscape | ❌ | Belum ada action PDF daftar pegawai yang mengatur paper landscape. |
| 42.14 | PDF — generate via DomPDF | ❌ | DomPDF terpasang untuk fitur lain, tetapi tidak ada route/action export PDF pegawai. |

### Issue #43 — Export Rekap Cuti (Excel + PDF)

Evidence utama: `CutiReportController`, `CutiRekapQuery`, `ExportCutiExcelAction`, `ExportCutiPdfAction`, `admin/cuti/pdf/laporan-cuti.blade.php`, `CutiExcelExportTest`, `CutiPdfExportTest`, dan `CutiRekapExportTest`. Export memiliki role gate, validasi UUID filter, batas data eksplisit, perlindungan formula spreadsheet, serta test source yang cukup rinci; gap di bawah tetap harus ditutup agar sesuai task issue.

| No | Task issue | Status | Bukti dan pekerjaan tersisa |
|---:|---|:---:|---|
| 43.1 | Excel — export dengan filter periode, unit kerja, pegawai | ✅ | Filter canonical diterapkan bersama pada preview, PDF, dan Excel melalui `ListCutiRekapRequest`/`CutiRekapQuery`. |
| 43.2 | Excel — Sheet 1 detail cuti | ✅ | Workbook memiliki sheet `Detail Cuti` dengan NIP, nama, jenis, tanggal, hari, dan label status resmi. |
| 43.3 | Excel — Sheet 2 ringkasan per pegawai | ❌ | Sheet kedua bernama `Saldo Cuti` dan berisi bucket saldo per tahun, bukan agregasi rekap cuti per pegawai sebagaimana task. |
| 43.4 | Excel — nama file `Rekap_Cuti_{periode}_{tanggal}.xlsx` | ❌ | Source menghasilkan `Laporan_Cuti_YYYYMMDD_HHMMSS.xlsx` dan test mengunci nama yang tidak sesuai requirement. |
| 43.5 | PDF — view dengan header institusi | ✅ | Template menampilkan identitas LLDIKTI Wilayah XVI, judul, periode, dan waktu cetak. |
| 43.6 | PDF — tabel rekap per pegawai | ❌ | Template merender satu baris per pengajuan cuti, bukan ringkasan/agregasi per pegawai. |
| 43.7 | PDF — bagian tanda tangan | ✅ | Area `Pembuat Laporan` dan `Mengetahui` tersedia. |
| 43.8 | PDF — footer halaman | ⚠️ | Footer fixed dengan waktu pembuatan tersedia, tetapi tidak ada nomor halaman/total halaman; lengkapi jika yang dimaksud acceptance criteria adalah footer bernomor. |

### Issue #46 — Kelola Reference Tables (CRUD Admin)

Migration/model/seeder referensi bukan bukti CRUD. Pencarian route, controller, FormRequest, Action, view, dan feature test tidak menemukan permukaan kelola reference table. `ReferenceSeederTest` memverifikasi seed dan hierarchy; PR #118 menambahkan hierarchy `ref_unit_kerja` serta migration/model `ref_notification_channels`. Model referensi belum memakai `SoftDeletes`, dan belum ada permukaan CRUD, audit, atau proteksi hapus.

| No | Task issue | Status | Bukti dan pekerjaan tersisa |
|---:|---|:---:|---|
| 46.1 | CRUD `ref_golongan` | ❌ | Tabel/model dipakai fitur pegawai, tetapi CRUD admin belum ada. |
| 46.2 | CRUD `ref_jenis_jabatan` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.3 | CRUD `ref_jabatan` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.4 | CRUD `ref_status_pegawai` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.5 | CRUD `ref_eselon` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.6 | CRUD `ref_unit_kerja` hierarkis | ⚠️ | PR #118 menambahkan field hierarchy, relasi parent/children, seeder hierarchy kanonis, dan test. CRUD Admin, FormRequest/Action, audit, dan policy/validasi pemakaian masih belum ada. |
| 46.7 | CRUD `ref_jenjang_pendidikan` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.8 | CRUD `ref_bup` | ❌ | Tabel/model tersedia; CRUD admin belum ada. |
| 46.9 | CRUD `ref_notification_channels` | ⚠️ | PR #118 menyediakan migration, model, seed, resolver kanal, dan test enable/disable. CRUD Admin, FormRequest/Action, audit, dan akses Super Admin masih belum ada. |
| 46.10 | Validasi tidak bisa hapus item yang sedang dipakai | ❌ | Tidak ada endpoint delete atau policy/reference usage checker. FK `restrictOnDelete` pada sebagian relasi belum menggantikan validasi UX/domain yang diminta. |
| 46.11 | Soft delete jika item sudah dipakai | ❌ | Migration/model referensi tidak memakai `deleted_at`/`SoftDeletes`; requirement ini juga perlu diselaraskan dengan larangan hapus item terpakai agar perilaku tidak ambigu. |
| 46.12 | Audit log mutasi reference table | ❌ | Tidak ada mutation action CRUD sehingga tidak ada audit create/update/delete/restore reference table. |
| 46.13 | Akses Super Admin saja | ❌ | Belum ada route/permukaan CRUD untuk diberi gate `role:super_admin`; tambahkan role gate kasar dan authorization di FormRequest/policy. |

### Urutan Pekerjaan Sprint 6 yang Masih Terbuka

1. **P0 — selesaikan Dashboard Admin real-data (#39):** buat Action agregasi, ganti seluruh angka/list/chart dummy, pertahankan controller tipis, tambahkan test akurasi, data kosong, role, dan regresi query.
2. **P0 — lengkapi Dashboard Pegawai (#40):** profile summary terscope, saldo ledger, cuti aktif/status resmi, lima notifikasi terbaru, lalu test pegawai tanpa linkage dan larangan melihat data orang lain.
3. **P0 — tutup export pegawai (#42):** pertahankan Excel yang ada, ubah custom column ordering mengikuti input tervalidasi, lalu buat PDF pegawai lengkap (header, tabel, footer X/Y, landscape, DomPDF, filter yang sama, dan test).
4. **P0 — selaraskan export cuti (#43):** ubah sheet kedua menjadi ringkasan cuti per pegawai atau dokumentasikan keputusan jika saldo memang dimaksudkan; perbaiki filename; ubah tabel PDF menjadi rekap per pegawai dan tambahkan footer halaman.
5. **P1 — quick action Kepala Bagian (#41):** tampilkan aksi yang aman/terotorisasi pada dashboard atau dokumentasikan bahwa link ke detail adalah definisi quick action yang disepakati. Jangan menduplikasi aturan transisi dari Action cuti yang sudah ada.
6. **P1 — CRUD reference table (#46):** foundation schema/hierarchy `ref_unit_kerja` dan `ref_notification_channels` telah tersedia di PR #118. Putuskan lebih dahulu hubungan delete protection vs soft delete, lalu implementasikan CRUD per vertical slice dengan FormRequest, Action, audit, Super Admin gate, test pemakaian FK, dan restore bila soft delete dipilih.

### Evidence QA Sprint 6 yang Belum Ada

- Seluruh stage Sprint 6 pada tracker masih `Not Started`; tidak ada status PR review, QA/retest, atau `Done` yang dapat diverifikasi dari dokumen delivery.
- Test source tersedia untuk Pimpinan, EWS dashboard pegawai, Kepala Bagian, Excel pegawai, dan export cuti; suite penuh terakhir lulus 848 test dengan 1 skip pada host lokal. Namun evidence browser QA responsif dan verifikasi PostgreSQL khusus agregasi/export Sprint 6 masih belum tersedia.
- Dashboard Admin real-data, dashboard Pegawai lengkap, PDF pegawai, dan CRUD reference table belum memiliki feature test karena implementasinya belum ada.
- Tidak ada evidence browser QA untuk desktop/tablet/mobile dan tidak ada evidence PostgreSQL 17 untuk query agregasi/export Sprint 6.

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
- ✅ Tambah/edit/daftar/detail pegawai memakai data nyata, validasi, upload, audit, dan test tersedia.
- ✅ **[UPDATE]** Status kelengkapan dokumen pegawai 4-nilai: `ListEmployeesAction` mengembalikan `'kosong'`/`'tersedia'`/`'tidak_lengkap'`/`'lengkap'`. `EmployeeIndexTest` diperluas dengan 8 test komprehensif termasuk verifikasi storage file dan endpoint `/status-dokumen`. `ShowEmployeeDocumentStatusAction` menggabungkan arsip `Document` dan riwayat SK menjadi daftar terpadu.
- ✅ Riwayat pangkat, jabatan, dan KGB bersifat append-only dengan `is_latest`, transaksi, dan audit.
- ✅ **[BARU] Issue #19 — riwayat hukuman disiplin append-only:** action, route DELETE, permission, dan kontrol hapus dihilangkan. Dokumen SK yang dipakai riwayat disiplin ditolak oleh `DeleteDocumentAction` pada server-side transaction; test menutup bypass route dan penghapusan dokumen/file.

### Sprint 3

- ✅ **[BARU] Issue #23 — profil sendiri read-only:** keluarga dan pendidikan hanya dibaca dari employee yang terhubung ke sesi; endpoint serta UI mutasi dihapus, dan test mengunci scope maupun ketiadaan route mutasi.
- ✅ CRUD keluarga admin memakai API v1, FormRequest, Action, data scope, soft delete, audit masking NIK, dan test.

### Sprint 4

- ✅ Saldo cuti berbasis ledger, initial balance, koreksi beralasan/audit, rollover tahunan, seed 2026, dan halaman/API saldo tersedia.
- ✅ Dokumen eksternal Kepala Lembaga: `KepalaLembagaSupportingDocument` model/migration/controller/action/FormRequest/test tersedia; Admin dapat mengunggah dokumen persetujuan eksternal untuk cuti Kepala Lembaga (Issue #31 sebagian).
- ✅ **[BARU]** Daftar cuti Pimpinan diperluas: `ListPimpinanLeavesAction` mengembalikan counter statistik (menunggu tindakan saya, total menunggu, disetujui, ditangguhkan), filter jenis cuti dan unit kerja tersedia.

### Sprint 5

- ✅ Halaman daftar EWS aktif: data nyata, sort urgensi, filter event/status, warna urgensi, detail pegawai, akses role, follow-up ber-catatan, audit, dan test tersedia.
- ✅ Perhitungan TMT pangkat/KGB/pensiun saat riwayat terbaru ditambahkan tersedia pada `EmployeeHistoryService`.
- ✅ Engine EWS memiliki lima trigger, threshold configurable, unique index alert, log eksekusi, command, dan test source.
- ✅ Toggle kinerja dan kelayakan Satyalancana menggunakan AJAX, validasi, Action, audit, serta test role/invalid request.
- ✅ Email memakai job queue dengan tiga retry, log kegagalan, template Bahasa Indonesia, dan CTA internal aman.
- ✅ Idle timeout 30 menit memiliki invalidasi sesi, redirect Keycloak, audit, respons JSON, dan test source.

### Laporan & Fitur Pimpinan (tersedia lebih awal dari jadwal)

- ✅ **[BARU]** Export nominatif custom Pimpinan (US-9.1B): `PimpinanCustomEmployeeExportAction` (whitelist 9 kolom, urutan kolom sesuai pilihan pengguna, anti-formula injection, streaming XLSX), route `pimpinan.laporan.pegawai.custom`, `CustomEmployeeExportRequest`, dan view kolom-selector tersedia.
- ✅ **[BARU]** GlobalSearch mendukung role Pimpinan: `GlobalSearchController` mendeteksi role pimpinan dan mengarahkan URL ke route yang sesuai; `GlobalSearchAuthorizationTest` tersedia.
- ✅ **[BARU]** `Gate::before` di `AppServiceProvider`: directive `@can()` Blade kini membaca `hasPermission()` model User secara konsisten di seluruh aplikasi.
- ✅ **[BARU]** Konsolidasi halaman pegawai Pimpinan: `pimpinan/pegawai/index.blade.php` dihapus; Pimpinan menggunakan `admin.pegawai.index` melalui `PimpinanEmployeeController::index()` (tidak ada duplikasi kode).

## Task yang Masih Belum Selesai atau Harus Dikoreksi

### Prioritas P0 — wajib sebelum Sprint 1–5 dinyatakan selesai

- ✅ **[SELESAI]** Hapus endpoint/API force delete, `DeleteEmployeeAction::forceDelete()`, `PurgeDeletedEmployeesAction`, command `employees:purge-deleted`, dan jadwal purge harian pukul 02:00 di `bootstrap/app.php`. Halaman Data Backup dikembalikan sebagai filter nonaktif permanen tanpa purge otomatis.
- ✅ Perbaiki mapping Keycloak: simpan `users.employee_id`, validasi uniqueness `keycloak_id`, audit persisten di `audit_logs`, dan test tersedia — **SELESAI commit #109**.
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

## Verifikasi Runtime dan Batasan yang Masih Ada

| Perintah/cek | Hasil | Penyebab |
|---|---|---|
| `php artisan route:list --path=api/v1 --json` | Berhasil pada 20 Juli 2026 | Output menegaskan API disiplin hanya GET/POST, serta profil keluarga/pendidikan mandiri hanya GET. |
| Test fokus perubahan `32ada6b` | Berhasil: 64 test, 211 assertion | Menjalankan `DisciplineRecordTest`, `EmployeeDocumentTest`, `ProfileTest`, `MyEducationHistoryTest`, `MyFamilyTest`, `EducationHistoryTest`, dan `RbacPermissionMiddlewareTest`. |
| `composer qa` dan test suite PostgreSQL penuh | Belum dijalankan ulang untuk analisis ini | Di luar perubahan dokumentasi ini; focused test SQLite tidak menggantikan evidence PostgreSQL 17. |
| Podman compose/PostgreSQL integration test | Tidak dapat dijalankan | Binary `podman` tidak ditemukan pada workstation. |
| PostgreSQL-sensitive test | Tidak dapat diverifikasi | Runtime container tidak tersedia dan `phpunit.xml` memakai SQLite in-memory. |

## Perbandingan Status Antar Versi Analisis

| Area | Status 14 Juli (`50b1e77`) | Status 17 Juli (`4ea27e8`) | Status 20 Juli (`32ada6b`) | Perubahan terbaru |
|---|:---:|:---:|:---:|---|
| Issue #4 Mapping User | ✅ (partial) | ✅ | ✅ | Tidak berubah |
| Dokumen eksternal Kepala Lembaga | ❌ | ⚠️ (Issue #31 naik) | ⚠️ | Tidak berubah; REJECT masih ada |
| Force delete pegawai backend | ❌ | ✅ | ✅ | Tidak berubah |
| Autocomplete pegawai cuti | — | ✅ baru | ✅ | Tidak berubah |
| Email `cuti.perlu_perubahan`/`cuti.tidak_disetujui` | ❌ absen | ⚠️ ditambahkan | ⚠️ | Tidak berubah; masih hardcoded |
| Kelengkapan dokumen pegawai | boolean `is_lengkap` | boolean `is_lengkap` | ✅ 4-nilai | `ListEmployeesAction` kini 4-nilai; `EmployeeIndexTest` +8 test |
| Export custom Pimpinan (US-9.1B) | ❌ | ❌ | ✅ baru | `PimpinanCustomEmployeeExportAction`, route, view tersedia |
| GlobalSearch Pimpinan | ❌ | ❌ | ✅ baru | Route Pimpinan, `GlobalSearchAuthorizationTest` tersedia |
| Gate::before AppServiceProvider | ❌ | ❌ | ✅ baru | `@can()` Blade konsisten membaca `hasPermission()` |
| Halaman daftar pegawai Pimpinan | duplikat | duplikat | ✅ dikonsolidasi | `pimpinan/pegawai/index.blade.php` dihapus |
| Daftar cuti Pimpinan | ⚠️ dasar | ⚠️ dasar | ⚠️ lebih lengkap | Counter statistik + filter lebih kaya |
| Dashboard Pimpinan | ⚠️ | ⚠️ | ⚠️ lebih lengkap | +744 baris widget; `PimpinanDashboardDataTest` tersedia, tetapi runtime/QA belum dijalankan ulang |
| Riwayat disiplin append-only (Issue #19) | ❌ | ❌ | ✅ | Route/action/permission/tombol DELETE dihapus; dokumen SK disiplin diblokir dari penghapusan; test route lama dan preservasi record/file tersedia |
| Profil self-service read-only (Issue #23) | ❌ | ❌ | ✅ | Route keluarga/pendidikan hanya GET, controller terscope sesi, UI/FormRequest mutasi dihapus; test scope dan ketiadaan route mutasi tersedia |
| REJECT nomenclature | ❌ | ❌ | ❌ | Belum diperbaiki |
| Double EWS scheduler | ❌ | ❌ | ❌ | Belum diperbaiki |
| SESSION_LIFETIME `.env.example` | ❌ | ❌ | ❌ | Belum diperbaiki |
| Profil self-service routes | ❌ | ❌ | ✅ | Tidak ada lagi route mutasi keluarga/pendidikan mandiri |
| Notification dispatcher hardcoded | ❌ | ❌ | ❌ | Belum diperbaiki (`emailEnabled` masih array) |

## Batasan Analisis

- Analisis menilai source yang ada pada commit `32ada6b`; tidak menggantikan QA manual, UAT, atau sign-off tim.
- Credential Keycloak, SMTP/email operasional, dan deployment Podman produksi tidak diuji.
- Ikon ✅ hanya menyatakan bukti implementasi source/test tersedia. Tracker tetap tidak dapat menyatakan `Done` tanpa review, QA/retest, dan evidence sesuai dokumen proses.
