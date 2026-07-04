# Issues — SIMPEG Fase 1 (Core)
## LLDIKTI Wilayah XVI

> Dokumen ini berisi daftar issues yang siap dipindahkan ke GitHub Issues / Notion Board.
> Setiap issue diturunkan dari User Stories dan dipecah menjadi task teknis yang actionable.
> Sinkron dengan PRD-SIMPEG-Fase1-Core.md v1.2: Keycloak hanya untuk SSO, RBAC internal aplikasi, approval cuti memakai chain dinamis per pegawai/unit, status cuti memakai label resmi, PostgreSQL development via container, production diarahkan ke Podman, notifikasi channel-configurable, dan laporan mendukung export nominatif Excel custom.

---

## Label Reference

| Label | Warna | Keterangan |
|-------|-------|-----------|
| `epic:auth` | 🔵 Blue | Modul Autentikasi & SSO |
| `epic:pegawai` | 🟢 Green | Modul Data Pegawai |
| `epic:import` | 🟡 Yellow | Modul Import Excel/CSV |
| `epic:cuti` | 🟠 Orange | Modul Manajemen Cuti |
| `epic:ews` | 🔴 Red | Modul Early Warning System |
| `epic:notifikasi` | 🟣 Purple | Modul Notifikasi |
| `epic:audit` | 💗 Pink | Modul Audit Log |
| `epic:dashboard` | ⚪ Default | Modul Dashboard |
| `epic:laporan` | 🟤 Brown | Modul Laporan & Export |
| `priority:P0` | 🔴 | Must Have |
| `priority:P1` | 🟡 | Should Have |
| `type:feature` | — | Fitur baru |
| `type:infra` | — | Setup infrastruktur |
| `type:ui` | — | Frontend / UI |
| `type:engine` | — | Business logic / engine |
| `type:test` | — | Testing |
| `sprint:1` s/d `sprint:7` | — | Sprint assignment |

---

# SPRINT 1 — FONDASI (9–20 Juni 2026)

---

## Issue #1 · Setup Project Laravel & Environment

| Field | Detail |
|-------|--------|
| **Story** | — (Infrastruktur) |
| **Labels** | `type:infra`, `sprint:1`, `priority:P0` |
| **Assignee** | Dion Kobi |
| **Story Points** | 3 |

**Deskripsi:**
Setup project Laravel baru beserta konfigurasi environment dasar untuk proyek SIMPEG.

**Tasks:**
- [ ] Inisialisasi project Laravel 12 (`composer create-project laravel/laravel simpeg`)
- [ ] Setup `.env` — database PostgreSQL, app name, timezone Asia/Makassar
- [ ] Setup koneksi PostgreSQL dan pastikan `php artisan migrate` berjalan
- [ ] Install dan konfigurasi packages awal:
  - `socialite` / `laravel-keycloak-web-guard` (SSO)
  - `owen-it/laravel-auditing` (Audit Log)
  - `maatwebsite/excel` (Export Excel)
  - `barryvdh/laravel-dompdf` (Export PDF)
  - `spatie/laravel-permission` (RBAC — opsional)
- [ ] Setup queue database driver Laravel untuk email dan scheduler EWS
- [ ] Buat file compose/container development (PostgreSQL 17 + Mailpit opsional)
- [ ] Buat `README.md` dengan instruksi setup lokal
- [ ] Setup `.editorconfig` dan `.php-cs-fixer` untuk konsistensi kode
- [ ] Push ke GitHub repository + setup branch `main` dan `develop`

**Definition of Done:**
- [ ] `php artisan serve` berjalan tanpa error
- [ ] Database connection verified
- [ ] Semua anggota tim bisa clone dan run project lokal

---

## Issue #2 · Setup Keycloak Client & SSO Middleware

| Field | Detail |
|-------|--------|
| **Story** | US-1.1 · Login via Keycloak SSO |
| **Labels** | `epic:auth`, `type:infra`, `sprint:1`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 5 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Implementasi login SSO menggunakan Keycloak. User mengakses SIMPEG → redirect ke Keycloak login → callback → session Laravel terbentuk. Role dan permission aplikasi tetap dibaca dari database SIMPEG.
User pertama yang berhasil login melalui SSO saat tabel user SIMPEG masih kosong otomatis dibootstrap sebagai `Super Admin`. Setelah bootstrap, user yang berhasil login tetapi belum memiliki role internal SIMPEG tetap memiliki session, namun akses dashboard/fitur normal ditolak dengan HTTP `403 Access Forbidden` dan pesan: *"Akun Anda belum memiliki role SIMPEG. Hubungi Admin."*

**Tasks:**
- [ ] Adaptasi trait/fungsi Keycloak dari LLDIKTI atau konfigurasi `socialite` + Keycloak provider
- [ ] Konfigurasi `.env`: KEYCLOAK_BASE_URL, KEYCLOAK_REALM, KEYCLOAK_CLIENT_ID, KEYCLOAK_CLIENT_SECRET, KEYCLOAK_REDIRECT_URI
- [ ] Buat middleware `EnsureKeycloakAuthenticated` — redirect ke Keycloak jika belum login
- [ ] Buat `AuthController` dengan method:
  - `redirectToKeycloak()` — redirect ke Keycloak login page
  - `handleCallback()` — proses callback dari Keycloak, buat/update session
  - `logout()` — hapus session Laravel + trigger Keycloak end-session
- [ ] Saat callback berhasil:
  - Cari pegawai di tabel `employees` berdasarkan email Keycloak
  - Jika ditemukan dan belum ada user lokal sama sekali → login, simpan `keycloak_id`, bootstrap role `Super Admin`
  - Jika ditemukan dan user lokal sudah ada → login, simpan `keycloak_id` jika belum ada, role tetap diatur dari SIMPEG
  - Jika role SIMPEG kosong / belum diset / tidak valid → session boleh aktif tetapi dashboard/fitur normal harus mengembalikan `403 Access Forbidden` dengan pesan hubungi admin
  - Jika tidak ditemukan → tampilkan halaman "Akun belum terdaftar"
- [ ] Abaikan role dasar / role claim dari SSO untuk otorisasi fitur; RBAC tetap memakai database SIMPEG
- [ ] Buat halaman "Akun belum terdaftar" (`resources/views/auth/unregistered.blade.php`)
- [ ] Register middleware di `bootstrap/app.php`
- [ ] Buat route group yang di-protect middleware auth

**Acceptance Criteria (dari US-1.1):**
- [ ] AC-1: Redirect ke Keycloak jika belum login
- [ ] AC-2: Session terbentuk setelah login sukses
- [ ] AC-3: Mapping via email ke tabel employees
- [ ] AC-4: Halaman info jika belum terdaftar
- [ ] AC-5: User SSO pertama dibootstrap sebagai Super Admin jika belum ada user lokal SIMPEG
- [ ] AC-6: User dengan role kosong/tidak valid mendapat 403 Access Forbidden dan pesan hubungi admin
- [ ] AC-7: Redirect ke dashboard sesuai role jika role SIMPEG valid
- [ ] AC-8: Login dicatat di audit log

---

## Issue #3 · Implementasi Logout & Session Management

| Field | Detail |
|-------|--------|
| **Story** | US-1.2 · Logout |
| **Labels** | `epic:auth`, `type:feature`, `sprint:1`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 2 |
| **Dependensi** | Issue #2 |

**Deskripsi:**
Implementasi logout yang menghapus session Laravel dan juga memicu single logout di Keycloak.

**Tasks:**
- [ ] Tambahkan method `logout()` di `AuthController`:
  - Hapus session Laravel (`Auth::logout()`, `session()->invalidate()`)
  - Redirect ke Keycloak end-session endpoint
- [ ] Tambahkan tombol "Keluar" di navbar layout (`layouts/app.blade.php`)
- [ ] Pastikan setelah logout, user kembali ke halaman login Keycloak
- [ ] Catat event logout di audit log

**Acceptance Criteria (dari US-1.2):**
- [ ] AC-1: Tombol "Keluar" ada di navbar
- [ ] AC-2: Session Laravel dihapus
- [ ] AC-3: Single logout Keycloak
- [ ] AC-4: Redirect ke halaman login
- [ ] AC-5: Audit log tercatat

---

## Issue #4 · Mapping User Keycloak & RBAC

| Field | Detail |
|-------|--------|
| **Story** | US-1.4 · Mapping User Keycloak ke Pegawai |
| **Labels** | `epic:auth`, `type:feature`, `sprint:1`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 5 |
| **Dependensi** | Issue #2 |

**Deskripsi:**
Halaman admin untuk memetakan akun Keycloak ke pegawai dan menetapkan role/permission internal SIMPEG.
Role dasar dari SSO tidak menjadi sumber otorisasi fitur; Super Admin menetapkan role internal SIMPEG melalui modul ini. User SSO baru setelah bootstrap dapat tercatat dengan role kosong sampai role ditetapkan di SIMPEG.

