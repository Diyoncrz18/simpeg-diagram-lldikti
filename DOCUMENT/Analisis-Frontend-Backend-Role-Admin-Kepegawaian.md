# Analisis Frontend dan Backend Role Admin Kepegawaian

> Status: audit read-only — **belum sepenuhnya sesuai** dengan kebutuhan Fase 1.
>
> Tanggal audit: 21 Juli 2026.
>
> Cakupan: seluruh halaman yang terlihat atau dapat diakses oleh role **Admin Kepegawaian**, halaman pendukung di dalam alurnya, Blade SSR, rute web/API, middleware/RBAC, controller, Action, FormRequest, layanan, audit, notifikasi, dan pengujian terarah. Dokumen ini mencatat kondisi implementasi saat diaudit dan tidak mengubah kebutuhan produk.

## 1. Legenda Status

| Ikon | Arti |
|:---:|---|
| ✅ | Sudah tersedia dengan data nyata dan secara umum sesuai requirement/pola kode yang berlaku. |
| ⚠️ | Sudah tersedia atau sebagian besar berfungsi, tetapi masih memiliki gap requirement, batasan hak akses yang belum konsisten, kekurangan arsitektur, keamanan, atau test. |
| ❌ | Belum tersedia, tidak dapat diselesaikan dari UI, memakai source data/kontrak yang salah, atau belum memenuhi requirement inti. |

Prioritas pada tabel menggunakan arti berikut.

| Prioritas | Arti |
|:---:|---|
| P0 | Risiko keamanan, data sensitif, atau kewajiban audit. Harus diperbaiki sebelum flow terkait dinyatakan diterima. |
| P1 | Requirement Fase 1 atau alur utama role tidak terpenuhi. |
| P2 | Alur utama telah ada, tetapi struktur code, pertahanan berlapis, konsistensi kontrak, atau bukti test perlu dituntaskan. |

## 2. Acuan dan Batas Peran

Urutan source of truth yang dipakai dalam audit ini adalah:

1. [PRD-SIMPEG-Fase1-Core.md](PRD-SIMPEG-Fase1-Core.md) versi 1.2 — sumber utama keputusan produk.
2. [Panduan-Penulisan-Kode-SIMPEG.md](Panduan-Penulisan-Kode-SIMPEG.md) — standar arsitektur, keamanan, audit, dan QA.
3. [User-Stories-SIMPEG-Fase1.md](User-Stories-SIMPEG-Fase1.md) — acceptance criteria dan dependensi.

Menurut matriks role PRD, Admin Kepegawaian harus dapat melakukan hal berikut:

| Area | Kewajiban Admin Kepegawaian menurut PRD |
|---|---|
| Dashboard | Melihat dashboard admin dengan data seluruh pegawai yang berada dalam cakupan administrasi. |
| Data pegawai | CRUD data pegawai, soft delete dan restore, riwayat append-only, import Excel/CSV, serta menetapkan atasan/Kepala Bagian per pegawai. |
| Dokumen | Mengelola dokumen dan SK pegawai dengan validasi berkas serta akses yang terotorisasi. |
| Cuti | Melihat seluruh pengajuan, mengelola saldo awal/koreksi saldo, dan mengunggah dokumen eksternal cuti Kepala Lembaga. Role ini **bukan** approver otomatis. |
| EWS | Melihat semua alert, melakukan follow-up, memperbarui flag kinerja yang diperlukan, serta menerima notifikasi EWS. |
| Laporan | Menjalankan laporan/export yang ditentukan PRD, termasuk nominatif, custom Excel aman, rekap cuti, dan riwayat kenaikan pangkat. |
| Audit | Membaca audit log resmi yang immutable sesuai hak akses. |

Halaman konfigurasi sistem berikut **bukan** scope normal Admin Kepegawaian menurut matriks role PRD dan memang seharusnya tetap Super Admin-only: Kelola Akses User, Role & Permission, Data Master, Hari Libur, Konfigurasi EWS, Pengaturan Sistem, dan pengelolaan backup khusus Super Admin. Implementasi sidebar dan route untuk pembatasan tersebut merupakan hal yang benar.

> Catatan konflik dokumen: bagian konfigurasi approval chain pada User Stories memberi penugasan yang dapat terbaca sebagai tugas Admin Kepegawaian, sedangkan matriks role PRD yang lebih utama menetapkannya sebagai konfigurasi Super Admin. Aplikasi saat ini membatasi konfigurasi tersebut ke Super Admin. Konflik ini perlu diselaraskan di dokumen sumber; audit ini mengikuti PRD dan tidak menyarankan membuka akses tanpa keputusan produk.

