# Analisis Kesesuaian Sprint 1 sampai Sprint 5 — SIMPEG

| Field | Nilai |
|---|---|
| Tanggal analisis | 14 Juli 2026 |
| Ruang lingkup | Sprint 1 — Fondasi; Sprint 2 — Data Pegawai Core; Sprint 3 — Import & Pelengkap; Sprint 4 — Cuti Core; Sprint 5 — EWS & Notifikasi |
| Kondisi repository | `SIMPEG/` pada branch `development`, commit `50b1e77` (`Fix/detail pegawai (#99)`) |
| Metode | Inspeksi source, migration, seeder, route, FormRequest, Action/Service, view Blade, konfigurasi scheduler, dan source test |
| Sumber acuan | `AGENTS.md`, PRD v1.2, panduan kode, user stories, issues, tracker vertical slice, dan pembagian tugas |
| Perubahan dalam analisis | Hanya laporan ini. Source aplikasi tidak diubah. |

## Legend Status

| Icon | Status | Arti |
|---|---|---|
| ✅ | Selesai pada source | Implementasi dan test/source relevan tersedia. Belum setara status tracker `Done`, karena QA/runtime belum dapat dieksekusi ulang. |
| ⚠️ | Sebagian | Fondasi tersedia, tetapi masih ada acceptance criteria, kesesuaian PRD, arsitektur, atau evidence QA yang belum terpenuhi. |
| ❌ | Belum selesai / tidak sesuai | Implementasi belum tersedia atau ada pelanggaran terhadap PRD, `AGENTS.md`, atau panduan kode. |

## Kesimpulan Eksekutif

Dari 38 issue Sprint 1–5, 14 berstatus ✅ pada level source, 20 ⚠️ sebagian, dan 4 ❌ tidak sesuai. Sprint 3 telah memiliki alur import ber-queue, CRUD keluarga, serta soft delete/restore. Sprint 4 telah memiliki kalkulator hari kerja, saldo cuti berbasis ledger, snapshot approval, dokumen QR, dan test yang cukup luas. Sprint 5 sudah memiliki mesin EWS, halaman alert aktif, flag kinerja/Satyalancana, email queue, dan middleware idle timeout, tetapi hanya Issue #35 yang memenuhi seluruh requirement source yang diperiksa.

Namun Sprint 3–5 belum dapat disebut selesai menurut Definition of Done dokumen proyek:

- Tracker masih menandai semua stage Sprint 3 dan Sprint 4 sebagai `Not Started`.
- Tracker Sprint 5 juga belum memiliki evidence review, QA/retest, atau sign-off yang dapat menjadikannya Done.
- Branch aktif adalah `development`, sedangkan tracker menyebut branch integrasi `develop`.
- Tidak ada bukti review/merge gate, QA/retest Grantly, atau sign-off slice.
- Runtime Laravel tidak dapat dijalankan pada workstation ini karena PHP host `8.3.30`, sedangkan `vendor/composer/platform_check.php` mensyaratkan PHP `>= 8.4.1`. Podman juga tidak tersedia.

Temuan paling penting:

1. Jalur force delete dan scheduler purge permanen pegawai tetap aktif. Ini melanggar batas Fase 1 dan membuat Issue #25 tidak sesuai.
2. Halaman/API profil sendiri menyediakan mutasi keluarga, pendidikan, serta perubahan password. US-2.5 mensyaratkan profil pegawai sepenuhnya read-only; self-service editing berada di luar batas Fase 1.
3. Import utama bekerja, tetapi manual column mapping, empat template lanjutan, dua contoh data, snapshot riwayat, dan kalkulasi TMT setelah import belum tersedia.
4. Skema cuti yang dipakai source berbeda dengan schema canonical PRD: `leave_request_steps` alih-alih `leave_approval_steps`, `leave_balance_ledger` alih-alih `leave_balance_adjustments`, dan `leave_proofs` alih-alih `leave_documents`. Karena `AGENTS.md` mengklasifikasikan konflik schema ini sebagai kasus eskalasi, keputusan canonical wajib didokumentasikan sebelum migration/kontrak API diubah.
5. Approval engine tersedia, tetapi masih memakai nama/action `REJECT`, `RejectLeaveAction`, dan route `/reject`. Ini melanggar label keputusan resmi Fase 1. Alur approval eksternal untuk cuti Kepala Lembaga juga belum ditemukan.
6. Konfigurasi approval source saat ini bersifat per pegawai dan global legacy stage 2/3; konfigurasi per unit yang diwajibkan PRD belum terbukti.

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
| PostgreSQL dan Podman | `compose.yml` dan konfigurasi PostgreSQL 17 tersedia; verifikasi runtime tidak bisa dilakukan karena Podman tidak ditemukan. |
| Test | Banyak Feature/Unit test tersedia, tetapi `phpunit.xml` masih memaksa SQLite in-memory. Tidak ada bukti test PostgreSQL atau Laravel Dusk. |