**Tasks:**
- [ ] Buat migration tabel `employees` (jika belum):
  - Tambah kolom `keycloak_id` (nullable, unique)
  - Tambah kolom `role_id` atau role enum awal: super_admin, admin_kepegawaian, pimpinan, kepala_bagian, pegawai
- [ ] Buat migration RBAC internal:
  - `roles`
  - `permissions`
  - `role_permissions`
  - pivot employee-role jika memakai relasi, atau `employees.role_id` bila satu role per pegawai
- [ ] Buat model `Employee` dengan fillable fields
- [ ] Buat `UserMappingController` dengan method:
  - `index()` — daftar pegawai + status mapping
  - `update()` — update keycloak_id dan role
- [ ] Buat view `admin/user-mapping/index.blade.php`:
  - Tabel: Nama, NIP, Email, Keycloak ID, Role, Status (Terhubung/Belum)
  - Tombol "Edit" per baris → modal/form edit
- [ ] Buat middleware `RoleMiddleware` — cek role user untuk akses halaman
- [ ] Pastikan middleware membaca role/permission dari database SIMPEG, bukan dari data otorisasi Keycloak
- [ ] User yang sudah login tetapi role SIMPEG kosong / tidak valid harus ditolak dari dashboard/fitur normal dengan `403 Access Forbidden` dan pesan hubungi admin
- [ ] Validasi: satu keycloak_id hanya bisa di-mapping ke satu pegawai
- [ ] Audit log: catat perubahan mapping dan role

**Acceptance Criteria (dari US-1.4):**
- [ ] AC-1: Halaman daftar pegawai + status mapping
- [ ] AC-2: Bisa isi Keycloak ID/email
- [ ] AC-3: Bisa tetapkan role internal aplikasi
- [ ] AC-4: Perubahan berlaku login berikutnya
- [ ] AC-5: Audit log tercatat
- [ ] AC-6: Validasi 1 akun = 1 pegawai
- [ ] AC-7: Role/permission dibaca dari database SIMPEG, bukan data otorisasi Keycloak
- [ ] AC-8: User SSO baru setelah bootstrap dibuat dengan role kosong sampai Super Admin menetapkan role di SIMPEG
- [ ] AC-9: Role dasar dari SSO tidak otomatis memberi permission SIMPEG

---

## Issue #5 · Setup Audit Log System

| Field | Detail |
|-------|--------|
| **Story** | US-7.1 · Pencatatan Otomatis Semua Perubahan |
| **Labels** | `epic:audit`, `type:engine`, `sprint:1`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 5 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Setup sistem audit log menggunakan package `owen-it/laravel-auditing` agar semua operasi CRUD tercatat otomatis.

**Tasks:**
- [ ] Install `owen-it/laravel-auditing` (`composer require owen-it/laravel-auditing`)
- [ ] Publish config dan migration (`php artisan vendor:publish --provider="OwenIt\Auditing\AuditingServiceProvider"`)
- [ ] Jalankan migration untuk tabel `audits`
- [ ] Buat trait atau base model `AuditableModel` yang implements `OwenIt\Auditing\Contracts\Auditable`
- [ ] Konfigurasi audit log:
  - Driver: database
  - Events: created, updated, deleted, restored
  - Record: old_values, new_values, user_id, ip_address, user_agent
- [ ] Buat custom events untuk:
  - LOGIN / LOGOUT
  - VERIFY / DECIDE / CHANGE_REQUESTED / DEFER / NOT_APPROVED (cuti)
  - IMPORT (Excel/CSV)
- [ ] Buat `AuditService` helper untuk log custom events
- [ ] Pastikan tabel `audits` tidak bisa di-delete via aplikasi (immutable)
- [ ] Test: create + update sebuah model → cek record di tabel audits

**Acceptance Criteria (dari US-7.1):**
- [ ] AC-1: Semua CRUD tercatat otomatis
- [ ] AC-2: Record berisi user_id, event, old_values, new_values, ip, user_agent
- [ ] AC-3: Immutable (tidak bisa edit/hapus via app)
- [ ] AC-4: Custom events untuk login/logout/verify/decide/import

---

## Issue #6 · Notifikasi In-App — Backend

| Field | Detail |
|-------|--------|
| **Story** | US-6.1 · Notifikasi In-App |
| **Labels** | `epic:notifikasi`, `type:engine`, `sprint:1`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Setup notification system Laravel menggunakan database channel.

**Tasks:**
- [ ] Buat migration tabel `notifications` (`php artisan notifications:table`)
- [ ] Buat base `SimpegNotification` class (extends `Notification`)
  - Channel: database
  - Method `toDatabase()` → return title, message, link, type
- [ ] Buat notification classes:
  - `CutiSubmittedNotification` — ke kepala bagian
  - `CutiApprovedNotification` — ke pegawai
  - `CutiPostponedNotification` — ke pegawai
  - `EwsAlertNotification` — ke admin/pegawai
- [ ] Tambahkan `Notifiable` trait ke model `Employee` (atau User)
- [ ] Buat `NotificationController`:
  - `getUnreadCount()` — return jumlah belum dibaca (untuk badge)
  - `getLatest()` — return 10 notifikasi terbaru
  - `markAsRead($id)` — tandai satu notifikasi dibaca
- [ ] Buat API routes untuk notifikasi (AJAX calls dari frontend)

---

## Issue #7 · Notifikasi In-App — Frontend (Bell Icon)

| Field | Detail |
|-------|--------|
| **Story** | US-6.1 · Notifikasi In-App |
| **Labels** | `epic:notifikasi`, `type:ui`, `sprint:1`, `priority:P0` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 2 |
| **Dependensi** | Issue #6, Issue #8 |

**Deskripsi:**
Implementasi UI bell icon notifikasi di navbar dengan dropdown dan badge counter.

**Tasks:**
- [ ] Tambahkan icon 🔔 (bell) di navbar `layouts/app.blade.php`
- [ ] Badge angka menunjukkan jumlah notifikasi belum dibaca
- [ ] Klik bell → dropdown menampilkan 10 notifikasi terbaru:
  - Judul, waktu relatif ("5 menit lalu"), indicator belum/sudah dibaca
- [ ] Klik item notifikasi → markAsRead + redirect ke halaman terkait
- [ ] Link "Lihat Semua Notifikasi" di bawah dropdown
- [ ] Polling AJAX setiap 30 detik untuk update badge count
- [ ] Styling: badge merah, dropdown shadow, bold untuk belum dibaca

---

## Issue #8 · Design System & Layout Master

| Field | Detail |
|-------|--------|
| **Story** | — (Infrastruktur UI) |
| **Labels** | `type:ui`, `type:infra`, `sprint:1`, `priority:P0` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 5 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Setup design system Laravel: layout master, komponen UI reusable, theming.

**Tasks:**
- [ ] Pilih dan install UI approach:
  - Opsi A: Filament (admin panel siap pakai)
  - Opsi B: Blade components + Vite + CSS/Tailwind manual
- [ ] Buat layout master `layouts/app.blade.php`:
  - Sidebar navigasi (collapsible)
  - Topbar/navbar (user info, bell notifikasi, tombol logout)
  - Main content area
  - Footer
- [ ] Buat Blade components reusable:
  - `<x-card>` — card container
  - `<x-table>` — data table dengan search, sort, pagination
  - `<x-form-input>` — input field dengan label dan error
  - `<x-form-select>` — dropdown select
  - `<x-form-date>` — date picker
  - `<x-modal>` — modal dialog
  - `<x-alert>` — alert/toast notification
  - `<x-badge>` — status badge (warna)
  - `<x-button>` — button variants (primary, danger, outline)
  - `<x-stat-card>` — KPI card untuk dashboard
- [ ] Setup warna tema dan typography
- [ ] Pastikan layout responsive (desktop, tablet, mobile)
- [ ] Navigasi sidebar berisi menu sesuai role user
- [ ] Bahasa UI: Bahasa Indonesia

---

## Issue #9 · Reusable Blade Components

| Field | Detail |
|-------|--------|
| **Story** | — (Infrastruktur UI) |
| **Labels** | `type:ui`, `sprint:1`, `priority:P0` |
| **Assignee** | Adriel Walintukan |
| **Story Points** | 3 |
| **Dependensi** | Issue #8 |

**Deskripsi:**
Buat komponen Blade tambahan yang akan dipakai berulang di seluruh aplikasi.