## 3. Peta Halaman yang Diaudit

Sidebar role `admin_kepegawaian` menyembunyikan menu privilese Super Admin. Dari UI, Admin Kepegawaian melihat halaman berikut.

| Urut | Grup | Halaman / route utama | Status akses saat audit |
|:---:|---|---|:---:|
| 1 | Dashboard | `dashboard` | ✅ |
| 2 | Kepegawaian | `data-pegawai`, tambah, edit, detail, riwayat, import | ✅ |
| 3 | Kepegawaian | `dokumen` | ✅ |
| 4 | Laporan | `laporan.export-pegawai` dan export Excel/custom | ✅ |
| 5 | Cuti | `cuti` | ✅ |
| 6 | Cuti | `cuti.rekap` dan saldo/ledger | ✅ |
| 7 | Cuti | `cuti.laporan` | ✅ |
| 8 | EWS | `notifications.index` | ✅ |
| 9 | EWS | `ews` | ✅ |
| 10 | Administrasi | `audit-log` | ✅ |
| 11 | Pendukung | `profil` | ✅ |

Selain halaman tersebut, terdapat route pendukung yang wajib dicatat karena merupakan bagian dari kewajiban role:

| Halaman/flow pendukung | Kondisi akses | Dampak audit |
|---|---|---|
| Pegawai nonaktif dan restore (`data-nonaktif`, `pegawai.restore`) | Route restore tunggal tersedia untuk Admin Kepegawaian, tetapi halaman tidak muncul di sidebar dan tidak ada tautan dari daftar pegawai. | Alur restore tidak dapat ditemukan/dituntaskan dari UI normal. |
| Nonaktifkan pegawai (`pegawai.destroy`) | Backend web mengizinkan Admin Kepegawaian dengan permission `employees.deactivate`. | Tombol antarmuka hanya ditampilkan untuk Super Admin; web dan API juga tidak konsisten. |
| Dokumen eksternal cuti Kepala Lembaga | Tersedia untuk Admin Kepegawaian dengan permission khusus. | Flow mendukung requirement cuti dan perlu dipertahankan. |
| Laporan riwayat kenaikan pangkat | Tidak tersedia di menu atau route Admin Kepegawaian. | Requirement laporan L3 untuk Admin belum terpenuhi. |

## 4. Ringkasan Eksekutif

Fondasi Admin Kepegawaian sudah cukup baik untuk pekerjaan operasional harian: daftar/tambah/edit/detail pegawai, import, dokumen inti, monitoring cuti, koreksi saldo, dokumen eksternal Kepala Lembaga, EWS aktif, notifikasi in-app, dan pembacaan audit log semuanya menggunakan data nyata serta mempunyai guard role/permission.

Namun, halaman ini belum dapat dinyatakan selesai Fase 1 karena ada empat gap utama.

1. **Penetapan atasan/Kepala Bagian** hanya dapat dilakukan Super Admin, padahal PRD menjadikannya tugas Admin Kepegawaian.
2. **Soft delete dan restore pegawai** tersedia sebagian di backend tetapi disembunyikan dari Admin Kepegawaian di UI; halaman pegawai nonaktif tidak ada di sidebar.
3. **Laporan pegawai** belum memenuhi kontrak L1/L1b/L3: PDF nominatif dan laporan riwayat kenaikan pangkat tidak tersedia untuk Admin, preview mengirim data kontak pribadi, dan modal custom Excel tidak dapat dipakai.
4. **Audit perubahan pegawai berisiko menyimpan NIK/No. KK plaintext** dan audit belum memiliki jaminan immutability/fail-safe yang memadai. Ini adalah temuan P0 lintas halaman.

## 5. Status Seluruh Halaman dan Flow

