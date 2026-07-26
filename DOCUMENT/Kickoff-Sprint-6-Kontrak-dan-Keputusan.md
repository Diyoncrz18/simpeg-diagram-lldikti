# Kickoff Sprint 6 — Kontrak & Keputusan (Dashboard & Laporan)

| Field | Detail |
|---|---|
| Tanggal | 26 Juli 2026 |
| Tahap | Kickoff slice Sprint 6 (tahap 1 alur vertical slice) |
| Disetujui oleh | Dion Kobi — System Analyst & Project Manager |
| Status | **Kanonis** — jika bertentangan dengan teks PRD/User Stories/Issues sebelumnya, dokumen ini yang berlaku sampai dokumen sumber direvisi |
| Acuan | PRD-SIMPEG-Fase1-Core.md v1.3 §13–14 & §16, User-Stories US-8.1/8.3/8.5 & US-9.x, Issues #39–#43 & #46, `AGENTS.md`, hasil verifikasi kode HEAD `1b2e5b6` (26 Juli 2026) |
| Cakupan | Dua keputusan produk (K-1, K-2) dan satu kontrak slice (K-3) yang memblokir pengerjaan Sprint 6 |

---

## K-1 — Kebijakan Penghapusan Reference Tables (Issue #46 / US-8.5)

### Latar belakang

Teks Issue #46 ambigu: satu bullet menyebut *"Validasi: tidak bisa menghapus item yang sedang dipakai"*, bullet lain menyebut *"Soft delete jika sudah dipakai"*. Audit kesesuaian mensyaratkan konflik ini diputuskan sebelum implementasi CRUD.

### Keputusan (disetujui 26 Juli 2026)

**Pola hybrid `is_active`** — tanpa menambah `SoftDeletes`/`deleted_at` ke reference tables.

### Aturan penerapan

