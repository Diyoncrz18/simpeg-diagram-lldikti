# PRD — SIMPEG Fase 1 (Core)
## Sistem Informasi Kepegawaian LLDIKTI Wilayah XVI

| Field | Detail |
|-------|--------|
| **Versi Dokumen** | 1.3 |
| **Tanggal** | 22 Juli 2026 |
| **Domain** | Disiapkan LLDIKTI saat tahap deployment |
| **Fase** | 1 — Core / Fondasi |
| **Target Go-Live** | 20 Agustus 2026 |
| **Tim Pengembang** | Tim Magang (Mahasiswa) dengan Supervisi |
| **Tech Stack** | Laravel 12 · Blade · PostgreSQL 17 · Keycloak SSO |

### Catatan Update Hasil Meeting Teknis

PRD ini menjadi **sumber kebenaran utama** untuk Fase 1. Keputusan meeting teknis terbaru yang sudah dimasukkan:

1. Keycloak digunakan sebagai gerbang autentikasi / SSO; role dan permission tetap dikelola di dalam aplikasi.
2. Pihak LLDIKTI akan menyediakan trait/fungsi Keycloak, akun SSO testing, Client ID, Client Secret, dan URL Keycloak.
3. Server dan domain production disiapkan oleh LLDIKTI ketika sistem mendekati tahap deployment.
4. Database development menggunakan PostgreSQL 17 melalui container.
5. Production diprioritaskan menggunakan Podman karena pertimbangan keamanan.
6. Notifikasi tidak boleh mengunci channel ke in-app/email saja; Fase 1 menyediakan in-app dan email, dengan arsitektur channel-configurable agar WhatsApp Business dapat ditambahkan saat layanan/credential tersedia.
7. Approval cuti tidak boleh hardcoded 3 stage seragam. Rantai verifikasi/approval harus dinamis per pegawai/unit, dapat memuat 1, 2, 3, atau lebih verifikator, dan dapat menunjuk pegawai tertentu seperti Ketua Tim Kerja tanpa membuat role baru.
8. Keputusan cuti mengikuti label resmi pada formulir: `Disetujui`, `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui`. Istilah formal `Ditolak` tidak digunakan pada keputusan cuti.
9. Cuti tahunan tidak boleh lintas tahun kalender; periode Desember–Januari harus dipecah menjadi dua pengajuan terpisah. Kuota tahunan 12 hari dengan carry-over maksimal 6 hari dari N-1, kecuali pegawai tidak mengambil cuti tahunan dua tahun berturut-turut sehingga total tahun berjalan dapat mencapai 24 hari.
10. Pengajuan cuti yang selesai diproses harus dapat menghasilkan formulir cuti resmi dengan QR Code verifikasi SIMPEG. Tanda tangan elektronik tersertifikasi belum wajib di Fase 1.
11. Struktur organisasi menggunakan `ref_unit_kerja` hierarkis (`parent_id`, `level`) untuk memuat Kepala Lembaga, Kepala Bagian Umum, Tim Kerja, urusan/sub-unit, dan kemungkinan level yang lebih dalam.
12. Status pegawai dan jabatan harus berbasis reference table, bukan enum/free text. Status pegawai menyimpan keterangan perubahan; jabatan dipilih dari `ref_jabatan` dan `kelas_jabatan` disimpan di riwayat jabatan.
13. EWS menambahkan Satyalancana 10/20/30 tahun dan setiap alert harus punya status tindak lanjut agar tidak muncul terus setelah ditangani.
14. Laporan menambahkan export nominatif Excel yang customizable: pengguna memilih kolom dan filter baris; PDF custom tidak masuk Fase 1.
15. BUP tidak di-hardcode; usia pensiun dihitung dari referensi jabatan / jenis jabatan.
16. Sample data pegawai, referensi jabatan, pangkat, golongan, struktur unit, saldo cuti awal, dan data mentah lainnya disediakan oleh bagian kepegawaian LLDIKTI.
17. Keputusan sesi langsung pengguna 22 Juli 2026 (kanonis, disetujui pengguna): import massal Fase 1 hanya mengaktifkan template Data Utama. Import membuat record pegawai beserta field snapshot awal (golongan, pangkat, jabatan, kelas jabatan, pendidikan, prodi, dan tanggal pensiun bila tersedia di kolom Excel), tetapi tidak membuat riwayat kepangkatan, riwayat jabatan, maupun riwayat KGB. Riwayat resmi diinput per pegawai melalui CRUD riwayat append-only. Kalkulasi TMT dijalankan saat riwayat/sumber resmi disimpan, bukan saat import selesai. Tanggal pensiun hasil import dipertahankan apa adanya dan tidak dihitung ulang atau ditimpa oleh proses import. Template lanjutan multi-jenis (Data Pelengkap, Riwayat Kepangkatan, Riwayat Jabatan, Riwayat KGB) tidak termasuk ruang lingkup saat ini dan tidak dipulihkan tanpa keputusan eksplisit baru. Keputusan ini menggantikan rincian import versi sebelumnya jika bertentangan.

---

## Daftar Isi