**Tasks:**
- [ ] `<x-file-upload>` — upload file dengan preview (foto, PDF, SK)
- [ ] `<x-confirm-dialog>` — dialog konfirmasi (single + double confirm)
- [ ] `<x-pagination>` — pagination dengan pilihan per-page (10/25/50)
- [ ] `<x-filter-bar>` — bar filter dengan dropdown + search
- [ ] `<x-breadcrumb>` — breadcrumb navigasi
- [ ] `<x-empty-state>` — tampilan kosong ("Belum ada data")
- [ ] `<x-loading>` — loading spinner/skeleton
- [ ] `<x-timeline>` — timeline vertikal (untuk approval cuti)
- [ ] `<x-toggle>` — toggle switch (untuk flag kinerja)
- [ ] `<x-tooltip>` — tooltip pada hover
- [ ] Dokumentasikan semua components di `resources/views/components/README.md`

---

## Issue #10 · CRUD Hari Libur Nasional & Cuti Bersama

| Field | Detail |
|-------|--------|
| **Story** | US-8.4 · Kelola Hari Libur |
| **Labels** | `epic:dashboard`, `type:feature`, `sprint:1`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
CRUD untuk tabel reference hari libur nasional dan cuti bersama.

**Tasks:**
- [ ] Buat migration tabel `ref_hari_libur`:
  - `id`, `tanggal` (date), `nama` (string), `tipe` (enum: libur_nasional, cuti_bersama), `tahun` (int), timestamps
- [ ] Buat model `HariLibur`
- [ ] Buat `HariLiburController` (CRUD):
  - `index()` — daftar per tahun, filter per tahun
  - `create()` / `store()` — form tambah
  - `edit()` / `update()` — form edit
  - `destroy()` — hapus
- [ ] Buat views: daftar, form tambah/edit
- [ ] Seed data hari libur 2026 (minimal: Idul Fitri, Idul Adha, Kemerdekaan, Natal, Tahun Baru, dll)
- [ ] Audit log untuk semua operasi
- [ ] Akses: Super Admin saja

---

## Issue #11 · Migration & Seeder Reference Tables

| Field | Detail |
|-------|--------|
| **Story** | — (Infrastruktur Database) |
| **Labels** | `type:infra`, `sprint:1`, `priority:P0` |
| **Assignee** | Jordan Sutarto + Dion Kobi |
| **Story Points** | 3 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Buat semua migration dan seeder untuk reference tables Fase 1.

**Tasks:**
- [ ] `ref_golongan` — I/a s/d IV/e (17 records)
- [ ] `ref_jenis_jabatan` — Struktural, Fungsional Tertentu, Fungsional Umum, Pimpinan Tinggi
- [ ] `ref_jabatan` — nama jabatan resmi per jenis jabatan, optional eselon/default BUP
- [ ] `ref_status_pegawai` — Aktif, Nonaktif, Pensiun, Mutasi, CLTN, Perpanjangan CLTN, Tugas Belajar, Pemberhentian Sementara, Wajib Militer, PNS Dinyatakan Hilang
- [ ] `ref_eselon` — I.a s/d V (jika relevan)
- [ ] `ref_unit_kerja` — hierarkis (`parent_id`, `level`, `jenis_unit`) untuk Kepala Lembaga, Bagian Umum, Tim Kerja, urusan/sub-unit
- [ ] `ref_jenis_pegawai` — PNS, PPPK
- [ ] `ref_jenis_cuti` — Tahunan, Sakit, Melahirkan, Besar, Alasan Penting, CLTN
- [ ] `ref_jenjang_pendidikan` — SD, SMP, SMA, D3, D4/S1, S2, S3
- [ ] `ref_agama` — Islam, Kristen, Katolik, Hindu, Buddha, Konghucu
- [ ] `ref_status_kawin` — Belum Kawin, Kawin, Cerai Hidup, Cerai Mati
- [ ] `ref_hubungan_keluarga` — Suami, Istri, Anak
- [ ] `ref_bup` — Batas Usia Pensiun per jenis jabatan (sesuai PP 49/2018)
- [ ] `ref_notification_channels` — in_app, email, whatsapp_business (future/configurable)
- [ ] Buat seeder: `php artisan db:seed --class=ReferenceTableSeeder`
- [ ] Test: seeder bisa dijalankan ulang tanpa error (idempotent)

---

## Issue #12 · Setup Testing Framework

| Field | Detail |
|-------|--------|
| **Story** | — (Infrastruktur QA) |
| **Labels** | `type:test`, `type:infra`, `sprint:1`, `priority:P0` |
| **Assignee** | Adriel Walintukan |
| **Story Points** | 2 |
| **Dependensi** | Issue #1 |

**Deskripsi:**
Setup testing framework untuk unit test dan browser test.

**Tasks:**
- [ ] Konfigurasi PHPUnit dengan database testing (SQLite in-memory atau PostgreSQL test DB)
- [ ] Install Laravel Dusk untuk browser testing (`composer require laravel/dusk --dev`)
- [ ] Buat base test class dengan helper methods
- [ ] Tulis test pertama: `AuthTest` — test redirect ke Keycloak, test halaman unregistered
- [ ] Buat `Makefile` atau script untuk menjalankan test (`php artisan test`)
- [ ] Dokumentasikan cara menjalankan test di README

---

# SPRINT 2 — DATA PEGAWAI (23 Juni – 4 Juli 2026)

---

## Issue #13 · Migration Tabel Utama Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-2.1 · Tambah Data Pegawai |
| **Labels** | `epic:pegawai`, `type:infra`, `sprint:2`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 3 |
| **Dependensi** | Issue #11 |

**Deskripsi:**
Buat migration untuk semua tabel terkait pegawai.

**Tasks:**
- [ ] Migration `employees`:
  - Data utama sesuai Excel: nama_lengkap, email_pribadi, nip (unique), tanggal_lahir, jenis_pegawai/status_kepegawaian, status_pegawai_id (FK), status_keterangan, golongan/pangkat/jabatan/kelas snapshot awal, pendidikan_terakhir, prodi_pendidikan_terakhir, tanggal_pensiun, person_label, person_formula_label, profil_status
  - Data pelengkap profil: nik, no_kk, tempat_lahir, jenis_kelamin, agama_id, status_kawin_id, golongan_darah, foto_path
  - Data kontak: alamat, no_hp, no_telepon_rumah
  - Data pengangkatan: jenis_pengangkatan, tmt_pengangkatan, no_sk_pengangkatan, tanggal_sk_pengangkatan, file_sk_pengangkatan
  - Auth: keycloak_id (nullable), role
  - Supervisor: kepala_bagian_id (FK self-referencing, nullable)
  - Kinerja: is_kinerja_baik (boolean, default true)
  - Kalkulasi: tanggal_kenaikan_pangkat_berikutnya, tanggal_kgb_berikutnya, tanggal_pensiun
  - SoftDeletes, timestamps
- [ ] Migration `rank_histories` (riwayat kepangkatan):
  - employee_id (FK), golongan_id (FK ref_golongan), tmt_pangkat, no_sk, tanggal_sk, file_sk, is_latest (bool), timestamps
- [ ] Migration `position_histories` (riwayat jabatan):
  - employee_id, jabatan_id (FK ref_jabatan), jenis_jabatan_id (FK), unit_kerja_id (FK ref_unit_kerja hierarkis), kelas_jabatan, tmt_jabatan, no_sk, tanggal_sk, file_sk, is_latest, timestamps
- [ ] Migration `kgb_histories` (riwayat KGB):
  - employee_id, tmt_kgb, gaji_pokok, no_sk, tanggal_sk, file_sk, is_latest, timestamps
- [ ] Migration `discipline_records` (hukuman disiplin):
  - employee_id, jenis_hukuman (enum: ringan/sedang/berat), deskripsi, tanggal_mulai, tanggal_berakhir (nullable), no_sk, tanggal_sk, file_sk, is_active (bool), timestamps
- [ ] Migration `family_members` (data keluarga):
  - employee_id, nama, hubungan (enum), nik, tempat_lahir, tanggal_lahir, jenis_kelamin, status_tunjangan (bool), pekerjaan, SoftDeletes, timestamps
- [ ] Migration `employee_education` (riwayat pendidikan):
  - employee_id, jenjang_id (FK), nama_institusi, jurusan, tahun_lulus, no_ijazah, timestamps
- [ ] Buat semua model dengan relationships + fillable + casts

---

## Issue #14 · Form Tambah Pegawai (Multi-Tab)

| Field | Detail |
|-------|--------|
| **Story** | US-2.1 · Tambah Data Pegawai Baru |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2) + Adithian Gunawan (UI) |
| **Story Points** | 8 |
| **Dependensi** | Issue #13, Issue #8 |

**Deskripsi:**
Form input multi-tab untuk menambahkan pegawai baru.

