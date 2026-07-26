# Rencana Eksekusi — Kelola Akses User Super Admin

> Status: Fase 1 sampai Fase 4 telah diimplementasikan dan diuji pada branch `fix/user-management-phase-1`. Release gate masih menunggu perbaikan style pada file workspace yang tidak terkait, sebagaimana dicatat pada `Bukti-QA-Kelola-Akses-User-Super-Admin.md`.
>
> Tanggal: 23 Juli 2026.
>
> Fokus: memperbaiki halaman `Kelola Akses User` (`user-management`) agar Super Admin dapat memetakan akun Keycloak ke pegawai dan menetapkan satu role internal SIMPEG secara benar, aman, teraudit, dan dapat diuji.

## 1. Tujuan dan Batas Fokus

### Tujuan

Menyelesaikan alur utama berikut:

```text
Super Admin membuka daftar pegawai
  -> memilih pegawai yang benar
  -> mengisi/mengubah identitas Keycloak sesuai kontrak yang disetujui
  -> memilih satu role internal SIMPEG
  -> backend memvalidasi dan menyimpan mapping secara atomik
  -> audit database tercatat aman
  -> daftar menampilkan status mapping yang benar setelah refresh
```

### Di dalam scope

- kontrak value role pada form Blade;
- pemetaan Keycloak ↔ pegawai dan satu role internal per pegawai;
- validasi, otorisasi, audit, integritas data, dan error state;
- filtering/pagination daftar User Management;
- test backend, kontrak HTML, dan QA manual halaman.

### Di luar scope rencana ini

- perubahan matriks `Role & Permission`;
- pengaturan sistem statis, Data Master, Hari Libur, atau konfigurasi channel notifikasi;
- konfigurasi server Keycloak, Client ID/Secret, atau pembuatan login manual;
- perubahan aturan role, dashboard, ataupun alur approval cuti;
- migrasi massal tanpa laporan konflik dan keputusan data owner.

## 2. Acuan dan Baseline Audit

### Dokumen sumber kebenaran

1. `PRD-SIMPEG-Fase1-Core.md` §4.1–4.2: Keycloak hanya autentikasi; SIMPEG adalah source of truth role/permission. Super Admin melakukan mapping user dan assignment role internal.
2. `User-Stories-SIMPEG-Fase1.md` US-1.4 (P0): daftar pegawai beserta status mapping, input Keycloak ID/email, satu role internal, unique Keycloak account, dan audit perubahan.
3. `Panduan-Penulisan-Kode-SIMPEG.md`: mutation menggunakan FormRequest dan Action; authorization berlapis; audit mutation penting; Alpine tidak menggantikan mutation backend.
4. `Tracking-Role/Role-Super-Admin.md`: tracking kesesuaian terkini role Super Admin (mengonsolidasi audit Administrasi Sistem 23 Juli yang menjadi dasar rencana ini).

### Bukti Graphify dan source saat ini

- **EXTRACTED:** node `UserMappingController` memiliki `index()` dan `update()`.
- **EXTRACTED:** node `Role` memiliki relasi `permissions()` dan node `Permission` memiliki relasi `roles()`.
- **SOURCE VERIFIED:** `UpdateUserMappingRequest` hanya menerima lima kode role internal: `super_admin`, `admin_kepegawaian`, `pimpinan`, `kepala_bagian`, dan `pegawai`.
- **SOURCE VERIFIED:** modal Blade saat ini mengirim label role (`Super Admin`, `Admin Kepegawaian`, dan seterusnya), bukan kode internal. Ini adalah penyebab utama submit UI gagal.
- **SOURCE VERIFIED:** controller saat ini memakai email sebagai payload utama dan dapat membuat/mengubah `User`; ia juga mencari pegawai dengan `email` atau `email_pribadi`.
- **SOURCE VERIFIED:** route sudah membatasi halaman dan mutation pada `role:super_admin`.

### Risiko baseline yang harus ditutup

1. Nilai role dari UI dan backend tidak sama.
2. Identifier employee yang dikirim form masih berupa email, sehingga tidak ideal sebagai key canonical.
3. Daftar memuat semua pegawai/user lalu difilter dan dipaginasi di browser.
4. FormRequest belum menjadi authorization layer karena `authorize()` selalu `true`.
5. Audit mapping saat ini merekam identitas Keycloak secara penuh; payload audit perlu ditinjau dan dimasking sesuai kebutuhan.
6. Role `null` dapat tersamarkan menjadi `pegawai` oleh state frontend, padahal keduanya memiliki arti otorisasi yang berbeda.
7. Mengosongkan `keycloak_id` belum memiliki arti disconnect yang konsisten, karena callback SSO dapat menghubungkan ulang user tertentu berdasarkan email terverifikasi.