| Kelompok | Halaman/flow | Status | Ringkasan kondisi |
|---|---|:---:|---|
| Dashboard | Dashboard Admin | ❌ | Hanya panel EWS memakai data nyata; widget utama lainnya masih data dummy di Blade. |
| Kepegawaian | Daftar Pegawai | ⚠️ | Data, filter, dan pagination nyata; Admin tidak memperoleh UI untuk menonaktifkan pegawai walau backend web mengizinkan. |
| Kepegawaian | Tambah/Edit Pegawai | ⚠️ | Form dan reference data nyata; audit update dapat merekam data sensitif plaintext. |
| Kepegawaian | Detail, riwayat, dan atasan | ❌ | Riwayat nyata, tetapi Admin tidak dapat menetapkan atasan/Kepala Bagian. |
| Kepegawaian | Pegawai Nonaktif/restore | ❌ | Route tersedia, namun tidak ada menu/tautan untuk Admin dan bulk restore dibatasi Super Admin. |
| Kepegawaian | Import Pegawai | ⚠️ | Alur nyata dan diuji; sebagian mutation masih memakai `Request` langsung di controller. |
| Kepegawaian | Dokumen & SK | ⚠️ | Upload/edit/detail/download nyata; delete/check-impact dibatasi Super Admin sehingga cakupan “kelola” perlu keputusan eksplisit. |
| Laporan | Export Pegawai | ❌ | Preview tidak kanonis dan mengekspos kontak pribadi; custom UI rusak; PDF nominatif belum ada. |
| Laporan | Riwayat Kenaikan Pangkat | ❌ | Tidak ada halaman/route laporan L3 untuk Admin Kepegawaian. |
| Cuti | Monitoring Pengajuan Cuti | ✅ | Query/action, scope `cuti.read_all`, status resmi, dan guard approver berjalan. |
| Cuti | Rekap Saldo dan Ledger | ✅ | Data nyata, filter/pagination, koreksi saldo, dan audit tersedia. |
| Cuti | Dokumen Eksternal Kepala Lembaga | ✅ | Upload dan pengelolaan dokumen tersedia untuk Admin dengan permission khusus. |
| Cuti | Laporan Cuti | ✅ | Preview dan fixed PDF/Excel menggunakan FormRequest dan Action. |
| EWS | EWS Aktif dan follow-up | ✅ | Alert nyata, filter, follow-up, dan akses Admin tersedia. |
| EWS | Flag Kinerja/Satyalancana pegawai | ✅ | Endpoint dan aksi tersedia bagi Admin melalui halaman detail pegawai. |
| EWS | Notifikasi | ⚠️ | Inbox dan ownership baik; email event Satyalancana belum didispatch. |
| Audit | Audit Log | ❌ | Pembacaan log tersedia, tetapi data sensitif, immutability, dan pagination backend belum memenuhi standar. |
| Pendukung | Profil Saya | ⚠️ | Profil nyata; perubahan password lokal tidak konsisten dengan SSO Keycloak dan belum teraudit. |

## 6. Analisis per Halaman

### 6.1 Dashboard Admin

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ❌ | `admin.dashboard` masih mendefinisikan sebagian besar statistik pegawai, distribusi, daftar pegawai, cuti terbaru, audit terbaru, dan hari libur sebagai array dummy. Beberapa tautan detail tidak mengarah ke flow nyata. |
| Backend | ⚠️ | `DashboardController` sudah menggunakan `ListActiveEwsAlertsAction` untuk panel EWS, tetapi belum menyediakan query/view model kanonis untuk widget dashboard lainnya. |
| Kesesuaian PRD | ❌ | PRD meminta dashboard admin berbasis data server-rendered untuk statistik dan widget operasional, bukan data contoh di Blade. |
| Task | ❌ P1 — buat Action/query dashboard yang memasok statistik pegawai, status/distribusi, cuti, daftar pegawai, audit terbaru yang dimasking, dan hari libur dari database. Hapus seluruh data dummy dan tautan `#`. |

### 6.2 Data Pegawai: Daftar, Tambah, dan Edit

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend daftar | ⚠️ | Filter, pencarian, pagination, dan tautan detail menggunakan data nyata. Namun opsi **Ubah Status** dan **Hapus → Backup** dibungkus pengecekan `super_admin` di Blade sehingga tidak tampil bagi Admin Kepegawaian. |
| Backend daftar | ⚠️ | `ListEmployeesAction` dipakai untuk list. Route web deactivate mengizinkan `super_admin,admin_kepegawaian` dengan permission `employees.deactivate`, tetapi route API destroy memasang pembatas Super Admin tambahan. Kontrak web/API tidak konsisten. |
| Frontend tambah/edit | ✅ | Form memakai route bernama, CSRF, `old()`, error validasi, dan pilihan reference dari database. Tidak ditemukan data dummy dalam flow utama. |
| Backend tambah/edit | ⚠️ | `StoreEmployeeRequest`/`CreateEmployeeAction` serta `UpdateEmployeeRequest`/`UpdateEmployeeAction` sudah memisahkan use case utama. Tetapi `UpdateEmployeeAction` mengambil old/new audit melalui `Employee::toArray()`. Karena cast enkripsi didekripsi ketika serialisasi, `nik` dan `no_kk` berisiko masuk ke audit sebagai plaintext. |
| Kesesuaian PRD | ⚠️ | CRUD utama tercapai, tetapi PRD mensyaratkan soft delete bagi Admin dan Panduan Kode melarang pencatatan data sensitif yang tidak perlu di audit. |
| Task | ❌ P0 — buat allowlist/masking audit pegawai sebelum `AuditService::log()`; pastikan NIK/No. KK tidak tersimpan maupun ditampilkan plaintext dan tambahkan test regresi. |
| Task | ❌ P1 — tampilkan aksi nonaktifkan bagi Admin hanya jika permission tersedia, atau ubah policy/route secara konsisten setelah keputusan produk. Selaraskan pembatas web dan API. |
| Task | ⚠️ P2 — pindahkan bulk deactivate/restore dari `Request` langsung di controller ke FormRequest dan Action terpisah dengan audit yang konsisten. |

