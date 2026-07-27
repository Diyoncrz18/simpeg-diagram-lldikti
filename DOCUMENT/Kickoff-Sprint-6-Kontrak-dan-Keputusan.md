# Kickoff Sprint 6 — Kontrak & Keputusan (Dashboard & Laporan)

| Field | Detail |
|---|---|
| Tanggal | 26 Juli 2026 |
| Tahap | Kickoff slice Sprint 6 (tahap 1 alur vertical slice) |
| Disetujui oleh | Dion Kobi — System Analyst & Project Manager |
| Status | **Kanonis** — jika bertentangan dengan teks PRD/User Stories/Issues sebelumnya, dokumen ini yang berlaku sampai dokumen sumber direvisi |
| Acuan | PRD-SIMPEG-Fase1-Core.md v1.3 §13–14 & §16, User-Stories US-8.1/8.3/8.5 & US-9.x, Issues #39–#43 & #46, `AGENTS.md`, hasil verifikasi kode HEAD `1b2e5b6` (26 Juli 2026) |
| Cakupan | Tiga keputusan produk (K-1, K-2, K-4) dan satu kontrak slice (K-3) yang memblokir pengerjaan Sprint 6 |

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

## K-4 — `ref_bup` Di-deprecate, Tidak Masuk Cakupan CRUD Fase 1 (Issue #46 / US-8.5)

### Latar belakang

Dari 9 tabel yang tercantum pada Issue #46, `ref_bup` tidak memiliki foreign key, relasi Eloquent, maupun pemanggilan runtime. Perhitungan batas usia pensiun yang berjalan mengambil nilai dari `ref_jabatan.default_bup` lalu jatuh ke `ref_jenis_jabatan.maks_usia_pensiun`. Kolom `jenis_jabatan` pada `ref_bup` berupa teks tanpa foreign key sehingga duplikat semantik dengan `ref_jenis_jabatan.nama`.

### Keputusan (disetujui 27 Juli 2026)

`ref_bup` tidak disambungkan ke perhitungan BUP dan tidak dibuatkan CRUD. Sumber BUP resmi Fase 1 adalah `ref_jabatan.default_bup` sebagai prioritas pertama dengan fallback `ref_jenis_jabatan.maks_usia_pensiun`. Cakupan CRUD Issue #46 berubah dari 9 tabel menjadi **8 tabel**.

### Aturan penerapan

1. Fase 1: hapus entri dari daftar dokumen, hapus baris `ref_bup` pada halaman Pengaturan Sistem, tandai model `RefBup` sebagai `@deprecated`, dan sesuaikan test yang memakainya.
2. Fase 1: tabel, migration, dan seeder `ref_bup` **tetap dipertahankan**. Migrasi `is_active` sudah menyentuh tabel ini dan sudah berjalan; migrasi destruktif menjelang go-live menambah risiko tanpa manfaat operasional.
3. Fase 2: drop tabel, hapus model, hapus seeder.
4. Dilarang membuat halaman kelola `ref_bup`. Bila halaman dibuat, Admin akan mengisi data dan mengira berpengaruh pada perhitungan pensiun padahal tidak dibaca kode mana pun. Pola "keberhasilan palsu" ini sudah ditandai P1 pada audit Role Super Admin dan tidak boleh ditambah.
5. Konsekuensi wajib: CRUD `ref_jabatan` tetap berada dalam cakupan Slice Issue #46 karena menjadi satu-satunya jalur Admin mengatur BUP per jabatan detail. Granularitas `ref_bup` (6 baris) lebih halus daripada `ref_jenis_jabatan` (4 kategori), sehingga tanpa CRUD `ref_jabatan` tingkat presedensi paling spesifik tidak dapat diisi Admin dan janji PRD §1742 belum tertepati.
6. Tab statis "Batas Usia Pensiun" pada halaman Data Master dicabut bersama form modalnya. Tab tersebut menampilkan data hardcoded beserta tombol Tambah, Edit, dan Hapus yang tidak menyimpan apa pun — bentuk keberhasilan palsu yang sama seperti aturan 4.

### Alasan

PRD memposisikan `ref_bup` sebagai opsi kondisional, bukan kewajiban: §16.13 memakai kata "dipakai bila", §16.2 memakai "dapat dioverride oleh `ref_jabatan` atau `ref_bup`", §10 memakai "atau detail `ref_bup`", dan §1 poin 15 tidak menyebutnya sama sekali. Kewajiban substantif PRD hanya dua, yaitu BUP tidak di-hardcode dan Admin dapat memperbaruinya tanpa ubah kode; keduanya sudah dipenuhi jalur yang berjalan. Menyambungkan `ref_bup` justru menciptakan jalur ketiga yang berebut sumber kebenaran dengan dua jalur yang sudah ada, tanpa aturan presedensi.

