# Audit Frontend dan Informasi Role Kepala Bagian — SIMPEG Fase 1

| Field | Nilai |
|---|---|
| Tanggal audit | 21 Juli 2026 |
| Ruang lingkup | Struktur frontend, navigasi, data yang ditampilkan, scope akses, dan alur cuti pada role `kepala_bagian` (Kabag) |
| Status keseluruhan | ⚠️ Sebagian besar sesuai, tetapi belum dapat dinyatakan sepenuhnya memenuhi acceptance criteria dashboard Kabag |
| Metode | Inspeksi PRD, user story, panduan kode, route, controller, Action/Service, FormRequest, Blade view, dan test feature terfokus |
| Perubahan kode/dokumen produk | Tidak ada. Dokumen ini merekam hasil audit, bukan mengubah requirement produk. |

## Ringkasan Eksekutif

Role Kepala Bagian sudah memiliki fondasi frontend yang konsisten dengan design system Super Admin: memakai layout bersama, card, tabel, badge, breadcrumb, tombol, pola responsif, serta navigasi berbasis role. Namun konsistensi yang benar bukan berarti Kabag harus memiliki halaman dan data yang sama dengan Super Admin. PRD menetapkan Super Admin melihat seluruh data, sedangkan Kabag hanya melihat bawahan langsung.

Pada sisi keamanan dan data scope, implementasi Kabag sudah baik. Daftar bawahan, pengajuan cuti, EWS, detail bawahan, detail cuti, serta lampiran cuti dibatasi di server ke bawahan langsung. Keputusan cuti juga tidak hanya dilindungi melalui tampilan Blade: service approval memastikan bahwa aktor memang approver pada step aktif.

Masih terdapat dua gap acceptance criteria yang harus ditutup agar dashboard Kabag dapat dianggap selesai sesuai user story:

1. Dashboard belum menyediakan quick action keputusan cuti dengan label resmi.
2. Status bawahan `Dinas Luar` belum tersedia pada data maupun tampilan.

Selain itu, ditemukan tiga masalah kualitas frontend: kontrol pagination/jumlah data yang tidak konsisten, halaman cuti yang lebih berorientasi riwayat daripada antrean tindakan, serta beberapa perbaikan minor untuk UX dan bahasa antarmuka.

## Acuan Audit

| Sumber | Ketentuan yang dipakai |
|---|---|
| `PRD-SIMPEG-Fase1-Core.md` bagian Kepala Bagian | Kabag menjadi approver tahap 1 untuk bawahan langsung, membaca data bawahan langsung, dan menerima notifikasi pengajuan cuti bawahan. |
| `PRD-SIMPEG-Fase1-Core.md` Modul Dashboard | Dashboard berbeda sesuai role; dashboard Kabag hanya memuat data bawahan langsung. |
| `User-Stories-SIMPEG-Fase1.md` US-4.4 | Daftar/detil pengajuan cuti, empat label keputusan resmi, catatan wajib, audit, notifikasi, scope bawahan langsung, dan skip approver duplikat. |
| `User-Stories-SIMPEG-Fase1.md` US-8.3 | Dashboard Kabag menampilkan daftar bawahan dengan status aktif/cuti/dinas luar, pengajuan pending dengan quick action, EWS bawahan, dan detail bawahan read-only. |
| `Panduan-Penulisan-Kode-SIMPEG.md` | Reuse Blade component, frontend tidak menjadi sumber keputusan akses, responsif, aksesibilitas, dan filtering/pagination dilakukan melalui data siap tampil dari backend. |

## Status Kesesuaian