### 6.3 Detail Pegawai, Riwayat Append-Only, dan Penetapan Atasan

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend detail dan riwayat | ⚠️ | Detail memuat relasi pegawai nyata (riwayat jabatan, pangkat, KGB, pendidikan, keluarga, disiplin, dokumen, dan status terkait). Tampilan menunjukkan data atasan saat ini, tetapi tidak menyediakan form/aksi penetapan atasan untuk Admin. |
| Backend riwayat | ⚠️ | Riwayat menggunakan use case tersendiri dan sifat append-only secara umum telah diterapkan. Namun endpoint tambah riwayat masih menerima `Request` langsung pada controller, sehingga tidak sepenuhnya mengikuti boundary FormRequest → Action. |
| Backend atasan | ❌ | `AssignSupervisorAction` sebenarnya tersedia dan memvalidasi self-supervisor serta membuat audit, tetapi route web/API dibatasi `super_admin` dan test yang ada secara eksplisit menolak Admin Kepegawaian. |
| Kesesuaian PRD | ❌ | PRD menyebut **Set supervisor/Kepala Bagian per pegawai** sebagai tanggung jawab Admin Kepegawaian. Pembatasan saat ini bertentangan langsung dengan matriks tersebut. |
| Task | ❌ P1 — setelah menambahkan policy/otorisasi yang tepat, buka route dan kontrol UI penetapan atasan untuk Admin Kepegawaian yang memiliki `employees.update`; gunakan `AssignSupervisorRequest` (baru) dan pertahankan validasi anti-self/anti-duplikasi serta audit aman. Perbarui test RBAC yang saat ini mengunci perilaku lama. |
| Task | ⚠️ P2 — ubah input tambah riwayat menjadi FormRequest spesifik agar validasi, otorisasi, dan audit berada di boundary yang konsisten. |

### 6.4 Pegawai Nonaktif, Soft Delete, dan Restore

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ❌ | Halaman `data-nonaktif` tidak berada di sidebar Admin Kepegawaian dan tidak ditemukan tautan dari daftar pegawai. Tombol nonaktif juga hanya terlihat bagi Super Admin. Admin tidak mempunyai alur normal untuk melihat, memulihkan, atau menyelesaikan lifecycle pegawai nonaktif. |
| Backend | ⚠️ | Route `data-nonaktif` dan restore tunggal tersedia bagi Admin dengan permission `employees.restore`; restore bulk dibatasi Super Admin. Soft delete tidak memakai permanent delete pada flow ini. |
| Kesesuaian PRD | ❌ | PRD memberikan soft delete **dan restore** kepada Admin Kepegawaian. Route tanpa UI yang dapat ditemukan bukan penyelesaian flow Fase 1. |
| Task | ❌ P1 — tambahkan menu/tautan **Pegawai Nonaktif** untuk Admin Kepegawaian, halaman list `onlyTrashed()` yang terotorisasi, dan aksi restore tunggal sesuai permission. Putuskan secara eksplisit apakah bulk restore memang hanya Super Admin; dokumentasikan alasan bila tetap dibatasi. |
| Task | ⚠️ P2 — tambah feature test HTML/sidebar dan test policy untuk memastikan UI serta route memiliki kontrak hak akses yang sama. |