1. [Ringkasan Eksekutif](#1-ringkasan-eksekutif)
2. [Latar Belakang & Permasalahan](#2-latar-belakang--permasalahan)
3. [Tujuan & Ruang Lingkup Fase 1](#3-tujuan--ruang-lingkup-fase-1)
4. [Pengguna & Role (RBAC)](#4-pengguna--role-rbac)
5. [Arsitektur Teknis](#5-arsitektur-teknis)
6. [Modul 1 — Autentikasi & SSO](#6-modul-1--autentikasi--sso)
7. [Modul 2 — Manajemen Data Pegawai](#7-modul-2--manajemen-data-pegawai)
8. [Modul 3 — Import Data Excel/CSV](#8-modul-3--import-data-excelcsv)
9. [Modul 4 — Manajemen Cuti](#9-modul-4--manajemen-cuti)
10. [Modul 5 — Early Warning System (EWS)](#10-modul-5--early-warning-system-ews)
11. [Modul 6 — Notifikasi](#11-modul-6--notifikasi)
12. [Modul 7 — Audit Log](#12-modul-7--audit-log)
13. [Modul 8 — Dashboard](#13-modul-8--dashboard)
14. [Modul 9 — Laporan & Export](#14-modul-9--laporan--export)
15. [Data Model (ERD)](#15-data-model-erd)
16. [Reference Tables (Seed Data)](#16-reference-tables-seed-data)
17. [API Specification](#17-api-specification)
18. [Non-Functional Requirements](#18-non-functional-requirements)
19. [Keamanan & Compliance](#19-keamanan--compliance)
20. [Batasan & Asumsi](#20-batasan--asumsi)
21. [Fase Selanjutnya (Out of Scope)](#21-fase-selanjutnya-out-of-scope)
22. [Regulasi yang Direferensi](#22-regulasi-yang-direferensi)
23. [Glosarium](#23-glosarium)

---

## 1. Ringkasan Eksekutif

SIMPEG adalah Sistem Informasi Kepegawaian yang dikembangkan untuk LLDIKTI Wilayah XVI guna mendigitalisasi pengelolaan data dan layanan kepegawaian yang saat ini masih berbasis manual dan berkas fisik.

**Fase 1 (Core)** berfokus pada fondasi sistem yang harus stabil sebelum fitur lanjutan dikembangkan. Cakupan Fase 1:

- **Autentikasi SSO** via Keycloak yang sudah tersedia di LLDIKTI XVI.
- **Data Pegawai Terpusat** — seluruh data pegawai (~46 orang) tersimpan dalam satu database.
- **Import Excel/CSV** — migrasi data awal dari spreadsheet yang sudah ada.
- **Manajemen Cuti** — pengajuan digital dengan rantai verifikasi/approval dinamis, status keputusan resmi, saldo tahunan, dan formulir cuti ber-QR Code.
- **Early Warning System** — notifikasi otomatis untuk kenaikan pangkat, KGB, pensiun, kontrak PPPK, dan Satyalancana.
- **Notifikasi** — in-app dan email via queue, dengan desain channel-configurable untuk penambahan WA Business.
- **Audit Log** — pencatatan semua perubahan data.
- **Dashboard** — ringkasan data kepegawaian untuk pimpinan dan admin.
- **Laporan & Export** — daftar pegawai, rekap cuti, riwayat kepangkatan, dan export nominatif Excel customizable.

---

## 2. Latar Belakang & Permasalahan

### 2.1 Kondisi Saat Ini

Pengelolaan data kepegawaian di LLDIKTI Wilayah XVI masih dilakukan secara manual:

| Aspek | Kondisi Saat Ini | Dampak |
|-------|-----------------|--------|
| **Data Pegawai** | Tersebar di berbagai spreadsheet dan dokumen manual | Sulit dicari, diperbarui, dan disinkronkan |
| **Kenaikan Pangkat & KGB** | Dihitung manual, tidak ada pengingat otomatis | Sering terlambat diproses |
| **Masa Pensiun** | Dipantau manual | Risiko momen penting terlewat |
| **Pengajuan Cuti** | Berkas fisik (paper-based) | Proses lambat, status sulit dipantau |
| **Administrasi** | Semua proses by-paper | Tidak efisien, rawan kesalahan |

### 2.2 Permasalahan Utama

1. **Data tidak terpusat** — informasi pegawai tersebar, tidak ada single source of truth.
2. **Momen penting sering terlewat** — tidak ada sistem pengingat otomatis untuk kenaikan pangkat, KGB, pensiun.
3. **Proses manual dan lambat** — pengajuan cuti dan administrasi masih paper-based.
4. **Tidak ada audit trail** — perubahan data tidak tercatat, sulit melacak siapa mengubah apa.

---

## 3. Tujuan & Ruang Lingkup Fase 1

### 3.1 Tujuan

1. Menyatukan seluruh data kepegawaian dalam satu sistem digital terpusat.
2. Mengotomasi pengingat momen penting kepegawaian (kenaikan pangkat, KGB, pensiun, kontrak PPPK).
3. Mendigitalisasi proses pengajuan dan persetujuan cuti.
4. Menyediakan audit trail untuk setiap perubahan data.
5. Menyediakan dashboard dan laporan dasar untuk pengambilan keputusan.

### 3.2 Ruang Lingkup Fase 1

**Termasuk (In Scope):**

| # | Modul | Keterangan |
|---|-------|------------|
| 1 | Autentikasi SSO | Login via Keycloak, RBAC internal aplikasi |
| 2 | Data Pegawai | Input oleh admin, view oleh pegawai |
| 3 | Riwayat Kepegawaian | Kepangkatan, Jabatan, KGB, Disiplin (append-only) |
| 4 | Import Excel/CSV | Migrasi data awal dari spreadsheet |
| 5 | Manajemen Cuti | 6 jenis cuti, approval/verifikasi dinamis, status keputusan resmi, formulir cuti QR |
| 6 | Early Warning System | 5 trigger otomatis, scheduler harian, lifecycle alert |
| 7 | Notifikasi | In-app + email via queue, channel-configurable untuk WA Business |
| 8 | Audit Log | Semua operasi CRUD + approval + login/logout |
| 9 | Dashboard | 7 widget untuk pimpinan dan admin |
| 10 | Laporan & Export | Daftar nominatif + rekap cuti + riwayat kepangkatan ke PDF/Excel, plus nominatif Excel custom |
| 11 | Soft Delete | Flag-based deletion; tidak ada penghapusan permanen di aplikasi |
| 12 | Reference Tables | Master data hierarkis dan referensi pegawai/jabatan/status/unit yang dapat dikelola |

**Tidak Termasuk (Out of Scope Fase 1):**

| Fitur | Fase Terencana | Sumber (Slide) |
|-------|:--------------:|:--------------:|
| Self-service pegawai (edit data sendiri) | Fase 2 | Slide 5 & 8 |
| Pending changes (approval perubahan data) | Fase 2 | Slide 5 |
| Klaim kehadiran | Fase 2 | Slide 8 |
| Surat tugas | Fase 2 | — |
| Notifikasi WhatsApp Business production | Fase 2 / setelah layanan tersedia | Hasil meeting teknis |
| Kalender virtual / Kalender tim | Fase 2 | Slide 10 |
| Log harian (catat kegiatan harian) | Fase 2 | Slide 8 |
| SKP & penilaian kinerja | Fase 3 | Slide 9 |
| Tracker 20 JP / tahun | Fase 3 | Slide 9 |
| Riwayat pelatihan | Fase 3 | — |
| Kalkulator IP-ASN | Fase 4 | Slide 9 |
| Asesmen kompetensi | Fase 4 | — |
| Integrasi API SIASN / BKN | Fase 4 | Slide 10 |
| Laporan pemenuhan 20 JP | Fase 3 | Slide 10 |
| Laporan data untuk SIASN | Fase 4 | Slide 10 |
| Modul khusus Dosen DPK | Fase 4 / modul terpisah | Hasil meeting teknis |

### 3.3 Target Pengguna

| Karakteristik | Detail |
|--------------|--------|
| Jumlah pengguna | ~46 pegawai internal |
| Jenis pegawai | PNS dan PPPK |
| Organisasi | LLDIKTI Wilayah XVI (single-tenant) |
| Ekspansi | Tidak ada rencana ekspansi ke PTS binaan |

---

## 4. Pengguna & Role (RBAC)

### 4.1 Hierarki Role

```
Super Admin
  └── Admin Kepegawaian
        └── Pimpinan
              └── Kepala Bagian
                    └── Pegawai
```

**Aturan:** 
- Keycloak hanya menjadi sumber autentikasi / SSO.
- SIMPEG tetap menjadi sumber kebenaran untuk role, permission, dan otorisasi fitur.
- Role dasar yang ada di SSO tidak dipakai sebagai sumber RBAC aplikasi; role SIMPEG hanya ditetapkan dan dibaca dari database SIMPEG.
- User pertama yang berhasil login melalui SSO saat tabel user SIMPEG masih kosong otomatis dibootstrap sebagai `Super Admin`.
- Setelah bootstrap pertama, user SSO baru yang berhasil teridentifikasi tetapi belum ditetapkan role oleh admin SIMPEG tetap boleh memiliki session login, tetapi tidak boleh mengakses dashboard/fitur normal.
- User dengan role kosong, belum diset, atau role tidak valid harus menerima HTTP `403 Access Forbidden` dengan pesan: *"Akun Anda belum memiliki role SIMPEG. Hubungi Admin."*
- Fase 1 menggunakan satu role utama per pegawai agar implementasi awal sederhana.
- Struktur permission internal tetap disiapkan agar akses fitur dapat diatur tanpa mengubah Keycloak.

### 4.2 Definisi Role & Hak Akses

> **Catatan RBAC vs approval:** Role aplikasi tetap sederhana untuk Fase 1. Kewenangan sebagai verifikator/approver cuti tidak harus menjadi role baru; sistem approval cuti dapat menunjuk pegawai tertentu pada chain, misalnya Ketua Tim Kerja, selama pegawai tersebut aktif dan memiliki akun SIMPEG.

#### Super Admin

| Hak Akses | Detail |
|-----------|--------|
| Konfigurasi Sistem | Kelola reference tables, konfigurasi EWS, hari libur nasional, channel notifikasi, dan chain approval cuti |
| User Management | Assign role ke user, mapping user Keycloak ↔ pegawai |
| Semua Fitur Admin | Semua yang bisa dilakukan Admin Kepegawaian |
| Soft Delete & Restore | Menonaktifkan data tanpa menghapus permanen, serta memulihkan data jika dibutuhkan |
| Audit Log | Akses penuh ke seluruh audit log |

#### Admin Kepegawaian

| Hak Akses | Detail |
|-----------|--------|
| Data Pegawai | CRUD semua data pegawai (create, read, update, soft-delete) |
| Riwayat | Tambah riwayat kepangkatan, jabatan, KGB, disiplin (append-only) |
| Import Excel/CSV | Upload dan mapping data dari Excel/CSV |
| Set Supervisor | Assign kepala bagian per pegawai |
| Cuti | Lihat semua pengajuan cuti, kelola saldo cuti, input saldo awal/prior year, upload dokumen cuti eksternal |
| EWS | Lihat semua peringatan, update flag kinerja |
| Notifikasi | Terima notifikasi EWS |
| Dokumen | Upload/kelola dokumen dan SK pegawai |
| Laporan | Generate dan export semua laporan |

#### Pimpinan (Kepala Lembaga)

| Hak Akses | Detail |
|-----------|--------|
| Dashboard | Akses dashboard dengan data semua pegawai |
| Cuti | Pejabat final/PYBMC untuk pegawai internal sesuai konfigurasi chain |
| Data Pegawai | Read-only semua data pegawai |
| Laporan | Generate dan export laporan |

#### Kepala Bagian

| Hak Akses | Detail |
|-----------|--------|
| Cuti | Approver Stage 1 — Mengetahui (hanya bawahan langsung) |
| Data Pegawai | Read-only data bawahan langsung |
| Notifikasi | Terima notifikasi pengajuan cuti dari bawahan |

#### Verifikator Cuti / Ketua Tim Kerja

| Hak Akses | Detail |
|-----------|--------|
| Cuti | Menjadi pihak mengetahui/verifikator jika ditunjuk pada chain approval cuti |
| Data Pegawai | Read-only terbatas pada data yang diperlukan untuk memverifikasi cuti |
| Notifikasi | Terima notifikasi pengajuan cuti yang menunggu tindakan verifikasi |

> **Catatan:** Ini bukan role wajib baru. Verifikator cuti dapat tetap memiliki role utama `Pegawai` atau `Kepala Bagian`; kewenangan verifikasi muncul dari tabel konfigurasi approval cuti.

#### Pegawai

| Hak Akses | Detail |
|-----------|--------|
| Data Pribadi | Read-only data sendiri |
| Cuti | Ajukan cuti, lihat status pengajuan, lihat saldo cuti |
| Notifikasi | Terima notifikasi EWS pribadi dan status cuti |

### 4.3 Mekanisme Set Supervisor & Approval Chain

Admin Kepegawaian meng-assign kepala bagian per pegawai. Mapping ini menentukan:
- Siapa yang menjadi pihak default pertama pada chain cuti pegawai tersebut.
- Siapa yang melihat data pegawai tersebut sebagai "bawahan langsung".

**Aturan:**
- Setiap pegawai harus memiliki tepat satu kepala bagian.
- Kepala bagian bisa memiliki banyak bawahan.
- Admin bisa mengubah mapping kapan saja (perubahan tercatat di audit log).
- Approval chain cuti tidak boleh hanya bergantung pada role. Chain dapat dikonfigurasi per pegawai atau unit, berisi urutan pihak mengetahui/verifikator/final approver.
- Jumlah verifikator fleksibel: 1, 2, 3, atau lebih sesuai kebutuhan LLDIKTI.
- Pegawai yang sama tidak boleh diminta menyetujui dua langkah berurutan; sistem harus otomatis melewati langkah duplikat atau mencegah konfigurasi yang tidak valid.
- Untuk cuti Kepala Lembaga sendiri, approval internal SIMPEG tidak berlaku sebagai keputusan final. Fase 1 cukup menyediakan pencatatan oleh Admin Kepegawaian dengan upload dokumen eksternal yang sudah disetujui oleh jalur kementerian/pejabat di atasnya.

---

## 5. Arsitektur Teknis

### 5.1 Tech Stack

| Layer | Teknologi | Keterangan |
|-------|-----------|------------|
| **Backend** | Laravel 12 (PHP) | Framework utama, business logic, API |
| **Database** | PostgreSQL 17 | Database relasional utama |
| **Autentikasi** | Keycloak SSO | Hanya untuk login / SSO |
| **Otorisasi** | RBAC internal aplikasi | Role dan permission disimpan di database SIMPEG |
| **File Storage** | Laravel Local Storage | `storage/app/public` untuk dokumen dan foto |
| **Email** | SMTP / email operasional LLDIKTI | Laravel Mail + Queue untuk notifikasi email |
| **Queue** | Laravel Queue (database driver) | Untuk email dan scheduler EWS |
| **Frontend** | Laravel Blade + CSS | Server-side rendering, responsive |
| **Scheduler** | Laravel Task Scheduling | Cron job untuk EWS harian |
| **Development DB** | PostgreSQL container | Menggunakan Docker/container selama development |
| **Production Runtime** | Podman | Diprioritaskan oleh LLDIKTI untuk deployment production |

### 5.2 Diagram Arsitektur

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
│   Browser   │────▶│   Laravel App   │────▶│  PostgreSQL  │
│  (Pegawai)  │◀────│  (Blade SSR)    │◀────│  (Database)  │
└─────────────┘     └────────┬────────┘     └──────────────┘
                             │
                    ┌────────┼────────┐
                    ▼        ▼        ▼
              ┌──────┐ ┌──────┐ ┌────────────┐
              │Keycloak│ │ SMTP │ │Local Storage│
              │ (SSO) │ │Server│ │  (Files)   │
              └──────┘ └──────┘ └────────────┘
```

### 5.3 Konvensi Teknis

| Aspek | Konvensi |
|-------|----------|
| **Bahasa UI** | Bahasa Indonesia |
| **Timezone** | Asia/Makassar (WITA, UTC+8) |
| **Soft Delete** | Kolom `deleted_at` (Laravel SoftDeletes) |
| **Timestamps** | Kolom `created_at`, `updated_at` otomatis |
| **UUID** | Primary key menggunakan UUID v4 |
| **Naming** | Database: snake_case · Model: PascalCase · Route: kebab-case |

### 5.4 Environment & Deployment

| Aspek | Keputusan Meeting |
|-------|-------------------|
| Development database | PostgreSQL 17 dijalankan melalui container agar konsisten antar perangkat developer |
| Production runtime | Podman diprioritaskan dibanding Docker karena dapat berjalan rootless dan lebih aman untuk host production |
| Server production | Disiapkan oleh pihak LLDIKTI ketika sistem mendekati tahap deployment |
| Domain & SSL | Disiapkan oleh pihak LLDIKTI ketika deployment; PRD tidak mengunci domain final sebelum konfirmasi |
| Deploy script / image / compose | Akan dibantu oleh pihak LLDIKTI pada tahap deployment |
| Email development | Mailpit dapat digunakan untuk testing email |
| Email production | Menggunakan user/password email operasional dari LLDIKTI |

---

## 6. Modul 1 — Autentikasi & SSO

### 6.1 Deskripsi

Autentikasi menggunakan Keycloak SSO yang disediakan LLDIKTI XVI. Tidak ada form login manual di SIMPEG — semua autentikasi di-redirect ke Keycloak. Setelah user berhasil login, SIMPEG melakukan mapping ke data pegawai lokal dan menggunakan RBAC internal untuk menentukan hak akses.

Pihak LLDIKTI akan membagikan trait/fungsi Keycloak yang sudah digunakan di lingkungan mereka. Tim pengembang juga akan menerima akun SSO testing, Client ID, Client Secret, dan URL Keycloak untuk kebutuhan integrasi.

### 6.2 User Stories

#### US-AUTH-01: Login via SSO

> **Sebagai** pegawai LLDIKTI XVI,
> **Saya ingin** login ke SIMPEG menggunakan akun Keycloak saya,
> **Sehingga** saya tidak perlu mengingat username/password terpisah.

**Acceptance Criteria:**
1. Saat mengakses SIMPEG tanpa session aktif, user di-redirect ke halaman login Keycloak.
2. Setelah berhasil login di Keycloak, user di-redirect kembali ke SIMPEG dengan session aktif.
3. Email user dari Keycloak di-cache di database lokal SIMPEG saat login pertama.
4. Jika user belum ter-mapping ke data pegawai di SIMPEG, tampilkan halaman "Akun belum terdaftar, hubungi Admin Kepegawaian".
5. Jika user sudah ter-mapping, redirect ke halaman sesuai role-nya.
6. Role dan permission yang dipakai aplikasi dibaca dari database SIMPEG, bukan dari Keycloak.

#### US-AUTH-02: Logout

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** bisa logout dari sistem,
> **Sehingga** akun saya aman saat meninggalkan perangkat.

**Acceptance Criteria:**
1. Tombol logout tersedia di semua halaman (header/navbar).
2. Logout menghapus session SIMPEG.
3. Logout juga memicu logout dari Keycloak (single logout).
4. Setelah logout, user di-redirect ke halaman login Keycloak.

#### US-AUTH-03: Session Timeout

> **Sebagai** admin,
> **Saya ingin** session user otomatis expired setelah periode inaktif,
> **Sehingga** keamanan sistem terjaga.

**Acceptance Criteria:**
1. Session expired setelah 30 menit tidak ada aktivitas (configurable).
2. Saat session expired, user di-redirect ke Keycloak untuk login ulang.
3. Event session timeout tercatat di audit log.

### 6.3 Flow Diagram

```
User akses SIMPEG
    │
    ▼
Session aktif? ──── Ya ──── Tampilkan halaman sesuai role
    │
   Tidak
    │
    ▼
Redirect ke Keycloak Login
    │
    ▼
User login di Keycloak
    │
    ▼
Keycloak redirect + token
    │
    ▼
SIMPEG validasi token
    │
    ▼
User ada di DB SIMPEG? ──── Tidak ──── "Akun belum terdaftar"
    │
   Ya
    │
    ▼
Cache email dari Keycloak
    │
    ▼
Buat session Laravel
    │
    ▼
Redirect ke dashboard sesuai role
```

---

## 7. Modul 2 — Manajemen Data Pegawai

### 7.1 Deskripsi

Modul ini menyimpan dan mengelola seluruh data kepegawaian secara terpusat. Di Fase 1, hanya Admin Kepegawaian yang bisa melakukan input dan edit data. Pegawai hanya bisa melihat data sendiri.

### 7.2 User Stories

#### US-PEG-01: Tambah Data Pegawai Baru

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menambahkan data pegawai baru ke sistem,
> **Sehingga** data pegawai tersimpan secara terpusat dan dapat dikelola.

**Acceptance Criteria:**
1. Form input dengan semua field wajib (lihat 7.3).
2. Validasi NIP unik (tidak boleh duplikat).
3. Upload foto pegawai (maks 10MB, format JPG/PNG).
4. Data tersimpan di database dengan status aktif.
5. Audit log mencatat: siapa yang menambahkan, kapan, data apa.

#### US-PEG-02: Edit Data Pegawai

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengubah data pegawai yang sudah ada,
> **Sehingga** data selalu akurat dan terbaru.

**Acceptance Criteria:**
1. Admin bisa mengedit semua field data pegawai.
2. Perubahan data tercatat di audit log (nilai sebelum dan sesudah).
3. Validasi data tetap berlaku saat edit.
4. Timestamp `updated_at` otomatis diperbarui.

#### US-PEG-03: Lihat Data Pegawai (Admin)

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat daftar dan detail semua pegawai,
> **Sehingga** saya bisa mengelola data kepegawaian secara efisien.

**Acceptance Criteria:**
1. Halaman daftar pegawai dengan tabel (nama, NIP, golongan, jabatan, unit kerja, status).
2. Search/filter berdasarkan: nama, NIP, golongan, unit kerja, status PNS/PPPK.
3. Klik nama pegawai menampilkan detail lengkap.
4. Pagination (10/25/50 per halaman).
5. Sorting berdasarkan kolom yang tersedia.

#### US-PEG-04: Lihat Data Sendiri (Pegawai)

> **Sebagai** pegawai,
> **Saya ingin** melihat data kepegawaian saya sendiri,
> **Sehingga** saya bisa memastikan data saya benar dan lengkap.

**Acceptance Criteria:**
1. Pegawai hanya bisa melihat data miliknya sendiri (tidak bisa lihat pegawai lain).
2. Tampilkan semua data: pribadi, kontak, keluarga, kepangkatan, jabatan, KGB, disiplin, dokumen.
3. Data bersifat read-only (tidak bisa edit di Fase 1).
4. Tampilkan informasi saldo cuti.

#### US-PEG-05: Soft Delete Pegawai

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menonaktifkan data pegawai yang sudah pensiun/mutasi,
> **Sehingga** data tidak muncul di daftar aktif tapi tetap tersimpan untuk riwayat.

**Acceptance Criteria:**
1. Soft delete mengisi kolom `deleted_at` (data tidak dihapus dari database).
2. Pegawai yang di-soft-delete tidak muncul di daftar pegawai aktif.
3. Admin bisa melihat daftar pegawai yang sudah dinonaktifkan (filter terpisah).
4. Admin bisa me-restore pegawai yang di-soft-delete.
5. Audit log mencatat soft delete dan restore.

#### US-PEG-06: Soft Delete oleh Super Admin

> **Sebagai** Super Admin,
> **Saya ingin** menonaktifkan data pegawai tanpa menghapus permanen,
> **Sehingga** data yang sewaktu-waktu dibutuhkan masih bisa ditemukan dan dipulihkan.

**Acceptance Criteria:**
1. Tidak ada fitur hapus permanen untuk data pegawai di aplikasi.
2. Super Admin hanya bisa melakukan soft delete/nonaktifkan pegawai dengan konfirmasi.
3. Data pegawai yang dinonaktifkan tetap tersimpan di database.
4. Data pegawai yang dinonaktifkan bisa ditemukan melalui filter pegawai non-aktif.
5. Super Admin bisa melakukan restore jika data perlu dipakai kembali.
6. Audit log mencatat soft delete dan restore.

### 7.3 Struktur Data Pegawai

> **Catatan penyesuaian format Excel:** File `daftar_pegawai.xlsx` sheet `Pegawai` menjadi acuan import awal. Header yang tersedia: `No`, `Nama Pegawai`, `Email Pegawai`, `Golongan`, `Jabatan`, `Kelas Jabatan`, `NIP`, `Nomor Telepon`, `Pangkat`, `Pendidikan Terakhir`, `Pensiun`, `Person`, `Person Formula`, `Prodi Pendidikan Terakhir`, `Status Kepegawaian`, dan `Tanggal Lahir`. Mapping detail tersedia di `Mapping-Data-Pegawai-Excel-SIMPEG.md`.
>
> Struktur di bawah tetap menjadi target profil lengkap SIMPEG. Untuk import awal dari Excel, field yang tidak tersedia seperti NIK, No. KK, tempat lahir, alamat, data SK, TMT, keluarga, unit kerja, dan kepala bagian boleh kosong dulu dan ditandai sebagai profil belum lengkap.

#### Data Pribadi

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `nama_lengkap` | string(255) | Ya | Nama sesuai SK |
| `nip` | string(18) | Ya | Unik, 18 digit |
| `email_pribadi` | string(255) | Ya untuk import Excel | Email dari kolom `Email Pegawai`; kandidat mapping awal Keycloak |
| `nik` | string(16) | Ya untuk profil lengkap | NIK KTP; belum tersedia di Excel awal |
| `no_kk` | string(16) | Tidak | Nomor Kartu Keluarga |
| `tempat_lahir` | string(100) | Ya untuk profil lengkap | Belum tersedia di Excel awal |
| `tanggal_lahir` | date | Ya | Untuk kalkulasi BUP |
| `jenis_kelamin` | enum | Ya untuk profil lengkap | L / P; belum tersedia di Excel awal |
| `agama` | ref_agama_id | Ya untuk profil lengkap | FK ke ref_agama; belum tersedia di Excel awal |
| `status_perkawinan` | ref_status_kawin_id | Ya untuk profil lengkap | FK ke ref_status_kawin; belum tersedia di Excel awal |
| `golongan_darah` | enum | Tidak | A / B / AB / O |
| `foto` | string(path) | Tidak | Path ke file foto |
| `jenis_pegawai` | enum/ref | Ya | PNS / PPPK / CPNS; dapat dibuat reference bila LLDIKTI ingin menambah kategori |
| `status_pegawai_id` | ref_status_pegawai_id | Ya | FK ke `ref_status_pegawai`; default `Aktif` |
| `status_keterangan` | text | Tidak | Keterangan/alasan saat status pegawai berubah, misalnya CLTN, tugas belajar, atau pemberhentian sementara |
| `golongan_terakhir` | snapshot | Ya untuk import Excel | Snapshot langsung dari kolom Excel saat import Data Utama. Setelah ada riwayat kepangkatan resmi, nilai terkini diturunkan dari riwayat terbaru berdasarkan TMT; import tidak membuat riwayat |
| `pangkat_terakhir` | snapshot | Tidak | Snapshot dari kolom Excel; boleh kosong untuk PPPK/CPNS. Setelah ada riwayat kepangkatan resmi, nilai terkini diturunkan dari riwayat terbaru; import tidak membuat riwayat |
| `jabatan_terakhir` | snapshot | Ya untuk import Excel | Snapshot dari kolom Excel saat import Data Utama. Setelah ada riwayat jabatan resmi, nilai terkini diturunkan dari riwayat terbaru berdasarkan TMT dan `ref_jabatan`; import tidak membuat riwayat |
| `kelas_jabatan_terakhir` | snapshot | Ya untuk import Excel | Snapshot dari kolom Excel; nilai kelas resmi disimpan pada riwayat jabatan, bukan pada master jabatan. Import tidak membuat riwayat jabatan |
| `pendidikan_terakhir` | string(20) | Ya untuk import Excel | Contoh: D3, S1, S2; snapshot dari kolom Excel |
| `prodi_pendidikan_terakhir` | string(255) | Ya untuk import Excel | Dari kolom `Prodi Pendidikan Terakhir`; snapshot dari kolom Excel |
| `tanggal_pensiun` | date | Tidak | Dari kolom `Pensiun`. Nilai hasil import dipertahankan apa adanya dan tidak dihitung ulang atau ditimpa oleh import. Bila kolom kosong, tanggal pensiun dihitung dari BUP hanya melalui kalkulasi TMT saat riwayat/sumber resmi disimpan, bukan saat import |
| `profil_status` | enum | Ya | `belum_lengkap` / `lengkap` untuk membedakan hasil import awal dan profil yang sudah dilengkapi |

#### Data Kontak

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `alamat` | text | Ya | Alamat lengkap |
| `no_hp` | string(20) | Ya | |
| `email_pribadi` | string(255) | Tidak | Email selain email Keycloak |
| `no_telepon_rumah` | string(20) | Tidak | |

#### Data Keluarga

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `nama_anggota` | string(255) | Ya | Nama pasangan/anak |
| `hubungan` | enum | Ya | Suami / Istri / Anak |
| `nik` | string(16) | Tidak | |
| `tempat_lahir` | string(100) | Tidak | |
| `tanggal_lahir` | date | Ya | |
| `jenis_kelamin` | enum | Ya | L / P |
| `status_tunjangan` | boolean | Ya | Apakah menerima tunjangan |
| `pekerjaan` | string(100) | Tidak | |

#### Data Pengangkatan

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `jenis_pengangkatan` | enum | Ya | CPNS / PNS / PPPK |
| `tmt_pengangkatan` | date | Ya | |
| `no_sk` | string(100) | Ya | |
| `tanggal_sk` | date | Ya | |
| `file_sk` | string(path) | Tidak | Path ke file SK |

#### Data Kepangkatan (Riwayat — Append-Only)

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `golongan_id` | ref_golongan_id | Ya | FK ke ref_golongan |
| `tmt_pangkat` | date | Ya | TMT untuk kalkulasi EWS |
| `no_sk` | string(100) | Ya | |
| `tanggal_sk` | date | Ya | |
| `file_sk` | string(path) | Tidak | |
| `is_latest` | boolean | Ya | Tandai record terbaru |

#### Data Jabatan (Riwayat — Append-Only)

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `jabatan_id` | ref_jabatan_id | Ya | FK ke `ref_jabatan`; nama jabatan tidak diinput free text |
| `jenis_jabatan_id` | ref_jenis_jabatan_id | Ya | FK ke ref_jenis_jabatan; dapat diisi otomatis dari `ref_jabatan` |
| `eselon_id` | ref_eselon_id | Tidak | FK ke ref_eselon |
| `unit_kerja_id` | ref_unit_kerja_id | Ya | FK ke ref_unit_kerja |
| `kelas_jabatan` | string(10) | Tidak | Disimpan di riwayat karena kelas jabatan bisa berbeda walau nama jabatan sama |
| `tmt_jabatan` | date | Ya | |
| `no_sk` | string(100) | Ya | |
| `tanggal_sk` | date | Ya | |
| `file_sk` | string(path) | Tidak | |
| `is_latest` | boolean | Ya | |

**Aturan riwayat jabatan:**
- Data utama pegawai menampilkan jabatan/golongan terkini dari riwayat terbaru berdasarkan TMT, bukan hasil edit manual terpisah.
- Satu pegawai hanya punya satu jabatan aktif/terkini pada satu waktu. Jika pegawai pindah dari fungsional ke struktural, jabatan aktif berubah dan jabatan lama tetap tersimpan sebagai riwayat.
- Upload SK menjadi bukti pendukung, tetapi sistem tidak melakukan parsing isi SK di Fase 1.

#### Data KGB (Riwayat — Append-Only)

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `tmt_kgb` | date | Ya | TMT untuk kalkulasi EWS |
| `gaji_pokok` | decimal(15,2) | Ya | Gaji pokok setelah KGB |
| `no_sk` | string(100) | Ya | |
| `tanggal_sk` | date | Ya | |
| `file_sk` | string(path) | Tidak | |
| `is_latest` | boolean | Ya | |

#### Hukuman Disiplin (Riwayat — Append-Only)

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `jenis_hukuman` | enum | Ya | Ringan / Sedang / Berat |
| `deskripsi` | text | Ya | Detail hukuman |
| `tanggal_mulai` | date | Ya | |
| `tanggal_berakhir` | date | Tidak | Null = masih aktif |
| `no_sk` | string(100) | Ya | |
| `tanggal_sk` | date | Ya | |
| `file_sk` | string(path) | Tidak | |
| `is_active` | boolean | Ya | Apakah masih berlaku |

#### Data Pendidikan

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `jenjang_id` | ref_jenjang_pendidikan_id | Ya | FK ke ref_jenjang_pendidikan |
| `nama_institusi` | string(255) | Ya | |
| `jurusan` | string(255) | Tidak | |
| `tahun_lulus` | year | Ya | |
| `no_ijazah` | string(100) | Ya | |
| `file_ijazah` | string(path) | Tidak | |

#### Dokumen & SK

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `jenis_dokumen` | string(100) | Ya | Kategori dokumen |
| `nama_dokumen` | string(255) | Ya | |
| `nomor_dokumen` | string(100) | Tidak | |
| `tanggal_dokumen` | date | Tidak | |
| `file_path` | string(path) | Ya | Path ke file |
| `keterangan` | text | Tidak | |

### 7.4 Aturan Upload File

| Aspek | Aturan |
|-------|--------|
| Ukuran maksimum | 10 MB per file |
| Format yang diizinkan | PDF, DOC, DOCX, JPG, JPEG, PNG |
| Penamaan file | `{pegawai_id}_{jenis}_{timestamp}.{ext}` |
| Storage | `storage/app/public/{pegawai_id}/{kategori}/` |
| Validasi | MIME type check (tidak hanya ekstensi) |

### 7.5 Set Supervisor (Assign Kepala Bagian)

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `pegawai_id` | UUID | Ya | Pegawai yang di-assign |
| `kepala_bagian_id` | UUID | Ya | FK ke pegawai yang menjadi kepala bagian |
| `tanggal_mulai` | date | Ya | Sejak kapan berlaku |
| `tanggal_berakhir` | date | Tidak | Null = masih berlaku |

---

## 8. Modul 3 — Import Data Excel/CSV

### 8.1 Deskripsi

Fitur import Excel/CSV memungkinkan Admin Kepegawaian melakukan migrasi data awal dari spreadsheet yang sudah ada ke database SIMPEG.

Format import awal mengikuti file `daftar_pegawai.xlsx` sheet `Pegawai`. Sistem tetap menyediakan mapping kolom agar format ini bisa disesuaikan jika LLDIKTI mengirim file versi berikutnya.

> **Ruang lingkup import Fase 1 (keputusan pengguna 22 Juli 2026):** Hanya template Data Utama yang aktif. Import membuat record pegawai beserta field snapshot awal, tetapi tidak membuat riwayat kepangkatan, riwayat jabatan, maupun riwayat KGB, dan tidak memanggil kalkulasi TMT. Riwayat resmi diinput per pegawai melalui CRUD riwayat append-only. Tanggal pensiun hasil import dipertahankan apa adanya. Template lanjutan multi-jenis tidak termasuk ruang lingkup saat ini dan tidak dipulihkan tanpa keputusan eksplisit baru.

### 8.2 User Stories

#### US-CSV-01: Import Data Pegawai dari Excel/CSV

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengimpor data pegawai dari file Excel/CSV,
> **Sehingga** data awal bisa dimasukkan secara massal tanpa input satu per satu.

**Acceptance Criteria:**
1. Admin bisa upload file Excel/CSV (maks 10MB).
2. Sistem menampilkan **preview data** (10 baris pertama) sebelum import.
3. Sistem auto-match header Excel `daftar_pegawai.xlsx` ke field SIMPEG, dan Admin tetap bisa melakukan **mapping kolom** manual jika diperlukan.
4. Validasi sebelum import: NIP unik, email pegawai terisi, format tanggal benar, field wajib dari Excel terisi.
5. Tampilkan **ringkasan validasi**: berapa baris valid, berapa baris error, detail error per baris.
6. Admin bisa memilih: import hanya yang valid, atau batalkan semua.
7. Import hanya memproses template Data Utama: membuat record pegawai beserta field snapshot awal. Import tidak membuat riwayat kepangkatan, riwayat jabatan, maupun riwayat KGB, dan tidak memanggil kalkulasi TMT.
8. Tanggal pensiun hasil import dipertahankan apa adanya; import tidak menghitung ulang atau menimpa tanggal pensiun.
9. Setelah import, tampilkan **laporan hasil**: berapa berhasil, berapa gagal.
10. Audit log mencatat import (siapa, kapan, berapa record).

#### US-CSV-02: Download Template Import

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengunduh template import,
> **Sehingga** saya tahu format yang benar untuk persiapan data.

**Acceptance Criteria:**
1. Tersedia tombol "Download Template Import".
2. Template utama berisi header sesuai `daftar_pegawai.xlsx`: `No`, `Nama Pegawai`, `Email Pegawai`, `Golongan`, `Jabatan`, `Kelas Jabatan`, `NIP`, `Nomor Telepon`, `Pangkat`, `Pendidikan Terakhir`, `Pensiun`, `Person`, `Person Formula`, `Prodi Pendidikan Terakhir`, `Status Kepegawaian`, `Tanggal Lahir`.
3. Sertakan 1-2 baris contoh data.
4. Format minimal CSV UTF-8 delimiter koma; bila disediakan Excel, gunakan `.xlsx` dengan header kolom yang sama.

### 8.3 Jenis Import yang Didukung (Fase 1)

Sesuai keputusan pengguna 22 Juli 2026, hanya satu jenis import yang aktif di Fase 1.

| Jenis Import | Status | Keterangan |
|-------------|--------|------------|
| Data Pegawai (Utama) | Aktif | Mengikuti `daftar_pegawai.xlsx`: nama, email, NIP, telepon, status kepegawaian, tanggal lahir, golongan, pangkat, jabatan, kelas jabatan, pendidikan, prodi, dan tanggal pensiun jika tersedia. Membuat record pegawai beserta snapshot awal; tidak membuat riwayat apa pun dan tidak memanggil kalkulasi TMT |

**Yang tidak termasuk ruang lingkup saat ini:** template import lanjutan Riwayat Kepangkatan, Riwayat Jabatan, dan Riwayat KGB tidak diaktifkan di Fase 1. Riwayat resmi diinput per pegawai melalui CRUD riwayat append-only, bukan melalui import massal. Template multi-jenis ini tidak dipulihkan tanpa keputusan eksplisit baru.

---

## 9. Modul 4 — Manajemen Cuti

### 9.1 Deskripsi

Modul cuti mendigitalisasi seluruh proses pengajuan dan persetujuan cuti sesuai PP 11/2017 jo PP 17/2020. Terdapat 6 jenis cuti dengan perbedaan hak antara PNS dan PPPK.

Berdasarkan meeting teknis terbaru, Fase 1 harus memakai engine approval cuti yang dinamis. Pengajuan dapat melalui kepala bagian, satu atau lebih verifikator, Ketua Tim Kerja bila ditunjuk, Kabag/Kepegawaian, dan final approver/PYBMC sesuai konfigurasi per pegawai/unit. Kepegawaian/verifikator wajib dapat memeriksa hak cuti, saldo, kelayakan, dan kelengkapan sebelum keputusan final.

Ketua Tim Kerja tidak memerlukan role baru. Jika perlu mengetahui atau memverifikasi cuti pegawai pada tim kerja substansi, Ketua Tim cukup ditunjuk sebagai salah satu step dalam approval chain. Untuk cuti pegawai internal biasa, final PYBMC tetap Kepala Lembaga sesuai konfigurasi. Untuk cuti Kepala Lembaga sendiri, SIMPEG Fase 1 hanya mencatat dan mengarsipkan dokumen persetujuan eksternal dari jalur kementerian/pejabat yang lebih tinggi.

### 9.2 Jenis Cuti

| Jenis Cuti | PNS | PPPK | Kuota / Aturan |
|------------|:---:|:----:|---------------|
| Cuti Tahunan | ✅ | ✅ | 12 hari kerja per tahun, carry-over mengikuti aturan N-1/N-2 |
| Cuti Sakit | ✅ | ✅ | Sesuai PP, surat dokter diperlukan (sesuai ketentuan) |
| Cuti Melahirkan | ✅ | ✅ | 3 bulan (anak ke-1 s.d. ke-3 sesuai PP) |
| Cuti Karena Alasan Penting | ✅ | ✅ | Sesuai PP, untuk kebutuhan mendesak keluarga |
| Cuti Besar | ✅ | ❌ | Minimal 5 tahun masa kerja, disembunyikan untuk PPPK |
| CLTN | ✅ | ❌ | Maks 3 tahun, disembunyikan untuk PPPK |

### 9.3 User Stories

#### US-CUT-01: Ajukan Cuti

> **Sebagai** pegawai,
> **Saya ingin** mengajukan cuti secara digital,
> **Sehingga** saya tidak perlu mengurus berkas fisik.

**Acceptance Criteria:**
1. Form pengajuan cuti dengan field: jenis cuti, tanggal mulai, tanggal selesai, alasan, dan lampiran opsional.
2. Jenis cuti yang ditampilkan sesuai status kepegawaian (PPPK: tanpa Cuti Besar dan CLTN).
3. Sistem otomatis **menghitung jumlah hari kerja** (exclude Sabtu, Minggu, hari libur nasional, cuti bersama).
4. Sistem menolak submit jika satu pengajuan melewati tahun kalender. Jika cuti berlangsung Desember–Januari, pegawai harus membuat dua pengajuan terpisah.
5. Validasi saldo cuti: jika saldo tidak cukup (untuk cuti tahunan), form tidak bisa di-submit dan sistem menampilkan pesan validasi. Ini bukan status keputusan `Tidak Disetujui`.
6. Upload lampiran opsional (misal: surat dokter untuk cuti sakit).
7. Setelah submit, status pengajuan = "Menunggu [step pertama sesuai chain]".
8. Notifikasi terkirim ke pihak pertama dalam approval chain.

#### US-CUT-02: Verifikasi / Mengetahui oleh Step Approval

> **Sebagai** pihak yang ditunjuk dalam approval chain,
> **Saya ingin** memeriksa pengajuan cuti yang menunggu tindakan saya,
> **Sehingga** setiap pertimbangan/verifikasi tercatat sebelum keputusan final.

**Acceptance Criteria:**
1. Pengguna hanya melihat pengajuan cuti yang sedang menunggu tindakan dirinya pada chain.
2. Detail pengajuan: nama pegawai, jenis cuti, tanggal, jumlah hari, alasan.
3. Opsi tindakan mengikuti label resmi formulir: **"Disetujui"**, **"Perubahan"**, **"Ditangguhkan"**, dan **"Tidak Disetujui"**.
4. Untuk step non-final, aksi disimpan sebagai rekomendasi/pertimbangan/verifikasi dan dapat menentukan apakah flow lanjut ke step berikutnya atau perlu tindak lanjut pegawai/admin.
5. `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui` wajib mengisi keterangan/alasan.
6. `Disetujui` tidak wajib mengisi keterangan.
7. Jika step yang sama menunjuk pegawai yang sudah bertindak di step sebelumnya, sistem melewati step duplikat atau menolak konfigurasi tersebut.
8. Semua tindakan dan keterangan tampil di timeline dan tercatat di audit log.

#### US-CUT-03: Verifikasi Kepegawaian

> **Sebagai** Admin Kepegawaian/verifikator,
> **Saya ingin** memverifikasi hak cuti, saldo, dan kelengkapan pengajuan,
> **Sehingga** pimpinan menerima pengajuan yang sudah jelas status kelayakannya.

**Acceptance Criteria:**
1. Verifikator dapat melihat saldo tahun berjalan, sisa N-1/N-2, cuti bersama, dan riwayat cuti tahunan pegawai.
2. Verifikator dapat memberi rekomendasi `Disetujui`, `Perubahan`, `Ditangguhkan`, atau `Tidak Disetujui`.
3. Rekomendasi selain `Disetujui` wajib memuat keterangan yang terlihat oleh pegawai, admin, dan approver berikutnya.
4. Jika verifikator merekomendasikan tidak lanjut, status dan alasan tetap disimpan sebagai bagian dari riwayat keputusan.
5. Chain dapat memiliki lebih dari satu verifikator dan urutannya mengikuti konfigurasi.

#### US-CUT-04: Keputusan Final Pimpinan/PYBMC

> **Sebagai** Pimpinan/PYBMC,
> **Saya ingin** memberikan keputusan akhir pengajuan cuti,
> **Sehingga** cuti bisa resmi berlaku.

**Acceptance Criteria:**
1. Final approver mengikuti konfigurasi approval chain; default untuk pegawai internal biasa adalah Kepala Lembaga/PYBMC.
2. Opsi keputusan final: **"Disetujui"**, **"Perubahan"**, **"Ditangguhkan"**, dan **"Tidak Disetujui"**.
3. Jika "Disetujui":
   - Status keputusan final menjadi `Disetujui`.
   - Saldo cuti tahunan **dikurangi otomatis** (jika cuti tahunan).
   - Notifikasi persetujuan terkirim ke pegawai.
   - Sistem menghasilkan formulir cuti resmi dengan QR Code verifikasi.
4. Jika `Perubahan`, `Ditangguhkan`, atau `Tidak Disetujui`:
   - Keterangan wajib diisi.
   - Saldo cuti tidak dikurangi.
   - Notifikasi terkirim ke pegawai beserta alasan/keterangan.
5. Keputusan final dan identitas pejabat final tampil pada halaman verifikasi QR.

#### US-CUT-05: Lihat Status Pengajuan Cuti

> **Sebagai** pegawai,
> **Saya ingin** melihat status pengajuan cuti saya,
> **Sehingga** saya tahu posisi approval saat ini.

**Acceptance Criteria:**
1. Daftar semua pengajuan cuti saya (aktif dan riwayat).
2. Status proses ditampilkan jelas: Draft / Menunggu [nama step] / Perlu Perubahan / Ditangguhkan / Selesai.
3. Keputusan final ditampilkan dengan label resmi: Disetujui / Perubahan / Ditangguhkan / Tidak Disetujui.
4. Timeline approval: siapa yang sudah bertindak, peran dalam chain, rekomendasi/keputusan, waktu, dan keterangan.
5. Filter berdasarkan status dan tahun.

#### US-CUT-06: Lihat Saldo Cuti

> **Sebagai** pegawai,
> **Saya ingin** melihat saldo cuti saya,
> **Sehingga** saya tahu berapa hari cuti yang tersisa.

**Acceptance Criteria:**
1. Tampilkan saldo cuti tahunan: jatah dasar 12 hari, carry-over N-1, hak tambahan jika memenuhi aturan N-2/N-1, total tersedia, terpakai, dan sisa.
2. Tampilkan riwayat penggunaan cuti tahun berjalan dan dua tahun sebelumnya yang memengaruhi carry-over.
3. Admin dapat mengisi/koreksi saldo awal dan riwayat N-1/N-2 saat setup awal dengan alasan dan audit log.

### 9.4 Aturan Bisnis Cuti

#### Carry-Over Cuti Tahunan

| Aturan | Detail |
|--------|--------|
| Jatah dasar | 12 hari kerja per tahun |
| Carry-over N-1 | Jika tahun sebelumnya masih ada sisa, maksimal 6 hari dapat dibawa ke tahun berjalan |
| Kondisi N-2 dan N-1 tidak mengambil cuti | Jika pegawai tidak mengambil cuti tahunan selama dua tahun berturut-turut, total hak tahun berjalan dapat mencapai 24 hari |
| Kondisi mengambil cuti pada salah satu dari N-2/N-1 | Carry-over tetap maksimal 6 hari, bukan 12 |
| Reset | Dihitung ulang otomatis di awal tahun (1 Januari) |
| Setup awal | Admin Kepegawaian dapat menginput saldo/riwayat tahun sebelumnya agar kalkulasi awal akurat |

#### Batas Tahun Kalender

- Satu pengajuan cuti tidak boleh melewati tahun kalender.
- Jika pegawai ingin cuti dari Desember sampai Januari, sistem harus meminta dua pengajuan: satu untuk tahun berjalan dan satu untuk tahun berikutnya.
- Validasi ini mencegah kebingungan reset kuota dan carry-over.

#### Kalkulasi Hari Kerja

Saat menghitung jumlah hari cuti, sistem harus **menghitung hari kerja saja**:

- ✅ Senin — Jumat dihitung
- ❌ Sabtu & Minggu tidak dihitung
- ❌ Hari Libur Nasional tidak dihitung (dari ref_hari_libur)
- ❌ Cuti Bersama tidak dihitung (dari ref_cuti_bersama)

#### Cuti Bersama

Cuti bersama otomatis mengurangi saldo cuti tahunan (sesuai kebijakan yang berlaku). Admin menginput daftar cuti bersama per tahun di reference table.

#### Status Flow Pengajuan Cuti

```
[Draft] → [Menunggu Step 1] → [Menunggu Step 2..n] → [Menunggu Final PYBMC] → [Selesai]
              │                      │                         │
              ├─ Perubahan           ├─ Perubahan              ├─ Perubahan
              ├─ Ditangguhkan        ├─ Ditangguhkan           ├─ Ditangguhkan
              └─ Tidak Disetujui     └─ Tidak Disetujui        └─ Tidak Disetujui
```

**Catatan konfigurasi:** Jika approver pada dua step adalah orang yang sama, step duplikat harus dilewati otomatis atau dicegah saat konfigurasi. Urutan approver harus dapat diatur agar tidak terjadi approval terbalik, misalnya Kepala Bagian → Ketua Tim → Kepegawaian/Kabag → Pimpinan.

**Catatan keputusan:** Istilah formal `Ditolak` tidak dipakai. Untuk keputusan negatif, gunakan `Tidak Disetujui`.

#### Dokumen Cuti & QR Verification

- Setelah pengajuan selesai diproses, sistem menghasilkan formulir cuti resmi sesuai format LLDIKTI.
- Formulir memuat QR Code yang mengarah ke halaman verifikasi SIMPEG.
- Halaman verifikasi menampilkan minimal: nama pegawai, jenis cuti, tanggal cuti, status keputusan, tanggal keputusan, pejabat final/approver, dan identitas instansi LLDIKTI Wilayah XVI.
- QR Code tidak menggantikan tanda tangan elektronik tersertifikasi. Fase 1 cukup menyediakan verifikasi digital berbasis halaman SIMPEG.
- Lampiran pendukung dari pegawai tetap opsional sesuai jenis cuti.
- Untuk cuti Kepala Lembaga atau kasus approval eksternal, Admin Kepegawaian dapat mengunggah scan dokumen yang sudah disetujui di luar SIMPEG; sistem menandainya sebagai `external_approval`.

---

## 10. Modul 5 — Early Warning System (EWS)

### 10.1 Deskripsi

EWS adalah scheduler otomatis yang berjalan setiap hari untuk memeriksa momen penting kepegawaian dan mengirim notifikasi di waktu yang telah ditentukan. Dasar perhitungan menggunakan TMT (Terhitung Mulai Tanggal) dari data riwayat pegawai.

### 10.2 Trigger Events

#### Kenaikan Pangkat Reguler

| Aspek | Detail |
|-------|--------|
| **Dasar hukum** | PP 99/2000 |
| **Rumus** | TMT pangkat terakhir + 4 tahun = tanggal kenaikan pangkat berikutnya |
| **Interval notifikasi** | H-90, H-60, H-30 sebelum tanggal kenaikan pangkat |
| **Syarat eligibility** | (1) 4 tahun sejak TMT terakhir, (2) Tidak ada hukuman disiplin aktif, (3) Flag kinerja baik ✅ |
| **Penerima** | Pegawai bersangkutan + Admin Kepegawaian |

**Catatan Fase 1:** Karena modul SKP belum tersedia, syarat "kinerja baik" ditangani dengan field manual `is_kinerja_baik` (boolean) di data pegawai yang diisi oleh Admin Kepegawaian. Default: `true`.

#### Kenaikan Gaji Berkala (KGB)

| Aspek | Detail |
|-------|--------|
| **Dasar hukum** | PP 99/2000 |
| **Rumus** | TMT KGB terakhir + 2 tahun = tanggal KGB berikutnya |
| **Interval notifikasi** | H-60, H-30, H-14 |
| **Penerima** | Pegawai bersangkutan + Admin Kepegawaian |

#### Batas Usia Pensiun (BUP)

| Aspek | Detail |
|-------|--------|
| **Dasar hukum** | PP 49/2018 |
| **Rumus** | Tanggal lahir + usia pensiun pada referensi jabatan = tanggal pensiun |
| **BUP** | Tidak di-hardcode; diambil dari `ref_jenis_jabatan.maks_usia_pensiun` atau detail `ref_bup` |
| **Interval notifikasi** | H-1 tahun, H-6 bulan, H-3 bulan |
| **Penerima** | Pegawai bersangkutan + Admin Kepegawaian |

**Keputusan meeting:** Secara umum BUP adalah 58 tahun, sedangkan beberapa jabatan tertentu terutama jabatan tinggi dapat menggunakan 60 tahun. Nilai final harus berasal dari reference table agar dapat disesuaikan oleh Admin tanpa perubahan kode.

#### Kontrak PPPK Berakhir

| Aspek | Detail |
|-------|--------|
| **Dasar hukum** | PP 49/2018 |
| **Rumus** | Tanggal berakhir kontrak PPPK |
| **Interval notifikasi** | H-6 bulan, H-3 bulan, H-1 bulan |
| **Penerima** | Pegawai PPPK bersangkutan + Admin Kepegawaian |
| **Khusus** | Hanya untuk pegawai dengan jenis_pegawai = PPPK |

#### Satyalancana Karya Satya

| Aspek | Detail |
|-------|--------|
| **Dasar** | Masa kerja pegawai berdasarkan TMT pengangkatan pertama |
| **Rumus** | TMT pengangkatan pertama + 10 / 20 / 30 tahun |
| **Interval notifikasi** | H-180, H-90, H-30 sebelum milestone |
| **Syarat eligibility** | Masa kerja terpenuhi dan flag kelayakan/pertimbangan manual dari Admin Kepegawaian |
| **Penerima** | Pegawai bersangkutan + Admin Kepegawaian |

**Catatan Fase 1:** Karena kelayakan Satyalancana dapat mempertimbangkan aspek kinerja/SKP dan data administratif lain yang belum seluruhnya dimodelkan, sistem menyediakan flag manual `is_satyalancana_eligible` atau catatan pertimbangan setara yang dapat diisi Admin Kepegawaian.

### 10.3 User Stories

#### US-EWS-01: Scheduler EWS Harian

> **Sebagai** sistem,
> **Saya ingin** menjalankan pengecekan EWS setiap hari secara otomatis,
> **Sehingga** tidak ada momen penting kepegawaian yang terlewat.

**Acceptance Criteria:**
1. Laravel scheduler berjalan setiap hari pukul 07:00 WITA.
2. Sistem memeriksa semua pegawai aktif terhadap 5 trigger di atas.
3. Jika ada pegawai yang memenuhi kriteria interval, buat notifikasi.
4. Notifikasi tidak duplikat (jika notifikasi H-90 sudah dikirim, tidak kirim ulang H-90 keesokan harinya).
5. Log eksekusi scheduler dicatat (waktu mulai, waktu selesai, jumlah notifikasi yang dihasilkan).
6. Alert memiliki status tindak lanjut (`aktif`, `ditangani`, `tidak_perlu`, `kedaluwarsa`) agar alert yang sudah selesai tidak terus muncul.

#### US-EWS-02: Dashboard EWS

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat daftar semua peringatan EWS yang aktif,
> **Sehingga** saya bisa menindaklanjuti sebelum deadline.

**Acceptance Criteria:**
1. Halaman daftar EWS aktif, diurutkan dari yang paling mendesak.
2. Informasi: nama pegawai, jenis event, tanggal target, sisa hari, status tindak lanjut.
3. Filter berdasarkan jenis event.
4. Indikator warna: merah (< 30 hari), kuning (30-90 hari), hijau (> 90 hari).
5. Admin dapat menandai alert sebagai ditangani/tidak perlu dengan catatan.

#### US-EWS-03: Update Flag Kinerja

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menandai status kinerja pegawai (baik/tidak baik),
> **Sehingga** EWS bisa menentukan eligibility kenaikan pangkat.

**Acceptance Criteria:**
1. Di halaman detail pegawai, ada toggle "Kinerja Baik" (default: Ya).
2. Jika diubah ke "Tidak", pegawai tidak muncul di EWS kenaikan pangkat.
3. Perubahan tercatat di audit log.

### 10.4 Kalkulasi TMT Otomatis

Sistem otomatis menghitung tanggal event berikutnya berdasarkan data riwayat:

```
tanggal_kenaikan_pangkat_berikutnya = tmt_pangkat_terakhir + 4 tahun
tanggal_kgb_berikutnya = tmt_kgb_terakhir + 2 tahun
tanggal_pensiun = tanggal_lahir + maks_usia_pensiun_pada_jabatan
tanggal_kontrak_berakhir = tanggal_berakhir_kontrak (langsung dari data)
tanggal_satyalancana = tmt_pengangkatan_pertama + 10/20/30 tahun
```

Kalkulasi ini dijalankan ulang setiap kali riwayat/sumber resmi disimpan:
- Data riwayat kepangkatan/KGB ditambahkan atau diperbarui.
- Data jabatan atau referensi BUP diperbarui (karena usia pensiun bisa berubah).

Kalkulasi TMT tidak dipicu oleh selesainya import massal. Import Data Utama hanya menyimpan field snapshot dan mempertahankan tanggal pensiun dari kolom Excel apa adanya. Tanggal pensiun hasil import tidak dihitung ulang atau ditimpa oleh import.

---

## 11. Modul 6 — Notifikasi

### 11.1 Deskripsi

Notifikasi dipicu oleh events di modul lain (cuti, EWS, import, audit penting, dll). Fase 1 mengirim notifikasi melalui in-app dan email via queue, tetapi implementasi tidak boleh mengunci channel hanya ke dua opsi tersebut. Sistem harus memakai konfigurasi channel agar WhatsApp Business atau channel lain dapat ditambahkan saat layanan/credential tersedia.

### 11.2 Channel Notifikasi

| Channel | Status Fase 1 | Mekanisme | Keterangan |
|---------|---------------|-----------|------------|
| **In-App** | Wajib | Database-backed, polling/SSE | Tampil di bell icon di navbar, badge count |
| **Email** | Wajib bila credential tersedia | SMTP/Gmail resmi via Laravel Mail + Queue | Development dapat memakai Mailpit; production memakai email operasional LLDIKTI atau Gmail credential yang disediakan |
| **WhatsApp Business** | Disiapkan sebagai konfigurasi/future channel | Adapter/service terpisah | Tidak wajib production di Fase 1; aktif bila LLDIKTI menyediakan layanan/library/credential |

**Aturan:** Event notifikasi memilih channel berdasarkan konfigurasi. Kode domain tidak boleh memanggil SMTP/WA langsung; gunakan dispatcher/notification service agar channel bisa dinyalakan/dimatikan per event.

### 11.3 Jenis Notifikasi

| Event | Penerima | In-App | Email | Keterangan |
|-------|----------|:------:|:-----:|------------|
| Cuti diajukan | Step pertama approval chain | ✅ | ✅ | Pegawai mengajukan cuti |
| Cuti menunggu verifikasi berikutnya | Verifikator/approver berikutnya | ✅ | ✅ | Lanjut sesuai chain dinamis |
| Cuti keputusan final `Disetujui` | Pegawai | ✅ | ✅ | Cuti resmi berlaku dan formulir QR tersedia |
| Cuti `Perubahan` / `Ditangguhkan` / `Tidak Disetujui` | Pegawai | ✅ | ✅ | Beserta alasan/keterangan |
| EWS: Kenaikan Pangkat | Pegawai + Admin | ✅ | ✅ | H-90, H-60, H-30 |
| EWS: KGB | Pegawai + Admin | ✅ | ✅ | H-60, H-30, H-14 |
| EWS: Pensiun | Pegawai + Admin | ✅ | ✅ | H-1thn, H-6bln, H-3bln |
| EWS: Kontrak PPPK | Pegawai + Admin | ✅ | ✅ | H-6bln, H-3bln, H-1bln |
| EWS: Satyalancana | Pegawai + Admin | ✅ | ✅ | H-180, H-90, H-30 |
| Data pegawai diubah | Admin (pembuat) | ✅ | ❌ | Konfirmasi audit trail |

### 11.4 User Stories

#### US-NOT-01: Lihat Notifikasi In-App

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** melihat notifikasi di dalam aplikasi,
> **Sehingga** saya segera tahu jika ada yang perlu ditindaklanjuti.

**Acceptance Criteria:**
1. Bell icon di navbar dengan badge count (jumlah notifikasi belum dibaca).
2. Klik bell menampilkan dropdown daftar notifikasi terbaru (10 terakhir).
3. Klik "Lihat Semua" menuju halaman daftar notifikasi lengkap.
4. Notifikasi bisa ditandai "sudah dibaca" (individu atau semua).
5. Klik notifikasi mengarahkan ke halaman terkait (misal: detail cuti).

#### US-NOT-02: Terima Notifikasi Email

> **Sebagai** pegawai,
> **Saya ingin** menerima notifikasi via email,
> **Sehingga** saya tetap informed meskipun tidak membuka aplikasi.

**Acceptance Criteria:**
1. Email dikirim via queue (tidak memblok request utama).
2. Email berisi: judul event, detail, link ke halaman terkait di SIMPEG.
3. Template email menggunakan Bahasa Indonesia.
4. Pengirim: configurable; saat production memakai alamat email operasional resmi dari LLDIKTI.

### 11.5 Struktur Data Notifikasi

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `user_id` | UUID | FK ke pegawai penerima |
| `type` | string | Jenis notifikasi (CUTI_DIAJUKAN, EWS_PANGKAT, dll) |
| `title` | string | Judul notifikasi |
| `body` | text | Isi detail notifikasi |
| `data` | json | Data tambahan (link, ID referensi) |
| `channels` | json | Channel yang dipakai untuk event ini |
| `delivery_status` | json | Status pengiriman per channel, termasuk retry/error |
| `is_read` | boolean | Status baca |
| `read_at` | timestamp | Waktu dibaca |
| `created_at` | timestamp | Waktu dibuat |

---

## 12. Modul 7 — Audit Log

### 12.1 Deskripsi

Setiap perubahan data di SIMPEG dicatat dalam audit log yang immutable. Audit log mencatat siapa, kapan, apa yang berubah (before/after), dan operasi apa yang dilakukan.

### 12.2 Scope Audit Log

| Operasi yang Dicatat | Contoh |
|----------------------|--------|
| **Create** | Tambah pegawai baru, tambah riwayat kepangkatan |
| **Update** | Edit data pribadi, update saldo cuti |
| **Soft Delete** | Nonaktifkan pegawai |
| **Restore** | Aktifkan kembali data pegawai non-aktif |
| **Verifikasi / Keputusan Cuti** | Setiap step approval, rekomendasi, keputusan final, dan perubahan status |
| **Login** | Login berhasil via Keycloak |
| **Logout** | Logout manual atau session timeout |
| **Import Excel/CSV** | Setiap batch import |

### 12.3 User Stories

#### US-AUD-01: Catat Semua Perubahan

> **Sebagai** sistem,
> **Saya ingin** otomatis mencatat setiap perubahan data,
> **Sehingga** ada audit trail yang lengkap dan tidak bisa dimanipulasi.

**Acceptance Criteria:**
1. Setiap operasi yang tercakup (lihat scope) otomatis membuat record audit log.
2. Audit log menyimpan: user ID, timestamp, jenis operasi, nama tabel, record ID, data sebelum (old_values), data sesudah (new_values).
3. Audit log **tidak bisa diedit atau dihapus** oleh siapa pun (termasuk Super Admin).
4. Implementasi menggunakan Laravel model events atau dedicated audit package.

#### US-AUD-02: Lihat Audit Log

> **Sebagai** Super Admin / Admin Kepegawaian,
> **Saya ingin** melihat audit log,
> **Sehingga** saya bisa melacak siapa mengubah data apa dan kapan.

**Acceptance Criteria:**
1. Halaman daftar audit log dengan tabel.
2. Filter berdasarkan: user, periode waktu, jenis operasi, nama tabel/modul.
3. Klik detail menampilkan perbandingan data sebelum dan sesudah (diff view).
4. Pagination dan sorting.
5. Akses: Super Admin dan Admin Kepegawaian.

### 12.4 Struktur Data Audit Log

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `user_id` | UUID | FK ke pegawai yang melakukan aksi |
| `user_name` | string | Snapshot nama user (untuk readability) |
| `event` | enum | CREATE / UPDATE / DELETE / SOFT_DELETE / RESTORE / LOGIN / LOGOUT / VERIFY / DECIDE / CHANGE_REQUESTED / DEFER / NOT_APPROVED / IMPORT / CONFIG_UPDATE |
| `auditable_type` | string | Nama model/tabel (misal: `Employee`, `LeaveRequest`) |
| `auditable_id` | UUID | ID record yang diubah |
| `old_values` | json | Data sebelum perubahan |
| `new_values` | json | Data sesudah perubahan |
| `ip_address` | string | IP address user |
| `user_agent` | string | Browser user agent |
| `created_at` | timestamp | Waktu operasi |

---

## 13. Modul 8 — Dashboard

### 13.1 Deskripsi

Dashboard menyajikan ringkasan informasi kepegawaian secara visual. Akses dan data yang ditampilkan berbeda per role.

### 13.2 Akses Dashboard per Role

| Role | Dashboard |
|------|-----------|
| Super Admin | Dashboard Admin (semua data) + konfigurasi sistem |
| Admin Kepegawaian | Dashboard Admin (semua data) |
| Pimpinan | Dashboard Pimpinan (semua data, read-only) |
| Kepala Bagian | Dashboard Kepala Bagian (data bawahan langsung) |
| Pegawai | Dashboard Pribadi (data sendiri) |

### 13.3 Widget Dashboard Admin / Pimpinan

#### W1: Jumlah Total Pegawai (PNS vs PPPK)

| Aspek | Detail |
|-------|--------|
| **Tipe** | KPI Card + Pie Chart |
| **Data** | Jumlah pegawai aktif, breakdown PNS vs PPPK (vs CPNS jika ada) |
| **Update** | Real-time (setiap load halaman) |

#### W2: KPI Card Kenaikan Pangkat

| Aspek | Detail |
|-------|--------|
| **Tipe** | KPI Card + List |
| **Data** | Siapa saja yang naik pangkat bulan ini dan tahun ini |
| **Detail** | Nama, golongan saat ini → golongan tujuan, tanggal kenaikan |

#### W3: Ringkasan Status Cuti

| Aspek | Detail |
|-------|--------|
| **Tipe** | KPI Card |
| **Data** | Jumlah pengajuan pending, disetujui bulan ini, ditunda |
| **Interaksi** | Klik card menuju halaman daftar cuti terkait |

#### W4: Daftar EWS Aktif

| Aspek | Detail |
|-------|--------|
| **Tipe** | Tabel ringkas (5 teratas) |
| **Data** | Event yang paling urgent, diurutkan dari sisa hari terkecil |
| **Indikator** | Merah (< 30 hari), Kuning (30-90 hari), Hijau (> 90 hari) |
| **Interaksi** | Link ke halaman EWS lengkap |

#### W5: Distribusi Pegawai per Golongan/Jabatan

| Aspek | Detail |
|-------|--------|
| **Tipe** | Bar Chart |
| **Data** | Jumlah pegawai per golongan (I/a — IV/e) |
| **Alternatif** | Bisa toggle ke distribusi per unit kerja |

#### W6: Statistik Audit Log

| Aspek | Detail |
|-------|--------|
| **Tipe** | List (5 terbaru) |
| **Data** | Perubahan data terbaru: siapa, kapan, apa yang berubah |
| **Interaksi** | Link ke audit log detail |

#### W7: Grafik Tren Pegawai

| Aspek | Detail |
|-------|--------|
| **Tipe** | Line Chart |
| **Data** | Jumlah pegawai aktif per bulan/tahun (dari data historis) |
| **Rentang** | 12 bulan terakhir |

### 13.4 Dashboard Pegawai (Pribadi)

| Widget | Data |
|--------|------|
| Profil ringkas | Nama, NIP, Golongan, Jabatan, Foto |
| Saldo cuti | Sisa cuti tahunan, carry-over |
| Status pengajuan cuti | Daftar pengajuan cuti aktif + statusnya |
| EWS pribadi | Peringatan yang relevan untuk diri sendiri |
| Notifikasi terbaru | 5 notifikasi terakhir |

---

## 14. Modul 9 — Laporan & Export

### 14.1 Deskripsi

Fase 1 menyediakan export laporan dasar ke format PDF dan Excel. Selain export fixed, Fase 1 menambahkan export nominatif Excel customizable agar Admin/Pimpinan dapat memilih kolom dan filter baris tanpa menunggu format laporan baru.

### 14.2 Laporan yang Tersedia

> Sesuai Slide 10, SIMPEG mendukung 5 jenis laporan. Di Fase 1, 3 laporan pertama tersedia. 2 sisanya dikembangkan di fase berikutnya.

#### L1: Daftar Nominatif Pegawai (Fase 1)

| Aspek | Detail |
|-------|--------|
| **Format** | PDF dan Excel (.xlsx) |
| **Isi** | Daftar semua pegawai aktif: NIP, Nama, Golongan, Jabatan, Unit Kerja, Jenis Pegawai |
| **Filter** | Per unit kerja, per golongan, per jenis pegawai |
| **Sorting** | Nama, NIP, Golongan |
| **Akses** | Admin Kepegawaian, Pimpinan |

#### L1b: Daftar Nominatif Custom Excel (Fase 1)

| Aspek | Detail |
|-------|--------|
| **Format** | Excel (.xlsx) saja |
| **Isi** | Kolom dipilih pengguna dari daftar kolom yang diizinkan, misalnya NIP, nama, status, jabatan, unit, golongan, pendidikan, tanggal pensiun |
| **Filter** | Filter baris berdasarkan field yang didukung, misalnya status = Tugas Belajar, unit kerja, jenis pegawai, golongan, jabatan |
| **Akses** | Admin Kepegawaian, Pimpinan |
| **Batasan** | PDF custom tidak masuk Fase 1 karena kompleksitas layout |

#### L2: Rekap Cuti (Fase 1)

| Aspek | Detail |
|-------|--------|
| **Format** | PDF dan Excel (.xlsx) |
| **Isi** | Rekap penggunaan cuti per pegawai: nama, jenis cuti, jumlah hari, sisa saldo |
| **Periode** | Per bulan atau per tahun (selectable) |
| **Filter** | Per pegawai, per unit kerja, per jenis cuti |
| **Akses** | Admin Kepegawaian, Pimpinan |

#### L3: Riwayat Kepangkatan (Fase 1)

| Aspek | Detail |
|-------|--------|
| **Format** | PDF dan Excel (.xlsx) |
| **Isi** | Daftar riwayat kepangkatan per pegawai: golongan, TMT, No. SK, tanggal SK |
| **Filter** | Per pegawai, per golongan, per periode |
| **Sorting** | TMT terbaru |
| **Akses** | Admin Kepegawaian, Pimpinan |

#### L4: Pemenuhan 20 JP (Fase 3 — belum tersedia di Fase 1)

> Laporan pemenuhan 20 Jam Pelajaran pengembangan kompetensi per tahun. Akan tersedia setelah modul Tracker 20 JP dikembangkan di Fase 3.

#### L5: Data untuk SIASN (Fase 4 — belum tersedia di Fase 1)

> Export data pegawai dalam format yang kompatibel dengan Sistem Informasi ASN (SIASN) BKN. Akan tersedia setelah integrasi API BKN di Fase 4.

### 14.3 User Stories

#### US-LAP-01: Export Daftar Nominatif Pegawai

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport daftar nominatif pegawai ke PDF atau Excel,
> **Sehingga** saya bisa membuat laporan cetak untuk kebutuhan administrasi.

**Acceptance Criteria:**
1. Tombol "Export PDF" dan "Export Excel" di halaman daftar pegawai.
2. Export mengikuti filter yang sedang aktif.
3. PDF memiliki header: nama instansi, judul laporan, tanggal cetak.
4. Excel berisi data mentah yang bisa diolah lebih lanjut.
5. File ter-download otomatis ke browser.

#### US-LAP-01B: Export Nominatif Custom ke Excel

> **Sebagai** Admin Kepegawaian/Pimpinan,
> **Saya ingin** memilih kolom dan filter pegawai sebelum export Excel,
> **Sehingga** saya bisa membuat laporan nominatif sesuai kebutuhan tanpa mengubah kode.

**Acceptance Criteria:**
1. Halaman export custom menampilkan daftar kolom yang boleh dipilih.
2. Pengguna dapat memilih satu atau lebih filter baris, misalnya status pegawai, unit/tim kerja, jenis pegawai, golongan, jabatan, atau periode pensiun.
3. Output hanya Excel `.xlsx`.
4. Export mengikuti urutan kolom yang dipilih pengguna.
5. Sistem menolak kolom sensitif yang tidak diizinkan untuk laporan.

#### US-LAP-02: Export Rekap Cuti

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport rekap cuti ke PDF atau Excel,
> **Sehingga** saya punya laporan penggunaan cuti yang bisa dilaporkan.

**Acceptance Criteria:**
1. Pilih periode (bulan/tahun) sebelum export.
2. PDF memiliki format laporan resmi dengan tanda tangan digital placeholder.
3. Excel berisi data detail per pegawai per jenis cuti.

#### US-LAP-03: Export Riwayat Kepangkatan

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport riwayat kepangkatan pegawai ke PDF atau Excel,
> **Sehingga** saya punya rekap lengkap kepangkatan untuk pelaporan dan arsip.

**Acceptance Criteria:**
1. Bisa export per pegawai atau keseluruhan.
2. PDF berisi tabel riwayat kepangkatan urut TMT terbaru.
3. Excel berisi data detail termasuk nomor SK dan tanggal SK.

---

## 15. Data Model (ERD)

### 15.1 Tabel Utama

```
┌─────────────────┐     ┌──────────────────────┐
│    employees     │     │  employee_families   │
│─────────────────│     │──────────────────────│
│ id (PK, UUID)   │──┐  │ id (PK, UUID)        │
│ nip              │  │  │ employee_id (FK)     │
│ nik              │  ├──│ nama_anggota         │
│ no_kk            │  │  │ hubungan             │
│ nama_lengkap     │  │  │ nik                  │
│ tempat_lahir     │  │  │ tanggal_lahir        │
│ tanggal_lahir    │  │  │ status_tunjangan     │
│ jenis_kelamin    │  │  └──────────────────────┘
│ agama_id (FK)    │  │
│ status_kawin_id  │  │  ┌──────────────────────┐
│ golongan_darah   │  │  │  rank_histories      │
│ foto             │  │  │  (Kepangkatan)       │
│ jenis_pegawai    │  │  │──────────────────────│
│ status_pegawai_id│  ├──│ id (PK, UUID)        │
│ status_keterangan│  │  │ employee_id (FK)     │
│ alamat           │  │  │ golongan_id (FK)     │
│ no_hp            │  │  │ tmt_pangkat          │
│ email_pribadi    │  │  │ no_sk                │
│ is_kinerja_baik  │  │  │ is_latest            │
│ keycloak_id      │  │  └──────────────────────┘
│ role             │  │
│ deleted_at       │  │
│ created_at       │  │  ┌──────────────────────┐
│ updated_at       │  │  │  position_histories  │
└─────────────────┘  │  │  (Jabatan)           │
                      │  │──────────────────────│
                      ├──│ id (PK, UUID)        │
                      │  │ employee_id (FK)     │
                      │  │ jabatan_id (FK)      │
                      │  │ jenis_jabatan_id(FK) │
                      │  │ unit_kerja_id (FK)   │
                      │  │ kelas_jabatan        │
                      │  │ tmt_jabatan          │
                      │  │ is_latest            │
                      │  └──────────────────────┘
                      │
                      │  ┌──────────────────────┐
                      │  │  salary_histories    │
                      │  │  (KGB)               │
                      │  │──────────────────────│
                      ├──│ id (PK, UUID)        │
                      │  │ employee_id (FK)     │
                      │  │ tmt_kgb              │
                      │  │ gaji_pokok           │
                      │  │ no_sk                │
                      │  │ is_latest            │
                      │  └──────────────────────┘
                      │
                      │  ┌──────────────────────┐
                      │  │  discipline_records  │
                      │  │──────────────────────│
                      ├──│ id (PK, UUID)        │
                      │  │ employee_id (FK)     │
                      │  │ jenis_hukuman        │
                      │  │ tanggal_mulai        │
                      │  │ tanggal_berakhir     │
                      │  │ is_active            │
                      │  └──────────────────────┘
                      │
                      │  ┌──────────────────────┐
                      │  │  education_histories │
                      │  │──────────────────────│
                      ├──│ id (PK, UUID)        │
                      │  │ employee_id (FK)     │
                      │  │ jenjang_id (FK)      │
                      │  │ nama_institusi       │
                      │  │ tahun_lulus          │
                      │  │ no_ijazah            │
                      │  └──────────────────────┘
                      │
                      │  ┌──────────────────────┐
                      │  │  documents           │
                      │  │──────────────────────│
                      ├──│ id (PK, UUID)        │
                      │  │ employee_id (FK)     │
                      │  │ jenis_dokumen        │
                      │  │ nama_dokumen         │
                      │  │ file_path            │
                      │  └──────────────────────┘
                      │
                      │  ┌──────────────────────┐
                      │  │  appointments        │
                      │  │  (Pengangkatan)      │
                      │  │──────────────────────│
                      └──│ id (PK, UUID)        │
                         │ employee_id (FK)     │
                         │ jenis_pengangkatan   │
                         │ tmt_pengangkatan     │
                         │ no_sk                │
                         └──────────────────────┘
```

### 15.2 Tabel Cuti

```
┌────────────────────────┐     ┌──────────────────────────┐
│    leave_requests      │     │  leave_approval_steps    │
│────────────────────────│     │──────────────────────────│
│ id (PK, UUID)          │     │ id (PK, UUID)            │
│ employee_id (FK)       │──┐  │ leave_request_id (FK)    │
│ jenis_cuti_id (FK)     │  │  │ step_order               │
│ tanggal_mulai          │  │  │ step_type                │
│ tanggal_selesai        │  ├──│ approver_id (FK)         │
│ jumlah_hari_kerja      │  │  │ decision_status          │
│ alasan                 │  │  │ keterangan              │
│ lampiran_path          │  │  │ acted_at                 │
│ workflow_status        │  │  └──────────────────────────┘
│ final_decision_status  │  │
│ qr_token               │  │  ┌──────────────────────────┐
│ generated_document_path│  │  │  leave_balances          │
└────────────────────────┘  │  │──────────────────────────│
                            └──│ id (PK, UUID)            │
                               │ employee_id (FK)         │
                               │ tahun                    │
                               │ jatah_dasar              │
                               │ carry_over_n1            │
                               │ hak_tambahan_n2_n1       │
                               │ terpakai                 │
                               │ sisa                     │
                               └──────────────────────────┘
```

| Tabel | Catatan |
|-------|---------|
| `leave_approval_chains` | Template/konfigurasi chain per pegawai/unit; memuat urutan step, tipe step, dan pegawai approver/verifikator |
| `leave_approval_steps` | Snapshot chain per pengajuan agar perubahan konfigurasi tidak mengubah riwayat pengajuan lama |
| `leave_documents` | Menyimpan metadata formulir cuti, QR token, path PDF, dan dokumen eksternal bila `external_approval = true` |
| `leave_balance_adjustments` | Menyimpan koreksi saldo manual beserta alasan dan audit trail |

`decision_status` dan `final_decision_status` hanya memakai label resmi: `Disetujui`, `Perubahan`, `Ditangguhkan`, `Tidak Disetujui`. Keterangan wajib untuk semua selain `Disetujui`.

### 15.3 Tabel EWS & Notifikasi

```
┌──────────────────────┐     ┌──────────────────────┐
│    ews_alerts         │     │   notifications      │
│──────────────────────│     │──────────────────────│
│ id (PK, UUID)        │     │ id (PK, UUID)        │
│ employee_id (FK)     │     │ user_id (FK)         │
│ type (enum)          │     │ type                 │
│ target_date          │     │ title                │
│ interval_days        │     │ body                 │
│ notified_at          │     │ data (json)          │
│ followup_status      │     │ channels (json)      │
│ handled_at           │     │ delivery_status(json)│
│ handled_by           │     │ is_read              │
│ created_at           │     │ read_at              │
└──────────────────────┘     │ created_at           │
                             └──────────────────────┘

┌──────────────────────┐
│    audit_logs        │
│──────────────────────│
│ id (PK, UUID)        │
│ user_id (FK)         │
│ user_name            │
│ event (enum)         │
│ auditable_type       │
│ auditable_id         │
│ old_values (json)    │
│ new_values (json)    │
│ ip_address           │
│ user_agent           │
│ created_at           │
└──────────────────────┘
```

`ews_alerts.type` mencakup `KENAIKAN_PANGKAT`, `KGB`, `BUP`, `KONTRAK_PPPK`, dan `SATYALANCANA`. `followup_status` minimal mendukung `aktif`, `ditangani`, `tidak_perlu`, dan `kedaluwarsa`.

### 15.4 Tabel Supervisor Mapping

```
┌──────────────────────────┐
│  supervisor_assignments  │
│──────────────────────────│
│ id (PK, UUID)            │
│ employee_id (FK)         │
│ supervisor_id (FK)       │
│ tanggal_mulai            │
│ tanggal_berakhir         │
│ created_at               │
│ updated_at               │
└──────────────────────────┘
```

### 15.5 Tabel RBAC Internal

Keycloak tidak menyimpan role dan permission aplikasi. Setelah login SSO berhasil, SIMPEG membaca role dan permission dari database internal.

| Tabel | Field Utama | Keterangan |
|-------|-------------|------------|
| `roles` | `id`, `name`, `guard_name`, `description` | Daftar role aplikasi: Super Admin, Admin Kepegawaian, Pimpinan, Kepala Bagian, Pegawai |
| `permissions` | `id`, `name`, `module`, `description` | Daftar permission per modul/aksi |
| `role_permissions` | `role_id`, `permission_id` | Pivot permission yang dimiliki role |
| `employee_roles` / `employees.role_id` | `employee_id`, `role_id` | Fase awal dapat memakai satu role utama per pegawai; pivot disiapkan bila perlu ekspansi |

**Aturan:** Keycloak hanya menghasilkan identitas login. Hak akses fitur, redirect dashboard, akses menu, dan otorisasi route harus mengacu ke RBAC internal SIMPEG.

---

## 16. Reference Tables (Seed Data)

Master data berikut menjadi seed awal dan harus dapat dikelola oleh Super Admin/Admin yang berwenang. Jumlah record dapat bertambah tanpa perubahan kode.

### 16.1 ref_golongan

| Kode | Nama |
|------|------|
| I/a | Juru Muda |
| I/b | Juru Muda Tingkat 1 |
| I/c | Juru |
| I/d | Juru Tingkat 1 |
| II/a | Pengatur Muda |
| II/b | Pengatur Muda Tingkat 1 |
| II/c | Pengatur |
| II/d | Pengatur Tingkat 1 |
| III/a | Penata Muda |
| III/b | Penata Muda Tingkat 1 |
| III/c | Penata |
| III/d | Penata Tingkat 1 |
| IV/a | Pembina |
| IV/b | Pembina Tingkat 1 |
| IV/c | Pembina Utama Muda |
| IV/d | Pembina Utama Madya |
| IV/e | Pembina Utama |

### 16.2 ref_jenis_jabatan

| ID | Nama | Maks Usia Pensiun | Catatan |
|----|------|:-----------------:|---------|
| 1 | Struktural | 60 | Dapat disesuaikan berdasarkan jabatan detail |
| 2 | Fungsional Tertentu | 58 / 60 | Mengikuti jenjang atau jabatan detail |
| 3 | Fungsional Umum / Pelaksana | 58 | Default umum |
| 4 | Jabatan Akademik / Dosen | 65 / 70 | Disiapkan untuk modul Dosen DPK di fase berikutnya |

> Field `maks_usia_pensiun` wajib tersedia agar EWS pensiun tidak mengunci usia pensiun secara statis di kode. Untuk jabatan yang lebih detail, nilai BUP dapat dioverride oleh `ref_jabatan` atau `ref_bup`.

### 16.3 ref_jabatan

`nama_jabatan` harus dipilih dari reference, bukan free text.

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `jenis_jabatan_id` | FK | Kategori jabatan untuk dependent select |
| `nama_jabatan` | string | Nama resmi jabatan |
| `eselon_id` | FK nullable | Untuk jabatan struktural |
| `default_bup` | integer nullable | Override BUP bila berbeda dari jenis jabatan |
| `is_active` | boolean | Bisa dinonaktifkan tanpa menghapus riwayat |

Contoh kategori: Fungsional Umum/Pelaksana, Fungsional Tertentu, Struktural, dan Jabatan Akademik/Dosen untuk pengembangan Dosen DPK di fase berikutnya. Untuk dosen, referensi jabatan akademik dapat memuat Asisten Ahli, Lektor, Lektor Kepala, dan Guru Besar.

### 16.4 ref_status_pegawai

Status pegawai tidak boleh berupa enum hardcoded. Seed awal minimal:

| Kode | Nama | Kelompok |
|------|------|----------|
| AKTIF | Aktif | Aktif |
| NONAKTIF | Nonaktif | Nonaktif |
| PENSIUN | Pensiun | Nonaktif |
| MUTASI | Mutasi | Nonaktif |
| CLTN | Cuti Luar Tanggungan Negara | Nonaktif |
| PERPANJANGAN_CLTN | Perpanjangan CLTN | Nonaktif |
| TUGAS_BELAJAR | Tugas Belajar | Aktif/khusus |
| PEMBERHENTIAN_SEMENTARA | Pemberhentian Sementara | Nonaktif |
| WAJIB_MILITER | Wajib Militer | Nonaktif/khusus |
| HILANG | PNS Dinyatakan Hilang | Nonaktif/khusus |

Setiap perubahan status pegawai wajib dapat menyimpan `status_keterangan` dan tercatat di audit log.

### 16.5 ref_eselon

| Kode | Nama |
|------|------|
| I.a | Eselon I.a |
| I.b | Eselon I.b |
| II.a | Eselon II.a |
| II.b | Eselon II.b |
| III.a | Eselon III.a |
| III.b | Eselon III.b |
| IV.a | Eselon IV.a |
| IV.b | Eselon IV.b |

### 16.6 ref_unit_kerja

`ref_unit_kerja` harus hierarkis dan tidak mengunci istilah "unit kerja" saja. UI boleh memakai label "Unit/Tim Kerja".

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `parent_id` | UUID nullable | FK ke `ref_unit_kerja.id`; null untuk root |
| `level` | integer | Level hierarki untuk sorting/tampilan |
| `nama` | string | Nama unit/tim/urusan |
| `jenis_unit` | enum/string | Contoh: lembaga, bagian, tim_kerja, urusan, sub_unit |
| `is_active` | boolean | Soft disable |

Contoh struktur awal:

| Level | Nama | Parent |
|:----:|------|--------|
| 0 | Kepala Lembaga | null |
| 1 | Kepala Bagian Umum | Kepala Lembaga |
| 1 | Ketua Tim Kerja Substansi | Kepala Lembaga |
| 2 | Urusan Organisasi Tata Laksana dan SDM | Kepala Bagian Umum |
| 2 | Urusan Keuangan | Kepala Bagian Umum |
| 2 | Urusan Humas / Layanan Pendukung | Kepala Bagian Umum |

Daftar final nama tim kerja, urusan, dan sub-unit mengikuti data bagian kepegawaian LLDIKTI.

### 16.7 ref_jenis_cuti

| ID | Nama | Khusus PNS |
|----|------|:----------:|
| 1 | Cuti Tahunan | Tidak |
| 2 | Cuti Sakit | Tidak |
| 3 | Cuti Melahirkan | Tidak |
| 4 | Cuti Karena Alasan Penting | Tidak |
| 5 | Cuti Besar | Ya |
| 6 | Cuti Luar Tanggungan Negara (CLTN) | Ya |

### 16.8 ref_agama

| ID | Nama |
|----|------|
| 1 | Islam |
| 2 | Kristen Protestan |
| 3 | Katolik |
| 4 | Hindu |
| 5 | Buddha |
| 6 | Konghucu |

### 16.9 ref_jenis_kelamin

| ID | Kode | Nama |
|----|------|------|
| 1 | L | Laki-laki |
| 2 | P | Perempuan |

### 16.10 ref_status_perkawinan

| ID | Nama |
|----|------|
| 1 | Belum Menikah |
| 2 | Menikah |
| 3 | Duda / Janda |

### 16.11 ref_jenjang_pendidikan

| ID | Nama |
|----|------|
| 1 | SD |
| 2 | SMP |
| 3 | SMA / SMK / Sederajat |
| 4 | D1 |
| 5 | D2 |
| 6 | D3 |
| 7 | D4 / S1 |
| 8 | S2 / Profesi |
| 9 | S3 |

### 16.12 ref_hari_libur

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `tanggal` | date | Tanggal libur |
| `nama` | string | Nama hari libur |
| `tahun` | year | Tahun |
| `is_cuti_bersama` | boolean | True jika cuti bersama |

> Diinput manual oleh Admin/Super Admin setiap awal tahun berdasarkan SKB Menteri.

### 16.13 ref_bup (Batas Usia Pensiun)

Reference ini dipakai bila aturan BUP perlu lebih detail dari `ref_jenis_jabatan.maks_usia_pensiun`.
Hasil meeting menyepakati bahwa BUP tidak di-hardcode di aplikasi.

| Jenis Jabatan | BUP (Tahun) |
|---------------|:-----------:|
| Pelaksana / Fungsional Umum | 58 |
| Fungsional Ahli Pertama | 58 |
| Fungsional Ahli Muda | 58 |
| Fungsional Ahli Madya | 60 |
| Pimpinan Tinggi | 60 |
| Struktural (Eselon I-II) | 60 |

> **Catatan:** Data BUP final mengikuti daftar jabatan yang diberikan oleh bagian kepegawaian LLDIKTI. Bila ada jabatan dengan usia pensiun berbeda, Admin harus dapat memperbaruinya melalui reference table.

### 16.14 ref_notification_channels

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | UUID | PK |
| `code` | string | `in_app`, `email`, `whatsapp_business`, dll |
| `name` | string | Nama channel |
| `is_enabled` | boolean | Aktif/nonaktif |
| `config` | json | Konfigurasi aman; credential sensitif tetap di `.env`/secret manager |

Channel notifikasi harus bisa dinyalakan/dimatikan tanpa mengubah kode domain.

---

## 17. API Specification

### 17.1 Konvensi API

Meskipun Fase 1 menggunakan Laravel Blade (server-side rendering), semua logika bisnis diimplementasikan sebagai **controller methods** yang juga bisa dipanggil sebagai API JSON di masa depan.

| Aspek | Konvensi |
|-------|----------|
| **URL Pattern** | `/api/v1/{resource}` (untuk JSON) atau `/{resource}` (untuk Blade) |
| **Method** | GET (list/show), POST (create), PUT (update), DELETE (soft-delete) |
| **Response** | JSON untuk API, Blade view untuk web |
| **Authentication** | Middleware auth via Keycloak session |
| **Authorization** | Middleware RBAC internal: role dan permission dari database SIMPEG |
| **Pagination** | `?page=1&per_page=25` |
| **Search** | `?search=keyword` |
| **Filter** | `?filter[field]=value` |

### 17.2 Route Map

#### Auth Routes

| Method | Route | Controller | Role | Keterangan |
|--------|-------|------------|------|------------|
| GET | `/auth/redirect` | AuthController@redirect | Public | Redirect ke Keycloak |
| GET | `/auth/callback` | AuthController@callback | Public | Callback dari Keycloak |
| POST | `/auth/logout` | AuthController@logout | Auth | Logout + SSO logout |

#### Employee Routes

| Method | Route | Controller | Role | Keterangan |
|--------|-------|------------|------|------------|
| GET | `/pegawai` | EmployeeController@index | Admin+ | Daftar pegawai |
| GET | `/pegawai/create` | EmployeeController@create | Admin | Form tambah |
| POST | `/pegawai` | EmployeeController@store | Admin | Simpan pegawai baru |
| GET | `/pegawai/{id}` | EmployeeController@show | Admin+/Self | Detail pegawai |
| GET | `/pegawai/{id}/edit` | EmployeeController@edit | Admin | Form edit |
| PUT | `/pegawai/{id}` | EmployeeController@update | Admin | Update data |
| DELETE | `/pegawai/{id}` | EmployeeController@destroy | Admin | Soft delete |
| POST | `/pegawai/{id}/restore` | EmployeeController@restore | Admin | Restore |
| GET | `/profil-saya` | EmployeeController@myProfile | Pegawai | Data sendiri |

#### History Routes (Riwayat)

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/pegawai/{id}/kepangkatan` | RankHistoryController@index | Admin+/Self |
| POST | `/pegawai/{id}/kepangkatan` | RankHistoryController@store | Admin |
| GET | `/pegawai/{id}/jabatan` | PositionHistoryController@index | Admin+/Self |
| POST | `/pegawai/{id}/jabatan` | PositionHistoryController@store | Admin |
| GET | `/pegawai/{id}/kgb` | SalaryHistoryController@index | Admin+/Self |
| POST | `/pegawai/{id}/kgb` | SalaryHistoryController@store | Admin |
| GET | `/pegawai/{id}/disiplin` | DisciplineController@index | Admin+/Self |
| POST | `/pegawai/{id}/disiplin` | DisciplineController@store | Admin |

#### Leave Routes (Cuti)

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/cuti` | LeaveController@index | All (filtered by role) |
| POST | `/cuti` | LeaveController@store | Pegawai |
| GET | `/cuti/{id}` | LeaveController@show | Related parties |
| POST | `/cuti/{id}/decision` | LeaveDecisionController@store | Current step actor |
| GET | `/cuti/{id}/document` | LeaveDocumentController@show | Related parties |
| GET | `/cuti/verify/{token}` | LeaveVerificationController@show | Public |
| GET | `/cuti/saldo` | LeaveBalanceController@show | Pegawai (self) |
| GET | `/admin/cuti/saldo` | LeaveBalanceController@index | Admin |
| POST | `/admin/cuti/saldo/{employeeId}/adjust` | LeaveBalanceController@adjust | Admin |

#### EWS Routes

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/ews` | EwsController@index | Admin |
| GET | `/ews/dashboard` | EwsController@dashboard | Admin+ |

#### Notification Routes

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/notifikasi` | NotificationController@index | Auth |
| POST | `/notifikasi/{id}/read` | NotificationController@markRead | Auth |
| POST | `/notifikasi/read-all` | NotificationController@markAllRead | Auth |

#### Import/Export Routes

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/import/template` | ImportController@downloadTemplate | Admin |
| POST | `/import/preview` | ImportController@preview | Admin |
| POST | `/import/execute` | ImportController@execute | Admin |
| GET | `/export/pegawai` | ExportController@employees | Admin+ |
| POST | `/export/pegawai/custom` | ExportController@customEmployees | Admin+ |
| GET | `/export/cuti` | ExportController@leaves | Admin+ |

#### Dashboard & Audit Routes

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/dashboard` | DashboardController@index | Auth (role-based view) |
| GET | `/audit-log` | AuditLogController@index | Admin+ |
| GET | `/audit-log/{id}` | AuditLogController@show | Admin+ |

#### Admin/Config Routes

| Method | Route | Controller | Role |
|--------|-------|------------|------|
| GET | `/admin/hari-libur` | HolidayController@index | SuperAdmin |
| POST | `/admin/hari-libur` | HolidayController@store | SuperAdmin |
| PUT | `/admin/hari-libur/{id}` | HolidayController@update | SuperAdmin |
| DELETE | `/admin/hari-libur/{id}` | HolidayController@destroy | SuperAdmin |
| GET | `/admin/supervisor` | SupervisorController@index | Admin |
| POST | `/admin/supervisor` | SupervisorController@store | Admin |
| GET | `/admin/approval-chain/cuti` | LeaveApprovalConfigController@index | SuperAdmin |
| POST | `/admin/approval-chain/cuti` | LeaveApprovalConfigController@store | SuperAdmin |
| GET | `/admin/notification-channels` | NotificationChannelController@index | SuperAdmin |
| PUT | `/admin/notification-channels/{id}` | NotificationChannelController@update | SuperAdmin |

---

## 18. Non-Functional Requirements

### 18.1 Performa

| Aspek | Target |
|-------|--------|
| Page load time | < 3 detik |
| API response time | < 1 detik (rata-rata) |
| Concurrent users | Mendukung 46 user simultan |
| Database query | Optimized dengan indexing pada kolom yang sering di-query |

### 18.2 Ketersediaan

| Aspek | Target |
|-------|--------|
| Uptime | 99% (mengizinkan downtime untuk maintenance) |
| Backup | Database backup otomatis harian |
| Recovery | Bisa restore dari backup terakhir |

### 18.3 Responsiveness

| Aspek | Target |
|-------|--------|
| Desktop | Fully functional pada resolusi ≥ 1024px |
| Tablet | Fully functional pada resolusi ≥ 768px |
| Mobile | Fully functional pada resolusi ≥ 375px |
| Browser | Chrome, Firefox, Safari, Edge (versi terbaru) |

### 18.4 Aksesibilitas

| Aspek | Target |
|-------|--------|
| Akses | Via internet (bukan hanya intranet) |
| HTTPS | Wajib (SSL/TLS) |
| Session timeout | 30 menit inaktif (configurable) |

---

## 19. Keamanan & Compliance

### 19.1 Autentikasi & Otorisasi

| Aspek | Implementasi |
|-------|-------------|
| Autentikasi | Keycloak SSO (OAuth 2.0 / OpenID Connect) |
| Otorisasi | Role dan permission internal di database SIMPEG, dijalankan melalui middleware Laravel |
| Session | Laravel session dengan encryption |
| CSRF | Laravel CSRF token protection |

### 19.2 Data Protection

| Aspek | Implementasi |
|-------|-------------|
| Enkripsi transport | HTTPS/TLS wajib |
| Enkripsi storage | Field sensitif (NIK, No. KK) di-encrypt at rest (Laravel Crypt) |
| File upload | MIME type validation, size limit, anti-malware filename |
| SQL Injection | Laravel Eloquent ORM (parameterized queries) |
| XSS | Laravel Blade auto-escaping |

### 19.3 Audit & Compliance

| Aspek | Implementasi |
|-------|-------------|
| Audit trail | Semua operasi tercatat (immutable) |
| Soft delete | Data tidak dihapus permanen secara default |
| Regulasi | PP 11/2017, PP 49/2018, PP 94/2021, PP 99/2000 |

---

## 20. Batasan & Asumsi

### 20.1 Asumsi

1. **Keycloak SSO disediakan oleh LLDIKTI** — tim pengembang membutuhkan trait/fungsi, Client ID, Client Secret, URL Keycloak, dan akun testing.
2. **Mapping user Keycloak ↔ pegawai via email atau ID Keycloak** — email/identifier dari Keycloak digunakan untuk mencocokkan dengan data pegawai di SIMPEG.
3. **Role dan permission dikelola di SIMPEG** — Keycloak tidak menjadi sumber otorisasi fitur aplikasi.
4. **Data pegawai awal tersedia dalam CSV/Excel** — sample dapat digunakan untuk import awal, tetapi field lengkap tetap mengikuti struktur PRD.
5. **Server/hosting dan domain production disiapkan LLDIKTI pada tahap deployment** — development tidak menunggu server production.
6. **Email production menggunakan email operasional LLDIKTI/Gmail resmi** — selama development testing email dapat memakai Mailpit.
7. **WhatsApp Business belum wajib aktif di Fase 1** — sistem hanya menyiapkan desain channel-configurable sampai layanan/credential disediakan.
8. **Referensi jabatan, pangkat, golongan, unit kerja hierarkis, status pegawai, dan BUP disediakan bagian kepegawaian** — sistem menyediakan struktur dan CRUD reference.
9. **Saldo cuti awal dan riwayat N-1/N-2 disediakan/diinput Admin Kepegawaian** — dibutuhkan agar carry-over tahun berjalan akurat.
10. **Format formulir cuti resmi dan konten halaman QR diverifikasi LLDIKTI** — tim pengembang menyediakan generator dan halaman verifikasi.

### 20.2 Batasan

1. **Single-tenant** — sistem hanya untuk LLDIKTI XVI, tidak ada multi-tenant.
2. **~46 pegawai** — arsitektur dioptimalkan untuk skala kecil.
3. **Tim magang** — PRD ditulis dengan detail teknis yang cukup untuk developer junior.
4. **Fase 1 saja** — fitur di luar scope Fase 1 tidak diimplementasikan.
5. **Tidak ada login manual** — autentikasi sepenuhnya via Keycloak.
6. **Dosen DPK belum menjadi modul operasional Fase 1** — struktur jabatan disiapkan agar tidak menutup pengembangan, tetapi alur khusus dosen dibuat di fase berikutnya.
7. **Tanda tangan elektronik tersertifikasi belum wajib** — QR Code verifikasi SIMPEG menjadi mekanisme verifikasi dokumen cuti Fase 1.

### 20.3 Dependensi

| Dependensi | Pihak |
|-----------|-------|
| Trait/fungsi Keycloak, Client ID, Client Secret, URL Keycloak, akun testing SSO | Tim IT LLDIKTI XVI |
| Server/hosting production | Tim IT LLDIKTI XVI |
| Domain & SSL production | Tim IT LLDIKTI XVI |
| User/password email operasional production atau Gmail resmi | Tim IT LLDIKTI XVI |
| Layanan/library/credential WhatsApp Business jika ingin diaktifkan | Tim IT LLDIKTI XVI |
| Data pegawai awal (CSV/Excel) | Admin Kepegawaian LLDIKTI XVI |
| Daftar unit/tim kerja hierarkis, jabatan, pangkat, golongan, status pegawai, dan BUP | Admin Kepegawaian LLDIKTI XVI |
| Saldo cuti awal, riwayat cuti N-1/N-2, dan aturan koreksi saldo awal | Admin Kepegawaian LLDIKTI XVI |
| Format formulir cuti resmi dan narasi verifikasi QR | Admin Kepegawaian LLDIKTI XVI |
| Daftar hari libur nasional 2026 | Admin / Super Admin |

---

## 21. Fase Selanjutnya (Out of Scope)

Untuk transparansi, berikut fitur yang direncanakan di fase berikutnya:

### Fase 2 — Aktivitas Harian
- Self-service pegawai (edit data sendiri)
- Pending changes (approval perubahan data oleh admin)
- Klaim kehadiran + kuota bulanan
- Surat tugas
- Kalender virtual (per pegawai / per tim)
- Log harian
- Aktivasi penuh WhatsApp Business notification bila layanan/credential sudah tersedia

### Fase 3 — Kinerja
- SKP & Rencana Hasil Kerja (RHK)
- Log harian ↔ RHK (many-to-many)
- Evaluasi kinerja oleh kepala bagian
- Riwayat pelatihan
- Tracker 20 JP pengembangan kompetensi
- Arsip dokumen (modul terpisah)
- Laporan PDF & Excel lengkap

### Fase 4 — Integrasi
- Kalkulator IP-ASN (4 dimensi penilaian)
- Asesmen kompetensi
- Ekspor data ke format SIASN (CSV/JSON BKN)
- Integrasi API SIASN (jika akses tersedia)
- Modul khusus Dosen DPK dan jabatan akademik bila diprioritaskan LLDIKTI

---

## 22. Regulasi yang Direferensi

| Regulasi | Konteks dalam Fase 1 |
|----------|----------------------|
| **PP 11/2017 jo PP 17/2020** | Manajemen PNS — jenis cuti, kuota, carry-over, aturan approval |
| **PP 49/2018** | Manajemen PPPK — BUP, kontrak PPPK, pembatasan jenis cuti |
| **PP 94/2021** | Disiplin PNS — jenis hukuman, dampak terhadap eligibility kenaikan pangkat |
| **PP 99/2000** | Kenaikan Pangkat — syarat 4 tahun TMT, syarat kinerja baik |
| **Ketentuan Satyalancana Karya Satya** | EWS milestone masa kerja 10/20/30 tahun; detail eligibility divalidasi Admin Kepegawaian |

> **Disclaimer:** Referensi regulasi berdasarkan yang disebutkan di dokumen sumber (paparan dan diagram). Tidak diklaim sebagai daftar lengkap semua regulasi terkait kepegawaian ASN.

---

## 23. Glosarium

| Istilah | Definisi |
|---------|----------|
| **ASN** | Aparatur Sipil Negara |
| **BKN** | Badan Kepegawaian Negara |
| **BUP** | Batas Usia Pensiun |
| **CLTN** | Cuti Luar Tanggungan Negara |
| **CPNS** | Calon Pegawai Negeri Sipil |
| **EWS** | Early Warning System — sistem peringatan dini |
| **KGB** | Kenaikan Gaji Berkala |
| **LLDIKTI** | Lembaga Layanan Pendidikan Tinggi |
| **NIP** | Nomor Induk Pegawai |
| **NIK** | Nomor Induk Kependudukan |
| **No. KK** | Nomor Kartu Keluarga |
| **PNS** | Pegawai Negeri Sipil |
| **PPPK** | Pegawai Pemerintah dengan Perjanjian Kerja |
| **PYBMC** | Pejabat Yang Berwenang Memberikan Cuti |
| **QR Verification** | Halaman verifikasi SIMPEG yang dibuka dari QR Code pada dokumen cuti |
| **Satyalancana** | Tanda kehormatan masa kerja ASN; di Fase 1 digunakan sebagai trigger EWS 10/20/30 tahun |
| **RBAC** | Role-Based Access Control |
| **SIASN** | Sistem Informasi ASN (BKN) |
| **SIMPEG** | Sistem Informasi Manajemen Kepegawaian |
| **SK** | Surat Keputusan |
| **SKP** | Sasaran Kinerja Pegawai |
| **SSO** | Single Sign-On |
| **TMT** | Terhitung Mulai Tanggal |
| **UAT** | User Acceptance Testing |
| **UUID** | Universally Unique Identifier |

---

*Dokumen ini disusun berdasarkan transkrip paparan SIMPEG, rekap PDF presentasi, dan diagram alur v0.4 SIMPEG LLDIKTI Wilayah XVI.*
