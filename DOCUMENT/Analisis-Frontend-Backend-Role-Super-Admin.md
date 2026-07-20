# Analisis Frontend dan Backend Role Super Admin

> Status: Audit read-only — **belum sepenuhnya sesuai** dengan dokumen Fase 1.
>
> Tanggal audit: 21 Juli 2026.
>
> Cakupan: menu sidebar dan halaman pendukung Super Admin, Blade SSR, rute, controller/action, FormRequest, service, model, migration/seeder, audit, notifikasi, serta pengujian terarah. Dokumen ini mencatat kondisi implementasi saat diaudit; tidak mengubah kebutuhan produk dan tidak menggantikan PRD.

## 1. Legenda Status

| Ikon | Arti |
|:---:|---|
| ✅ | Sudah tersedia, berfungsi dengan data nyata, dan secara umum sesuai dokumen/pola kode. |
| ⚠️ | Sudah dikerjakan sebagian atau berfungsi, tetapi masih ada ketidaksesuaian, kekurangan kualitas kode, cakupan test, atau requirement yang belum tuntas. |
| ❌ | Belum selesai, tidak berfungsi dari UI, memakai data dummy, atau memberi keberhasilan palsu tanpa perubahan data yang nyata. |

## 2. Acuan Audit

Urutan sumber kebenaran yang digunakan:

1. [PRD-SIMPEG-Fase1-Core.md](PRD-SIMPEG-Fase1-Core.md) versi 1.2.
2. [Panduan-Penulisan-Kode-SIMPEG.md](Panduan-Penulisan-Kode-SIMPEG.md).
3. [User-Stories-SIMPEG-Fase1.md](User-Stories-SIMPEG-Fase1.md).

Hak utama Super Admin yang menjadi dasar audit:

| Area | Ketentuan PRD |
|---|---|
| Konfigurasi sistem | Mengelola reference table, konfigurasi EWS, hari libur nasional, channel notifikasi, dan chain approval cuti. |
| User management | Memetakan akun Keycloak dengan pegawai dan menetapkan satu role internal SIMPEG. |
| Fitur admin | Memiliki seluruh kemampuan Admin Kepegawaian. |
| Soft delete/restore | Menonaktifkan tanpa hapus permanen dan memulihkan data bila diperlukan. |
| Audit log | Memiliki akses penuh ke audit log yang immutable. |

Audit juga menilai aturan teknis berikut:

- Mutation menggunakan FormRequest, Action, dan audit database.
- Blade hanya menangani presentasi/state UI ringan; tidak menjadi sumber data/domain utama.
- Reference table, hari libur, status pegawai, jabatan, unit kerja, BUP, dan channel notifikasi menggunakan source of truth database.
- Data dummy tidak boleh tersisa pada primary flow yang telah memiliki backend nyata.
- List besar menggunakan filtering/pagination dari backend, bukan seluruh data dikirim ke browser.

## 3. Ringkasan Eksekutif Frontend

Frontend Super Admin mempunyai fondasi yang baik pada data pegawai, dokumen, cuti, EWS aktif, konfigurasi approval cuti, notifikasi, dan sebagian laporan. Banyak alur tersebut sudah memakai Action, FormRequest, data database, CSRF, pagination, serta guard akses backend.

Namun, frontend belum dapat dinyatakan siap sepenuhnya karena beberapa halaman konfigurasi utama masih statis atau semu:

1. **Data Master** belum memiliki CRUD nyata untuk reference table.
2. **Hari Libur web** menampilkan data statis dan tidak menulis ke `ref_hari_libur`, walaupun API yang benar sudah tersedia.
3. **Dashboard** hanya memakai data EWS nyata; mayoritas widget lain masih dummy.
4. **Kelola Akses User** memiliki nilai role UI yang tidak cocok dengan validasi backend sehingga pemetaan bisa gagal dari browser.
5. **Pengaturan Sistem** menampilkan data statis dan memberi pesan sukses tanpa menyimpan konfigurasi.
6. **Laporan Pegawai** memiliki preview yang tidak konsisten dengan export backend serta modal custom export yang tidak dapat dipakai.
7. **Role & Permission** menyimpan perubahan RBAC, tetapi auditnya masih session dan bukan audit database immutable.

## 4. Status Kesesuaian Seluruh Halaman

| Kelompok | Halaman | Status | Kondisi saat audit |
|---|---|:---:|---|
| Dashboard | Dashboard Super Admin | ❌ | Hanya panel EWS memakai data nyata. Statistik pegawai, cuti, distribusi golongan, daftar pegawai, audit terbaru, dan hari libur masih array dummy di Blade. |
| Kepegawaian | Data Pegawai | ⚠️ | Data, filter, pagination, detail, dan aksi utama menggunakan backend nyata. Controller masih terlalu besar dan memuat sejumlah query, transaksi, validasi, serta bulk mutation langsung. |
| Kepegawaian | Tambah Pegawai | ✅ | Memakai `StoreEmployeeRequest`, `CreateEmployeeAction`, dan reference data database. |
| Kepegawaian | Edit Pegawai | ✅ | Memakai request/action khusus dan pilihan reference dari database. |
| Kepegawaian | Detail Pegawai dan Riwayat | ⚠️ | Relasi data nyata tersedia. Endpoint tambah riwayat dan assign atasan masih memakai `Request` serta sebagian logic di controller. |
| Kepegawaian | Pegawai Nonaktif | ✅ | Menampilkan soft-deleted employee dan menyediakan alur restore. |
| Kepegawaian | Data Backup | ✅ | Menggunakan `Employee::onlyTrashed()`, pencarian, pagination, dan data riwayat jabatan nyata. |
| Kepegawaian | Import Pegawai | ✅ | Alur import tersedia dan test terarah lulus. Perlu tetap diuji di PostgreSQL untuk UUID, file, dan transaksi. |
| Kepegawaian | Dokumen & SK | ✅ | Upload, update, download, pemeriksaan dampak, dan delete menggunakan Action/FormRequest. |
| Laporan | Export Pegawai | ❌ | Preview dibuat melalui closure route, memuat pegawai nonaktif/pensiun, dan mengirim email pribadi/no. HP ke browser. Modal custom export memiliki state/handler Alpine yang tidak ada. |
| Cuti | Monitoring Pengajuan Cuti | ✅ | Menggunakan Action, filter backend, pagination, serta scope akses berdasarkan role/permission. |
| Cuti | Detail dan Persetujuan Cuti | ✅ | Otorisasi approver aktif dijaga backend; keputusan memakai Action/Service dan istilah resmi cuti. |
| Cuti | Rekap Saldo Cuti | ✅ | Menggunakan query/action, paginator, ledger saldo, dan koreksi saldo melalui FormRequest/Action. |
| Cuti | Laporan Cuti PDF/Excel | ✅ | Controller tipis: FormRequest → Action → preview/PDF/Excel. Tidak ditemukan data dummy pada view laporan. |
| Cuti | Konfigurasi Approval Cuti | ✅ | Chain dinamis per pegawai, pencarian kandidat dibatasi, alasan wajib, PYBMC global, backfill, dan bukti audit tersedia. |
| EWS | EWS Aktif | ✅ | Data alert nyata melalui Action, filter, status tindak lanjut, dan endpoint follow-up khusus. |
| EWS | Konfigurasi EWS | ⚠️ | Nilai konfigurasi dan audit database tersimpan benar. Namun validasi/rule panjang masih berada di controller `Request`, dan daftar audit konfigurasi belum dipaginasi. |
| Notifikasi | Pusat Notifikasi | ✅ | Data nyata, paginator backend, endpoint tandai dibaca, CSRF, dan ownership notifikasi diterapkan. |
| Administrasi | Kelola Akses User | ❌ | Backend pemetaan dan audit tersedia, tetapi nilai role dari UI tidak cocok dengan kode role yang divalidasi backend. |
| Administrasi | Role & Permission | ⚠️ | Permission matrix tersimpan ke database dan mencegah self-lockout Super Admin. Audit perubahan masih hanya session. |
| Administrasi | Data Master | ❌ | Data referensi adalah array Blade statis; tidak ada controller/action/route CRUD nyata. |
| Administrasi | Hari Libur | ❌ | Halaman web memakai controller statis/session; tidak menggunakan API/Action yang benar untuk `ref_hari_libur`. |
| Administrasi | Pengaturan Sistem | ❌ | Data instansi, SMTP, mapping user, dan master summary statis; controller tidak menyimpan request payload. |
| Administrasi | Audit Log | ⚠️ | Audit berasal dari database, tetapi semua log dikirim ke browser lalu difilter/dipaginasi oleh Alpine. |
| Pendukung | Profil Saya | ⚠️ | Profil/data pegawai nyata. Form ubah password hanya mengubah password lokal, bukan Keycloak, dan belum dicatat sebagai audit mutation. |