## Status Issue Sprint 1

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #1 | Setup Laravel dan environment | ⚠️ | Laravel 12, PostgreSQL 17, queue, compose, README, Pint, PHPStan, Vite, dan helper Podman tersedia. Runtime terblokir oleh PHP host/PODMAN. |
| #2 | Keycloak SSO dan middleware | ⚠️ | Controller, Action callback/logout/redirect, middleware, config Socialite, dan test tersedia. Login IdP nyata belum diuji; bootstrap first mapped employee masih membutuhkan keputusan terdokumentasi. |
| #3 | Logout dan session management | ⚠️ | Logout POST, invalidasi session, dan audit tersedia. Route GET `/logout` juga memutasi session tanpa CSRF. |
| #4 | Mapping user Keycloak dan RBAC | ✅ | FormRequest UpdateUserMappingRequest terpisah, employee_id disimpan otomatis ke tabel users saat mapping, dan penulisan audit log menggunakan AuditService ke DB. |
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
| #25 | Soft delete dan restore pegawai | ❌ | Soft delete, restore, audit, daftar nonaktif, serta EWS exclusion tersedia. Namun `force-destroy`, `DeleteEmployeeAction`, `PurgeDeletedEmployeesAction`, dan scheduler purge 30 hari tetap menghapus data/berkas permanen. |

## Status Issue Sprint 4

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #26 | Migration tabel cuti | ⚠️ | Tabel/model chain, snapshot step, balance, ledger, dan QR proof tersedia serta diuji. Schema runtime memakai `leave_request_steps`, `leave_balance_ledger`, `leave_proofs`; PRD mensyaratkan `leave_approval_steps`, `leave_balance_adjustments`, `leave_documents`, serta field canonical request. Konflik schema perlu keputusan terdokumentasi. |
| #27 | Assign Kepala Bagian per pegawai | ⚠️ | `kepala_bagian_id`, `SupervisorAssignment`, riwayat perubahan, larangan self-assign, audit, dan test tersedia. Route dibatasi Super Admin, padahal PRD/US menunjuk Admin Kepegawaian; tanggal mulai tidak dapat ditentukan pengguna; tidak ada enforcement bahwa semua pegawai memiliki tepat satu kepala bagian aktif. |
| #28 | Konfigurasi approval chain | ⚠️ | Chain per pegawai, step, PYBMC global, backfill, audit, FormRequest, dan skip duplicate tersedia. Konfigurasi per unit belum ada; form utama masih legacy stage 2/3; UI untuk menyusun chain per pegawai secara penuh tidak terbukti; step type dibatasi `kepala_bagian`/ `verifier`. |
| #29 | Kalkulasi hari kerja otomatis | ⚠️ | `WorkdayCalculator`, endpoint `/api/v1/cuti/calculate-workdays`, warning weekend/libur, validasi request, dan test unit/feature tersedia. Tetapi Blade menetapkan `workDays = result.data`, padahal kontrak API mengembalikan `data.jumlah_hari_kerja`; nilai/warning real-time tidak dirender sesuai kontrak. |
| #30 | Form pengajuan cuti | ⚠️ | Form, upload lampiran tervalidasi, hitung server-side, saldo, snapshot chain, status menunggu, audit, in-app/email queue, dan test tersedia. Dropdown tidak terbukti menyaring jenis Cuti Besar/CLTN untuk PPPK; validasi lintas tahun hanya diterapkan pada cuti tahunan, bukan semua pengajuan. |
| #31 | Approval engine cuti dinamis | ❌ | Snapshot per request, data-scope approver, notifikasi, audit, final deduction, QR proof, timeline, dan test E2E tersedia. Namun source memakai `REJECT`, `RejectLeaveAction`, dan route `/reject`, yang dilarang untuk keputusan cuti. Alur approval eksternal dan upload dokumen untuk cuti Kepala Lembaga belum ditemukan. |
| #32 | Saldo cuti dan daftar cuti pegawai | ✅ | Balance ledger, bucket N-2/N-1/tahun berjalan, seeder 2026, koreksi awal/manual ber-audit, rollover scheduler, halaman/API saldo, riwayat, dan test tersedia. |

