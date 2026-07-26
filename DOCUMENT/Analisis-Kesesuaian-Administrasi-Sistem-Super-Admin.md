# Analisis Kesesuaian Administrasi Sistem — Super Admin

> Status: Audit read-only — **belum sepenuhnya sesuai** dengan dokumen Fase 1.
>
> Tanggal audit: 23 Juli 2026.
>
> Cakupan: informasi dan alur frontend pada menu **Administrasi Sistem** untuk role Super Admin, beserta route, controller, model, request, audit, dan test yang mendukungnya. Tidak ada perubahan kode atau konfigurasi yang dilakukan oleh audit ini.

## 1. Ringkasan Eksekutif

Menu **Administrasi Sistem** Super Admin memuat enam halaman:

1. Kelola Akses User;
2. Role & Permission;
3. Data Master;
4. Hari Libur;
5. Pengaturan Sistem; dan
6. Audit Log.

Tidak ada halaman yang dapat dinyatakan sepenuhnya siap untuk flow Fase 1. Dua halaman menggunakan data nyata tetapi masih memiliki kekurangan material (`Role & Permission` dan `Audit Log`). Empat halaman lain memiliki masalah alur utama, informasi statis, atau keberhasilan palsu.

| Halaman | Status | Ringkasan kondisi saat audit |
|---|:---:|---|
| Kelola Akses User | ❌ | Backend mapping dan audit database tersedia, tetapi nilai role dari form UI tidak sesuai kontrak validasi backend sehingga submit dari browser gagal. |
| Role & Permission | ⚠️ | Matriks menggunakan role/permission database dan `sync()` nyata, tetapi mutation memakai request mentah dan audit hanya disimpan di session. |
| Data Master | ❌ | Data reference, jumlah data, dan interaksi CRUD masih berupa array/mutation lokal dalam Blade. |
| Hari Libur | ❌ | Halaman web memakai daftar statis dan mutation session; API CRUD database yang benar belum dipakai oleh UI web. |
| Pengaturan Sistem | ❌ | Profil instansi, SMTP, mapping SSO, dan ringkasan master statis. Simpan memberi sukses tanpa persistence nyata. |
| Audit Log | ⚠️ | Mengambil data `audit_logs` nyata dan memiliki diff/filter, tetapi semua data dikirim ke browser; immutability dan masking belum cukup ditegakkan oleh source yang diperiksa. |

## 2. Dokumen Acuan

Urutan acuan yang dipakai dalam audit:

1. `PRD-SIMPEG-Fase1-Core.md`:
   - §4.1: Keycloak hanya autentikasi; SIMPEG adalah source of truth untuk role, permission, dan otorisasi.
   - §4.2: Super Admin mengelola reference table, konfigurasi EWS, hari libur, channel notifikasi, chain approval cuti, user mapping, serta audit log.
   - §12: audit log immutable, menyimpan before/after, dan menyediakan filter, detail, pagination, serta sorting.
   - §15.5–16: RBAC internal dan master/reference data berasal dari database dan dapat dikelola tanpa perubahan kode saat record bertambah.
2. `User-Stories-SIMPEG-Fase1.md`, khususnya US-1.4 tentang pemetaan Keycloak ke pegawai dan satu role internal per pegawai.
3. `Panduan-Penulisan-Kode-SIMPEG.md`:
   - mutation memakai FormRequest dan Action;
   - authorization berlapis;
   - Alpine hanya untuk peningkatan UX, bukan menggantikan mutation/audit backend;
   - mutation penting harus memiliki audit resmi dan payload sensitif harus dimasking.
4. `Analisis-Frontend-Backend-Role-Super-Admin.md` sebagai audit terdahulu, bukan pengganti PRD. Kondisi major yang dicatat di sana masih terkonfirmasi pada source saat ini.

## 3. Cakupan Menu dan Akses

### Bukti Graphify dan source

- **EXTRACTED:** node `views/layouts/app.blade.php` menunjuk file `resources/views/layouts/app.blade.php`.
- **SOURCE VERIFIED:** grup `Administrasi Sistem` di layout memuat route `user-management`, `rbac`, `data-master`, `hari-libur`, `pengaturan`, dan `audit-log`.
- **SOURCE VERIFIED:** Konfigurasi EWS berada di grup berbeda, yaitu `EWS & Notifikasi`; tidak dihitung sebagai halaman dalam kelompok Administrasi Sistem ini.
- **SOURCE VERIFIED:** `AdminKepegawaianAccessTest` menguji bahwa Admin Kepegawaian tidak dapat membuka halaman khusus Super Admin, sementara akses Audit Log tetap tersedia sesuai role/permission terkait.

Node view layout berderajat 0 pada graph, sehingga graph tidak mengekstrak seluruh relasi menu → route. Verifikasi source dilakukan setelah file ditunjuk Graphify.