## 3. Keputusan yang Harus Dikunci Sebelum Tahap Hardening

Keputusan berikut telah disetujui pada 23 Juli 2026 sebelum mengubah kontrak mapping, melakukan backfill data, atau menjalankan Fase 2:

| Keputusan | Opsi | Keputusan disetujui | Alasan |
|---|---|---|---|
| Identifier canonical Keycloak | hanya `sub`/Keycloak ID stabil; atau menerima ID dan email sebagai tipe berbeda | `users.keycloak_id` menyimpan `sub` stabil. `users.employee_id` menjadi target payload canonical. Email hanya discovery legacy/tampilan dan bukan identifier POST. | PRD/US menyebut Keycloak ID atau email, sedangkan implementasi sekarang menyimpan keduanya dalam satu `keycloak_id`. Semantik harus jelas agar tidak salah mapping. |
| User legacy tanpa `employee_id` | backfill otomatis dari email; atau laporan konflik untuk verifikasi admin | Tidak ada backfill atau migrasi massal pada Fase 2. Discovery email hanya untuk satu kecocokan aman; konflik/null dilaporkan dan tidak dipetakan otomatis. | Hindari mengikat akun ke pegawai yang salah. |
| Permission granular mapping | hanya gate `role:super_admin`; atau role + permission khusus | Pertahankan gate `role:super_admin`; FormRequest memeriksa role yang sama sebagai defense in depth. Permission baru tidak dibuat sampai nama canonical disetujui. | Panduan meminta defense in depth. Nama permission harus diverifikasi dari seed, bukan diasumsikan. |
| Semantik disconnect Keycloak | izinkan re-link otomatis setelah disconnect; tandai `manual_unmap` agar re-link ditolak; atau larang pengosongan identifier dari UI | Pengosongan `keycloak_id` ditolak/nonaktif pada Fase 2. Desain `manual_unmap` dan callback re-link menjadi pekerjaan terpisah. | Mengosongkan identifier saat ini tidak selalu memutus koneksi secara permanen; login berikutnya dapat memetakan ulang user melalui email terverifikasi. |
| Source of truth mapping/role | `users` canonical dengan field legacy `employees` dipertahankan sementara; atau sinkronisasi/migrasi terpisah yang disetujui | `users` adalah sumber kebenaran akun aplikasi, `keycloak_id`, role internal, dan `employee_id`. Field legacy `employees` tidak disinkronkan otomatis pada Fase 2. | Flow aktif callback dan halaman admin memakai `users`; memperbarui dua sumber tanpa keputusan akan menimbulkan drift data. |
| Kebijakan kegagalan audit | strict audit: rollback mapping; atau best-effort audit dengan monitoring dan pemulihan | Strict audit: mapping dan audit berada dalam satu transaksi; kegagalan audit membatalkan mapping. Identifier Keycloak dimasking. | Audit service yang menelan exception tidak otomatis membuat mapping dan audit atomik. Kebijakan ini menentukan perilaku transaction dan release gate. |

Tidak ada perubahan Keycloak eksternal, penebakan identity mapping, sinkronisasi field legacy, atau definisi disconnect yang boleh diimplementasikan sebelum keputusan ini tersedia.

## 4. Urutan Eksekusi

### Fase 0 — Kontrak dan baseline terukur

**Tujuan:** mengunci perilaku yang sudah benar sebelum perubahan.

| ID | Task | Output | Kriteria selesai |
|---|---|---|---|
| UA-00 | Dokumentasikan seluruh keputusan pada §3. | Keputusan tertulis di dokumen produk/teknis yang disetujui. | Tidak ada ambiguitas tentang `sub`, email SSO, legacy mapping, source of truth `users`, permission gate, semantik disconnect, dan kebijakan audit. |
| UA-01 | Tambahkan regression test yang menangkap bug UI saat ini. | Test HTML/feature yang memeriksa `option value` role dan submit sesuai form. | Test gagal pada value label lama dan akan lulus setelah perbaikan. |
| UA-02 | Ambil inventaris mapping legacy secara read-only. | Laporan jumlah user: punya `employee_id`, hanya cocok email tunggal, konflik, dan tidak cocok. | Tidak ada data diubah; konflik dapat ditinjau pemilik data. |

### Fase 1 — Perbaikan frontend P0: kontrak role dan error state