| Area | Status | Hasil audit |
|---|:---:|---|
| Layout, sidebar, card, tabel, breadcrumb, dan gaya UI | ✅ | Halaman Kabag memakai layout dan komponen UI bersama yang juga dipakai role lain. |
| Menu sesuai role | ✅ | Menu Kabag hanya memuat Dashboard, Daftar Bawahan, Cuti Bawahan, Pengajuan Cuti pribadi, EWS Bawahan, dan Notifikasi. |
| Pembatasan data bawahan langsung | ✅ | Query, halaman detail, lampiran, EWS, dan keputusan cuti memakai scope server-side. |
| Detail bawahan read-only | ✅ | Tidak ada kontrol edit atau hapus data bawahan pada halaman Kabag. |
| Detail dan keputusan cuti | ✅ | Informasi lengkap, label keputusan resmi, konfirmasi setuju, catatan wajib, timeline, audit, notifikasi, dan validasi approver aktif tersedia. |
| Quick action cuti di dashboard | ❌ | Widget pending hanya membawa pengguna ke antrian/detail, belum memiliki tindakan cepat. |
| Status `Aktif/Cuti/Dinas Luar` | ❌ | Dashboard dan filter baru mendukung Aktif dan Cuti; `Dinas Luar` belum direpresentasikan dalam kode. |
| Pagination/jumlah data | ⚠️ | Beberapa pilihan jumlah data ditampilkan tetapi tidak diterapkan konsisten oleh backend. |
| Fokus antrian tindakan cuti | ⚠️ | Halaman Cuti Bawahan menampilkan seluruh riwayat, bukan antrean tindakan Kabag sebagai fokus/default. |

## Hal yang Sudah Sesuai

### 1. Struktur frontend dan navigasi

Layout `resources/views/components/layouts/app.blade.php` memiliki menu khusus Kabag:

```text
Kepala Bagian
├── Dashboard
├── Daftar Bawahan
├── Cuti Bawahan
├── Pengajuan Cuti
├── EWS Bawahan
└── Notifikasi
```

Struktur tersebut sudah tepat karena Kabag tidak memperoleh menu administrasi global, konfigurasi sistem, laporan global, atau konfigurasi approval yang merupakan kewenangan Super Admin. Halaman Kabag menggunakan `<x-layouts.app>`, `x-ui.card`, `x-ui.table`, `x-ui.stat-card`, `x-ui.badge`, `x-ui.button`, dan breadcrumb sehingga pola visualnya sejalan dengan Super Admin tanpa menyamakan scope data.

Kode Blade juga menunjukkan pola responsif yang konsisten:

- grid dimulai dari `grid-cols-1` dan meningkat pada breakpoint `sm`, `md`, atau `lg`;
- tabel memakai `overflow-x-auto` pada layar sempit;
- header halaman menggunakan `flex-col` lalu beralih ke `sm:flex-row`;
- tombol ikon umumnya memiliki tooltip, `title`, atau `aria-label`;
- foto bawahan memiliki teks alternatif.

### 2. Scope bawahan langsung dan route gate

Route Kabag berada di bawah middleware `role:kepala_bagian`, prefix `/kepala-bagian`, dan binding UUID untuk detail bawahan maupun cuti. Controller juga menolak akun Kabag yang belum tertaut ke record pegawai.

`app/Services/Employees/KepalaBagianScopeService.php` menjadi sumber pembatasan data bawahan. Service tersebut dipakai oleh Action dashboard, daftar bawahan, daftar cuti, EWS, dan detail bawahan. Akses detail bawahan, detail cuti, serta unduh lampiran kembali diperiksa di backend; user tidak cukup hanya mengetahui URL untuk membuka data pegawai di luar scope.

Hal ini sesuai dengan PRD bahwa mapping kepala bagian menentukan siapa yang dapat melihat pegawai sebagai bawahan langsung.

### 3. Halaman bawahan bersifat read-only

`resources/views/kabag/bawahan/show.blade.php` menampilkan informasi pegawai seperti:

- foto, nama, NIP, dan jabatan;
- status kepegawaian;
- unit kerja, golongan, jabatan terakhir, dan jenis pegawai;
- riwayat cuti ringkas;
- EWS aktif yang relevan.

Tidak ada form mutasi data, tombol edit, maupun tombol hapus. Halaman ini konsisten dengan hak akses read-only data bawahan langsung.

### 4. Alur keputusan cuti

Halaman `resources/views/kabag/cuti/show.blade.php` memenuhi kebutuhan US-4.4:

- menampilkan nama, NIP, jabatan, golongan, jenis cuti, tanggal, jumlah hari kerja, alasan, dan lampiran;
- memakai label keputusan resmi `Disetujui`, `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui`;
- tidak memakai tombol formal `Tolak`;
- menampilkan konfirmasi sebelum keputusan `Disetujui` dikirim;
- mewajibkan catatan minimal lima karakter pada keputusan selain `Disetujui`;
- menampilkan timeline approval, alasan tindakan, waktu tindakan, dan step yang dilewati;
- hanya menampilkan form keputusan jika Kabag adalah approver step aktif.

`KepalaBagianLeaveDecisionRequest` memvalidasi empat nilai keputusan dan kewajiban catatan. `LeaveApprovalService` juga memeriksa approver snapshot pada step aktif dan menolak actor yang tidak berwenang. Karena itu, menyembunyikan tombol di Blade tidak menjadi satu-satunya lapisan authorization.

Service approval menangani skip approver duplikat, menulis approval/audit, dan mengirim notifikasi kepada approver berikutnya atau pemohon sesuai hasil keputusan.

## Temuan yang Belum Sesuai

### P1 — Quick action keputusan cuti belum ada di dashboard

**Requirement:** US-8.3 AC-2 meminta daftar pengajuan cuti pending yang menunggu tindakan Kabag dengan quick action sesuai label resmi keputusan cuti.

**Kondisi saat ini:** Widget `Pengajuan Cuti Bawahan` pada `resources/views/kabag/dashboard.blade.php` hanya menyediakan tautan `Lihat Antrean` dan tautan menuju detail cuti. Tidak ada tombol atau menu tindakan `Disetujui`, `Perubahan`, `Ditangguhkan`, atau `Tidak Disetujui` pada dashboard.

**Dampak:** Proses tetap dapat diselesaikan melalui detail cuti, tetapi acceptance criteria dashboard belum terpenuhi dan Kabag membutuhkan satu langkah tambahan untuk setiap pengajuan.

**Rekomendasi:** Tambahkan quick action pada widget dashboard dengan tetap memakai endpoint dan workflow yang sudah ada. `Disetujui` harus tetap memakai konfirmasi; `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui` harus menyediakan modal/drawer catatan wajib. Jangan membuat endpoint atau alur keputusan terpisah.

### P1 — Status `Dinas Luar` belum tersedia

**Requirement:** US-8.3 AC-1 meminta daftar bawahan dengan status `aktif`, `cuti`, atau `dinas luar`.

**Kondisi saat ini:**

- Dashboard hanya merender badge `Cuti` bila bawahan sedang memiliki cuti disetujui; selain itu merender `Aktif`.
- Filter daftar bawahan hanya menawarkan `Aktif` dan `Cuti`.
- Pencarian repository tidak menemukan istilah `Dinas Luar`, `dinas luar`, atau `dinas_luar` pada kode aplikasi, migration, seeder, maupun test.

**Dampak:** Requirement tampilan status pada dashboard belum lengkap; pengguna tidak dapat membedakan pegawai yang sedang dinas luar dari pegawai aktif biasa.

**Keputusan yang dibutuhkan sebelum implementasi:** Dokumen belum menetapkan sumber data `Dinas Luar` secara rinci. Harus diputuskan apakah ia merupakan:

1. nilai pada reference status pegawai;
2. status operasional yang terpisah dari status kepegawaian; atau
3. data yang berasal dari modul penugasan/dinas luar.

Keputusan tersebut perlu dicatat di dokumen produk sebelum menambah enum, reference table, atau logika tampilan baru.

### P2 — Kontrol jumlah data dan pagination tidak konsisten

| Halaman | Kondisi UI | Kondisi backend | Dampak |
|---|---|---|---|
| Daftar Bawahan | Menawarkan 10, 25, 50, dan 100 data | FormRequest menerima 100, tetapi `ListKepalaBagianEmployeesAction` hanya menerima 10, 25, 50 dan mengubah nilai 100 menjadi 10 | Pilihan 100 tampak aktif tetapi hasilnya tidak sesuai ekspektasi. |
| Cuti Bawahan | Menawarkan 10, 25, 50, dan 100 data | `ListKepalaBagianLeavesAction` selalu memakai `paginate(20)` | Semua pilihan jumlah data di UI tidak berpengaruh. |
| EWS Bawahan | Menampilkan select jumlah data dan area pagination | Data dikirim sebagai array tanpa paginator; select tidak terhubung ke request dan state `totalPages` selalu 1 | Kontrol merupakan UI kosmetik/nonfungsional. |