## 5. Task yang Sudah Sesuai

| Status | Task/kapabilitas | Bukti implementasi | Catatan |
|:---:|---|---|---|
| ✅ | CRUD data pegawai utama | `StoreEmployeeRequest`, `UpdateEmployeeRequest`, Action pegawai, data reference | Struktur dasar create/update sudah memakai data nyata. |
| ✅ | Soft delete dan restore pegawai | `Employee::onlyTrashed()`, backup page, audit soft delete/restore | Tidak ada permanent delete pegawai pada alur ini. |
| ✅ | Pengelolaan dokumen pegawai | Action dokumen, FormRequest upload/update, download terotorisasi | Alur utama sudah tidak memakai data dummy. |
| ✅ | Monitoring dan approval cuti | Action/Service cuti, FormRequest keputusan, guard approver aktif | Keputusan memakai label resmi `Disetujui`, `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui`. |
| ✅ | Konfigurasi approval chain dinamis | Form chain pegawai, PYBMC global, backfill, audit | Salah satu implementasi frontend paling lengkap. |
| ✅ | Rekap saldo dan ledger cuti | Query/action rekap, paginator, adjustment request/action | Data saldo dan ledger berasal dari database. |
| ✅ | EWS aktif dan follow-up | `ListActiveEwsAlertsAction`, request follow-up | Data alert nyata dan dapat ditindaklanjuti sesuai role. |
| ✅ | Notifikasi in-app | Paginator, mark read, mark all read, ownership test | Endpoint dan UI terhubung. |
| ✅ | Laporan cuti fixed PDF/Excel | Request filter dan Action export | Sesuai pola controller tipis. |

## 6. Task yang Sudah Dikerjakan tetapi Belum Tuntas

| Status | Task/kapabilitas | Kondisi saat ini | Pekerjaan lanjutan |
|:---:|---|---|---|
| ⚠️ | Data Pegawai dan riwayat | Mayoritas nyata, tetapi controller masih gemuk; beberapa mutation memakai `Request`. | Pecah bulk mutation, riwayat, dan assign atasan ke FormRequest + Action yang spesifik. |
| ⚠️ | Konfigurasi EWS | Persistensi dan audit database benar, tetapi rule validasi masih panjang di controller. | Buat `UpdateEwsConfigRequest`; pindahkan rule/penyimpanan ke Action/Service; paginate audit konfigurasi. |
| ⚠️ | Role & Permission | `sync()` permission bekerja, tetapi audit masih `dynamic_audit_logs` session. | Gunakan `AuditService`/`audit_logs` immutable dengan old/new permission dan alasan perubahan. |
| ⚠️ | Audit Log UI | Detail audit nyata dan aman ditampilkan, tetapi seluruh record dimuat ke browser. | Gunakan query/filter/pagination API atau Action backend pada halaman Blade. |
| ⚠️ | Profil Saya | Data profil nyata, tetapi fitur password tidak sesuai dengan SSO Keycloak dan belum diaudit. | Hapus/sembunyikan form lokal atau arahkan ke manajemen akun Keycloak; auditkan mutation keamanan yang relevan. |
| ⚠️ | Role restore pegawai | Super Admin memiliki bulk restore; route restore tunggal juga masih dapat diakses Admin Kepegawaian. | Putuskan dan dokumentasikan interpretasi PRD yang konsisten sebelum mengubah policy/route. |
| ⚠️ | Test frontend | Test request/API penting tersedia, namun beberapa kontrak HTML tidak diuji. | Tambah test halaman Blade/browser untuk Hari Libur web, nilai role user mapping, dashboard, settings, dan custom export modal. |

## 7. Task yang Belum Selesai atau Tidak Berfungsi

### 7.1 Dashboard Super Admin

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Widget W1–W3 data kepegawaian | Nilai/statistik distribusi masih hardcoded di Blade. | Buat query/action dashboard untuk jumlah pegawai, status, golongan, jabatan, dan unit. |
| ❌ | Widget W5 ringkasan cuti | Daftar cuti masih array `$cutiList`. | Ambil pengajuan cuti nyata dengan batas jumlah dan tautan detail yang sah. |
| ❌ | Widget W6 daftar pegawai | Daftar pegawai serta tombol detail masih dummy/href `#`. | Ambil data pegawai nyata dan gunakan route detail yang terotorisasi. |
| ❌ | Widget W7 audit terbaru | Audit dashboard masih array statis. | Ambil audit log terbaru dari database dengan data yang sudah dimasking. |
| ❌ | Hari libur pada dashboard | Menggunakan array hari libur statis. | Gunakan `ref_hari_libur` dan hapus data contoh. |

### 7.2 Data Master

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Membaca reference table dari database | Blade mendefinisikan array statis untuk golongan, jabatan, eselon, cuti, unit, BUP, dan lainnya. | Buat controller/action per master atau generic service yang tetap menjaga rule domain per tabel. |
| ❌ | Tambah/edit/hapus master | Tombol hanya membuka modal tanpa request penyimpanan. | Sediakan route deklaratif, FormRequest, Action, policy, audit, dan response/error state. |
| ❌ | Validasi referensi yang sedang dipakai | Tidak ada guard pemakaian relasi. | Tolak delete atau soft delete item yang masih direferensikan, sesuai jenis master. |
| ❌ | Kelola `ref_notification_channels` | Tidak ada menu/tab/form untuk channel notifikasi. | Implementasikan konfigurasi channel berdasarkan requirement PRD: in-app, email, dan WhatsApp Business future/configurable. |
| ❌ | Unit kerja hierarkis | Tampilan master tidak menunjukkan pengelolaan `parent_id`, `level`, dan struktur unit. | Tambahkan pengelolaan pohon unit kerja dan validasi hierarchy. |
| ❌ | Jabatan/BUP aktif dan override | Master statis tidak menampilkan lifecycle/status aktif atau aturan BUP secara utuh. | Gunakan data `ref_jabatan`, `ref_jenis_jabatan`, dan `ref_bup` nyata sesuai PRD. |