**Tasks Backend 2 (Grantly):**
- [ ] Buat `EmployeeController@create` dan `@store`
- [ ] Buat `StoreEmployeeRequest` (Form Request) dengan validasi:
  - NIP: required, unique, 18 digit
  - NIK: required, 16 digit
  - No. KK: nullable, 16 digit
  - Nama: required, string, max 255
  - Tanggal lahir: required, date, before today
  - Foto: nullable, image, max 10MB, mimes: jpg,png
  - File SK: nullable, file, max 10MB, mimes: pdf,jpg,png
- [ ] Simpan foto ke `storage/app/public/photos/`
- [ ] Simpan file SK ke `storage/app/public/sk/`
- [ ] Set status = Aktif setelah simpan
- [ ] Flash message sukses
- [ ] Audit log

**Tasks Frontend (Adithian):**
- [ ] Buat view `employees/create.blade.php` — form multi-tab:
  - Tab 1: Data Pribadi (nama, NIP, NIK, No. KK, TTL, kelamin, agama, kawin, goldar, foto, jenis pegawai)
  - Tab 2: Data Kontak (alamat, HP, email, telepon)
  - Tab 3: Data Pengangkatan (jenis, TMT, no SK, tanggal SK, upload SK)
- [ ] Preview foto sebelum upload
- [ ] Validasi client-side (required fields, format NIK/NIP)
- [ ] Navigasi antar tab tanpa kehilangan data
- [ ] Responsive

---

## Issue #15 · Form Edit Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-2.2 · Edit Data Pegawai |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 5 |
| **Dependensi** | Issue #14 |

**Deskripsi:**
Form edit data pegawai dengan validasi dan audit trail.

**Tasks:**
- [ ] Buat `EmployeeController@edit` dan `@update`
- [ ] Buat `UpdateEmployeeRequest` — sama dengan store tapi NIP unique ignore current
- [ ] Audit log mencatat old_values dan new_values
- [ ] Foto bisa diganti (old foto di-replace)
- [ ] Flash message sukses
- [ ] Re-use view dari Issue #14 (mode edit)

---

## Issue #16 · Halaman Daftar Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-2.3 · Daftar Pegawai (Admin View) |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2) + Adithian Gunawan (UI) |
| **Story Points** | 5 |
| **Dependensi** | Issue #13 |

**Tasks Backend 2 (Grantly):**
- [ ] Buat `EmployeeController@index` dengan:
  - Search: nama, NIP (ILIKE query)
  - Filter: golongan, unit_kerja, jenis_pegawai, status
  - Sorting: klik header kolom
  - Pagination: 10/25/50 per halaman
  - Default: hanya pegawai aktif (belum soft-delete)
- [ ] Eager load: golongan terkini, jabatan terkini, unit kerja

**Tasks Frontend (Adithian):**
- [ ] Buat view `employees/index.blade.php`:
  - Tabel: Foto (thumbnail), Nama, NIP, Golongan, Jabatan, Unit Kerja, Jenis, Status
  - Search bar di atas tabel
  - Dropdown filter
  - Tombol "Tambah Pegawai"
  - Klik nama → detail pegawai
- [ ] Responsive tabel (horizontal scroll di mobile)

---

## Issue #17 · Halaman Detail Pegawai (Tabbed)

| Field | Detail |
|-------|--------|
| **Story** | US-2.4 · Detail Pegawai (Admin View) |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2) + Adithian Gunawan (UI) |
| **Story Points** | 5 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Buat `EmployeeController@show` — load semua relasi
- [ ] Buat view `employees/show.blade.php` — layout tabbed:
  - Tab 1: Profil & Kontak
  - Tab 2: Data Keluarga (tabel + tombol "Tambah")
  - Tab 3: Riwayat Kepangkatan (tabel urut terbaru + tombol "Tambah")
  - Tab 4: Riwayat Jabatan
  - Tab 5: Riwayat KGB
  - Tab 6: Hukuman Disiplin
  - Tab 7: Riwayat Pendidikan
  - Tab 8: Dokumen & SK
  - Tab 9: Data Pengangkatan
- [ ] Tampilkan info kalkulasi: tanggal naik pangkat, tanggal KGB, tanggal pensiun
- [ ] Tampilkan flag "Kinerja Baik" (toggle)
- [ ] Tampilkan kepala bagian
- [ ] Tombol "Edit" → ke form edit

---

## Issue #18 · CRUD Riwayat Kepangkatan (Append-Only)

| Field | Detail |
|-------|--------|
| **Story** | US-2.6 · Tambah Riwayat Kepangkatan/Jabatan/KGB |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 5 |
| **Dependensi** | Issue #13 |

**Deskripsi:**
Implementasi penambahan riwayat kepangkatan, jabatan, dan KGB. Data bersifat append-only.

**Tasks:**
- [ ] Buat `RankHistoryController@store`:
  - Validasi: golongan_id wajib ada di ref_golongan, TMT wajib, file SK
  - Saat simpan: set `is_latest = false` pada record sebelumnya, set record baru `is_latest = true`
  - Hitung `tanggal_kenaikan_pangkat_berikutnya = tmt_pangkat + 4 tahun` → update di employees
  - Audit log
- [ ] Buat `PositionHistoryController@store`:
  - Validasi: nama jabatan, jenis, unit kerja
  - Logic is_latest sama
  - Hitung ulang tanggal_pensiun berdasarkan BUP jenis jabatan baru
- [ ] Buat `KgbHistoryController@store`:
  - Validasi: TMT KGB, gaji pokok
  - Logic is_latest sama
  - Hitung `tanggal_kgb_berikutnya = tmt_kgb + 2 tahun` → update di employees
- [ ] Buat form partial untuk masing-masing (modal di halaman detail pegawai)
- [ ] Upload file SK
- [ ] Record **tidak bisa diedit atau dihapus** setelah dibuat

---

## Issue #19 · CRUD Hukuman Disiplin

| Field | Detail |
|-------|--------|
| **Story** | US-2.7 · Tambah Riwayat Hukuman Disiplin |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:2`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Buat `DisciplineController@store`:
  - Input: jenis (ringan/sedang/berat), deskripsi, tanggal mulai, tanggal berakhir, SK
  - `is_active = true` jika tanggal_berakhir null atau belum lewat
- [ ] Buat Laravel Scheduler command `DeactivateExpiredDiscipline`:
  - Jalan harian
  - Set `is_active = false` untuk record yang tanggal_berakhir sudah lewat
- [ ] Register command di `routes/console.php` atau `app/Console/Kernel.php`
- [ ] Form partial (modal di tab disiplin halaman detail)
- [ ] Data append-only
- [ ] Audit log

---

# SPRINT 3 — IMPORT EXCEL/CSV & PELENGKAP (7–18 Juli 2026)

---

## Issue #20 · Download Template Import

| Field | Detail |
|-------|--------|
| **Story** | US-3.1 · Download Template Import |
| **Labels** | `epic:import`, `type:feature`, `sprint:3`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 2 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Buat `ImportTemplateController@download` — generate template CSV dan/atau Excel
- [ ] Template utama Data Pegawai mengikuti header `daftar_pegawai.xlsx`: `No`, `Nama Pegawai`, `Email Pegawai`, `Golongan`, `Jabatan`, `Kelas Jabatan`, `NIP`, `Nomor Telepon`, `Pangkat`, `Pendidikan Terakhir`, `Pensiun`, `Person`, `Person Formula`, `Prodi Pendidikan Terakhir`, `Status Kepegawaian`, `Tanggal Lahir`
- [ ] Template lanjutan: Data Pelengkap, Riwayat Kepangkatan, Riwayat Jabatan, Riwayat KGB
- [ ] Header kolom mengikuti `Mapping-Data-Pegawai-Excel-SIMPEG.md`
- [ ] Sertakan 2 baris contoh data dummy
- [ ] Format minimal CSV UTF-8 delimiter koma; opsi Excel memakai `.xlsx`
- [ ] Tombol download di halaman Import

---

## Issue #21 · Upload, Preview & Validasi Excel/CSV

| Field | Detail |
|-------|--------|
| **Story** | US-3.2, US-3.3 · Upload & Preview + Validasi |
| **Labels** | `epic:import`, `type:feature`, `sprint:3`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2) + Adithian Gunawan (UI) |
| **Story Points** | 10 |
| **Dependensi** | Issue #13, Issue #20 |

**Tasks Backend 2 (Grantly):**
- [ ] Buat `EmployeeImportController` dengan method: upload, preview, validate, execute
- [ ] Upload: terima file Excel/CSV, simpan sementara, parse header
- [ ] Auto-match kolom Excel/CSV ke field SIMPEG berdasarkan header `daftar_pegawai.xlsx`
- [ ] Validasi semua baris:
  - NIP unik (cek DB + antar baris)
  - Email Pegawai terisi dan berformat email
  - Tanggal Lahir valid
  - Pensiun valid jika terisi
  - Status Kepegawaian valid: PNS/CPNS/PPPK
  - Field wajib Excel terisi
  - Golongan ada di reference table jika reference sudah tersedia
- [ ] Return ringkasan: total, valid ✅, error ❌, skip (NIP duplikat)
- [ ] Untuk baris error: nomor baris, kolom, jenis error

**Tasks Frontend (Adithian):**
- [ ] Buat view `import/index.blade.php`:
  - Step 1: Upload file + tombol download template
  - Step 2: Preview 10 baris pertama + mapping kolom (dropdown)
  - Step 3: Hasil validasi — ringkasan + detail error
  - Step 4: Tombol "Import yang Valid" atau "Batal"
- [ ] Progress indicator (step wizard)
- [ ] Tabel error dengan highlight baris bermasalah

---

## Issue #22 · Eksekusi Import Excel/CSV (Queue Job)

| Field | Detail |
|-------|--------|
| **Story** | US-3.4 · Eksekusi Import & Laporan Hasil |
| **Labels** | `epic:import`, `type:engine`, `sprint:3`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 7 |
| **Dependensi** | Issue #21 |

**Tasks:**
- [ ] Buat `ImportEmployeesJob` (queue job):
  - Process baris valid → insert ke employees
  - Skip baris NIP duplikat
  - Simpan snapshot awal dari Excel: golongan, pangkat, jabatan, kelas jabatan, pendidikan, prodi, tanggal pensiun jika tersedia
  - Set `profil_status = belum_lengkap` untuk pegawai hasil import yang belum punya data pelengkap PRD
  - Hitung tanggal pensiun hanya untuk baris yang kolom `Pensiun` kosong dan referensi BUP sudah tersedia
  - Track progress (jumlah berhasil/gagal)
- [ ] Dispatch job dari controller setelah user klik "Import"
- [ ] Tampilkan progress bar atau loading indicator
- [ ] Setelah selesai, tampilkan laporan: berhasil, gagal, skip
- [ ] Laporan bisa di-download sebagai CSV/Excel (baris gagal + alasan)
- [ ] Audit log: user, timestamp, nama file, jumlah berhasil/gagal

---

## Issue #23 · Halaman Profil Sendiri (Read-Only)

| Field | Detail |
|-------|--------|
| **Story** | US-2.5 · Lihat Profil Sendiri |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:3`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #17 |