**Rekomendasi:** Normalisasi kontrak `per_page` end-to-end: FormRequest, Action, paginator, view, dan test harus memakai daftar nilai yang sama. Bila pagination belum didukung untuk EWS, hilangkan kontrol jumlah data dan pagination hingga backend mendukungnya.

### P2 — Halaman Cuti Bawahan lebih berfungsi sebagai riwayat daripada antrean tindakan

**Requirement:** US-4.4 AC-1 menekankan daftar pengajuan yang menunggu tindakan Kabag.

**Kondisi saat ini:** `resources/views/kabag/cuti/index.blade.php` menjelaskan bahwa halaman memuat seluruh riwayat dan pengajuan cuti bawahan langsung. Action `ListKepalaBagianLeavesAction` memang membatasi data ke bawahan langsung, tetapi tidak membatasi default data pada step aktif yang menjadi tindakan Kabag.

**Catatan:** Ini bukan kebocoran data dan histori bawahan tetap berguna. Dashboard juga telah memiliki query pending yang benar. Masalahnya adalah fokus halaman: permohonan yang selesai atau sedang ditangani approver lain bercampur dengan permohonan yang memerlukan tindakan Kabag.

**Rekomendasi:** Jadikan default halaman sebagai `Menunggu Tindakan Saya`, lalu sediakan tab/filter eksplisit `Seluruh Riwayat Bawahan`. Alternatif lain adalah memisahkan halaman antrean dan halaman riwayat.

### P3 — Perbaikan kecil UX dan bahasa

- Subtitle Daftar Bawahan memakai frasa `Kelola dan pantau`, padahal role ini tidak mempunyai aksi kelola data. Frasa yang lebih akurat adalah `Pantau seluruh pegawai bawahan langsung Anda`.
- Opsi filter unit kerja dan jenis pegawai diambil dari seluruh reference table, bukan hanya opsi yang ada pada bawahan Kabag. Ini tidak membocorkan data pegawai, tetapi dapat menghasilkan banyak filter dengan hasil nol.
- Ada atribut aksesibilitas berbahasa Inggris seperti `Toggle Detail` pada halaman EWS. Karena UI SIMPEG menggunakan Bahasa Indonesia, sebaiknya memakai `Tampilkan/Sembunyikan Detail`.

## Audit per Halaman

| Halaman | Informasi/aksi yang sudah tersedia | Status |
|---|---|:---:|
| Dashboard Kabag | KPI bawahan aktif, cuti menunggu tindakan, bawahan sedang cuti, daftar lima bawahan, EWS bawahan, dan tautan detail | ⚠️ Quick action cuti serta status Dinas Luar belum ada |
| Daftar Bawahan | Pencarian, filter golongan/unit/jenis/status, tabel bawahan, status, detail read-only, dan pagination | ⚠️ Pilihan pagination 100 tidak konsisten; status Dinas Luar belum ada |
| Detail Bawahan | Profil ringkas, informasi kepegawaian, riwayat cuti, dan EWS aktif | ✅ Read-only dan scope sesuai |
| Cuti Bawahan | Filter, seluruh daftar cuti bawahan, status badge, serta tautan detail | ⚠️ Default bukan antrean tindakan; pilihan per-page tidak berfungsi |
| Detail Cuti | Data pemohon, lampiran, keputusan resmi, catatan wajib, modal konfirmasi, dan timeline | ✅ Sesuai |
| EWS Bawahan | Filter event/status, target tanggal, sisa hari, status tindak lanjut, eligibility, dan detail bawahan | ⚠️ Pagination/jumlah data hanya kosmetik |

## Perbandingan dengan Super Admin

### Konsisten secara struktur

Kabag sudah konsisten dengan Super Admin pada:

- layout utama dan sidebar;
- pattern welcome banner dan KPI card;
- penggunaan komponen card, table, badge, button, filter bar, tooltip, dan breadcrumb;
- pola responsive desktop, tablet, dan mobile;
- gaya Bahasa Indonesia pada konten utama.

### Berbeda secara data dan kewenangan secara sengaja

Kabag tidak boleh disamakan dengan Super Admin pada data atau menu. Perbedaan berikut adalah benar dan wajib dipertahankan:

| Aspek | Super Admin | Kepala Bagian |
|---|---|---|
| Scope data | Seluruh pegawai | Bawahan langsung saja |
| Data pegawai | Kelola/admin global | Baca detail bawahan secara read-only |
| Cuti | Konfigurasi dan pengawasan global sesuai permission | Approval tahap 1 untuk bawahan langsung dan pengajuan cuti pribadi |
| EWS | Global/konfigurasi sesuai permission | EWS bawahan langsung |
| Konfigurasi sistem | Tersedia | Tidak tersedia |

### Catatan terhadap dashboard Super Admin

Dashboard Super Admin masih memiliki angka statis dan detail yang menampilkan pesan data dummy. Karena itu, Super Admin tidak boleh dijadikan patokan untuk akurasi informasi. Kabag sudah lebih tepat dalam satu aspek penting: `BuildKepalaBagianDashboardAction` mengambil data bawahan, cuti pending, dan EWS dari query nyata yang sudah dibatasi scope.

## Bukti Verifikasi

Perintah yang dijalankan:

```text
php artisan test tests/Feature/KepalaBagianFrontendTest.php tests/Feature/KepalaBagianRouteGateTest.php --compact
```

Hasil:

```text
PASS  Tests\Feature\KepalaBagianFrontendTest
PASS  Tests\Feature\KepalaBagianRouteGateTest

Tests: 7 passed (55 assertions)
Duration: 1.82s
```

Test tersebut membuktikan:

- redirect Kabag menuju dashboard khusus;
- navigasi menampilkan Cuti Bawahan dan Pengajuan Cuti pribadi;
- daftar dan detail bawahan dibatasi ke bawahan langsung;
- route detail bawahan di luar scope ditolak;
- daftar/detail cuti memakai data scoped;
- catatan keputusan wajib untuk keputusan yang membutuhkannya;
- keputusan menulis step, approval, dan audit;
- EWS hanya menampilkan alert bawahan langsung;
- role selain Kabag tidak dapat membuka URL Kabag secara langsung.

`phpunit.xml` lokal menetapkan SQLite in-memory. Oleh karena itu, hasil ini adalah bukti fungsional terfokus, bukan bukti khusus kompatibilitas PostgreSQL 17. Tidak ada perubahan database atau kode aplikasi yang dilakukan dalam audit ini.

## Prioritas Tindak Lanjut

1. **P1 — Tambahkan quick action cuti pada dashboard Kabag** dengan reuse workflow keputusan yang sudah tersedia.
2. **P1 — Tetapkan keputusan produk untuk sumber status Dinas Luar**, catat pada dokumen sumber, kemudian implementasikan model/reference data, query, tampilan, dan test.
3. **P2 — Selaraskan pagination/jumlah data** pada Daftar Bawahan, Cuti Bawahan, dan EWS Bawahan.
4. **P2 — Tegaskan pemisahan antrean tindakan dan riwayat cuti** agar fokus tugas Kabag sesuai US-4.4.
5. **P3 — Rapikan copy, opsi filter, dan bahasa aksesibilitas** untuk meningkatkan kejelasan antarmuka.

## Kesimpulan

Frontend Kabag saat ini sudah aman secara scope, konsisten secara visual, dan memiliki workflow keputusan cuti yang baik. Halaman detail bawahan dan detail cuti dapat dinyatakan sesuai secara substansial. Akan tetapi, dashboard Kabag belum lengkap terhadap US-8.3 karena belum mempunyai quick action keputusan dan belum menampilkan status Dinas Luar. Perbaikan pagination serta pemisahan antrean versus riwayat akan membuat pengalaman pengguna lebih konsisten dan siap dijadikan evidence completion untuk vertical slice dashboard Kabag.