### 7.3 Hari Libur

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Daftar hari libur dari `ref_hari_libur` | Web controller memakai `public static $hariLiburData`. | Ganti sumber halaman Blade ke `ListHariLiburAction`/model `RefHariLibur`. |
| ❌ | Tambah hari libur nyata | Web `store()` hanya menulis `dynamic_audit_logs` session. | Gunakan `StoreHariLiburRequest` dan `CreateHariLiburAction`. |
| ❌ | Edit hari libur nyata | Web `update()` tidak memutasi database. | Gunakan UUID binding, request update, dan `UpdateHariLiburAction`. |
| ❌ | Hapus hari libur nyata | Web `destroy()` tidak menghapus `ref_hari_libur`. | Gunakan `DeleteHariLiburAction`, audit database, dan refresh data dari server. |
| ❌ | Audit immutable hari libur | Audit hanya disimpan pada session. | Gunakan `AuditService` sehingga muncul di Audit Log resmi. |
| ⚠️ | API Hari Libur | API CRUD, Action, request, UUID binding, dan test sudah tersedia. | Hubungkan frontend Blade ke implementasi API/Action ini; hindari dua implementasi paralel. |

### 7.4 Kelola Akses User dan Role Internal

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Kontrak nilai role UI/backend | UI memakai value label `Super Admin`, sedangkan FormRequest meminta `super_admin`. | Gunakan value kode internal pada setiap `<option>` dan label Bahasa Indonesia sebagai teks. |
| ⚠️ | Pemetaan email pegawai | Index hanya memakai `Employee::email`, tetapi update juga mencari `email_pribadi`. | Gunakan strategi email yang konsisten serta tampilkan status tidak dapat dipetakan secara jujur. |
| ✅ | Unik Keycloak ID | Backend mencegah satu Keycloak ID dipetakan ke dua user. | Pertahankan test uniqueness dan audit database. |
| ✅ | Role berasal dari database SIMPEG | Validasi role internal sudah tersedia dan tidak memakai role claim Keycloak. | Perbaiki UI agar mengirim kode yang sama. |

### 7.5 Pengaturan Sistem

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Persistensi konfigurasi instansi | Semua nilai ditulis statis di Blade. | Jangan tampilkan sebagai editable sebelum storage/config contract disetujui. |
| ❌ | Simpan konfigurasi | `SettingsController::update()` tidak membaca ataupun menyimpan input. | Implementasikan hanya setting yang berada dalam scope dan telah memiliki keputusan produk. |
| ❌ | Audit pengaturan | Hanya `dynamic_audit_logs` session. | Gunakan audit database immutable dengan before/after yang aman. |
| ❌ | Mapping SSO dari halaman settings | Data mapping statis dan aksi tidak terhubung ke backend mapping user. | Hilangkan duplikasi; gunakan halaman Kelola Akses User yang benar setelah diperbaiki. |

### 7.6 Laporan dan Export Pegawai

| Status | Task | Kondisi | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | Preview nominatif konsisten dengan export | Preview route memuat seluruh employee, termasuk Pensiun/nonaktif; export standard memakai sumber berbeda. | Gunakan satu Action/query kanonis untuk preview dan export; default hanya pegawai aktif sesuai requirement. |
| ❌ | Perlindungan data pada preview | Email pribadi dan no. HP dikirim ke Alpine/browser; kolom dapat diaktifkan pada preview. | Hapus field kontak pribadi dari payload browser dan dari column picker yang tidak diizinkan. |
| ❌ | Export custom dari UI | Modal mengacu pada `showCustomExportModal`, `customColumns`, `filterOptions`, dan `submitCustomExport` yang tidak didefinisikan. | Tambahkan state/handler dan tombol pembuka modal, atau sederhanakan menjadi form server-side yang benar. |
| ⚠️ | Export Excel standard | Endpoint, request, dan test file Excel tersedia. | Selaraskan preview, filter, active scope, dan UI dengan endpoint standard. |
| ✅ | Validasi kolom custom di backend | Backend menolak kolom sensitif seperti NIK. | Pertahankan allowlist canonical dan tambahkan test browser/UI. |

## 8. Catatan Kualitas Kode dan Frontend

### Hal yang sudah baik

- Komponen UI seperti button, table, filter bar, breadcrumb, modal, dan pagination telah dipakai secara luas sehingga tampilan cukup konsisten.
- Banyak form utama memakai `@csrf`, route bernama, FormRequest, dan pesan validasi server.
- Halaman cuti, notifikasi, dokumen, dan EWS aktif sudah memisahkan sebagian besar rule domain dari Blade.
- Notifikasi menerapkan ownership dengan baik: pengguna hanya dapat melihat serta menandai notifikasi miliknya sendiri.
- Konfigurasi approval cuti menerapkan progressive enhancement: form utama tetap valid di backend, Alpine hanya mendukung UX.

### Hal yang perlu dituntaskan

| Status | Area teknis | Temuan | Tindakan |
|:---:|---|---|---|
| ⚠️ | Route | Data Master dan preview laporan pegawai masih memakai closure yang melakukan pekerjaan aplikasi. | Pindahkan ke controller/action yang deklaratif dan teruji. |
| ⚠️ | Mutation boundary | RBAC, Hari Libur web, Settings, bulk pegawai, assign atasan, dan konfigurasi EWS masih memakai `Request` langsung pada sebagian alur. | Gunakan FormRequest per use case termasuk otorisasi dan validasi input. |
| ❌ | Audit resmi | `dynamic_audit_logs` session dipakai oleh Hari Libur web, Settings, dan RBAC. | Hentikan pemakaian session sebagai audit resmi; gunakan `AuditService` dan `audit_logs`. |
| ⚠️ | Pagination/frontend payload | Audit log dan laporan pegawai memuat collection besar lalu filter/paginate di browser. | Terapkan query/filter/paginator backend serta kirim payload minimum. |
| ❌ | Data dummy primary flow | Dashboard, Data Master, Hari Libur, dan Settings masih berisi data dummy/statis. | Ganti dengan view model dari Action/database atau sembunyikan fitur yang belum diselesaikan. |
| ⚠️ | Keycloak SSO | Ubah password lokal dapat menimbulkan kesan password Keycloak ikut berubah. | Hapus/redirect fitur dan dokumentasikan alur manajemen akun SSO. |

## 9. Hasil Pengujian

Pengujian terarah yang dijalankan:

```powershell
php artisan test --compact `
  tests/Feature/UserMappingControllerTest.php `
  tests/Feature/HariLiburCrudTest.php `
  tests/Feature/DashboardEwsTest.php `
  tests/Feature/EwsActivePageTest.php `
  tests/Feature/CutiConfigPageTest.php `
  tests/Feature/EmployeeDocumentTest.php `
  tests/Feature/EmployeeImportTest.php `
  tests/Feature/EmployeeReportExportTest.php `
  tests/Feature/NotificationInboxTest.php `
  tests/Feature/ProfileTest.php
```

Hasil:

```text
98 passed (446 assertions)
```

Interpretasi hasil test:

| Status | Temuan cakupan test |
|:---:|---|
| ✅ | API Hari Libur, notifikasi, EWS, dokumen, import, profil, konfigurasi cuti, dan sebagian laporan memiliki test yang lulus. |
| ⚠️ | Test Hari Libur menguji API yang benar, bukan halaman web `/hari-libur` yang masih memakai controller statis. |
| ⚠️ | Test user mapping mengirim kode role internal secara langsung sehingga tidak menangkap nilai `<option>` UI yang salah. |
| ⚠️ | Test laporan saat ini bahkan mengunci preview yang menampilkan pegawai pensiun; test perlu diselaraskan dengan requirement active employee. |
| ❌ | Belum ada bukti test browser/end-to-end untuk memastikan modal custom export, form settings, form Hari Libur web, dan kontrak HTML user mapping berfungsi dari UI. |