## Status Issue Sprint 5

| Issue | Deliverable | Status | Bukti dan catatan |
|---:|---|:---:|---|
| #33 | Kalkulasi TMT otomatis | ⚠️ | EmployeeHistoryService menghitung kenaikan pangkat dari TMT pangkat +4 tahun, KGB +2 tahun, serta pensiun dari tanggal lahir dan BUP jenis jabatan ketika riwayat terbaru dibuat. Namun tidak ada TmtCalculatorService/observer/listener, milestone Satyalancana tidak disimpan, tidak ada rekalkulasi saat import, dan perubahan referensi BUP tidak memicu hitung ulang. |
| #34 | EWS scheduler harian | ⚠️ | EwsEngineService memeriksa pegawai aktif untuk lima trigger, threshold configurable, unique alert, log run, error notification, command, dan test tersedia. Command yang sama dijadwalkan di routes/console.php serta bootstrap/app.php sehingga berisiko dieksekusi dua kali. Alert tidak menyimpan is_eligible/trigger_days/tahun seperti task issue; in-app notification hanya dibuat untuk pegawai, bukan Admin Kepegawaian. |
| #35 | Halaman daftar EWS aktif | ✅ | Controller/action/view menampilkan nama, NIP, event, target, sisa hari, eligibility, dan follow-up; urutan sisa hari, warna urgensi, filter event/status, detail pegawai, akses Admin/Super Admin/Pimpinan, aksi ditangani/tidak perlu ber-catatan, audit, dan test role tersedia. |
| #36 | Flag kinerja baik dan kelayakan Satyalancana | ⚠️ | Toggle AJAX, endpoint JSON, FormRequest, Action, RBAC route, audit, flag/catatan Satyalancana, dan test tersedia. Deskripsi UI bukan tooltip dengan teks yang diminta. Saat flag kinerja false, source tetap membuat dan menampilkan alert pangkat sebagai Tidak Eligible, bertentangan dengan PRD yang menyatakan pegawai tidak muncul pada EWS kenaikan pangkat. |
| #37 | Notifikasi email | ⚠️ | NotificationService, job queue, tiga retry, log kegagalan final, template responsive Bahasa Indonesia, CTA aman, Mailpit development, dan test tersedia. Pemilihan event email masih hardcoded, bukan membaca reference notification channels; EWS Satyalancana serta keputusan cuti Perubahan/Tidak Disetujui tidak terdaftar sebagai channel email. Admin EWS hanya menerima email tambahan, bukan notifikasi in-app. |
| #38 | Session timeout | ⚠️ | Middleware menegakkan idle timeout 30 menit, logout/invalidate session, flash, redirect route login Keycloak, JSON 401, audit SESSION_TIMEOUT, dan test polling tersedia. Tetapi .env.example dan default Laravel masih menetapkan SESSION_LIFETIME=120, bukan 30 sebagaimana Issue #38. |

## Task yang Sudah Selesai pada Source

Status berikut tetap memerlukan QA/retest pada PHP kompatibel dan PostgreSQL sebelum tracker dapat diubah menjadi `Done`.

### Sprint 1