**Tasks:**
- [ ] Buat `ProfileController@show` — load data pegawai yang login
- [ ] Re-use layout dari halaman detail (Issue #17) tapi semua **read-only**
- [ ] Tidak ada tombol edit/tambah
- [ ] Tampilkan saldo cuti tahun berjalan
- [ ] Tampilkan tanggal naik pangkat & KGB berikutnya
- [ ] Pegawai tidak bisa akses profil pegawai lain (authorize)

---

## Issue #24 · CRUD Data Keluarga

| Field | Detail |
|-------|--------|
| **Story** | US-2.8 · Kelola Data Keluarga |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:3`, `priority:P1` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Buat `FamilyMemberController` — CRUD
- [ ] Form: nama, hubungan, NIK, TTL, kelamin, status tunjangan, pekerjaan
- [ ] Soft delete untuk data keluarga
- [ ] Di halaman detail pegawai, tab "Keluarga"
- [ ] Audit log

---

## Issue #25 · Soft Delete & Restore Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-2.9 · Soft Delete Pegawai |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:3`, `priority:P0` |
| **Assignee** | Adriel Walintukan |
| **Story Points** | 3 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Tombol "Nonaktifkan" di halaman detail pegawai
- [ ] Confirm dialog: "Apakah Anda yakin?"
- [ ] Soft delete → set `deleted_at`
- [ ] Tidak muncul di daftar default
- [ ] Filter "Tampilkan Non-Aktif" di daftar pegawai
- [ ] Tombol "Aktifkan Kembali" (restore) dari daftar non-aktif
- [ ] Pegawai non-aktif tidak diproses EWS
- [ ] Audit log

---

# SPRINT 4 — CUTI (21 Juli – 1 Agustus 2026)

---

## Issue #26 · Migration Tabel Cuti

| Field | Detail |
|-------|--------|
| **Story** | US-4.1 · Ajukan Cuti |
| **Labels** | `epic:cuti`, `type:infra`, `sprint:4`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 2 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Migration `leave_requests`:
  - employee_id, jenis_cuti_id (FK), tanggal_mulai, tanggal_selesai, jumlah_hari_kerja, alasan, file_lampiran, workflow_status, final_decision_status, qr_token, generated_document_path, external_approval, timestamps
- [ ] Migration `leave_approval_chains`:
  - scope_type (global/unit/employee), scope_id nullable, step_order, step_type, approver_employee_id, is_active, timestamps
- [ ] Migration `leave_approval_steps`:
  - leave_request_id, step_order, step_type, approver_id, decision_status (`Disetujui`/`Perubahan`/`Ditangguhkan`/`Tidak Disetujui`), keterangan, acted_at, timestamps
- [ ] Migration `leave_balances`:
  - employee_id, tahun (int), jatah_dasar, carry_over_n1, hak_tambahan_n2_n1, terpakai, sisa, timestamps
- [ ] Migration `leave_balance_adjustments`:
  - employee_id, tahun, adjustment_days, reason, created_by, timestamps
- [ ] Migration `leave_documents`:
  - leave_request_id, document_path, qr_token, verification_payload, external_document_path, timestamps
- [ ] Buat semua model dengan relationships

---

## Issue #27 · Assign Kepala Bagian per Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-4.11 · Assign Kepala Bagian |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:4`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #13 |

**Tasks:**
- [ ] Di halaman detail pegawai, section "Kepala Bagian"
- [ ] Dropdown: semua pegawai (kecuali diri sendiri)
- [ ] Simpan ke `employees.kepala_bagian_id`
- [ ] Riwayat perubahan kepala bagian tersimpan
- [ ] Satu pegawai = satu kepala bagian aktif
- [ ] Audit log

---

## Issue #28 · Konfigurasi Approval Chain

| Field | Detail |
|-------|--------|
| **Story** | US-4.10 · Konfigurasi Approval Chain Cuti |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:4`, `priority:P0` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #26 |

**Tasks:**
- [ ] Halaman konfigurasi approval chain cuti (Super Admin only)
- [ ] Konfigurasi chain per pegawai/unit: Kepala Bagian, Ketua Tim Kerja, satu atau lebih verifikator, Kabag/Kepegawaian, Pimpinan/PYBMC
- [ ] Ketua Tim Kerja dapat dipilih sebagai verifikator tanpa role baru
- [ ] Simpan urutan approval per pegawai/unit
- [ ] Tambahkan flag/logic skip jika approver pada dua step adalah orang yang sama
- [ ] Simpan ke tabel `leave_approval_chains`
- [ ] Audit log

---

## Issue #29 · Kalkulasi Hari Kerja Otomatis

| Field | Detail |
|-------|--------|
| **Story** | US-4.12 · Kalkulasi Hari Kerja |
| **Labels** | `epic:cuti`, `type:engine`, `sprint:4`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 5 |
| **Dependensi** | Issue #10 (hari libur) |

**Tasks:**
- [ ] Buat `WorkdayCalculator` service class:
  - Input: tanggal_mulai, tanggal_selesai
  - Hitung: total hari - Sabtu - Minggu - hari libur (dari ref_hari_libur) - cuti bersama
  - Return: jumlah hari kerja
- [ ] Buat API endpoint `/api/calculate-workdays?start=&end=` — AJAX dari form cuti
- [ ] Warning jika tanggal mulai/selesai jatuh di weekend/libur
- [ ] Unit test untuk kalkulasi

---

## Issue #30 · Form Pengajuan Cuti

| Field | Detail |
|-------|--------|
| **Story** | US-4.1 · Ajukan Cuti |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:4`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Backend 1) + Adithian Gunawan (UI) |
| **Story Points** | 5 |
| **Dependensi** | Issue #26, Issue #27, Issue #29 |

**Tasks Backend 1 (Jordan):**
- [ ] Buat `LeaveRequestController@create` dan `@store`
- [ ] Validasi:
  - Jenis cuti sesuai jenis pegawai (PNS/PPPK)
  - Saldo cukup (untuk cuti tahunan)
  - Pegawai harus punya approval chain aktif
  - Tanggal valid
  - Tanggal mulai/selesai tidak boleh lintas tahun kalender
- [ ] Setelah submit: status = "Menunggu [step pertama approval chain]"
- [ ] Snapshot approval chain ke `leave_approval_steps`
- [ ] Kirim notifikasi ke pihak pertama pada chain (channel mengikuti konfigurasi)