Pemeriksaan format juga dijalankan:

```powershell
composer format:check
```

Hasil belum hijau karena ada perubahan lokal yang sudah ada di worktree SIMPEG:

```text
bootstrap/app.php
test_show.php
```

Dokumen audit ini tidak mengubah kedua file tersebut. `git diff --check` tidak menemukan error whitespace.

> Batas verifikasi: test lokal tidak menggantikan verifikasi PostgreSQL 17 dan browser end-to-end. Perubahan terkait UUID, JSON, FK/index, date casting, pagination query, route model binding, serta upload tetap perlu diuji pada PostgreSQL.

## 10. Urutan Task Perbaikan yang Direkomendasikan

| Urutan | Status | Task perbaikan | Hasil yang harus tercapai |
|:---:|:---:|---|---|
| 1 | ❌ | Satukan Hari Libur web dengan API/Action `RefHariLibur`. | Semua perubahan dari UI benar-benar mengubah database, mengubah kalkulasi hari kerja, dan menghasilkan audit immutable. |
| 2 | ❌ | Implementasikan Data Master sebagai CRUD reference table nyata. | Super Admin dapat mengelola master sesuai scope dengan validasi pemakaian, hierarchy, audit, dan tanpa data statis. |
| 3 | ❌ | Ganti widget dashboard dummy dengan query/action nyata. | W1–W7 menampilkan angka/tautan yang dapat dipercaya dan tidak ada `href="#"`/alert dummy. |
| 4 | ❌ | Perbaiki value role pada form Kelola Akses User. | Browser mengirim `super_admin`, `admin_kepegawaian`, `pimpinan`, `kepala_bagian`, atau `pegawai`; mapping sukses dan teraudit. |
| 5 | ❌ | Nonaktifkan atau desain ulang Pengaturan Sistem. | Tidak ada pesan sukses palsu; hanya konfigurasi Fase 1 yang punya storage, security, audit, dan keputusan produk yang jelas. |
| 6 | ❌ | Satukan preview/export laporan pegawai dan aktifkan custom export UI. | Preview, Excel standard, dan Excel custom memakai filter/scope kanonis tanpa kontak sensitif di browser. |
| 7 | ⚠️ | Pindahkan audit RBAC dari session ke database. | Perubahan permission menyimpan actor, IP, user-agent, old/new values, dan timestamp pada `audit_logs`. |
| 8 | ⚠️ | Ubah Audit Log Blade menjadi server-side filtering/pagination. | UI tidak mengirim semua audit records ke browser dan tetap bisa filter user/periode/event/modul. |
| 9 | ⚠️ | Rapikan mutation controller lama. | Setiap use case mutasi memiliki FormRequest, Action, otorisasi, audit, dan test fokus. |
| 10 | ⚠️ | Tambahkan test UI/Blade dan PostgreSQL. | Cakupan test memastikan kontrak halaman benar, bukan hanya API/direct request. |

## 11. Kesimpulan Frontend

Role Super Admin memiliki banyak fondasi backend yang sudah baik, terutama pada kepegawaian, dokumen, cuti, EWS aktif, notifikasi, dan konfigurasi approval cuti. Akan tetapi, halaman yang paling penting untuk konfigurasi sistem belum konsisten dengan source of truth database.

Prioritas penerimaan perbaikan harus berada pada **Data Master**, **Hari Libur**, **Dashboard**, **Kelola Akses User**, **Pengaturan Sistem**, dan **Laporan Pegawai**. Selama halaman tersebut masih menunjukkan data dummy atau keberhasilan palsu, frontend Super Admin belum dapat dianggap selesai sesuai Fase 1.

## 12. Ringkasan Eksekutif Backend

Backend Super Admin tidak sepenuhnya tertinggal dari frontend. Fondasi penting telah ada dan beberapa di antaranya mengikuti arsitektur yang diharapkan: Keycloak hanya menjadi sumber identitas, RBAC berasal dari database SIMPEG, CRUD hari libur yang benar tersedia pada API, konfigurasi approval cuti memakai Action/FormRequest, EWS dan channel notifikasi menggunakan database, serta export Excel memakai allowlist kolom.

Namun, backend belum dapat dinyatakan sepenuhnya sesuai Fase 1 karena terdapat temuan keamanan dan auditabilitas yang material:

1. **Edit pegawai dapat menulis NIK dan No. KK dalam bentuk plaintext ke `audit_logs`.** Hal ini terjadi karena `UpdateEmployeeAction` mengaudit hasil `Employee::toArray()` tanpa payload masking, lalu `AuditController` meneruskan `old_values`/`new_values` ke view apa adanya.
2. **Audit resmi belum konsisten dan belum fail-closed.** `AuditService` menelan kegagalan tulis audit; RBAC, Pengaturan Sistem, dan halaman web Hari Libur masih menulis pseudo-audit ke session, bukan ke `audit_logs`.
3. **Jalur web Hari Libur dan Pengaturan Sistem tidak memutasi source of truth**, walaupun API Hari Libur yang benar telah tersedia.
4. **Notifikasi email EWS Satyalancana belum diaktifkan di resolver**, padahal PRD mewajibkan in-app dan email untuk event tersebut.
5. **Preview laporan pegawai masih bypass Action/FormRequest**, mengekspor payload kontak pribadi ke browser dan tidak memakai query kanonis yang dipakai Excel.

Dengan demikian, fondasi backend cukup kuat untuk dilanjutkan, tetapi acceptance Super Admin harus menahan fitur yang terkena temuan ❌ sampai audit data sensitif, persistensi konfigurasi, dan kontrak delivery notifikasi diperbaiki.

## 13. Status Kesesuaian Backend per Area