- ✅ Audit log database, service, immutable model, dan halaman audit tersedia.
- ✅ Notifikasi in-app, inbox/read/unread, notification bell, serta queue email pendukung tersedia.
- ✅ Layout master, design system, dan komponen UI/form dasar tersedia.
- ✅ CRUD hari libur/cuti bersama dengan RBAC, audit, seeder, dan test tersedia.

### Sprint 2

- ✅ Migration/model data pegawai dan relasi riwayat tersedia.
- ✅ Tambah/edit/daftar/detail pegawai memakai data nyata, validasi, upload, audit, dan test tersedia.
- ✅ Riwayat pangkat, jabatan, dan KGB bersifat append-only dengan `is_latest`, transaksi, dan audit.

### Sprint 3

- ✅ CRUD keluarga admin memakai API v1, FormRequest, Action, data scope, soft delete, audit masking NIK, dan test.

### Sprint 4

- ✅ Saldo cuti berbasis ledger, initial balance, koreksi beralasan/audit, rollover tahunan, seed 2026, dan halaman/API saldo tersedia.

### Sprint 5

- ✅ Halaman daftar EWS aktif: data nyata, sort urgensi, filter event/status, warna urgensi, detail pegawai, akses role, follow-up ber-catatan, audit, dan test tersedia.
- ✅ Perhitungan TMT pangkat/KGB/pensiun saat riwayat terbaru ditambahkan tersedia pada EmployeeHistoryService.
- ✅ Engine EWS memiliki lima trigger, threshold configurable, unique index alert, log eksekusi, command, dan test source.
- ✅ Toggle kinerja dan kelayakan Satyalancana menggunakan AJAX, validasi, Action, audit, serta test role/invalid request.
- ✅ Email memakai job queue dengan tiga retry, log kegagalan, template Bahasa Indonesia, dan CTA internal aman.
- ✅ Idle timeout 30 menit memiliki invalidasi sesi, redirect Keycloak, audit, respons JSON, dan test source.

## Task yang Masih Belum Selesai atau Harus Dikoreksi

### Prioritas P0 — wajib sebelum Sprint 1–5 dinyatakan selesai

- ❌ Hapus endpoint/API force delete, `DeleteEmployeeAction`, `PurgeDeletedEmployeesAction`, command `employees:purge-deleted`, jadwal purge, dan UI backup yang menjanjikan hard delete. Pegawai Fase 1 hanya soft delete/restore.
- ❌ Hapus route DELETE, UI, dan action penghapusan `DisciplineRecord`; riwayat disiplin wajib append-only.
- ✅ Perbaiki mapping Keycloak: simpan `users.employee_id`, validasi satu user–satu pegawai, audit persisten di `audit_logs`, dan test login semua role mapped.
- ❌ Ubah profil pegawai menjadi read-only. Hapus atau keluarkan endpoint self-service family/education serta perubahan data lain dari scope Fase 1; pisahkan perubahan password dari fitur profil jika tetap diperlukan secara administratif.
- ❌ Ganti semua istilah/action `REJECT`, `RejectLeaveAction`, dan route `/reject` pada domain cuti dengan nomenklatur `Tidak Disetujui`/ `NOT_APPROVED`; pertahankan keterangan wajib dan audit.
- ❌ Putuskan lalu dokumentasikan schema canonical cuti sebelum membuat migration baru: apakah PRD memakai `leave_approval_steps`, `leave_balance_adjustments`, `leave_documents`, atau proyek mempertahankan nama semantic alternatif saat ini. Konflik ini tidak boleh diselesaikan dengan asumsi.

### Prioritas P0 — tutup acceptance criteria Sprint 5

