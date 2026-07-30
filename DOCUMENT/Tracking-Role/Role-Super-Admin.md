# Tracking Role: Super Admin

| Field | Nilai |
|---|---|
| Role internal | `super_admin` |
| Tanggal analisis ulang | 27 Juli 2026 |
| Basis verifikasi | Source `development` @ `0b94960` (PR #147) + working tree Slice 4 terverifikasi lokal pada PostgreSQL dan browser, 29 Juli 2026 |
| Dokumen asal (dikonsolidasi ke file ini) | `Analisis-Frontend-Backend-Role-Super-Admin.md` (audit 21 Juli) dan `Analisis-Kesesuaian-Administrasi-Sistem-Super-Admin.md` (audit 23 Juli) |
| Acuan produk | PRD v1.3 §4.2 (konfigurasi sistem, user management, seluruh kemampuan Admin), §12 (audit immutable), §15–16 (RBAC & master dari database), US-1.4 |
| Status keseluruhan | ⚠️ **Belum sepenuhnya sesuai** — operasional harian kuat; halaman konfigurasi sistem masih titik terlemah |

## Ringkasan

Super Admin punya dua wajah. Alur operasional (pegawai, dokumen, cuti, EWS, notifikasi, approval
chain) sudah nyata, ter-guard, dan teruji. Sebaliknya, sebagian menu **Administrasi Sistem** masih
statis atau memberi keberhasilan palsu. Sejak audit Juli, satu halaman ❌ terbesar — **Kelola Akses
User** — sudah dituntaskan penuh (PR #126, QA 23 Juli lulus termasuk verifikasi PostgreSQL 17).
Yang tersisa terkonsentrasi pada: Data Master, Hari Libur web, Pengaturan Sistem, RBAC, dashboard,
dan pengerasan audit log.

## ✅ Sudah Sesuai (terverifikasi 27 Juli)

| # | Area | Bukti |
|---|---|---|
| 1 | **Kelola Akses User / US-1.4** (dulu ❌ P0) — option value kode internal, paginasi+filter server-side (`ListUserMappingsAction` → `paginate()->withQueryString()`), controller tipis ber-FormRequest, audit dengan identifier ter-masking, `logOrFail()` di dalam `DB::transaction` sehingga kegagalan audit membatalkan mutasi, `lockForUpdate` anti-race, constraint unik terverifikasi di PostgreSQL 17 | `admin/user-management/index.blade.php:172-176,407-411`; `ListUserMappingsAction.php:39-60`; `UpdateUserMappingAction.php:33,71-82,216-230`; PR #126 (`a4e00dd`); `Bukti-QA-Kelola-Akses-User-Super-Admin.md` |
| 2 | SSO Keycloak = autentikasi saja; role/permission dari DB SIMPEG; bootstrap Super Admin pertama; collision guard | `HandleKeycloakCallbackAction` |
| 3 | CRUD pegawai utama + soft delete/restore + halaman Data Backup (`onlyTrashed()`, tanpa permanent delete) | `StoreEmployeeRequest`/`UpdateEmployeeRequest` + Action |
| 4 | **Riwayat pangkat/jabatan/KGB append-only ditegakkan** — id riwayat lama ditolak validasi, action selalu membuat record baru | PR #129 (`1b3dfd6`), `UpdateEmployeeRequest.php:42-44`, `EmployeeUpdateTest` |
| 5 | Dokumen & SK — upload/update/detail/download/check-impact/delete via Action + FormRequest | `DokumenController` |
| 6 | Import pegawai — wizard lengkap, template 2 baris contoh (PNS+PPPK, PR #130), **laporan hasil import permanen + unduhan CSV** (PR #131, tabel `import_batches`) | `GenerateImportTemplateAction`, `ExecuteImportBatchAction`, `DownloadImportReportAction` |
| 7 | Monitoring + keputusan cuti, rekap saldo/ledger + koreksi ber-audit, laporan cuti PDF/Excel fixed, konfigurasi approval chain dinamis (PYBMC, backfill, audit) | `Actions/Cuti/*`, `CutiConfigPageTest` |
| 8 | EWS aktif + follow-up + flag eligibility; scheduler + deduplikasi alert | `ListActiveEwsAlertsAction` |
| 9 | **Konfigurasi EWS: controller kini tipis** (dulu ⚠️ validasi inline panjang) — `UpdateEwsConfigRequest` + `ShowEwsConfigPageAction`/`UpdateEwsConfigAction` | `EwsConfigController.php:15-28` |
| 10 | **Kanal notifikasi berbasis database** — allowlist email hardcoded diganti `notification_event_channels` (fail-closed); `ews.satyalancana` kini in-app + email sesuai PRD 11.3 | `NotificationRecipientResolver.php:48-55`, `NotificationChannelResolver.php:24-33`, `ReferenceSeeder.php:247-260` |
| 11 | Export Excel pegawai (standard/custom) dengan allowlist kolom aman — NIK/No. KK ditolak backend | `ExportPegawaiExcelAction`, `ExportCustomPegawaiExcelAction` |
| 12 | API Hari Libur CRUD nyata (`ref_hari_libur`) + audit + UUID binding + test | `Api/V1/HariLiburController`, `HariLiburCrudTest` |
| 13 | Notifikasi in-app — paginator, tandai dibaca, ownership | `notifications.index` |
| 14 | `CreateEmployeeAction` tidak lagi mengaudit NIK/No. KK plaintext — memakai `getRawOriginal()` (ciphertext) | `CreateEmployeeAction.php:253` |

## ❌ / ⚠️ Belum Sesuai

| # | Prioritas | Temuan (terverifikasi 27 Juli) | Tindak lanjut |
|---|:---:|---|---|
| 1 | P0 | **Audit update pegawai masih bocor NIK/No. KK plaintext** — `UpdateEmployeeAction.php:35,349` masih `toArray()` pada model bercast `encrypted` tanpa `$hidden`; `AuditController` meneruskan `old/new_values` mentah ke browser. (Sisi create sudah aman.) | Task tracker #2, Sprint 7 (7.2-1) |
| 2 | P0 | **Immutability audit belum ditegakkan** — `AuditLog` (41 baris) tanpa guard `updating`/`deleting`, tanpa observer; murni konvensi | Task tracker #3, Sprint 7 (7.2-2) |
| 3 | P0/P2 | **Halaman Audit Log memuat semua record ke browser** — `AuditController.php:13-18` `->get()` tanpa paginate, filter/paginasi di Alpine, payload tanpa masking | Sprint 7 slice 7.1 (7.1-1) |
| 4 | P1 | **Audit fail-closed baru parsial** — `AuditService` kini punya `logOrFail()`, tapi baru dipakai 2 tempat (user mapping, assign supervisor); 39 pemakaian lain masih `log()` yang menelan kegagalan | Task tracker #12 |
| 5 | P1 | **Data Master — sebagian besar ditutup PR #134 (merged 27 Juli)**: CRUD lima tabel referensi (golongan, jenis jabatan, eselon, status pegawai, jenjang pendidikan) + halaman Data Master berbasis database + guard baris sistem, sesuai keputusan K-1. **Tab unit kerja hierarkis MERGED ke `development` via PR #147 pada 29 Juli (squash `0b94960`)**: membaca database dengan tampilan berjenjang, nama unik se-sistem, penonaktifan induk ditolak selama masih ada sub-unit aktif, jenis unit terkunci empat nilai resmi, kedalaman dihitung otomatis dan seluruh sub-unit ikut disesuaikan saat induk dipindahkan. **Channel & event notifikasi selesai implementasi lokal 29 Juli dan menunggu commit/PR.** Sisa: tab referensi lain (jenis cuti, agama, status perkawinan, hari libur) dan CRUD `ref_jabatan` (naik prioritas per K-4, satu-satunya jalur Admin mengatur BUP per jabatan detail). **Keputusan `ref_bup` SUDAH DITUTUP per K-4 (27 Juli 2026):** di-deprecate, tidak dibuatkan CRUD, tab statis BUP pada halaman Data Master dicabut bersama form modalnya karena menampilkan data hardcoded beserta tombol aksi yang tidak menyimpan apa pun | Task tracker #22 (Issue #46) — lanjutan slice |
| 6 | P1 | **Hari Libur web masih statis + audit session** — `Admin/HariLiburController` masih `static $hariLiburData`, store/update/destroy hanya menulis `dynamic_audit_logs`; API/model nyata sudah ada tapi tidak dipakai halaman | Task tracker #5 |
| 7 | P1 | **Pengaturan Sistem = keberhasilan palsu** — `SettingsController::update()` tidak membaca input; input `x-model` tanpa `name=`; profil instansi/SMTP hardcode; panel mapping SSO di halaman ini masih memakai `<option value="Super Admin">` (bug lama yang di Kelola Akses User sudah diperbaiki) dan hanya memutasi array JS lokal | Belum ada task khusus — kandidat: sembunyikan/disable form sampai kontrak storage diputuskan (usulan audit 23 Juli tetap berlaku) |
| 8 | P1 | **RBAC** — `RbacController::update()` masih `Request` mentah, `sync()` dalam loop tanpa transaction, audit hanya session, tanpa invariant anti-lockout server-side | Belum ada task khusus — perlu diangkat ke backlog (P1) |
| 9 | P1 | **Dashboard masih dummy kecuali panel EWS** — statistik pegawai/golongan/cuti/pegawai/audit/hari-libur hardcoded; `href="#"` + `alert('Ini adalah data dummy')`; tombol Setuju/Tunda cuti hanya mengubah `innerHTML` klien | Task tracker #16 (Issue #39); kontrak payload W1–W7 sudah dikunci di K-3 |
| 10 | P1 | **Preview laporan pegawai masih closure lama** — `routes/web.php:246-284` query semua pegawai (termasuk nonaktif) + kirim `email_pribadi`/`no_hp` ke browser. `ExportPegawaiPreviewAction` + `LaporanController::exportPegawai()` **sudah ada tapi tidak terdaftar di route mana pun** (dead code); view juga belum sinkron dengan kontrak action baru | Task tracker #18 (Issue #42.3) |
| 11 | P1 | **Modal custom export tidak bisa dipakai** — `showCustomExportModal`/`customColumns`/`filterOptions`/`submitCustomExport` direferensikan di `admin/laporan/export-pegawai.blade.php:386-442` tapi tidak ada di komponen Alpine `exportPegawai` (:473-637) | Task tracker #20 (Issue #42.5, #42.8) |
| 12 | P1 | **PDF nominatif L1 belum ada** — seluruh pemakaian DomPDF hanya untuk cuti + riwayat kepangkatan Pimpinan; tidak ada PDF daftar pegawai | Task tracker #19 (Issue #42.9–42.14) |
| 13 | P1 | **Laporan L3 (riwayat kenaikan pangkat) belum tersedia untuk Admin/Super Admin** — route hanya di prefix `pimpinan.` | Belum ada task khusus — perlu diangkat ke backlog |
| 14 | P2 | **Konfigurasi EWS belum atomik** — `UpdateEwsConfigAction` loop `log()`+`setVal()` per key tanpa `DB::transaction`, masih menulis cermin `dynamic_audit_logs`; histori audit konfigurasi `->get()` tanpa paginasi | Gabungkan dengan task #12/#22 |
| 15 | P2 | **Import: tahap validate masih `$request->validate()` inline; tahap execute tanpa validasi input sama sekali** (hanya baca batch dari cache) | Backlog P2 (lanjutan Issue #22) |
| 16 | P2 | **Form ubah password lokal** — kini via `UpdatePasswordRequest`/`UpdatePasswordAction` (struktur membaik), tetapi tetap hanya memutasi password lokal tanpa Keycloak dan tanpa audit; berpotensi menyesatkan pengguna SSO | Keputusan produk: hapus/sembunyikan atau dokumentasikan |

### Update 29 Juli 2026 — Channel Notifikasi

UI channel dan kebijakan event notifikasi **selesai implementasi lokal dan menunggu commit/PR**. Super Admin memperoleh halaman database-backed untuk master kill-switch dan matriks 13 event, dengan desired state, status efektif fail-closed, audit atomik, pagination, queued email recheck, serta perlindungan config/credential. WhatsApp Business tetap disabled karena adapter belum tersedia. Dengan ini, item channel notifikasi tidak lagi menjadi sisa implementasi #46; sisa #46 adalah CRUD `ref_jabatan` dan tab referensi lain. Verifikasi final lokal dan reviewer telah lulus.

## Riwayat Temuan Audit 21–23 Juli → Kondisi Sekarang

| Temuan lama | Kondisi 27 Juli |
|---|---|
| ❌ P0 Kelola Akses User (value role salah, filter browser) | ✅ Diperbaiki penuh (PR #126 + QA PostgreSQL 17) |
| ❌ Email EWS Satyalancana tidak terkirim | ✅ Diperbaiki — resolusi channel dari DB, event ter-seed in-app+email |
| ⚠️ EwsConfigController gemuk | ✅ Controller dirapikan; ⚠️ transaction + paginasi audit tersisa |
| ❌ Audit create pegawai berisiko plaintext | ✅ `getRawOriginal()`; ❌ jalur **update** masih bocor |
| ❌ Data Master statis | ⚠️ Sebagian besar ditutup PR #134 dan PR #147; channel notifikasi selesai implementasi lokal 29 Juli dan menunggu commit/PR. Sisa tab lanjutan dan CRUD `ref_jabatan`. Keputusan `ref_bup` sudah ditutup per K-4 |
| ❌ Hari Libur web, Pengaturan Sistem, Dashboard, RBAC, preview laporan, modal custom, PDF L1/L3 | ❌ Semua masih berlaku (lihat tabel di atas) |
| ⚠️ AuditService menelan kegagalan | ⚠️ Parsial — `logOrFail()` lahir tapi belum menyebar |