### 6.5 Import Pegawai

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ✅ | Tahap unggah, preview, validasi, eksekusi batch, status, dan template import tersedia dari halaman Data Pegawai. |
| Backend | ⚠️ | Alur import, validasi data, dan test feature tersedia. Namun tahap validate dan execute masih menggunakan validasi `Request` langsung di controller, bukan FormRequest per use case. |
| Kesesuaian PRD | ⚠️ | Kemampuan import dasar ada, tetapi perlu memastikan transaksi, UUID, file, dan error row diuji pada PostgreSQL 17, bukan SQLite saja. |
| Task | ⚠️ P2 — buat FormRequest untuk validate/execute import dan jalankan regresi pada PostgreSQL 17 untuk file, UUID, foreign key, serta transaksi batch. |

### 6.6 Dokumen & SK Pegawai

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ✅ | Indeks dokumen, upload, edit metadata, detail, download, dan state error menggunakan data nyata. Admin dapat upload/edit/download sesuai permission. |
| Backend | ⚠️ | `DokumenController` memakai Action/FormRequest untuk flow utama, termasuk validasi file. Namun `destroy` dan `check-impact` dibatasi hanya untuk Super Admin. |
| Kesesuaian PRD | ⚠️ | PRD menyebut Admin Kepegawaian “mengelola dokumen dan SK”. Upload/edit/download sudah memenuhi inti tersebut, tetapi kata *mengelola* belum menjelaskan secara eksplisit apakah delete dokumen harus diberikan kepada Admin. Pembatasan lebih ketat dapat valid demi integritas dokumen, namun perlu keputusan agar tidak menjadi gap tersembunyi. |
| Task | ⚠️ P1 — tetapkan dan dokumentasikan matriks delete/check-impact dokumen. Bila Admin memang harus dapat menghapus, gunakan Action yang sudah ada, policy/permission granular, audit immutable, dan konfirmasi dampak; bila Super Admin-only adalah kebijakan final, tulis pengecualian itu di PRD/acceptance criteria dan tampilkan informasi yang jelas di UI. |

### 6.7 Laporan dan Export Pegawai

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend preview standar | ❌ | Preview `/laporan/export-pegawai` dibangun dari closure route dan menerima seluruh data pegawai, termasuk scope yang tidak selaras dengan export aktif. Email pribadi dan nomor HP dikirim ke browser. Ini tidak memenuhi prinsip payload minimum dan perlindungan data sensitif. |
| Frontend custom Excel | ❌ | Modal merujuk state/handler Alpine (`showCustomExportModal`, `customColumns`, `filterOptions`, `submitCustomExport`) yang tidak didefinisikan sehingga pengguna tidak dapat menjalankan custom export dari UI. |
| Backend Excel | ⚠️ | Export Excel standard/custom telah memakai request/action dan allowlist kolom; test menolak kolom sensitif. Namun route preview belum memakai `ExportPegawaiPreviewAction`/query kanonis. Endpoint export pegawai lama juga memiliki kontrak kolom kontak yang berbeda. |
| PDF nominatif L1 | ❌ | PRD L1 dan acceptance criteria meminta laporan nominatif standard **PDF dan Excel**. Untuk Admin saat ini hanya tersedia alur Excel; PDF standard belum tersedia. |
| Laporan L3 riwayat kenaikan pangkat | ❌ | PRD meminta PDF/Excel L3 bagi Admin dan Pimpinan. Route/menu yang tersedia hanya berada pada area Pimpinan, bukan Admin Kepegawaian. |
| Kesesuaian PRD | ❌ | L1, L1b, dan L3 belum selesai secara terpadu, walaupun sebagian backend custom Excel sudah baik. |
| Task | ❌ P1 — pindahkan preview ke controller/action dan gunakan satu query/filter/scope kanonis untuk preview, Excel standard, dan Excel custom. Default hanya pegawai aktif jika tidak ada filter eksplisit; payload browser hanya berisi kolom aman yang diperlukan. |
| Task | ❌ P1 — perbaiki atau ganti modal custom dengan form server-side yang benar; pastikan kolom hanya berasal dari allowlist backend dan seluruh error tampil di UI. |
| Task | ❌ P1 — implementasikan/otorisasikan PDF nominatif fixed-format L1 serta laporan L3 PDF/Excel untuk Admin Kepegawaian. Jangan menambah custom PDF karena berada di luar batas Fase 1. |

### 6.8 Monitoring Cuti, Rekap Saldo, Dokumen Eksternal, dan Laporan Cuti