**Tujuan:** membuat form yang sekarang ada benar-benar mengirim kontrak yang diterima backend tanpa menunggu refactor besar.

| ID | Task frontend | File/area sasaran | Kriteria penerimaan |
|---|---|---|---|
| UA-10 | Ubah setiap `option value` **dan nilai filter role** ke kode internal; label tetap Bahasa Indonesia. | `resources/views/admin/user-management/index.blade.php`: modal dan filter tabel. | Browser/filter memakai salah satu dari `super_admin`, `admin_kepegawaian`, `pimpinan`, `kepala_bagian`, `pegawai`; label tampilan tidak dipakai sebagai payload/filter value. |
| UA-11 | Gunakan pemetaan kode → label yang konsisten untuk `x-model`, filter, badge, dan warning Super Admin; tangani role `null` secara eksplisit. | State `selectedEmployee`, modal, filter, badge role, conditional warning. | Role dari database tampil dengan label yang benar; warning Super Admin muncul saat value `super_admin`; role `null` tampil sebagai **Belum diberi role** dan tidak otomatis berubah menjadi `pegawai` ketika modal dibuka. |
| UA-12 | Tampilkan error validasi server per field, pesan uniqueness, dan state modal setelah redirect. | Input target pegawai/Keycloak ID/role, area flash/error, dan inisialisasi Alpine. | Pengguna mengetahui field yang salah tanpa harus menebak; modal dibuka kembali dengan `old()` yang aman; error terhubung secara aksesibel melalui label dan `aria-describedby`. |
| UA-13 | Tegaskan arti field identitas Keycloak serta disconnect berdasarkan keputusan §3. | Label, placeholder, help text. | UI tidak menyatakan “ID / Email” bila backend hanya menganggapnya `sub`; UI tidak menjanjikan disconnect permanen bila callback masih dapat me-link ulang; bila dua jenis didukung, keduanya dibedakan eksplisit. |
| UA-14 | Pastikan mapping tidak dapat disimpan sebagai perubahan lokal Alpine saja. | Form modal dan submit server. | Satu-satunya aksi simpan mengirim form ke backend dan halaman memuat ulang data nyata. |

**Batas fase 1:** tidak menambahkan data dummy atau endpoint baru. Perbaikan value role harus compatible dengan `UpdateUserMappingRequest` yang sudah ada.

### Fase 2 — Kontrak payload canonical dan hardening backend

**Tujuan:** menghapus ketergantungan email sebagai identifier mutation serta memastikan setiap mapping terkait tepat satu pegawai.

| ID | Task | Rencana implementasi | Kriteria penerimaan |
|---|---|---|---|
| UA-20 | Ganti payload target dari email menjadi `employee_id` UUID. | View menyertakan employee UUID; FormRequest memvalidasi UUID dan keberadaan pegawai; email hanya menjadi informasi tampilan atau fallback legacy terkontrol. | Request tidak dapat memetakan user ke pegawai yang dipilih melalui email yang dimodifikasi; input email yang tidak dikenal/tidak relevan tidak dapat membuat user orphan. |
| UA-21 | Buat `UpdateUserMappingAction`. | Controller menjadi adapter HTTP: request tervalidasi → action → redirect. Action menangani lookup user, uniqueness, relasi employee, persistence, dan audit. | Tidak ada orchestration mapping/decision audit besar di controller. |
| UA-22 | Jadikan `employee_id` relasi canonical dan `users` sebagai source of truth mapping/role. | Cari user terlebih dahulu dengan `users.employee_id`; fallback legacy email hanya sesuai keputusan §3, tidak boleh ambigu, dan tidak menyinkronkan field legacy `employees` tanpa strategi yang disetujui. | Satu user tidak terikat ke pegawai yang salah; user baru tidak dibuat bila employee target tidak valid; hanya satu sumber data aktif yang diperbarui. |
| UA-23 | Verifikasi dan gunakan uniqueness Keycloak/employee yang sudah ada secara benar. | Inventaris constraint/index PostgreSQL aktual; gunakan transaction, lock/validasi yang sesuai, dan penanganan exception unique. Buat migrasi hanya bila ditemukan drift skema yang nyata. | Dua pegawai tidak bisa memakai identifier Keycloak yang sama atau satu employee dipetakan ke dua user, termasuk request paralel; konflik database berubah menjadi error domain/validasi yang aman, bukan HTTP 500. |
| UA-24 | Perkuat authorization. | `authorize()` memeriksa Super Admin dan permission canonical yang disetujui; route memakai gate yang sama. | Request langsung dari role/permission yang tidak berhak menerima 403. |
| UA-25 | Buat audit mapping resmi, aman, dan sesuai kebijakan kegagalan audit §3. | Audit berisi actor, employee ID, role lama/baru, status mapping; identifier Keycloak dimasking atau hanya metadata yang diperlukan. Action menerapkan pilihan strict rollback atau best-effort dengan monitoring/pemulihan. | Record audit database tercipta untuk create/update/disconnect tanpa payload sensitif berlebih; kegagalan audit mengikuti kebijakan yang disetujui dan dapat diuji. |

