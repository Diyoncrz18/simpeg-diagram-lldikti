# Tracking Role: Admin Kepegawaian

| Field | Nilai |
|---|---|
| Role internal | `admin_kepegawaian` |
| Tanggal analisis ulang | 27 Juli 2026 |
| Basis verifikasi | Branch `development` @ `e82b527` (sesudah PR #121–#131) |
| Dokumen asal (dikonsolidasi ke file ini) | `Analisis-Frontend-Backend-Role-Admin-Kepegawaian.md` — audit 21 Juli 2026 |
| Acuan produk | PRD v1.3 matriks role (CRUD pegawai + soft delete/restore + set atasan, dokumen, cuti non-approver, EWS, laporan, audit read) |
| Status keseluruhan | ⚠️ **Belum sepenuhnya sesuai** — dari 4 gap utama audit lama, 1 sudah tuntas (set atasan); lifecycle nonaktif, laporan, dan audit-sensitif masih terbuka |

## Ringkasan

Audit 21 Juli menyimpulkan 4 gap utama: (1) penetapan atasan hanya Super Admin, (2) soft
delete/restore tidak bisa dituntaskan dari UI Admin, (3) laporan L1/L1b/L3 belum memenuhi kontrak,
(4) audit pegawai berisiko menyimpan NIK/No. KK plaintext. Verifikasi ulang 27 Juli: **gap 1 sudah
ditutup penuh**, gap 2 membaik di lapisan route tapi memburuk di navigasi, gap 3 dan 4 masih
terbuka dan sudah terpetakan ke task tracker Sprint 6–7.

## ✅ Sudah Sesuai (terverifikasi 27 Juli)

| # | Area | Bukti |
|---|---|---|
| 1 | **Penetapan atasan/Kepala Bagian oleh Admin** (dulu ❌ P1 — bertentangan langsung dengan PRD) — route web + API kini `role:super_admin,admin_kepegawaian` + `permission:employees.update`, UI form tampil untuk Admin, memakai `AssignSupervisorRequest`, dan audit-nya `logOrFail()` | `routes/web.php:344-347`, `routes/api/v1/pegawai.php:137-140`, `admin/pegawai/show.blade.php:28-30,877`, `PegawaiController.php:604` |
| 2 | RBAC halaman — sidebar menyembunyikan menu privilese Super Admin; route konfigurasi menolak Admin | `AdminKepegawaianAccessTest` |
| 3 | Daftar/tambah/edit/detail pegawai dengan data & reference nyata, filter + pagination backend | `ListEmployeesAction`, `StoreEmployeeRequest`/`UpdateEmployeeRequest` |
| 4 | **Append-only riwayat ditegakkan dari form web** — id riwayat lama ditolak, action selalu membuat record baru | PR #129 |
| 5 | Import pegawai — wizard unggah→preview→validasi→eksekusi, template 2 baris contoh PNS+PPPK (PR #130), laporan hasil import permanen + CSV per-baris (PR #131) dengan kontrol kepemilikan batch | `EmployeeImportController::report()`, `EmployeeImportReportTest` |
| 6 | Tambah riwayat pangkat/jabatan/KGB via API kini ber-FormRequest (dulu `Request` mentah) | `StoreRankHistoryRequest`, `StorePositionHistoryRequest`, `StoreKgbHistoryRequest` |
| 7 | Dokumen inti — upload/edit metadata/detail/download sesuai permission | `DokumenController` (kecuali destroy/check-impact, lihat ⚠️ #4) |
| 8 | Monitoring cuti (bukan approver otomatis — sesuai PRD), rekap saldo + ledger + koreksi ber-audit, dokumen eksternal Kepala Lembaga, laporan cuti PDF/Excel fixed | `Actions/Cuti/*`, permission `cuti.read_all`, `cuti.kepala_lembaga_documents.manage` |
| 9 | EWS aktif + follow-up + pembaruan flag kinerja/Satyalancana dari detail pegawai | `ListActiveEwsAlertsAction`, `EmployeeSatyalancanaEligibilityTest` |
| 10 | **Email EWS Satyalancana** (dulu ❌ P1) — resolusi channel kini dari tabel `notification_event_channels`; `ews.satyalancana` ter-seed in-app + email | `NotificationChannelResolver.php:24-33`, `ReferenceSeeder.php:258` |
| 11 | Inbox notifikasi — paginator, tandai dibaca, ownership, CSRF | `notifications.index` |
| 12 | Audit log dapat dibaca Admin sesuai permission (kualitas halamannya lihat ⚠️ #6) | `routes/web.php:485-491` |

## ❌ / ⚠️ Belum Sesuai

| # | Prioritas | Temuan (terverifikasi 27 Juli) | Tindak lanjut |
|---|:---:|---|---|
| 1 | P0 | **Audit update pegawai bocor NIK/No. KK plaintext** — `UpdateEmployeeAction.php:35,349` masih `toArray()`; create sudah aman (`getRawOriginal()`) | Task tracker #2, Sprint 7 (7.2-1) |
| 2 | P1 | **Lifecycle pegawai nonaktif belum bisa dituntaskan dari UI** — route `data-nonaktif` + restore per-baris **sudah dibuka** untuk Admin (`routes/web.php:196-198,333-336`), tetapi menu `data-nonaktif` kini **hilang dari sidebar untuk semua role** (hanya muncul sebagai entri terkunci Kabag di `app.blade.php:98`), tidak ada tautan dari daftar pegawai, dan bulk restore tetap super_admin-only via halaman Data Backup | Perlu diangkat ke backlog (regresi navigasi — lebih luas dari temuan lama) |
| 3 | P1 | **Tombol Ubah Status / Hapus ke Backup disembunyikan dari Admin** — Blade masih cek `role === 'super_admin'` (`admin/pegawai/index.blade.php:632,673`) padahal route `pegawai.destroy`/`bulkDestroy` mengizinkan Admin ber-permission `employees.deactivate` (`routes/web.php:326-332`); UI dan route tidak konsisten | Backlog P1 (keputusan: tampilkan sesuai permission) |
| 4 | P1 | **Dokumen destroy/check-impact masih super_admin-only** (`routes/web.php:468-475`) — PRD menyebut Admin "mengelola dokumen"; batas kata *mengelola* belum pernah diputuskan resmi | Eskalasi keputusan produk (dicatat sejak audit 21 Juli) |
| 5 | P1 | **Dashboard Admin masih dummy kecuali panel EWS** — statistik/distribusi/cuti/pegawai/audit/hari-libur hardcoded di Blade | Task tracker #16 (Issue #39); kontrak K-3 sudah tersedia |
| 6 | P0/P2 | **Halaman Audit Log** — semua record `->get()` ke browser, filter/paginasi Alpine, payload tanpa masking | Sprint 7 (7.1-1); guard immutable = task #3 |
| 7 | P1 | **Laporan pegawai belum memenuhi L1/L1b/L3** — (a) preview masih closure lama yang memuat pegawai nonaktif + `email_pribadi`/`no_hp` ke browser; pengganti (`ExportPegawaiPreviewAction` + `LaporanController`) sudah ditulis tapi belum di-wire ke route; (b) modal custom Excel mati (state Alpine tidak didefinisikan); (c) PDF nominatif L1 belum ada; (d) L3 riwayat kenaikan pangkat belum punya route/menu untuk Admin (hanya `pimpinan.*`) | Task tracker #18, #20, #19 (Issue #42); L3-Admin perlu diangkat ke backlog |
| 8 | P2 | **Import: `validate` masih inline `$request->validate()`, `execute` tanpa validasi input** (`EmployeeImportController.php:57-63,73`) | Backlog P2 |
| 9 | P2 | **Endpoint web tambah riwayat pendidikan** (`pegawai.riwayat.store`) masih `Request` mentah dan `StoreEmployeeHistoryAction` tidak memvalidasi input (jalur pangkat/jabatan/KGB sudah ditutup dengan exception) | Backlog P2 |
| 10 | P2 | Form ubah password lokal di Profil — tidak sinkron dengan SSO Keycloak, tanpa audit | Keputusan produk (sama dengan Super Admin ⚠️ #16) |

## Riwayat Temuan Audit 21 Juli → Kondisi Sekarang

| Gap utama audit lama | Kondisi 27 Juli |
|---|---|
| ❌ Penetapan atasan Super Admin-only | ✅ Diperbaiki penuh (route + UI + FormRequest + audit strict) |
| ❌ Soft delete/restore tak tersentuh dari UI | ⚠️ Route dibuka untuk Admin; ❌ navigasi justru hilang untuk semua role — perlu menu/tautan |
| ❌ Laporan L1/L1b/L3 | ❌ Masih — preview/PII/modal/PDF L1 (task #18–#20), L3-Admin belum ada |
| ❌ Audit NIK/No. KK plaintext | ⚠️ Create aman; ❌ update masih (task #2) |
| ❌ Email Satyalancana | ✅ Diperbaiki (channel berbasis DB) |
| ⚠️ Riwayat via `Request` mentah | ✅ API pangkat/jabatan/KGB ber-FormRequest; ⚠️ sisa endpoint web pendidikan |