| Halaman/flow | Frontend | Backend | Status dan kesesuaian |
|---|---|---|:---:|
| Pengajuan/monitoring cuti | Filter, list, detail, dan label keputusan resmi ditampilkan dengan data nyata. | `ListLeaveRequestsAction`, scope role/permission, serta service keputusan menjaga Admin sebagai pemantau, bukan approver otomatis. | ✅ |
| Rekap saldo dan ledger | Rekap, filter, pagination, saldo, dan ledger data nyata. | Query/action koreksi saldo memakai request/action dan audit. | ✅ |
| Dokumen eksternal Kepala Lembaga | Flow upload tersedia sebagai halaman pendukung dan memakai UI/form yang nyata. | Controller/request/action diberi permission `cuti.kepala_lembaga_documents.manage` untuk Super/Admin. | ✅ |
| Laporan cuti | Preview dan pilihan PDF/Excel fixed-format tersedia. | `LaporanCutiController` mengikuti FormRequest → Action → response. | ✅ |
| Konfigurasi approval cuti | Tidak ada di menu Admin; sesuai matriks role PRD utama. | Route Super Admin-only. | ✅* |

`✅*` berarti sesuai PRD utama, dengan catatan konflik dokumentasi pada bagian 2 yang perlu diputuskan oleh pemilik produk.

Tidak ditemukan kebutuhan agar Admin Kepegawaian dapat menyetujui atau mengambil keputusan cuti hanya karena ia dapat melihat semua pengajuan. Mempertahankan penolakan terhadap Admin yang bukan approver merupakan perilaku yang benar.

### 6.9 EWS Aktif, Flag Pegawai, dan Notifikasi

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend EWS Aktif | ✅ | Alert aktual, filter, warna/status, dan follow-up tersedia bagi Admin Kepegawaian. |
| Backend EWS Aktif | ✅ | `ListActiveEwsAlertsAction` dan action follow-up dipakai; role Admin memiliki akses baca/follow-up yang sesuai. |
| Flag kinerja dan Satyalancana | ✅ | Route/action dari detail pegawai mengizinkan Admin memperbarui flag yang dibutuhkan untuk kondisi EWS. Test feature yang relevan tersedia. |
| Inbox notifikasi | ✅ | Paginator backend, tandai dibaca, CSRF, dan ownership per pengguna telah diterapkan. |
| Email Satyalancana | ❌ | `NotificationRecipientResolver` tidak memasukkan event `ews.satyalancana` pada allowlist email, padahal PRD mewajibkan alert Satyalancana disampaikan lewat in-app dan email kepada penerima yang tepat. |
| Task | ❌ P1 — masukkan event Satyalancana ke jalur dispatch email berbasis channel reference, pastikan penerima pegawai dan Admin Kepegawaian sesuai aturan, lalu tambah test notification/queue/mail untuk H-180, H-90, dan H-30. |

### 6.10 Audit Log

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ⚠️ | Data audit berasal dari database dan Admin diizinkan membacanya. Namun halaman memuat seluruh koleksi log ke browser lalu melakukan filter/pagination di Alpine; ini tidak skalabel dan memperluas data yang dikirim ke klien. |
| Backend | ❌ | `AuditService` menelan kegagalan tulis audit; beberapa mutation masih memakai `dynamic_audit_logs` session alih-alih `audit_logs`; model audit belum secara eksplisit menolak update/delete aplikasi; dan audit update pegawai dapat memuat NIK/No. KK plaintext. |
| Kesesuaian PRD/Panduan | ❌ | PRD mewajibkan audit signifikan yang immutable, berguna, dan aman. Panduan melarang audit data sensitif yang tidak diperlukan. Kondisi ini adalah gap P0. |
| Task | ❌ P0 — masking/allowlist audit pegawai; tidak boleh ada NIK atau No. KK plaintext pada `old_values`/`new_values`, response audit, ataupun Blade. Tambah test tulis dan baca. |
| Task | ❌ P0 — tetapkan mekanisme audit yang tidak dapat dimutasi lewat aplikasi, hilangkan audit session sebagai sumber resmi, dan jangan menelan kegagalan audit tanpa kebijakan eksplisit. |
| Task | ⚠️ P2 — gunakan Action/query filter/pagination backend pada halaman Blade serta kirim payload audit minimum yang sudah dimasking. |

### 6.11 Profil Saya

| Aspek | Status | Temuan |
|---|:---:|---|
| Frontend | ⚠️ | Halaman profil dan data pegawai aktual dapat ditampilkan. Form ubah password memberi kesan bahwa password SSO Keycloak dapat dikelola dari SIMPEG. |
| Backend | ⚠️ | Perubahan password hanya memutasi password lokal, tidak Keycloak, dan belum tercatat sebagai audit mutation keamanan. |
| Task | ⚠️ P2 — putuskan apakah password lokal masih memiliki fungsi yang sah. Bila tidak, sembunyikan/hapus form atau arahkan pengguna ke manajemen akun Keycloak; jika dipertahankan, dokumentasikan kontrak dan auditkan perubahan. |