### Dokumen yang direvisi

| Dokumen | Lokasi | Tindakan |
|---|---|---|
| `Issues-SIMPEG-Fase1.md` | Issue #46, task CRUD | `ref_bup` dikeluarkan; cakupan menjadi 8 tabel |
| `Issues-SIMPEG-Fase1.md` | Issue #11, baris seeder `ref_bup` | **Tidak dicoret**, hanya ditandai deprecated. Baris ini adalah cakupan migration/seeder, dan aturan 2 mempertahankan keduanya di Fase 1 |
| `User-Stories-SIMPEG-Fase1.md` | US-8.5 AC-1 | `ref_bup` dikeluarkan; cakupan menjadi 8 tabel |
| `PRD-SIMPEG-Fase1-Core.md` | §10 baris BUP | Sumber BUP diubah menjadi `ref_jabatan.default_bup` dengan fallback `ref_jenis_jabatan.maks_usia_pensiun` |
| `PRD-SIMPEG-Fase1-Core.md` | §16.2 catatan | `ref_bup` dihapus dari jalur override |
| `PRD-SIMPEG-Fase1-Core.md` | §16.13 | Ditandai DEPRECATED; daftar nilai dipertahankan sebagai acuan domain |
| `PRD-SIMPEG-Fase1-Core.md` | §16.13 catatan penutup | Reference table yang dimaksud diperjelas menjadi `ref_jabatan` dan `ref_jenis_jabatan` |
| `Tracking-Role/Role-Super-Admin.md` | Baris sisa Data Master | Open question `ref_bup` ditutup |
| `Tracking-Sprint-1-7/Sprint-6-Dashboard-dan-Laporan.md` | Status #46 | Open question `ref_bup` ditutup |

### Catatan koreksi rujukan

Instruksi awal menyebut "coret `ref_bup` dari Issue #46 baris 417 dan 1324" serta "CRUD `ref_jabatan` di Issue #46 baris 407". Setelah diverifikasi, baris 407 dan 417 berada di **Issue #11** (cakupan migration/seeder), bukan Issue #46. Task CRUD Issue #46 berada pada satu baris yang memuat `ref_bup` sekaligus `ref_jabatan`. Karena aturan 2 mempertahankan seeder di Fase 1, baris Issue #11 tidak dicoret melainkan ditandai deprecated agar tidak bertentangan dengan keputusan ini.

---

## Konsekuensi ke Backlog & Dokumen

| Item | Dampak |
|---|---|
| Task #16 (Dashboard Admin) | **Unblocked** — kontrak K-3 menjadi acuan FE (dummy fixture) & BE |
| Issue #46 cakupan CRUD | **Berubah** — 9 tabel menjadi 8 tabel per K-4; `ref_bup` keluar, CRUD `ref_jabatan` tetap wajib |
| CRUD `ref_jabatan` | **Prioritas dinaikkan** — syarat sah K-4; tanpa ini Admin tidak dapat mengatur BUP per jabatan detail dan PRD §1742 belum tertepati |
| Backlog Sprint 7 | Tambah P1: presedensi BUP global versus per jabatan. `EwsEngineService` memperlakukan `pensiun_required_age_years` sebagai override penuh, sehingga bila diisi 58 maka pejabat tinggi ikut dihitung 58 dan bertentangan dengan keputusan meeting PRD §926. Urutan yang benar: `ref_jabatan.default_bup`, lalu `ref_jenis_jabatan.maks_usia_pensiun`, lalu global sebagai fallback paling kalah. Catatan teknis: `EwsSchedulerTest` saat ini mengunci perilaku override tersebut sehingga test itu harus diubah lebih dulu |
| Backlog Fase 2 | Tambah: drop tabel `ref_bup`, hapus model, hapus seeder (K-4 aturan 3) |
| Task #22 (Reference Tables) | **Unblocked** — implementasi mengikuti K-1; tidak ada migrasi `SoftDeletes` |
| Task #23 (Kabag) | **Scope berubah** — sub-item "Dinas Luar" menjadi: bersihkan kode setengah jadi PR #125 + revisi dokumen (K-2); sisa pekerjaan koding: pagination EWS bawahan |
| US-8.3 AC-1 | Direvisi per K-2 (catatan ditambahkan di User-Stories-SIMPEG-Fase1.md) |
| Issue #46 | Catatan kebijakan K-1 ditambahkan di Issues-SIMPEG-Fase1.md |
| PRD §16 | Tidak perlu perubahan — K-1 justru menegaskan pola `is_active` yang sudah ada di PRD §16.3 |
| Backlog Fase 2 | Tambah: modul Surat Tugas menurunkan status "Dinas Luar" (K-2 aturan 4) |