### Fase 3 — Daftar data nyata yang scalable

**Tujuan:** membuat informasi pada halaman selalu berasal dari database dan tetap aman ketika data bertambah.

| ID | Task | Rencana implementasi | Kriteria penerimaan |
|---|---|---|---|
| UA-30 | Buat `ListUserMappingsAction` atau query object. | Eager load data reference yang dibutuhkan; gunakan `users.employee_id` sebagai relasi employee-user prioritas; fallback email legacy hanya bila diperlukan dan aman; filter `search`, `role`, dan `status`; urutkan deterministik. | Blade menerima paginator/data mapping, bukan seluruh `Employee::get()` dan `User::all()`; mapping aktif tidak lagi bergantung pada exact email equality. |
| UA-31 | Pindahkan filter/pagination ke query parameter backend. | Gunakan named route dan pertahankan filter saat berpindah halaman. | Filter nama/NIP/email, role, dan status benar setelah reload dan pada data lebih dari satu halaman. |
| UA-32 | Tampilkan status mapping dengan jujur. | Bedakan: belum ada user lokal, user lokal tanpa role, identifier Keycloak kosong, dan terhubung. | Role kosong tidak ditampilkan sebagai `pegawai` default bila belum benar-benar ditetapkan. |
| UA-33 | Tinjau payload PII halaman. | Kirim hanya nama, NIP yang diperlukan untuk Super Admin, status, role, dan identifier yang diperlukan; hindari data pribadi tambahan. | Data browser minimal namun tetap memenuhi US-1.4. |

### Fase 4 — Validasi, QA, dan release gate

**Tujuan:** memastikan feature benar pada UI, domain, security, dan audit.

| ID | Test/QA | Skenario minimum |
|---|---|---|
| UA-40 | Authorization test | Guest redirect; Admin Kepegawaian/Pimpinan/Kepala Bagian/Pegawai menerima 403; Super Admin berhak akses. |
| UA-41 | Kontrak UI test | HTML mengandung kode role internal pada `option value` modal dan filter; label Bahasa Indonesia tetap tampil; form submit aktual menyimpan role yang dipilih; role `null` tampil sebagai **Belum diberi role** dan tidak di-default menjadi `pegawai` saat modal dibuka. |
| UA-42 | Validation test | Role label lama/invalid ditolak; employee UUID invalid ditolak; identifier Keycloak invalid/duplikat ditolak; payload email legacy yang tidak dikenal atau dimodifikasi tidak dapat membuat user orphan; error tampil di field dan modal kembali terbuka. |
| UA-43 | Integrity test | User tidak dapat dipetakan ke employee yang tidak ada; tidak membuat user orphan; satu identifier Keycloak tidak dapat dipakai dua employee; satu employee tidak dapat dipetakan ke dua user; konflik unique PostgreSQL dan kasus parallel diterjemahkan menjadi respons domain yang aman. |
| UA-44 | Audit test | Create/update menghasilkan audit database dengan before/after relevan, actor, IP/UA, dan identifier yang sudah dimasking; test membuktikan perilaku saat audit gagal sesuai keputusan strict rollback. Sesuai keputusan §3, disconnect masih ditolak sehingga tidak boleh menghasilkan mutation atau audit mapping. |
| UA-45 | Pagination/filter test | Hasil pencarian/status/role berasal dari query backend, page size stabil, dan filter bertahan saat pindah halaman. |
| UA-46 | Manual QA desktop/mobile | Pilih semua lima role, ubah mapping, reload halaman, cek badge/status, keyboard modal, label/ID/error accessibility, serta tampilan lebar kecil. |
| UA-47 | Regression suite | Jalankan test terarah terlebih dahulu, lalu `composer qa` bila environment memungkinkan. |

## 5. Urutan Pull Request yang Direkomendasikan