## 7. Task Berdasarkan Status

### 7.1 Sudah Sesuai

| Status | Task/kapabilitas | Bukti kondisi |
|:---:|---|---|
| ✅ | RBAC halaman operasional Admin | Sidebar menyembunyikan konfigurasi Super Admin dan route privilese menolak akses Admin secara langsung. |
| ✅ | Daftar, tambah, edit, dan detail inti pegawai | Data/reference nyata, route bernama, CSRF, validasi, Action utama, filter, serta pagination tersedia. |
| ✅ | Import pegawai | Tahap unggah sampai eksekusi batch dan test feature tersedia. |
| ✅ | Upload/edit/download dokumen inti | FormRequest/Action dan validasi file dipakai pada flow utama. |
| ✅ | Monitoring, saldo, dokumen eksternal, dan laporan cuti | Action/Service/request serta guard keputusan cuti berjalan sesuai pemisahan peran. |
| ✅ | EWS aktif/follow-up dan flag eligibility | Alert dan follow-up nyata; Admin dapat memperbarui flag yang diperlukan. |
| ✅ | Inbox notifikasi | Pagination, tandai dibaca, CSRF, dan ownership tersedia. |

### 7.2 Sudah Dikerjakan tetapi Belum Tuntas

| Status | Task/kapabilitas | Kekurangan yang harus diselesaikan |
|:---:|---|---|
| ⚠️ | CRUD data pegawai | Tombol/status Admin tidak konsisten dengan route, API dan UI; audit data sensitif harus diamankan. |
| ⚠️ | Riwayat pegawai | Data nyata dan append-only tersedia, tetapi mutation belum semuanya memakai FormRequest spesifik. |
| ⚠️ | Dokumen & SK | Tetapkan secara eksplisit apakah delete/check-impact dokumen bagian dari hak Admin atau sengaja Super-only. |
| ⚠️ | Import | Validasi/eksekusi perlu dipindahkan dari `Request` controller dan dibuktikan di PostgreSQL 17. |
| ⚠️ | Export Excel | Backend safe allowlist ada, tetapi preview, scope, modal UI, dan kontrak kolom belum disatukan. |
| ⚠️ | Notifikasi EWS | In-app berfungsi, tetapi email Satyalancana tidak dijalankan. |
| ⚠️ | Audit list | Sumber log database ada, namun pagination/filter harus server-side setelah audit keamanan diperbaiki. |
| ⚠️ | Profil | Data nyata, tetapi password lokal versus Keycloak dan audit mutation belum jelas. |

### 7.3 Belum Sesuai atau Belum Selesai

| Status | Task/kapabilitas | Perbaikan yang diperlukan |
|:---:|---|---|
| ❌ | Dashboard Admin berbasis data nyata | Ganti seluruh widget dummy dengan Action/query dashboard dan tautan detail terotorisasi. |
| ❌ | Penetapan atasan/Kepala Bagian oleh Admin | Buka permission/UI/route sesuai PRD, gunakan FormRequest, dan update test RBAC. |
| ❌ | Soft delete/restore yang dapat dituntaskan dari UI Admin | Tampilkan aksi nonaktif dan halaman Pegawai Nonaktif/restore dalam navigasi yang benar. |
| ❌ | Audit pegawai aman dan immutable | Masking NIK/No. KK, tidak menelan error audit, larang mutation audit, dan hilangkan pseudo-audit session. |
| ❌ | Nominatif L1 PDF/Excel yang kanonis dan aman | Satu source query untuk preview/export, tanpa email/no. HP di browser; tambah PDF fixed-format. |
| ❌ | Custom Excel yang dapat digunakan dari UI | Implementasikan state/handler/modal atau form SSR yang benar dengan allowlist backend. |
| ❌ | Laporan L3 riwayat kenaikan pangkat untuk Admin | Tambahkan route, UI, policy, fixed PDF, dan Excel dengan filter yang ditetapkan PRD. |
| ❌ | Email EWS Satyalancana | Tambahkan event ke dispatcher/channel dan bukti queue/mail test. |

## 8. Prioritas Implementasi yang Disarankan