1. Setiap reference table yang dikelola dari UI wajib memiliki kolom `is_active` (sudah tersedia di `ref_jabatan`, `ref_unit_kerja`, `ref_notification_channels`; tabel lain ditambah melalui migrasi baru dengan default `true`).
2. **Item yang sudah direferensikan** oleh data pegawai/riwayat/cuti: permintaan hapus **ditolak** (validasi pemakaian di Action + FK `restrictOnDelete` tetap dipertahankan). Aksi yang tersedia hanya **Nonaktifkan** (`is_active = false`) dan **Aktifkan kembali**.
3. **Item yang belum pernah dipakai**: boleh dihapus permanen dengan dialog konfirmasi.
4. Item nonaktif tidak muncul di dropdown input baru, tetapi data historis yang mereferensikannya tetap tampil utuh.
5. Interpretasi resmi frasa Issue #46 *"soft delete jika sudah dipakai"* = **nonaktif via `is_active`**, bukan `SoftDeletes` Laravel.
6. Audit: `CREATE`, `UPDATE`, `DELETE` (hapus permanen item tak terpakai), dan `CONFIG_UPDATE` (nonaktif/aktif) dicatat via `AuditService`; akses seluruh CRUD digerbang `role:super_admin`.
7. Kebijakan ini juga berlaku untuk pengelolaan `ref_notification_channels` dan `notification_event_channels` (kebijakan kanal per event dari PR #122) yang UI-nya dibuat pada Issue #46.

### Alasan

- Sejalan PRD §16.3 (`is_active` — *"Bisa dinonaktifkan tanpa menghapus riwayat"*) dan pola skema yang sudah ada.
- Menghindari migrasi `deleted_at` pada 9+ tabel dan penyesuaian scope query di banyak dropdown.
- Memenuhi kedua kalimat Issue #46 tanpa kontradiksi.

---

## K-2 — Status "Dinas Luar" Ditunda ke Fase 2 (US-8.3 AC-1 / Issue #41)

### Latar belakang

US-8.3 AC-1 meminta daftar bawahan berstatus *aktif/cuti/dinas luar*, tetapi tidak ada dokumen yang mendefinisikan sumber datanya. Kondisi kode saat ini: enum `status_aktif` = `[Aktif, Non-Aktif, Pensiun, Mutasi]`, seeder `ref_status_pegawai` tidak memuat "Dinas Luar", dan PR #125 sempat menyiapkan filter backend `dinas_laut`-nya tanpa sumber data (fitur mati).

### Keputusan (disetujui 26 Juli 2026)

**Status "Dinas Luar" ditunda ke Fase 2.** Sumber datanya kelak adalah **modul Surat Tugas/penugasan** (Fase 2), bukan input manual dan bukan nilai baru di `ref_status_pegawai`.

### Aturan penerapan

1. Fase 1: dashboard dan daftar bawahan Kepala Bagian hanya menampilkan status **`Aktif`** dan **`Cuti`**.
2. **Revisi resmi US-8.3 AC-1 untuk Fase 1**: *"Daftar Bawahan: Nama, jabatan, status (aktif/cuti)"* — "dinas luar" dipindah ke backlog Fase 2.
3. Kode setengah jadi dari PR #125 dibersihkan agar tidak menjadi dead code yang menyesatkan: opsi `dinas_luar` di `KepalaBagianEmployeeFilterRequest`, cabang filter di `ListKepalaBagianEmployeesAction:63-67`, dan badge kondisional `sedang_dinas_luar` di `kabag/bawahan/show.blade.php` (dieksekusi pada task perbaikan Issue #41).
4. Saat modul Surat Tugas dibangun di Fase 2, status dinas luar diturunkan otomatis dari rentang tanggal penugasan aktif — tidak pernah menjadi status yang di-toggle manual.

### Alasan

- Tidak ada sumber data sahih di Fase 1; menambah nilai manual mencampur status kepegawaian resmi dengan status kehadiran operasional dan rawan basi.
- `AGENTS.md` (Fase 1 Boundaries) eksplisit melarang implementasi *assignments/surat tugas* tanpa perubahan scope.

---

## K-3 — Kontrak Payload Dashboard Admin (Issue #39 / US-8.1)

### Arsitektur

- Dibuat **`BuildAdminDashboardAction`** (pola sama dengan `BuildPimpinanDashboardAction`), dipanggil dari `DashboardController@index` untuk role `super_admin` dan `admin_kepegawaian` (satu Dashboard Admin bersama — perbedaan kedua role ada di permission/menu, bukan halaman terpisah, sesuai catatan tracker).
- Controller tetap tipis; Blade tidak menjalankan query/kalkulasi domain; seluruh data dummy, `href="#"`, dan `alert('Ini adalah data dummy')` dihapus.
- Widget W2/W5/W7 boleh me-reuse query yang sudah terbukti benar di `BuildPimpinanDashboardAction` (ekstrak ke service/query object bersama bila mulai duplikat).

### Kontrak field per widget

| Widget | Key payload | Tipe | Sumber data & aturan |
|---|---|---|---|
| W1 Komposisi Pegawai | `totalPegawaiAktif` | int | Pegawai aktif (default scope aktif) |
| | `komposisiPegawai` | array kode jenis → int | Breakdown `PNS` / `PPPK` (+`CPNS` bila ada) dari relasi jenis pegawai |
| W2 Kenaikan Pangkat | `kenaikanPangkatBulanIni`, `kenaikanPangkatTahunIni` | int | `employees.tanggal_kenaikan_pangkat_berikutnya` dalam bulan/tahun berjalan |
| | `daftarKenaikanPangkat` | array maks 5 | `{nama, nip, golongan_awal, golongan_tujuan, tanggal}` — pola golongan asal→tujuan sama dengan Pimpinan |
| W3 Status Cuti | `cutiMenunggu`, `cutiDisetujuiBulanIni`, `cutiDitangguhkan` | int | `leave_requests` per status resmi |
| W4 EWS | `dashboardEwsAlerts` (5 teratas), `dashboardEwsTotal`, `dashboardEwsUrgent/Warning/Info` | existing | **Kontrak existing dipertahankan** (sudah real) |
| W5 Distribusi Golongan | `distribusiGolongan` | array kode → int | Kode golongan **penuh** `I/a`…`IV/e` + `Belum Diisi`; dilarang memotong dengan `explode('/')` |
| W6 Audit Terbaru | `auditTerbaru` | array maks 5 | `{user_name, event, modul, waktu}` — **tanpa** `old_values`/`new_values` (masking; jangan kirim payload audit mentah ke dashboard) |
| W7 Tren Pegawai | `trenPegawai` | array 12 titik | `{label, jumlah}` per bulan, 12 bulan terakhir; metode sama dengan Pimpinan saat ini (basis `created_at` + `tanggal_pensiun`) — batasannya terdokumentasi, dan penyempurnaan basis riwayat (task regresi #24) wajib diberlakukan serempak untuk Admin & Pimpinan |

### Aturan interaksi & tampilan

1. Link widget: W1 → daftar pegawai; W2 → daftar pegawai/EWS pangkat; W3 → monitoring cuti; W4 → halaman EWS; W6 → audit log. **Tidak boleh ada `href="#"`.**
2. **Keputusan cuti tidak dilakukan dari Dashboard Admin** — Admin Kepegawaian bukan approver (PRD §4.2); tombol dummy "Setuju/Tunda" dihapus dan diganti tautan ke halaman monitoring cuti.
3. Widget bonus non-PRD di dashboard lama ("Pegawai Terbaru", "Hari Libur") dihapus, atau bila dipertahankan wajib memakai data database (`ListHariLiburAction`) — bukan array statis.
4. Setiap widget punya empty state jujur; tidak ada fallback angka literal.
5. Responsive desktop/tablet/mobile (AC-9 US-8.1).

### Kriteria selesai slice

- Feature test: akurasi angka tiap widget dengan seed deterministik, kondisi data kosong, role access (`pegawai`/`kepala_bagian` tidak melihat dashboard admin); agregasi diverifikasi juga pada PostgreSQL 17.
- Tidak ada string dummy tersisa di `admin/dashboard.blade.php` (grep `dummy|Ahmad Fauzi|href="#"` = nol).
- Review PR (Adriel) + QA/retest (Grantly) sesuai gate slice.

---

## Konsekuensi ke Backlog & Dokumen

| Item | Dampak |
|---|---|
| Task #16 (Dashboard Admin) | **Unblocked** — kontrak K-3 menjadi acuan FE (dummy fixture) & BE |
| Task #22 (Reference Tables) | **Unblocked** — implementasi mengikuti K-1; tidak ada migrasi `SoftDeletes` |
| Task #23 (Kabag) | **Scope berubah** — sub-item "Dinas Luar" menjadi: bersihkan kode setengah jadi PR #125 + revisi dokumen (K-2); sisa pekerjaan koding: pagination EWS bawahan |
| US-8.3 AC-1 | Direvisi per K-2 (catatan ditambahkan di User-Stories-SIMPEG-Fase1.md) |
| Issue #46 | Catatan kebijakan K-1 ditambahkan di Issues-SIMPEG-Fase1.md |
| PRD §16 | Tidak perlu perubahan — K-1 justru menegaskan pola `is_active` yang sudah ada di PRD §16.3 |
| Backlog Fase 2 | Tambah: modul Surat Tugas menurunkan status "Dinas Luar" (K-2 aturan 4) |