## 4. Hasil Detail per Halaman

### 4.1 Kelola Akses User

#### Yang sudah sesuai

- **EXTRACTED:** node `UserMappingController` memiliki method `index()` dan `update()`.
- **SOURCE VERIFIED:** index mengambil pegawai dan user lokal; update menyimpan `keycloak_id`, role, dan relasi employee, lalu menulis audit database dengan `AuditService::log()`.
- **SOURCE VERIFIED:** `UpdateUserMappingRequest` membatasi role ke lima kode internal: `super_admin`, `admin_kepegawaian`, `pimpinan`, `kepala_bagian`, dan `pegawai`.
- **SOURCE VERIFIED:** route index dan update digerbang `role:super_admin`.
- **SOURCE VERIFIED:** test controller memverifikasi akses, validasi, uniqueness Keycloak ID, persistence, dan audit database.

#### Ketidaksesuaian

- **SOURCE VERIFIED — P0:** form Blade mengirim value `Super Admin`, `Admin Kepegawaian`, `Pimpinan`, `Kepala Bagian`, atau `Pegawai`, sedangkan backend hanya menerima kode internal huruf kecil/underscore. Submit role dari UI akan gagal validasi.
- **SOURCE VERIFIED — P2:** daftar memuat seluruh pegawai/user lalu filter dan pagination dilakukan di browser.
- **SOURCE VERIFIED — P2:** `UpdateUserMappingRequest::authorize()` selalu `true`; mutation hanya bergantung pada gate route tanpa pertahanan berlapis pada request boundary.

#### Dampak

User story US-1.4 adalah P0: Super Admin harus dapat memasangkan Keycloak dengan pegawai dan menetapkan satu role internal. Backend lulus test ketika diberi kode internal secara langsung, tetapi form browser tidak mengirim kode tersebut. Karena itu test controller saja belum cukup menjadi bukti UI berfungsi.

#### Perbaikan

1. Ganti value option menjadi kode internal, misalnya `value="super_admin"` dengan label `Super Admin`.
2. Tambahkan test HTTP/Blade atau browser yang mensubmit nilai option aktual.
3. Gunakan pagination/filter backend bila daftar pegawai membesar.
4. Tegakkan authorization pada FormRequest/policy sesuai pola panduan.

### 4.2 Role & Permission

#### Yang sudah sesuai

- **EXTRACTED:** node `RbacController` memiliki method `index()` dan `update()`.
- **EXTRACTED:** model `Role` memiliki relasi `permissions()`, dan model `Permission` memiliki relasi `roles()`.
- **SOURCE VERIFIED:** controller mengambil role dan permission dari database, mengelompokkan per modul, dan menjalankan `sync()` pada pivot permission role.
- **SOURCE VERIFIED:** kolom Super Admin dibuat checked/disabled pada UI untuk mengurangi risiko lockout lewat interaksi normal.

#### Ketidaksesuaian

- **SOURCE VERIFIED — P1:** update memakai `Request` biasa, tanpa validasi matrix/permission ID melalui FormRequest dan tanpa transaksi eksplisit.
- **SOURCE VERIFIED — P1:** perubahan RBAC hanya dicatat sebagai `dynamic_audit_logs` pada session, bukan ke tabel `audit_logs` immutable.
- **INFERRED — P1:** proteksi self-lockout hanya ada pada Blade. Karena controller menerima matrix mentah dan langsung melakukan `sync()`, request yang dimodifikasi manual berpotensi menghilangkan permission Super Admin. Bukti source menunjukkan tidak ada invariant backend yang memaksa permission inti tetap terpasang.

#### Perbaikan

1. Buat `UpdateRolePermissionMatrixRequest` untuk validasi shape matrix dan ID permission.
2. Pindahkan orchestration ke Action dengan transaction atomik.
3. Wajibkan alasan perubahan dan tulis old/new permission ke audit database.
4. Tegakkan invariant server-side agar Super Admin tidak dapat kehilangan permission minimum.
5. Tambahkan test langsung untuk `POST /rbac/update`; Graphify tidak menemukan node test yang langsung meliput mutation ini.

### 4.3 Data Master

#### Bukti

- **EXTRACTED:** node `data-master/index.blade.php` menunjuk `resources/views/admin/data-master/index.blade.php`.
- **SOURCE VERIFIED:** route `/data-master` hanya merender Blade; tidak menunjuk controller/action CRUD pada route web halaman tersebut.
- **SOURCE VERIFIED:** Blade mendeklarasikan array statis untuk golongan, jenis jabatan, eselon, jenis cuti, agama, unit kerja, BUP, dan hari libur.
- **SOURCE VERIFIED:** tombol simpan modal hanya menutup modal; tidak ada route mutation yang menyimpan reference data.