| Area backend | Status | Bukti implementasi | Penilaian terhadap dokumen |
|---|:---:|---|---|
| SSO Keycloak dan bootstrap user | ✅ | `HandleKeycloakCallbackAction` memakai subject Keycloak stabil, email terverifikasi, proteksi collision, dan bootstrap Super Admin pertama. | Sesuai: role claim Keycloak tidak dipakai untuk memberikan akses; role internal SIMPEG menjadi sumber otorisasi. |
| Gerbang route dan permission inti | ✅ | Grup utama memakai `keycloak.auth`, timeout, `role`; route data pegawai, cuti konfigurasi, Hari Libur API, dan audit memiliki gate role/permission. | Secara umum sesuai pola RBAC internal dan UUID binding untuk route yang relevan. |
| Pemetaan user dan role internal | ⚠️ | `UpdateUserMappingRequest` memvalidasi lima kode role; controller mencegah Keycloak ID dipakai dua user dan membuat audit database. | Sudah berfungsi bila operator memasukkan subject Keycloak stabil. Namun field tidak mendukung resolusi **email Keycloak** sebagaimana AC, `authorize()` masih selalu `true` dan route hanya mengandalkan role gate. |
| RBAC matrix | ⚠️ | `roles.permissions()->sync()` menyimpan permission secara nyata. | Persistensi ada, tetapi mutation memakai `Request` langsung dan audit perubahan hanya `dynamic_audit_logs` pada session; tidak memenuhi audit immutable. |
| CRUD pegawai, soft delete/restore, dokumen, import | ⚠️ | Mayoritas flow memakai FormRequest, Action, transaksi, storage service, audit, dan test. | Dasar kuat, tetapi controller pegawai masih memuat bulk mutation/riwayat/assign atasan dengan `Request` langsung; audit edit pegawai memiliki kebocoran data sensitif. |
| Hari Libur API | ✅ | `StoreHariLiburRequest`/`UpdateHariLiburRequest` → Action → `RefHariLibur`, UUID binding, validasi tanggal duplikat, audit database. | Sesuai source of truth dan dapat dipakai kalkulasi cuti. |
| Hari Libur web | ❌ | `Admin\\HariLiburController` memakai array statis dan hanya menyimpan `dynamic_audit_logs` di session. | Tidak sesuai karena create/update/delete dari halaman web tidak mengubah `ref_hari_libur` dan tidak memberi audit resmi. |
| Data Master/reference table | ⚠️ | Migration melengkapi hierarki unit, status aktif jabatan, `default_bup`, katalog status pegawai, serta `ref_notification_channels`; `ReferenceSeeder` bersifat idempoten. | Fondasi skema/seed sudah tersedia. Namun belum ada controller, Action, FormRequest, atau CRUD backend untuk pengelolaan master oleh Super Admin; seed jabatan awal juga belum memberi contoh nilai override `default_bup`. |
| Konfigurasi dan engine EWS | ⚠️ | Nilai `EwsConfig`, run scheduler, alert deduplikasi, dan audit konfigurasi berbasis database tersedia. | Fungsional, tetapi `EwsConfigController` masih menaruh validasi/rule/loop persistensi di controller dengan `Request` langsung tanpa satu transaction atomik. |
| Channel dan pengiriman notifikasi | ⚠️ | `NotificationChannelResolver` membaca `ref_notification_channels`; `NotificationService` mendispatch email melalui queue setelah commit. | Arsitektur channel sesuai. Allowlist email tidak mencakup `ews.satyalancana`, sehingga satu event wajib PRD hanya masuk in-app. |
| Laporan dan export pegawai | ⚠️ | Endpoint Excel memakai `ExportPegawaiRequest`/`CustomEmployeeExportRequest`, Action, service query, dan allowlist kolom aman. | Preview web masih closure route yang mengambil semua pegawai dan kontak pribadi ke browser; ia bypass request/action dan tidak konsisten dengan endpoint export. |
| Konfigurasi approval dan saldo cuti | ✅ | FormRequest dan Action khusus menjaga chain dinamis, alasan, backfill/PYBMC, serta audit. | Salah satu vertical slice backend yang paling dekat dengan pola dokumen. |
| Audit Log | ❌ | Tabel/model dan halaman/API baca tersedia. | Tidak memenuhi sepenuhnya: payload pegawai tidak dimasking, beberapa mutation hanya audit session, kegagalan `AuditService` tidak menghentikan/menandai operasi, dan model tidak mencegah update/delete secara eksplisit. |
| Pengujian | ⚠️ | Test feature terarah lulus; seeder, API Hari Libur, mapping, RBAC, EWS, notifikasi, laporan, dan audit memiliki bukti test. | `phpunit.xml` masih memakai SQLite in-memory; belum menjadi bukti PostgreSQL 17 untuk UUID, JSON, FK/index, date casting, transaksi, dan binding rute. |

## 14. Temuan Backend yang Perlu Diperbaiki

### 14.1 Prioritas P0 — keamanan data sensitif pada audit pegawai

| Status | Bukti | Risiko | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | `Employee` mengenkripsi `nik` dan `no_kk`, tetapi tidak menyembunyikannya dari serialisasi. `UpdateEmployeeAction` mengambil nilai lama dan baru dengan `toArray()` lalu mengirimnya ke `AuditService`. `AuditController` juga meneruskan payload audit mentah ke Blade. | Cast `encrypted` didekripsi saat serialisasi Eloquent; akibatnya NIK/No. KK plaintext dapat tersimpan di JSON audit dan terlihat oleh role yang berhak membaca audit. Ini bertentangan dengan Panduan Kode yang melarang menyimpan data sensitif yang tidak perlu serta mewajibkan masking. | Buat payload audit pegawai khusus berbasis allowlist/masking; jangan masukkan `nik` dan `no_kk` ke `old_values`/`new_values`. Masking harus diterapkan saat menulis **dan** saat membaca data audit historis. Tambahkan test regresi yang membuktikan NIK/No. KK tidak muncul pada record audit maupun HTML/API audit. |

### 14.2 Prioritas P1 — audit resmi, immutable, dan dapat diandalkan

| Status | Temuan | Dampak | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | `AuditService::log()` dan `logAs()` menangkap semua exception lalu hanya menulis warning. | Mutation penting dapat berhasil walaupun tidak ada audit record, bertentangan dengan scope audit PRD. | Tentukan kebijakan audit fail-closed untuk mutation yang wajib diaudit, atau gunakan transactional outbox dengan retry yang dapat dipantau; jangan hanya menelan error. |
| ❌ | `RbacController`, `SettingsController`, dan `Admin\\HariLiburController` menggunakan `dynamic_audit_logs` session. | Rekam jejak hilang antar-session, tidak muncul pada audit resmi, dan mudah tidak konsisten dengan data database. | Pindahkan setiap flow nyata ke Action + `AuditService`/`audit_logs`; hapus audit session sebagai sumber kebenaran. |
| ⚠️ | `AuditLog` hanya menghilangkan `updated_at`; tidak ada event model/guard aplikasi yang menolak `update()` atau `delete()`. | Tidak ada route mutasi audit saat ini, tetapi immutability bergantung pada konvensi dan bisa rusak oleh kode mendatang. | Tambahkan guard model untuk menolak update/delete dan test immutability; gunakan kontrol database bila deployment membutuhkan perlindungan tambahan. |
| ⚠️ | `AuditController` memuat seluruh audit log dan mengirim old/new values tanpa filtering/masking server-side. | Payload besar dan data audit sensitif dapat terekspos lebih luas dari yang diperlukan. | Gunakan `ListAuditLogsAction` atau query object untuk filter/pagination server-side serta display payload yang telah dimasking. |

### 14.3 Prioritas P1 — integritas konfigurasi dan source of truth

| Status | Temuan | Perbaikan yang diperlukan |
|:---:|---|---|
| ❌ | Halaman web `/hari-libur` menjalankan implementasi statis yang berbeda dari API CRUD `RefHariLibur`. | Jadikan satu Action/service sebagai jalur tunggal bagi Blade dan API; gunakan binding UUID, request khusus, database, dan audit resmi. |
| ❌ | `SettingsController::update()` tidak membaca dan tidak menyimpan nilai form. | Jangan tampilkan form editable sebelum kontrak setting Fase 1 disetujui; setelah itu implementasikan storage, request, action, audit, dan test. |
| ❌ | Data Master hanya route closure/view statis; tidak ada backend CRUD untuk reference table atau channel notifikasi. | Implementasikan per domain dengan validasi pemakaian relasi, hierarchy unit, lifecycle aktif/nonaktif, audit, dan permission. |
| ⚠️ | `EwsConfigController` mencampur validasi, rule urutan threshold, audit, dan update loop dalam controller. | Pindahkan otorisasi/validasi ke `UpdateEwsConfigRequest`, rule/persistensi ke Action/Service, dan jalankan batch perubahan dalam transaction. |

### 14.4 Prioritas P1 — notifikasi dan laporan