**Tasks Frontend (Adithian):**
- [ ] Form: jenis cuti (dropdown), tanggal mulai, tanggal selesai, alasan, upload lampiran
- [ ] Kalkulasi hari kerja realtime (AJAX ke Issue #29)
- [ ] Validasi saldo client-side
- [ ] Error message jika belum punya kepala bagian

---

## Issue #31 · Approval Engine Cuti Dinamis

| Field | Detail |
|-------|--------|
| **Story** | US-4.4, US-4.5, US-4.6 · Approval Chain Cuti |
| **Labels** | `epic:cuti`, `type:engine`, `sprint:4`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2 engine) + Adithian Gunawan (UI) |
| **Story Points** | 13 |
| **Dependensi** | Issue #30 |

**Tasks Backend 2 (Grantly):**
- [ ] Buat `LeaveApprovalService`:
  - `decide(leaveRequest, actor, decisionStatus, keterangan)`:
    - Validasi actor adalah approver/verifikator pada step aktif
    - `Disetujui` pada step non-final → lanjut ke step berikutnya yang valid
    - `Disetujui` pada step final → status final `Disetujui`, kurangi saldo cuti, generate dokumen QR, notifikasi ke pegawai
    - `Perubahan`, `Ditangguhkan`, `Tidak Disetujui` → keterangan wajib, saldo tidak dikurangi, status dan alasan tampil di timeline
    - Jika approver step berikutnya sama dengan approver saat ini, skip step duplikat
- [ ] Buat `LeaveApprovalController`:
  - `index()` — daftar pengajuan pending untuk approver
  - `decision($id)` — simpan keputusan/rekomendasi
- [ ] Audit log untuk setiap aksi
- [ ] Saldo cuti: hanya dikurangi setelah keputusan final `Disetujui` (cuti tahunan saja)
- [ ] Generate formulir cuti dan halaman verifikasi QR

**Tasks Frontend (Adithian):**
- [ ] Halaman "Pengajuan Cuti Bawahan" — daftar pending
- [ ] Detail: nama, jenis cuti, tanggal, hari, alasan, lampiran
- [ ] Tombol/aksi: "Disetujui", "Perubahan", "Ditangguhkan", "Tidak Disetujui"
- [ ] Aksi selain "Disetujui" → textarea keterangan wajib
- [ ] Tidak ada tombol formal "Tolak"; gunakan "Tidak Disetujui"
- [ ] Badge status berwarna untuk menunggu/disetujui/perubahan/ditangguhkan/tidak disetujui

---

## Issue #32 · Saldo Cuti & Daftar Cuti Pegawai