| Urutan | Prioritas | Task | Kriteria selesai |
|:---:|:---:|---|---|
| 1 | P0 | Amankan audit pegawai dan audit resmi. | Audit tidak pernah menyimpan/menampilkan NIK/No. KK plaintext; audit signifikan persisten, tidak dapat diubah aplikasi, dan kegagalan audit memiliki kebijakan eksplisit serta test regresi. |
| 2 | P1 | Penuhi hak penetapan atasan untuk Admin. | Admin dengan permission tepat dapat menetapkan atasan dari detail pegawai; self-assignment/duplikasi ditolak; audit dan test RBAC/UI lulus. |
| 3 | P1 | Pulihkan lifecycle soft delete/restore di UI Admin. | Menu/list pegawai nonaktif, nonaktifkan, restore, audit, policy, dan route web/API memiliki kontrak yang sama. |
| 4 | P1 | Selesaikan laporan L1/L1b/L3. | Preview, Excel standard/custom, PDF L1, dan L3 menggunakan scope/filter kanonis, payload aman, dan UI yang dapat diselesaikan. |
| 5 | P1 | Lengkapi email EWS Satyalancana. | Event H-180/H-90/H-30 terkirim ke penerima sesuai channel reference; test notification, queue, dan mail lulus. |
| 6 | P2 | Rapikan mutation legacy. | Riwayat, bulk pegawai, assign atasan, serta import memakai FormRequest → Action → Service/model dengan test terarah. |
| 7 | P2 | Validasi PostgreSQL dan UX browser. | Test relevan berhasil pada PostgreSQL 17 dan ada test HTML/browser untuk sidebar Admin, modal export, role action, upload, dan responsive primary flow. |

## 9. Hasil Pengujian pada Audit Ini

Pengujian terarah yang dijalankan dari folder `SIMPEG`:

```powershell
php artisan test --compact `
  tests/Feature/AdminKepegawaianAccessTest.php `
  tests/Feature/SupervisorAssignmentTest.php `
  tests/Feature/EmployeeDeactivateRestoreTest.php `
  tests/Feature/EmployeeImportTest.php `
  tests/Feature/EmployeeDocumentTest.php `
  tests/Feature/EmployeeReportExportTest.php `
  tests/Feature/CutiRbacTest.php `
  tests/Feature/CutiRekapExportTest.php `
  tests/Feature/KepalaLembagaSupportingDocumentTest.php `
  tests/Feature/EwsActivePageTest.php `
  tests/Feature/EmployeeSatyalancanaEligibilityTest.php `
  tests/Feature/NotificationInboxTest.php `
  tests/Feature/AuditLogIndexTest.php `
  tests/Feature/ProfileTest.php
```

Hasil: command selesai tanpa kegagalan pada lingkungan lokal audit.

| Status | Interpretasi |
|:---:|---|
| ✅ | Test membuktikan banyak route/flow existing berjalan: gate Admin, import, dokumen, cuti, EWS, notifikasi, report Excel, audit index, dan profil. |
| ⚠️ | Test dapat mengunci perilaku yang justru belum sesuai PRD. Contoh: test penetapan atasan saat ini mengharapkan Admin ditolak, sehingga hasil lulus bukan bukti kesesuaian terhadap matriks role PRD. |
| ⚠️ | `phpunit.xml` menggunakan SQLite in-memory. Hasil ini bukan bukti PostgreSQL 17 untuk UUID, encrypted cast/audit JSON, FK/index, transaksi, maupun upload. |
| ❌ | Belum ada bukti test browser/end-to-end khusus Admin untuk tombol nonaktif/restore, sidebar Pegawai Nonaktif, form penetapan atasan, custom export modal, PDF L1/L3, dan masking NIK/No. KK di audit. |

## 10. Kesimpulan

Role Admin Kepegawaian telah memiliki fondasi operasional yang nyata dan cukup luas, terutama untuk data pegawai dasar, cuti, dokumen inti, EWS, dan notifikasi. Pembatasan terhadap konfigurasi Super Admin juga pada umumnya konsisten dengan PRD utama.

Namun status keseluruhannya tetap **belum sepenuhnya sesuai Fase 1**. Prioritas pertama bukan penambahan tampilan baru, melainkan menutup risiko P0 audit sensitif dan memastikan hak yang eksplisit diberikan PRD — penetapan atasan serta soft delete/restore — benar-benar dapat dijalankan dari UI Admin yang terotorisasi. Sesudah itu, laporan L1/L1b/L3 dan email EWS Satyalancana perlu dituntaskan dengan source query yang sama, data minimum, dan bukti test PostgreSQL/browser.

Tidak ada kode aplikasi yang diubah dalam audit ini.