| Status | Temuan | Bukti terhadap requirement | Perbaikan yang diperlukan |
|:---:|---|---|---|
| ❌ | `NotificationRecipientResolver::emailEnabled()` mengizinkan empat event EWS, tetapi tidak `ews.satyalancana`. | PRD tabel 11.3 mewajibkan in-app **dan email** untuk EWS Satyalancana H-180/H-90/H-30. | Tambahkan event Satyalancana ke allowlist, pastikan pegawai dan admin penerima mendapatkan email sesuai channel aktif, dan tambah test queue/mail. |
| ❌ | Route preview `/laporan/export-pegawai` adalah closure yang memuat semua employee, email pribadi, dan no. HP; ia tidak memakai `ExportPegawaiRequest` atau `ExportPegawaiPreviewAction`. | Panduan meminta route deklaratif, controller tipis, payload minimum, dan perlindungan data sensitif. | Alihkan route ke `LaporanController::exportPegawai`, gunakan query kanonis `EmployeeExportDataService`, dan batasi payload preview pada kolom yang diizinkan. |
| ⚠️ | `ExportPegawaiPreviewAction` sudah ada tetapi belum dipakai route web; default UI status `Aktif` perlu diselaraskan dengan parameter query yang benar. | Preview dan Excel dapat menghasilkan scope yang berbeda. | Jadikan satu sumber query/filter untuk preview, Excel standard, dan Excel custom; kunci kontrak ini dengan test. |

### 14.5 Prioritas P2 — konsistensi arsitektur dan kontrak input

| Status | Temuan | Perbaikan yang diperlukan |
|:---:|---|---|
| ⚠️ | Bulk deactivate/restore, tambah riwayat, assign atasan, validasi/eksekusi import, RBAC, dan konfigurasi EWS masih memiliki `Request`/rule bisnis langsung di controller. | Pecah per use case menjadi FormRequest → Action → Service/model agar otorisasi, transaksi, audit, dan test lebih mudah dijaga. |
| ⚠️ | `UpdateUserMappingRequest::authorize()` selalu `true`; keamanan semata-mata berasal dari middleware route. | Tambahkan otorisasi request/policy sebagai pertahanan berlapis dan dokumentasikan siapa yang boleh mengubah mapping. |
| ⚠️ | Mapping user menyimpan string sebagai `keycloak_id` tanpa membedakan subject Keycloak dengan email SSO. | Tegaskan kontrak: resolve email SSO secara aman atau ubah label/form agar hanya menerima subject Keycloak; tambahkan validasi dan test untuk kedua jalur yang didukung. |
| ⚠️ | `default_bup` telah tersedia di skema `ref_jabatan`, tetapi seed jabatan awal belum menggunakan override. | Isi override bila memang diperlukan secara bisnis dan uji kalkulasi BUP; nullable tetap dapat dipertahankan untuk jabatan yang mengikuti default jenis jabatan. |

## 15. Task Backend Berdasarkan Status

### Sudah sesuai

| Status | Task backend | Keterangan |
|:---:|---|---|
| ✅ | Keycloak sebagai autentikasi, RBAC internal sebagai otorisasi | Tidak memberi akses berdasarkan role claim Keycloak; account tanpa role valid ditolak middleware. |
| ✅ | API Hari Libur berbasis database | Request, Action, audit, UUID binding, dan validasi tanggal duplikat tersedia. |
| ✅ | Fondasi reference table Fase 1 | Hierarki unit, status pegawai, lifecycle jabatan, BUP override, dan channel notifikasi telah dimigrasikan; seeder bersifat idempoten. |
| ✅ | Export Excel dengan allowlist kolom | Standard/custom export memakai FormRequest dan Action, serta tidak mengizinkan kolom sensitif seperti NIK. |
| ✅ | Konfigurasi approval cuti dan saldo | Menggunakan use case terpisah, validasi, scope/otorisasi, audit, dan test. |
| ✅ | Dispatcher notifikasi + queue | Channel dibaca dari reference table dan email didispatch lewat queue setelah commit, bukan SMTP langsung dari domain. |

### Sudah dikerjakan tetapi belum tuntas

| Status | Task backend | Kekurangan yang tersisa |
|:---:|---|---|
| ⚠️ | CRUD pegawai dan dokumen | Mayoritas use case mengikuti Action/FormRequest, tetapi controller lama masih gemuk dan audit edit harus diamankan. |
| ⚠️ | RBAC matrix | `sync()` database bekerja, tetapi request/action/audit immutable belum lengkap. |
| ⚠️ | EWS | Engine, deduplikasi, konfigurasi database, dan flag eligibility ada; controller konfigurasi perlu dirapikan secara atomik. |
| ⚠️ | Reference data | Skema dan seed tersedia, tetapi pengelolaan Super Admin belum memiliki backend CRUD. |
| ⚠️ | User mapping | Unique ID dan role internal tersedia, tetapi dukungan email Keycloak/subject dan otorisasi request perlu diperjelas. |
| ⚠️ | Audit list/API | Filter/paginator Action tersedia pada API, tetapi endpoint masih menerima query tanpa FormRequest dan Blade masih memuat semua record. |
| ⚠️ | Notifikasi EWS | Empat event EWS email tersedia; Satyalancana belum masuk daftar email. |

### Belum selesai atau tidak sesuai

| Status | Task backend | Alasan |
|:---:|---|---|
| ❌ | Masking audit data pegawai | NIK/No. KK dapat masuk ke audit plaintext pada update dan ditampilkan kembali. |
| ❌ | Jaminan audit mutation | Kegagalan audit ditelan, beberapa flow hanya audit session, dan immutability belum dijaga eksplisit. |
| ❌ | Hari Libur dari halaman web | Implementasi web tidak menyimpan ke `ref_hari_libur`. |
| ❌ | Pengaturan Sistem | Tidak ada persistensi request payload maupun audit resmi. |
| ❌ | Data Master CRUD | Tidak terdapat endpoint/use case backend untuk mengelola master sesuai scope Super Admin. |
| ❌ | Email EWS Satyalancana | Event wajib PRD tidak pernah melewati allowlist pengiriman email. |
| ❌ | Preview laporan pegawai kanonis dan minimal data | Closure route bypass arsitektur dan mengirim kontak pribadi ke browser. |

## 16. Hasil Pengujian Backend

Pengujian terarah tambahan yang dijalankan untuk backend Super Admin:

```powershell
php artisan test --compact `
  tests/Feature/ReferenceSeederTest.php `
  tests/Feature/DatabaseSeederTest.php `
  tests/Feature/HariLiburCrudTest.php `
  tests/Feature/UserMappingControllerTest.php `
  tests/Feature/RbacPermissionMiddlewareTest.php `
  tests/Feature/CutiAuditCoverageTest.php `
  tests/Feature/LeaveDecisionAuditTest.php `
  tests/Feature/EmailNotificationTest.php `
  tests/Feature/EwsSchedulerTest.php `
  tests/Feature/EmployeeReportExportTest.php `
  tests/Feature/EmployeeSatyalancanaEligibilityTest.php `
  tests/Feature/AuditLogIndexTest.php `
  tests/Feature/AuditPageIntegrationTest.php