- ❌ Buat satu mekanisme kalkulasi TMT yang dipanggil saat riwayat dibuat/diubah, import selesai, dan referensi BUP berubah. Simpan atau modelkan milestone Satyalancana secara konsisten tanpa menduplikasi aturan di engine.
- ❌ Hapus salah satu registrasi jadwal app:run-ews. Pertahankan satu scheduler configurable pada timezone Asia/Makassar dan tambahkan test bahwa hanya satu event terdaftar.
- ❌ Lengkapi data alert EWS yang diperlukan requirement, termasuk eligibility saat alert dibuat, lalu pastikan semua penerima yang diwajibkan mendapat notifikasi in-app dan email untuk kelima event EWS.
- ❌ Terapkan source of truth PRD untuk flag kinerja false atau dokumentasikan keputusan yang mengubahnya: PRD menyebut alert pangkat tidak tampil, sedangkan issue dan source saat ini memilih tampil sebagai Tidak Eligible.
- ❌ Buat notification dispatcher yang membaca konfigurasi reference notification channels. Daftarkan seluruh keputusan cuti dan kelima event EWS; jangan gunakan daftar event email hardcoded.

### Prioritas P1 — tutup acceptance criteria Sprint 3

- ❌ Tambahkan template Data Pelengkap, Riwayat Kepangkatan, Riwayat Jabatan, dan Riwayat KGB.
- ❌ Tambahkan dua contoh data dummy pada setiap template yang relevan.
- ❌ Tambahkan manual column mapping dropdown, warning kolom tidak cocok, kontrak API, dan test-nya.
- ❌ Saat import sukses, buat snapshot riwayat yang tersedia, hitung TMT pangkat/KGB, dan hitung pensiun jika kolom kosong serta reference BUP tersedia.
- ❌ Sediakan laporan hasil import yang persisten/downloadable untuk error eksekusi, bukan hanya CSV yang dibangun dari state browser.

### Prioritas P1 — tutup acceptance criteria Sprint 4

- ❌ Beri Admin Kepegawaian hak assign Kepala Bagian sesuai PRD/US, dukung input tanggal mulai, dan tetapkan proses untuk memastikan satu Kepala Bagian aktif per pegawai.
- ❌ Tambahkan konfigurasi approval chain per unit dan UI penyusunan chain per pegawai yang lengkap. Ketua Tim Kerja tetap dipilih sebagai verifier tanpa role baru.
- ❌ Saring dropdown jenis cuti berdasarkan PNS/PPPK pada form. Pertahankan validasi server sebagai sumber kebenaran.
- ❌ Tolak semua pengajuan yang melintasi tahun kalender, bukan hanya Cuti Tahunan.
- ❌ Perbaiki integrasi form kalkulasi hari kerja agar membaca `result.data.jumlah_hari_kerja` dan menampilkan `result.data.warnings`; tambahkan test browser/feature kontrak respons.
- ❌ Implementasikan pencatatan/upload dokumen persetujuan eksternal untuk cuti Kepala Lembaga dengan audit dan akses Admin Kepegawaian.
- ❌ Lengkapi tampilan status awal dengan label `Menunggu [nama step pertama]` yang berasal dari snapshot chain.

### Prioritas P1 — pulihkan verifikasi dan delivery gate

- ❌ Jalankan project dengan PHP `>= 8.4.1` atau image/container yang sesuai `composer.lock`.
- ❌ Pulihkan Podman, jalankan PostgreSQL 17, migration, seeder, test fokus Sprint 1–4, `composer qa`, dan `php artisan route:list --path=api/v1 --json`.
- ❌ Sediakan dan jalankan suite PostgreSQL untuk UUID, JSON, FK, index, transaction, pagination, dan route binding. SQLite in-memory bukan bukti tunggal yang cukup.
- ❌ Kumpulkan evidence PR review/merge dan QA/retest, lalu perbarui tracker dari `Not Started` hanya setelah Definition of Done terpenuhi.
- ❌ Samakan SESSION_LIFETIME dengan 30 menit atau dokumentasikan alasan penggunaan lifetime Laravel 120 menit bersama idle timeout custom 30 menit; uji perilaku pada Keycloak dan database session nyata.

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

## Batasan Analisis

- Analisis menilai source yang ada pada commit `50b1e77`; tidak menggantikan QA manual, UAT, atau sign-off tim.
- Credential Keycloak, SMTP/email operasional, dan deployment Podman produksi tidak diuji.
- Ikon ✅ hanya menyatakan bukti implementasi source/test tersedia. Tracker tetap tidak dapat menyatakan `Done` tanpa review, QA/retest, dan evidence sesuai dokumen proses.