#### Ketidaksesuaian

Halaman tidak memenuhi PRD bahwa reference tables dapat dikelola oleh Super Admin/Admin berwenang dan record baru dapat bertambah tanpa perubahan kode. Informasi jumlah maupun isi master di halaman bukan source of truth database.

Di halaman ini juga tidak terdapat tab untuk konfigurasi channel notifikasi. PRD memasukkan channel notifikasi dalam hak konfigurasi Super Admin.

#### Batas bukti Graphify

Graphify menemukan `RefNotificationChannel`, migration seed, dan layanan notifikasi, tetapi tidak memberi hubungan yang cukup untuk membuktikan ada atau tidaknya UI konfigurasi channel di seluruh aplikasi. Kesimpulan audit dibatasi menjadi: **halaman Data Master ini tidak menyediakannya**.

#### Perbaikan

1. Ganti data Blade dengan query/reference model nyata.
2. Sediakan CRUD per domain master: route → FormRequest → Action → model → audit.
3. Tolak atau soft-delete secara aman master yang masih direferensikan.
4. Tambahkan pengelolaan hierarki unit kerja, lifecycle jabatan/BUP, dan channel notifikasi.

### 4.4 Hari Libur

#### Yang sudah sesuai pada domain/API

- **EXTRACTED:** graph menunjuk `RefHariLibur`, `CreateHariLiburAction`, `UpdateHariLiburAction`, dan controller API V1 Hari Libur.
- **SOURCE VERIFIED:** `HariLiburCrudTest` menargetkan `/api/v1/hari-libur`, menulis ke `ref_hari_libur`, dan memverifikasi audit database pada create/update/delete.

#### Ketidaksesuaian halaman web

- **EXTRACTED:** node `HariLiburController` web memiliki index/store/edit/update/destroy.
- **SOURCE VERIFIED:** controller web menggunakan `public static $hariLiburData`, bukan query model `RefHariLibur`.
- **SOURCE VERIFIED:** store/update/destroy hanya menambah `dynamic_audit_logs` dalam session dan redirect sukses; tidak memutasi tabel `ref_hari_libur`.
- **SOURCE VERIFIED:** route web `/hari-libur` menunjuk controller statis tersebut, walaupun role dan permission gate route sudah benar.
- **SOURCE VERIFIED:** Blade web juga membangun daftar/filter/pagination dari data lokal.

#### Dampak

UI dapat mengklaim hari libur berhasil ditambah, diubah, atau dihapus, tetapi kalender resmi yang dipakai perhitungan hari kerja/cuti tidak berubah. API yang benar telah tersedia, namun belum menjadi implementasi canonical halaman web.

#### Perbaikan

Gunakan Action dan FormRequest yang sama dengan API (`ListHariLiburAction`, `CreateHariLiburAction`, `UpdateHariLiburAction`, `DeleteHariLiburAction`) pada controller Blade. Hapus array statis dan audit session, lalu tambahkan test untuk route web `/hari-libur`.

### 4.5 Pengaturan Sistem

#### Bukti

- **EXTRACTED:** node `SettingsController` memiliki `index()` dan `update()`.
- **SOURCE VERIFIED:** `update()` tidak membaca atau menyimpan payload request. Ia hanya membuat `dynamic_audit_logs` session generik dan memberi flash sukses.
- **SOURCE VERIFIED:** Blade mendefinisikan profil instansi, data user/Keycloak, ringkasan master, session lifetime, host SMTP `mailpit`, dan port SMTP secara hardcoded.
- **SOURCE VERIFIED:** form menampilkan input editable dan menyatakan audit real-time, tetapi tidak ada persistence atau old/new values aktual.
- **SOURCE VERIFIED:** tab mapping SSO hanya mengubah array Alpine lokal dan tidak mengirim request ke `UserMappingController`.

#### Ketidaksesuaian

Halaman memberi informasi dan feedback seolah konfigurasi sudah nyata, padahal sumber data bukan database/config resmi dan perubahan tidak disimpan. Halaman juga menduplikasi Kelola Akses User serta Data Master dalam bentuk statis.

PRD memberi hak konfigurasi sistem kepada Super Admin, tetapi tidak menetapkan kontrak storage untuk editor profil instansi atau SMTP. Karena itu jangan menambah tabel/konfigurasi baru secara spekulatif.

#### Perbaikan

1. Sembunyikan atau disable form yang belum mempunyai kontrak persistence produk.
2. Hilangkan duplikasi mapping SSO; arahkan ke Kelola Akses User yang telah diperbaiki.
3. Arahkan ringkasan master ke query canonical atau halaman Data Master nyata.
4. Sebelum implementasi persistence baru, tetapkan source of truth dan requirement produk.
5. Setelah keputusan tersedia, gunakan FormRequest → Action → storage → audit database.