```

Hasil:

```text
99 passed (349 assertions)
```

| Status | Interpretasi |
|:---:|---|
| ✅ | Seeder/reference, API Hari Libur, mapping user, RBAC middleware, audit cuti, keputusan cuti, email yang telah diaktifkan, scheduler EWS, report export, flag Satyalancana, dan halaman audit memiliki bukti test lulus. |
| ⚠️ | Lulusnya test tidak meniadakan temuan desain: test Hari Libur menarget API, bukan controller web statis; test email saat ini belum mengunci kewajiban email Satyalancana; dan test audit belum memastikan NIK/No. KK dimasking. |
| ⚠️ | `phpunit.xml` menetapkan `DB_CONNECTION=sqlite` dan `DB_DATABASE=:memory:`. Seluruh hasil di atas adalah bukti perilaku pada SQLite, bukan validasi PostgreSQL 17. |
| ❌ | Belum ada bukti test PostgreSQL terpisah untuk migration reference terbaru, UUID binding, JSON audit, foreign key/index, transaksi konfigurasi EWS, atau perlindungan payload audit sensitif. |

## 17. Urutan Perbaikan Terpadu Frontend–Backend

| Urutan | Status | Task | Kriteria selesai |
|:---:|:---:|---|---|
| 1 | ❌ | Amankan audit pegawai. | `nik`/`no_kk` tidak pernah tersimpan atau ditampilkan dalam plaintext pada audit; ada test regresi penulisan dan pembacaan. |
| 2 | ❌ | Jadikan audit mutation resmi, persist, dan immutable. | Tidak ada lagi `dynamic_audit_logs` sebagai sumber resmi; kegagalan audit ditangani dengan kebijakan yang dapat dipertanggungjawabkan; update/delete audit ditolak dan teruji. |
| 3 | ❌ | Satukan Hari Libur web dengan use case `RefHariLibur`. | UI, API, kalkulasi hari kerja, dan audit memakai data database yang sama. |
| 4 | ❌ | Implementasikan Data Master dan channel notifikasi. | Setiap master Fase 1 yang dikelola memiliki route, permission, FormRequest, Action, audit, guard relasi, dan UI nyata. |
| 5 | ❌ | Perbaiki email EWS Satyalancana. | Saat channel email aktif, pegawai dan admin penerima memperoleh email H-180/H-90/H-30; queue/mail test lulus. |
| 6 | ❌ | Satukan preview dan export laporan pegawai. | Semua format memakai Action/filter/scope kanonis; browser tidak menerima kontak pribadi atau kolom sensitif yang tidak diperlukan. |
| 7 | ⚠️ | Rapikan controller legacy. | RBAC, EWS config, bulk pegawai, riwayat, atasan, dan import mutation memakai FormRequest/Action/Service yang fokus dan teruji. |
| 8 | ⚠️ | Perjelas kontrak Keycloak user mapping. | Operator tidak dapat salah memasukkan email sebagai subject; AC email SSO/Keycloak ID memiliki perilaku yang eksplisit dan teruji. |
| 9 | ⚠️ | Uji PostgreSQL 17 dan browser E2E. | CI/local menjalankan suite relevan pada PostgreSQL serta test browser untuk halaman konfigurasi utama. |

## 18. Kesimpulan Terpadu

Implementasi Super Admin sudah mempunyai banyak **vertical slice backend** yang sehat, khususnya SSO/RBAC internal, data pegawai inti, dokumen, approval cuti, saldo cuti, EWS, API Hari Libur, reference schema, dan export Excel terproteksi. Hasil test terarah juga menunjukkan fondasi tersebut berjalan pada lingkungan test saat ini.

Namun, kondisi keseluruhan masih **belum sepenuhnya sesuai Fase 1**. Dua penghalang utama adalah kebocoran NIK/No. KK melalui audit pegawai dan audit resmi yang belum konsisten/terjamin. Setelah itu, perbaikan perlu berfokus pada Hari Libur web, Data Master, Pengaturan Sistem, email Satyalancana, dan preview laporan agar tampilan Super Admin dan backend menggunakan source of truth yang sama.

Tidak ada perubahan kode aplikasi dalam audit ini. Semua temuan harus dijadikan dasar perubahan terpisah yang diawali keputusan produk bila menyentuh kontrak Keycloak, kebijakan audit fail-closed, atau desain konfigurasi sistem.

## 19. Ketidaksesuaian per Halaman Super Admin

Bagian ini mengurutkan halaman mengikuti sidebar Super Admin, dimulai dari **Dashboard**. Halaman pendukung yang dibuka dari halaman utama—seperti tambah/edit/detail pegawai dan import—diletakkan tepat di bawah induknya. Kolom frontend membahas data, interaksi, dan kontrak HTML/Blade; kolom backend membahas route, otorisasi, request, Action/Service, persistensi, audit, serta keamanan data.

Keterangan pada kolom **Tidak ada temuan material** berarti tidak ada ketidaksesuaian penting yang ditemukan pada audit ini untuk flow halaman tersebut. Hal itu tidak menghapus kewajiban regression test saat kode halaman diubah kelak.

### 19.1 Dashboard

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 1 | **Dashboard** (`dashboard`) | ❌ Widget statistik pegawai, distribusi golongan, pengajuan cuti, daftar pegawai, audit terbaru, dan hari libur masih array dummy di Blade. Sebagian tautan detail masih `href="#"`/dummy. | ⚠️ Data EWS telah nyata, tetapi tidak ada use case/query dashboard terpadu untuk statistik pegawai, cuti, audit terbaru, dan hari libur. Data dummy dari backend/view model tidak boleh menjadi source of truth. | P1 |

### 19.2 Kepegawaian

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 2 | **Data Pegawai** (`data-pegawai`) | ⚠️ Daftar, filter, pagination, dan aksi utama sudah memakai data nyata. Kontrak modal/halaman tetap perlu diuji untuk seluruh variasi status dokumen dan bulk action. | ⚠️ CRUD utama telah memakai request/action, tetapi bulk deactivate/restore masih berada di controller dengan `Request` langsung. Audit perubahan pegawai harus diperbaiki karena dapat memuat NIK/No. KK tanpa masking. | P0 untuk audit; P2 arsitektur |
| 2a | **Tambah Pegawai** (`pegawai.create`) | ✅ Form dan reference data sudah berasal dari database; tidak ditemukan ketidaksesuaian frontend material. | ⚠️ `StoreEmployeeRequest` dan `CreateEmployeeAction` sudah benar, tetapi payload audit create perlu ditinjau agar tidak menyimpan atribut sensitif yang tidak diperlukan, walaupun nilai terenkripsi belum tentu plaintext. | P1 |
| 2b | **Edit Pegawai** (`pegawai.edit`) | ✅ Form, `old()`, error validasi, dan pilihan reference sudah nyata. | ❌ `UpdateEmployeeAction` menggunakan `Employee::toArray()` sebagai nilai audit lama/baru. Cast encrypted membuka nilai `nik`/`no_kk` ketika serialisasi sehingga berisiko tersimpan plaintext. | P0 |
| 2c | **Detail Pegawai & Riwayat** (`pegawai.show`) | ⚠️ Relasi dan tab data nyata tersedia. Tambah riwayat/assign atasan masih perlu test interaksi HTML yang lebih lengkap. | ⚠️ Tambah riwayat dan assign atasan masih memakai `Request` dan logic controller langsung, bukan FormRequest/Action yang terpisah. | P2 |
| 2d | **Pegawai Nonaktif** (`data-nonaktif`) | ✅ List dan alur restore tersedia dari data soft-delete. | ⚠️ Restore tunggal masih dapat diakses Admin Kepegawaian, sedangkan bulk restore dibatasi Super Admin. Interpretasi terhadap hak restore Super Admin perlu diputuskan/didokumentasikan agar konsisten. | P2 |
| 3 | **Data Backup** (`data-backup`) | ✅ Menampilkan pegawai terhapus, pencarian, pagination, dan data riwayat nyata; tidak ditemukan ketidaksesuaian frontend material. | ✅ Menggunakan `onlyTrashed()` dan restore tanpa permanent delete pada flow ini. Tetap pertahankan audit soft delete/restore pada setiap perubahan. | — |
| 4 | **Dokumen & SK** (`dokumen`) | ✅ Upload, edit metadata, detail, download, dan pemeriksaan dampak memakai data nyata. | ✅ Action/FormRequest dan pemeriksaan dampak digunakan untuk flow utama. Tidak ditemukan ketidaksesuaian backend material pada audit ini. | — |
| 2e | **Import Pegawai** (`pegawai.import`, dibuka dari Data Pegawai) | ✅ Alur unggah, preview, validasi, eksekusi, dan status batch tersedia. | ⚠️ Unggah memakai request khusus, tetapi validasi/eksekusi batch masih memvalidasi `Request` langsung di controller. Flow perlu dipisah ke FormRequest/Action dan diuji kembali pada PostgreSQL untuk UUID, file, queue, dan transaksi. | P2 |

### 19.3 Cuti

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 5 | **Pengajuan Cuti** (`cuti`) | ✅ Monitoring, detail, dan tindakan approval yang tersedia untuk Super Admin terhubung ke data nyata dan label keputusan resmi. | ✅ Otorisasi approver per tahap, Action/Service keputusan, audit, serta notifikasi inti tersedia. Tidak ditemukan gap material khusus halaman ini. | — |
| 6 | **Rekap Cuti** (`cuti.rekap`) | ✅ Filter, pagination, rekap saldo, dan ledger menggunakan data nyata. | ✅ Query/action rekap serta FormRequest/Action koreksi saldo tersedia. | — |
| 7 | **Export Cuti** (`cuti.laporan`) | ✅ Preview dan pilihan fixed PDF/Excel terhubung; tidak ditemukan data dummy pada flow utama. | ✅ Controller tipis memakai FormRequest → Action → response preview/PDF/Excel. | — |
| 8 | **Konfigurasi Approval Cuti** (`cuti.config`) | ✅ Form chain global/per pegawai, pencarian kandidat, alasan, PYBMC, dan backfill tersedia. | ✅ Action/FormRequest menjaga chain dinamis, scope, audit, dan backfill. Tidak ditemukan ketidaksesuaian material khusus halaman ini. | — |

### 19.4 EWS dan Notifikasi

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 9 | **Notifikasi** (`notifications.index`) | ✅ Paginator, status baca, CSRF, dan ownership halaman tersedia. | ⚠️ Halaman inbox benar, tetapi delivery event **EWS Satyalancana** belum mengirim email karena `ews.satyalancana` tidak ada dalam allowlist `NotificationRecipientResolver`. PRD mewajibkan in-app dan email untuk event ini. | P1 |
| 10 | **EWS Aktif** (`ews`) | ✅ Alert nyata, filter, follow-up, dan scope role tersedia. | ⚠️ Engine EWS, deduplikasi alert, dan flag eligibility tersedia. Kekurangan email Satyalancana memengaruhi outcome EWS walaupun daftar alert halaman ini tetap tampil benar. | P1 |
| 11 | **Konfigurasi EWS** (`ews.config`) | ⚠️ Nilai konfigurasi dan histori audit nyata, tetapi histori belum dipaginasi dan menambah payload halaman. | ⚠️ `EwsConfigController` mencampur validasi, rule urutan threshold, audit, dan loop persistensi melalui `Request` langsung. Perlu `UpdateEwsConfigRequest`, Action/Service, dan transaction atomik. | P2 |

### 19.5 Administrasi Sistem

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 12 | **Kelola Akses User** (`user-management`) | ❌ Nilai role pada `<option>` UI tidak sesuai kode internal yang divalidasi backend; pemetaan dapat gagal dari browser. Tampilan juga perlu membedakan status subject Keycloak dan email SSO secara jujur. | ⚠️ Validasi lima role internal dan unique Keycloak ID sudah ada. Namun field `keycloak_id` hanya diperlakukan sebagai subject stabil, bukan resolusi email Keycloak sebagaimana acceptance criteria; `authorize()` request selalu `true` dan route hanya mengandalkan role gate. | P1 |
| 13 | **Role & Permission** (`rbac`) | ⚠️ Matrix permission tersimpan dan UX mencegah self-lockout. Namun pesan/audit halaman memberi kesan audit resmi padahal bukti audit hanya session. | ⚠️ `sync()` permission nyata, tetapi mutation memakai `Request` langsung dan menulis `dynamic_audit_logs` session, bukan `audit_logs` immutable. | P1 |
| 14 | **Data Master** (`data-master`) | ❌ Tab, tabel, dan modal masih array Blade statis; tombol tambah/edit/hapus tidak memiliki flow penyimpanan nyata. Pengelolaan hierarchy unit, lifecycle jabatan, BUP override, dan channel notifikasi belum ada. | ❌ Skema/seed reference table tersedia, tetapi tidak ada route controller/action/FormRequest CRUD untuk master, validasi master yang masih dipakai, permission granular, maupun audit resmi. | P1 |
| 15 | **Hari Libur** (`hari-libur`) | ❌ Halaman menampilkan data statis dan memberi keberhasilan palsu setelah form dikirim. | ❌ Controller web hanya memodifikasi `dynamic_audit_logs` session, tidak `ref_hari_libur`. API Hari Libur sebenarnya sudah benar tetapi belum digunakan oleh halaman ini. | P1 |
| 16 | **Pengaturan Sistem** (`pengaturan`) | ❌ Data instansi, SMTP, mapping, dan ringkasan master bersifat statis; form mengklaim berhasil disimpan padahal tidak ada perubahan nyata. | ❌ `SettingsController::update()` tidak membaca atau menyimpan input, hanya menulis pseudo-audit session. Storage/kontrak setting Fase 1 juga belum diputuskan. | P1 |
| 17 | **Audit Log** (`audit-log`) | ⚠️ Data audit database tampil, tetapi semua record dikirim ke browser lalu difilter/dipaginasi oleh Alpine. | ❌ Audit tidak sepenuhnya immutable/andal: error tulis audit ditelan, beberapa flow memakai audit session, `AuditLog` belum menolak update/delete secara eksplisit, dan payload audit pegawai belum dimasking. | P0 untuk masking/audit; P2 pagination |

### 19.6 Halaman profil yang selalu tersedia di navigasi pengguna

Halaman Profil Saya bukan menu administrasi utama, tetapi dapat diakses ketika Super Admin aktif sehingga tetap dicatat untuk kelengkapan audit.

| Urut | Halaman | Frontend yang belum sesuai | Backend yang belum sesuai | Prioritas |
|:---:|---|---|---|:---:|
| 18 | **Profil Saya** (`profil`) | ⚠️ Data profil nyata. Namun form ubah password memberi kesan password SSO Keycloak dapat diubah dari SIMPEG. | ⚠️ Mutation hanya mengubah password lokal dan belum tercatat sebagai audit mutation. Alur ini perlu dihapus/disembunyikan atau diarahkan ke manajemen akun Keycloak sesuai keputusan produk. | P2 |

### 19.7 Cara membaca prioritas per halaman

| Prioritas | Arti untuk pengerjaan |
|:---:|---|
| **P0** | Tidak boleh menunggu perbaikan kosmetik: risiko data sensitif atau audit wajib belum terpenuhi. Perbaiki dan tambah test regresi sebelum penerimaan fitur terkait. |
| **P1** | Flow utama Super Admin tidak nyata, tidak konsisten dengan PRD, atau menghasilkan outcome yang salah. Selesaikan sebelum halaman dinyatakan selesai Fase 1. |
| **P2** | Fondasi berfungsi, tetapi struktur arsitektur, pertahanan berlapis, payload, atau test perlu dirapikan agar aman dipelihara. |
| **—** | Tidak ada ketidaksesuaian material yang ditemukan dalam audit saat ini; tetap lakukan regression test bila flow diubah. |