| Field | Detail |
|-------|--------|
| **Story** | US-4.3 · Lihat Saldo Cuti |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:4`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Backend 1) + Adriel Walintukan (UI) |
| **Story Points** | 3 |
| **Dependensi** | Issue #26 |

**Tasks:**
- [ ] Buat `LeaveBalanceController@show` — saldo pegawai yang login
- [ ] Tampilkan: jatah dasar 12 hari, carry-over N-1 maksimal 6, hak tambahan N-2/N-1, total, terpakai, sisa
- [ ] Riwayat penggunaan cuti tahun berjalan
- [ ] Data diperbarui real-time setelah cuti disetujui
- [ ] Buat seeder/input awal saldo 2026 untuk semua pegawai (jatah dasar = 12) + saldo/riwayat N-1/N-2 jika tersedia

---

# SPRINT 5 — EWS & NOTIFIKASI (4–15 Agustus 2026)

---

## Issue #33 · Kalkulasi TMT Otomatis

| Field | Detail |
|-------|--------|
| **Story** | US-5.5 · Kalkulasi TMT Otomatis |
| **Labels** | `epic:ews`, `type:engine`, `sprint:5`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 5 |
| **Dependensi** | Issue #18 |

**Tasks:**
- [ ] Buat `TmtCalculatorService`:
  - Saat riwayat pangkat ditambahkan → `tmt_pangkat + 4 tahun`
  - Saat riwayat KGB ditambahkan → `tmt_kgb + 2 tahun`
  - Saat jabatan ditambahkan → `tanggal_lahir + BUP`
  - Saat data pengangkatan pertama tersedia → milestone Satyalancana 10/20/30 tahun
- [ ] Register sebagai Model Observer atau Event Listener
- [ ] Simpan hasil ke kolom di tabel `employees`
- [ ] Juga jalankan saat import CSV selesai (batch)
- [ ] Unit test

---

## Issue #34 · EWS Scheduler Harian

| Field | Detail |
|-------|--------|
| **Story** | US-5.1 · Scheduler EWS Harian |
| **Labels** | `epic:ews`, `type:engine`, `sprint:5`, `priority:P0` |
| **Assignee** | Grantly Sorongan |
| **Story Points** | 8 |
| **Dependensi** | Issue #33, Issue #6 |

**Tasks:**
- [ ] Buat migration `ews_alerts`:
  - employee_id, jenis_event, tanggal_target, sisa_hari, is_eligible, notified_at, trigger_days (H-90/H-60/etc), tahun, followup_status, handled_at, handled_by, handled_note, timestamps
- [ ] Buat Artisan command `app:run-ews`:
  - Cek semua pegawai aktif
  - 5 trigger: kenaikan pangkat, KGB, pensiun, kontrak PPPK, Satyalancana
  - Threshold: H-90/H-60/H-30 (pangkat), H-60/H-30/H-14 (KGB), H-1thn/H-6bln/H-3bln (pensiun), H-6bln/H-3bln/H-1bln (PPPK), H-180/H-90/H-30 (Satyalancana)
  - Eligibility pangkat: 4 tahun + no disiplin aktif + kinerja baik
  - Eligibility Satyalancana: masa kerja 10/20/30 tahun + flag/catatan kelayakan manual
  - No duplicate: cek `ews_alerts` sebelum kirim
  - Kirim notifikasi ke admin + pegawai bersangkutan
- [ ] Register di scheduler: `$schedule->command('app:run-ews')->dailyAt('07:00');`
- [ ] Log eksekusi: waktu mulai/selesai, jumlah alert baru
- [ ] Error handling: jika gagal, notifikasi ke Super Admin

---

## Issue #35 · Halaman Daftar EWS Aktif

| Field | Detail |
|-------|--------|
| **Story** | US-5.2 · Halaman Daftar EWS Aktif |
| **Labels** | `epic:ews`, `type:feature`, `sprint:5`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Backend 1) + Adithian Gunawan (UI) |
| **Story Points** | 5 |
| **Dependensi** | Issue #34 |

**Tasks:**
- [ ] Buat `EwsController@index`
- [ ] Tabel: Nama, NIP, Jenis Event, Tanggal Target, Sisa Hari, Eligibility
- [ ] Urut dari sisa hari terkecil
- [ ] Warna baris: 🔴 <30 hari, 🟡 30-90 hari, 🟢 >90 hari
- [ ] Filter: jenis event dan status tindak lanjut
- [ ] Aksi: tandai `ditangani` / `tidak_perlu` dengan catatan
- [ ] Klik nama → detail pegawai
- [ ] Akses: Admin, Super Admin, Pimpinan

---

## Issue #36 · Flag Kinerja Baik (Toggle)

| Field | Detail |
|-------|--------|
| **Story** | US-5.4 · Update Flag Kinerja Baik |
| **Labels** | `epic:ews`, `type:feature`, `sprint:5`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Backend 1) + Adriel Walintukan (UI) |
| **Story Points** | 2 |
| **Dependensi** | Issue #17 |

**Tasks:**
- [ ] Toggle "Kinerja Baik" di halaman detail pegawai
- [ ] Flag/catatan kelayakan Satyalancana di halaman detail pegawai
- [ ] AJAX update tanpa reload halaman
- [ ] Tooltip: "Flag ini menggantikan penilaian SKP yang belum tersedia di Fase 1"
- [ ] Jika false → tidak eligible kenaikan pangkat di EWS
- [ ] Audit log

---

## Issue #37 · Email Notification

| Field | Detail |
|-------|--------|
| **Story** | US-6.3 · Notifikasi Email |
| **Labels** | `epic:notifikasi`, `type:feature`, `sprint:5`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Backend 1) + Adriel Walintukan (template) |
| **Story Points** | 5 |
| **Dependensi** | Issue #6 |

**Tasks Backend 1 (Jordan):**
- [ ] Konfigurasi Laravel Mail di `.env`: Mailpit untuk development, email operasional/Gmail resmi LLDIKTI untuk production
- [ ] Tambahkan notification dispatcher yang membaca `ref_notification_channels`
- [ ] Tambahkan mail channel ke notification classes melalui dispatcher, bukan hardcoded di domain service
- [ ] Queue: dispatch email via queue (non-blocking)
- [ ] Retry: maks 3x jika gagal
- [ ] Log error jika semua retry gagal

**Tasks Frontend (Adriel):**
- [ ] Buat email template HTML responsive (`resources/views/emails/`)
- [ ] Template: judul, detail singkat, tombol "Lihat di SIMPEG"
- [ ] Bahasa Indonesia
- [ ] Template untuk: cuti submitted, keputusan cuti (`Disetujui`/`Perubahan`/`Ditangguhkan`/`Tidak Disetujui`), EWS alert

---

## Issue #38 · Session Timeout

| Field | Detail |
|-------|--------|
| **Story** | US-1.3 · Session Timeout Otomatis |
| **Labels** | `epic:auth`, `type:feature`, `sprint:5`, `priority:P1` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 3 |
| **Dependensi** | Issue #2 |

**Tasks:**
- [ ] Set `SESSION_LIFETIME=30` di `.env`
- [ ] Buat middleware `SessionTimeoutMessage` — flash "Sesi Anda telah berakhir"
- [ ] Redirect ke Keycloak login saat session expired
- [ ] Audit log: catat session timeout event

---

# SPRINT 6-7 — DASHBOARD, LAPORAN, STABILIZATION & UAT

> Catatan revisi vertical slice: beberapa issue P1/UAT di bagian ini dipindahkan ke Sprint 7, sementara issue yang lebih tepat masuk ke Sprint 3-5 diberi label sprint sesuai rencana terbaru. Nomor issue dipertahankan agar referensi tetap stabil.

---

## Issue #39 · Dashboard Admin & Pimpinan (7 Widget)

| Field | Detail |
|-------|--------|
| **Story** | US-8.1 · Dashboard Admin |
| **Labels** | `epic:dashboard`, `type:feature`, `sprint:6`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Backend 2 queries) + Adithian Gunawan (UI/charts) |
| **Story Points** | 8 |
| **Dependensi** | Issue #16, Issue #31, Issue #35 |

**Tasks Backend 2 (Grantly):**
- [ ] Buat `DashboardController@admin` — hitung semua data widget
- [ ] W1: Komposisi pegawai (total aktif, PNS vs PPPK)
- [ ] W2: Kenaikan pangkat bulan ini / tahun ini
- [ ] W3: Status cuti (pending, disetujui bulan ini, ditunda)
- [ ] W4: Top 5 EWS paling urgent
- [ ] W5: Distribusi golongan (count per golongan)
- [ ] W6: 5 audit log terbaru
- [ ] W7: Tren pegawai 12 bulan terakhir

**Tasks Frontend (Adithian):**
- [ ] Layout dashboard responsive (grid)
- [ ] W1: KPI card + pie chart (Chart.js/ApexCharts)
- [ ] W2: KPI card + mini list
- [ ] W3: KPI card
- [ ] W4: Tabel mini dengan warna indikator
- [ ] W5: Bar chart horizontal
- [ ] W6: List item
- [ ] W7: Line chart

---

## Issue #40 · Dashboard Pegawai (Pribadi)

| Field | Detail |
|-------|--------|
| **Story** | US-8.2 · Dashboard Pegawai |
| **Labels** | `epic:dashboard`, `type:feature`, `sprint:6`, `priority:P0` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 5 |
| **Dependensi** | Issue #23, Issue #32, Issue #35 |

**Tasks:**
- [ ] Profil ringkas: foto, nama, NIP, golongan, jabatan, unit kerja
- [ ] Card saldo cuti
- [ ] Daftar pengajuan cuti aktif + status
- [ ] EWS pribadi
- [ ] 5 notifikasi terbaru
- [ ] Responsive

---

## Issue #41 · Dashboard Kepala Bagian

| Field | Detail |
|-------|--------|
| **Story** | US-8.3 · Dashboard Kepala Bagian |
| **Labels** | `epic:dashboard`, `type:feature`, `sprint:6`, `priority:P1` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 5 |
| **Dependensi** | Issue #31, Issue #27 |

**Tasks:**
- [ ] Daftar bawahan langsung
- [ ] Pengajuan cuti pending (quick action sesuai label resmi keputusan cuti)
- [ ] EWS bawahan
- [ ] Klik nama → detail ringkas (read-only)

---

## Issue #42 · Export Daftar Pegawai (Excel + PDF)

| Field | Detail |
|-------|--------|
| **Story** | US-9.1 + US-9.1B + US-9.2 · Export Pegawai |
| **Labels** | `epic:laporan`, `type:feature`, `sprint:6`, `priority:P0` |
| **Assignee** | Grantly Sorongan (Excel) + Adriel Walintukan (PDF) |
| **Story Points** | 6 |
| **Dependensi** | Issue #16 |

**Tasks Excel (Grantly):**
- [ ] Buat `EmployeeExport` class (Maatwebsite/Excel)
- [ ] Kolom: No, NIP, Nama, Golongan, Jabatan, Unit Kerja, Jenis, Status
- [ ] Mengikuti filter aktif
- [ ] Nama file: `Daftar_Pegawai_LLDIKTI_XVI_{tanggal}.xlsx`
- [ ] Buat `CustomEmployeeExport`:
  - Pilihan kolom dari whitelist aman
  - Filter status pegawai, unit/tim kerja, jenis pegawai, golongan, jabatan, periode pensiun
  - Output Excel saja
  - Urutan kolom mengikuti pilihan user

**Tasks PDF (Adriel):**
- [ ] Buat Blade view untuk PDF (`exports/employees-pdf.blade.php`)
- [ ] Header: nama instansi, judul, tanggal cetak
- [ ] Tabel data
- [ ] Footer: halaman X dari Y
- [ ] Orientasi landscape
- [ ] Generate PDF via DomPDF

---

## Issue #43 · Export Rekap Cuti (Excel + PDF)

| Field | Detail |
|-------|--------|
| **Story** | US-9.3 + US-9.4 · Export Cuti |
| **Labels** | `epic:laporan`, `type:feature`, `sprint:6`, `priority:P0` |
| **Assignee** | Jordan Sutarto (Excel) + Adriel Walintukan (PDF) |
| **Story Points** | 6 |
| **Dependensi** | Issue #31 |

**Tasks Excel (Jordan):**
- [ ] Buat `LeaveReportExport` — filter: periode, unit kerja, pegawai
- [ ] Sheet 1: detail cuti
- [ ] Sheet 2: ringkasan per pegawai
- [ ] Nama file: `Rekap_Cuti_{periode}_{tanggal}.xlsx`

**Tasks PDF (Adriel):**
- [ ] View PDF dengan header institusi
- [ ] Tabel rekap per pegawai
- [ ] Bagian tanda tangan
- [ ] Footer halaman

---

## Issue #44 · Daftar Cuti Pegawai + Timeline Approval

| Field | Detail |
|-------|--------|
| **Story** | US-4.2 + US-4.7 · Daftar Cuti + Timeline |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:4`, `priority:P0` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 6 |
| **Dependensi** | Issue #31 |

**Tasks:**
- [ ] Tabel daftar pengajuan cuti pegawai: jenis, tanggal, hari, status, tanggal ajuan
- [ ] Badge warna status untuk menunggu, disetujui, perubahan, ditangguhkan, tidak disetujui
- [ ] Filter: tahun, status
- [ ] Klik baris → detail + timeline approval
- [ ] Timeline visual vertikal: step, nama approver/verifikator, aksi, waktu, keterangan
- [ ] Pagination

---

## Issue #45 · Daftar Cuti Admin View + Kelola Saldo

| Field | Detail |
|-------|--------|
| **Story** | US-4.8 + US-4.9 · Admin Cuti |
| **Labels** | `epic:cuti`, `type:feature`, `sprint:7`, `priority:P1` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 8 |
| **Dependensi** | Issue #31 |

**Tasks:**
- [ ] Daftar semua pengajuan cuti (semua pegawai) — admin monitor only
- [ ] Filter: status, jenis cuti, unit kerja, periode
- [ ] Search: nama/NIP
- [ ] Halaman kelola saldo cuti semua pegawai
- [ ] Koreksi manual saldo (alasan wajib)
- [ ] Auto carry-over awal tahun (scheduler 1 Januari): N-1 maksimal 6 hari, N-2/N-1 tidak mengambil cuti dapat mencapai 24 hari
- [ ] Input/koreksi saldo awal dan riwayat N-1/N-2
- [ ] Audit log koreksi

---

## Issue #46 · Kelola Reference Tables (CRUD Admin)

| Field | Detail |
|-------|--------|
| **Story** | US-8.5 · Kelola Reference Tables |
| **Labels** | `epic:dashboard`, `type:feature`, `sprint:6`, `priority:P1` |
| **Assignee** | Jordan Sutarto |
| **Story Points** | 5 |
| **Dependensi** | Issue #11 |