| PR | Isi | Ketergantungan | Catatan review |
|---|---|---|---|
| PR-1 | UA-01 dan UA-10 s.d. UA-14: perbaikan value role modal/filter, label tampilan, role `null`, error state, dan test kontrak UI. | Tidak menunggu keputusan identifier canonical, tetapi tidak boleh menjanjikan disconnect yang belum diputuskan. | PR kecil, aman, dan segera memulihkan alur P0 yang ada. |
| PR-2 | UA-00, UA-20 s.d. UA-25: identifier canonical, request/action, authorization, transaction, audit. | Keputusan §3 harus selesai. | Jangan merge bila strategi legacy/backfill belum terdokumentasi. |
| PR-3 | UA-30 s.d. UA-33: query backend, paginator, status jujur, payload minimal. | PR-2. | Uji data lebih dari satu halaman dan user legacy. |
| PR-4 | UA-40 s.d. UA-47: kelengkapan test/QA/retest dan bukti release. | PR-1 sampai PR-3. | Tidak ada defect P0/P1 terbuka. |

## 6. Definition of Done

Kelola Akses User dapat dinyatakan selesai hanya jika seluruh kondisi berikut terpenuhi:

- Super Admin dapat menyimpan salah satu dari lima role internal melalui form browser yang nyata.
- Nilai yang ditampilkan UI, dikirim request, disimpan database, dan dibaca middleware adalah kode role yang sama.
- Mapping selalu mengarah ke employee UUID canonical; tidak ada pembuatan user orphan atau fallback email ambigu.
- `users` menjadi sumber kebenaran mapping/role; setiap penggunaan atau pengakhiran field legacy `employees` mengikuti keputusan terdokumentasi.
- Satu identifier Keycloak tidak dapat dipetakan ke lebih dari satu employee.
- Perilaku disconnect Keycloak dan perilaku login sesudahnya sesuai keputusan yang disetujui serta tercermin pada UI, callback, dan audit.
- User yang belum memiliki role tetap tampil dengan status/role kosong secara jujur dan tidak memperoleh akses fitur hanya dari role Keycloak.
- Setiap mutation mapping tercatat pada audit database dengan payload aman/masked.
- Gate route, FormRequest, dan policy/service menolak actor yang tidak berhak.
- List, filter, dan pagination menggunakan query backend serta tidak mengirim seluruh data ke browser.
- Test UA-40 sampai UA-47 lulus dan QA manual memiliki bukti retest.

## 7. Risiko dan Mitigasi

| Risiko | Mitigasi |
|---|---|
| Semantik `keycloak_id` bercampur antara `sub` dan email. | Kunci keputusan §3 sebelum mengubah data/kontrak. Pisahkan field atau migrasi hanya jika kebutuhan produk disetujui. |
| User legacy belum terkait `employee_id`. | Laporan read-only; auto-backfill hanya untuk kecocokan tunggal; konflik diputuskan pemilik data. |
| Field mapping/role legacy pada `employees` berbeda dengan `users`. | Tetapkan `users` sebagai source of truth; jangan sinkronkan dua lokasi tanpa strategi migrasi/deprecation yang disetujui. |
| Disconnect tampak berhasil tetapi callback melakukan re-link otomatis pada login berikutnya. | Kunci semantik disconnect sebelum Fase 2; samakan UI, callback, audit, dan test dengan keputusan tersebut. |
| Update paralel memakai identifier Keycloak sama. | Constraint database dan transaction; tangani exception unique dengan pesan validasi yang aman. |
| Perbaikan UI lulus test controller tetapi gagal browser. | Tambahkan assertion HTML/form submit dan QA manual pada PR-1. |
| Audit menyimpan identifier/payload berlebihan. | Bentuk payload audit khusus dan test masking sebelum release. |
| Audit gagal setelah mapping disimpan. | Terapkan kebijakan strict rollback atau best-effort dengan monitoring/pemulihan yang disetujui; verifikasi melalui test kegagalan audit. |
| Permission mapping belum mempunyai nama canonical yang konsisten. | Inventaris seed/middleware sebelum menambah gate permission; jangan menebak nama permission. |

## 8. Catatan Graphify

Graphify memberikan node dan method yang mendukung baseline controller/model. Namun beberapa Blade view berderajat 0, sehingga hubungan controller → view tidak terekstrak langsung. Source view hanya dibaca setelah file ditunjuk hasil query Graphify.

Kesimpulan yang diberi label **INFERRED** pada rencana ini adalah risiko request yang dimodifikasi manual dan self-lockout/ambigu identity; keduanya harus dibuktikan dengan test keamanan sebelum diperlakukan sebagai defect terselesaikan.