### 4.6 Audit Log

#### Yang sudah sesuai

- **EXTRACTED:** node `AuditController` memiliki `index()`, `show()`, dan mapper view.
- **SOURCE VERIFIED:** controller mengambil data dari `AuditLog`, mengurutkan dari terbaru, lalu membentuk payload view.
- **SOURCE VERIFIED:** view menyediakan filter event/user/modul/periode, sorting, pagination UI, drawer detail, dan diff before/after.
- **SOURCE VERIFIED:** test Audit Log API memverifikasi filter dan batas `per_page`; test akses memverifikasi Admin Kepegawaian dapat mengakses audit sesuai kontrak.

#### Ketidaksesuaian dan risiko

- **SOURCE VERIFIED — P2:** `AuditController::index()` memuat seluruh audit log dengan `->get()`. Semua record lalu di-embed sebagai JSON dan difilter/sort/dipaginasi oleh Alpine di browser.
- **SOURCE VERIFIED — P0:** model `AuditLog` hanya menonaktifkan timestamp update; source model tidak menolak operasi `update()` atau `delete()` secara eksplisit. PRD mewajibkan audit tidak dapat diedit atau dihapus, termasuk oleh Super Admin.
- **SOURCE VERIFIED — P0:** mapper view meneruskan `old_values`/`new_values` mentah ke browser. Panduan mewajibkan masking data sensitif yang tidak dibutuhkan.
- **SOURCE VERIFIED — P1:** `AuditService` menangkap kegagalan tulis audit dan hanya menulis warning log. Mutation dapat selesai meskipun audit resmi gagal dibuat.

#### Batas bukti

Source yang diperiksa tidak membuktikan ada/tidaknya constraint PostgreSQL yang melarang update/delete audit log. Kesimpulan yang didukung audit ini adalah: **model dan controller yang diperiksa belum menegakkan immutability/masking secara eksplisit**.

#### Perbaikan

1. Tegakkan immutability pada aplikasi dan database.
2. Mask payload sebelum disimpan atau sebelum dikirim ke view sesuai kebutuhan akses.
3. Gunakan Action/query backend dengan filter, sort, dan paginator database.
4. Tentukan kebijakan kegagalan audit untuk mutation yang auditnya wajib.

## 5. Hasil Verifikasi Test

Test terarah yang dijalankan pada 23 Juli 2026:

| Test | Hasil | Interpretasi |
|---|---:|---|
| `UserMappingControllerTest` | 9 test / 14 assertion lulus | Backend menerima kode role internal, tetapi test belum mensimulasikan nilai option Blade yang saat ini salah. |
| `HariLiburCrudTest` | 21 test / 71 assertion lulus | API `/api/v1/hari-libur` benar; tidak membuktikan route web `/hari-libur` memakai API/Action tersebut. |
| `AuditLogIndexTest` + `AdminKepegawaianAccessTest` | 9 test / 64 assertion lulus | Filter API dan gate akses utama bekerja; tidak membuktikan payload Blade sudah server-paginated/masked. |

Total: **39 test dan 149 assertion lulus** untuk subset yang dijalankan.

## 6. Backlog Prioritas

### P0

1. Perbaiki value role pada form Kelola Akses User dan tambahkan test browser/Blade.
2. Tegakkan audit immutable dan masking payload Audit Log.

### P1

1. Ubah Data Master dari static Blade menjadi CRUD reference data nyata.
2. Hubungkan Hari Libur web ke implementasi API/Action yang sudah benar.
3. Hentikan form dan audit palsu pada Pengaturan Sistem sampai contract setting disetujui.
4. Gunakan FormRequest, Action, transaction, audit database, dan invariant Super Admin pada update RBAC.
5. Tambahkan test langsung untuk mutation RBAC dan halaman web Hari Libur.

### P2

1. Terapkan filter/pagination backend untuk User Management dan Audit Log.
2. Lengkapi authorization berlapis pada FormRequest/policy untuk mutation administrasi.
3. Rapikan controller agar mengikuti pola tipis: Request → Action → response.

## 7. Status Akhir

Status audit terdahulu **masih relevan secara material**. Perubahan terbaru yang perlu segera ditangani bukan kosmetik UI, melainkan integritas informasi dan keterhubungan frontend dengan source of truth:

- User Management memiliki backend nyata tetapi kontrak form rusak.
- Role & Permission menyimpan matrix nyata tetapi belum memiliki mutation audit yang andal.
- Data Master, Hari Libur web, dan Pengaturan Sistem masih menampilkan atau menyimpan informasi yang tidak nyata.
- Audit Log menampilkan data nyata tetapi masih memerlukan hardening security, masking, dan pagination server-side.