**Tasks:**
- [ ] CRUD untuk: ref_golongan, ref_jenis_jabatan, ref_jabatan, ref_status_pegawai, ref_eselon, ref_unit_kerja hierarkis, ref_jenjang_pendidikan, ref_bup, ref_notification_channels
- [ ] Validasi: tidak bisa hapus item yang sedang dipakai
- [ ] Soft delete jika sudah dipakai
- [ ] Audit log
- [ ] Akses: Super Admin

---

## Issue #47 · Halaman Audit Log + Diff View

| Field | Detail |
|-------|--------|
| **Story** | US-7.2 + US-7.3 · Daftar + Detail Audit Log |
| **Labels** | `epic:audit`, `type:feature`, `sprint:7`, `priority:P1` |
| **Assignee** | Adriel Walintukan |
| **Story Points** | 6 |
| **Dependensi** | Issue #5 |

**Tasks:**
- [ ] Tabel audit log: Waktu, User, Event, Modul, Ringkasan
- [ ] Filter: event, user, modul, periode
- [ ] Pagination (25/halaman)
- [ ] Klik baris → detail/modal:
  - Info: user, waktu, IP, browser, event, model, record ID
  - Diff view: tabel "Sebelum" vs "Sesudah" (highlight perubahan)
  - CREATE → tampilkan new_values
  - DELETE → tampilkan old_values
- [ ] Tombol "Lihat Record" → navigasi ke record
- [ ] Akses: Super Admin, Admin Kepegawaian

---

## Issue #48 · Halaman Semua Notifikasi + Tandai Dibaca

| Field | Detail |
|-------|--------|
| **Story** | US-6.2 + US-6.4 · Notifikasi |
| **Labels** | `epic:notifikasi`, `type:feature`, `sprint:7`, `priority:P1` |
| **Assignee** | Adithian Gunawan (halaman) + Adriel Walintukan (tandai dibaca) |
| **Story Points** | 5 |
| **Dependensi** | Issue #6 |

**Tasks:**
- [ ] Halaman daftar semua notifikasi (urut terbaru)
- [ ] Visual: bold/highlight untuk belum dibaca
- [ ] Tombol "Tandai Semua Sudah Dibaca"
- [ ] Klik notifikasi → redirect ke halaman terkait
- [ ] Tombol per-item "Tandai dibaca" tanpa redirect
- [ ] Badge navbar berkurang setelah aksi
- [ ] Pagination

---

## Issue #49 · EWS Pribadi (Pegawai)

| Field | Detail |
|-------|--------|
| **Story** | US-5.3 · EWS Pribadi |
| **Labels** | `epic:ews`, `type:feature`, `sprint:5`, `priority:P1` |
| **Assignee** | Adithian Gunawan |
| **Story Points** | 3 |
| **Dependensi** | Issue #34 |

**Tasks:**
- [ ] Section "Peringatan Penting" di dashboard/profil pegawai
- [ ] Tampilkan: jenis event, tanggal target, sisa hari, eligibility
- [ ] Hanya EWS milik pegawai yang login
- [ ] Warna indikator sesuai urgency

---

## Issue #50 · Redirect Berdasarkan Role

| Field | Detail |
|-------|--------|
| **Story** | US-1.5 · Redirect Setelah Login |
| **Labels** | `epic:auth`, `type:feature`, `sprint:7`, `priority:P1` |
| **Assignee** | Adriel Walintukan |
| **Story Points** | 3 |
| **Dependensi** | Issue #2, Issue #39, Issue #40, Issue #41 |

**Tasks:**
- [ ] Super Admin / Admin Kepegawaian → Dashboard Admin
- [ ] Pimpinan → Dashboard Pimpinan
- [ ] Kepala Bagian → Dashboard Kepala Bagian
- [ ] Pegawai → Dashboard Pribadi
- [ ] Implementasi di AuthController callback

---

## Issue #51 · Kebijakan Soft Delete Pegawai (Super Admin)

| Field | Detail |
|-------|--------|
| **Story** | US-2.10 · Soft Delete Pegawai oleh Super Admin |
| **Labels** | `epic:pegawai`, `type:feature`, `sprint:3`, `priority:P1` |
| **Assignee** | Jordan Sutarto (backend/delete policy) + Adriel Walintukan (UI/review) |
| **Story Points** | 2 |
| **Dependensi** | Issue #25 |

**Tasks:**
- [ ] Pastikan tidak ada tombol "Hapus Permanen" untuk role apa pun, termasuk Super Admin
- [ ] Super Admin hanya bisa soft delete/nonaktifkan pegawai dengan dialog konfirmasi
- [ ] Data pegawai yang dinonaktifkan tetap tersimpan di database dan file fisik tetap ada di storage
- [ ] Data non-aktif bisa ditemukan melalui filter "Tampilkan Pegawai Non-Aktif"
- [ ] Super Admin bisa restore data pegawai yang dinonaktifkan
- [ ] Audit log mencatat soft delete dan restore

---

## Issue #52 · Full Regression Test (UAT)

| Field | Detail |
|-------|--------|
| **Story** | — (QA) |
| **Labels** | `type:test`, `sprint:7`, `priority:P0` |
| **Assignee** | Grantly Sorongan (QA) + Dion Kobi (UAT coordination) + Adriel Walintukan (release gate) |
| **Story Points** | 8 |

**Deskripsi:**
Full end-to-end testing seluruh sistem sebelum go-live.

**Test Scenarios:**
- [ ] **Auth:** Login SSO → mapping → redirect per role → logout → session timeout
- [ ] **CRUD Pegawai:** Tambah → edit → detail → daftar → search → filter → sort
- [ ] **Riwayat:** Tambah pangkat → is_latest update → TMT terhitung → tampil di detail
- [ ] **Import Excel/CSV:** Download template → upload → preview → validasi → import → verify data
- [ ] **Cuti E2E:** Ajukan → step verifikasi dinamis → keputusan final `Disetujui` → saldo berkurang → dokumen QR muncul → notifikasi diterima
- [ ] **Cuti Edge Cases:** Saldo habis, tanggal weekend, lintas tahun kalender, PPPK jenis cuti terbatas, keputusan `Perubahan`/`Ditangguhkan`/`Tidak Disetujui`
- [ ] **EWS:** Jalankan scheduler manual → verify alert termasuk Satyalancana muncul → notifikasi terkirim → no duplicate → alert bisa ditandai ditangani/tidak perlu
- [ ] **Dashboard:** Semua widget menampilkan data akurat
- [ ] **Export:** Excel + PDF untuk pegawai dan cuti
- [ ] **Audit Log:** Setiap operasi tercatat → diff view akurat
- [ ] **Notifikasi:** In-app badge + email terkirim
- [ ] **Responsive:** Test di Chrome, Firefox, Edge — desktop + tablet
- [ ] **RBAC:** Setiap role hanya bisa akses halaman yang diizinkan
- [ ] **Soft Delete & Restore:** Nonaktifkan → tidak muncul di daftar aktif → tampil di filter non-aktif → restore → muncul lagi

**Output:**
- [ ] Bug report list (severity: critical/major/minor)
- [ ] Test coverage summary
- [ ] Sign-off document (jika semua critical fix resolved)

---

## Ringkasan Issues per Sprint

| Sprint | Issues | Total |
|--------|--------|:-----:|
| Sprint 1 — Fondasi | #1 – #12 | 12 |
| Sprint 2 — Data Pegawai | #13 – #19 | 7 |
| Sprint 3 — Import & Pelengkap | #20 – #25, #51 | 7 |
| Sprint 4 — Cuti | #26 – #32, #44 | 8 |
| Sprint 5 — EWS & Notifikasi | #33 – #38, #49 | 7 |
| Sprint 6 — Dashboard & Laporan | #39 – #43, #46 | 6 |
| Sprint 7 — Stabilization, Regression & UAT | #45, #47, #48, #50, #52 | 5 |
| **Total** | | **52** |

## Ringkasan Issues per Assignee

| Assignee | Issues |
|----------|--------|
| **Dion Kobi** | #1, #11, #52 |
| **Jordan Sutarto — Lead Backend** | #4, #6, #10, #11, #18, #19, #20, #23, #24, #26, #27, #28, #30, #32, #35, #36, #37, #38, #43, #45, #46, #51 |
| **Grantly Sorongan — Support Backend + QA Tester** | #2, #3, #5, #13, #14, #15, #16, #17, #21, #22, #29, #31, #33, #34, #39, #42, #52 |
| **Adithian Gunawan** | #7, #8, #14, #16, #17, #21, #30, #31, #35, #39, #40, #41, #44, #48, #49 |
| **Adriel Walintukan** | #9, #12, #25, #32, #36, #37, #42, #43, #47, #48, #50, #51, #52 |
