# User Stories — SIMPEG Fase 1 (Core)
## LLDIKTI Wilayah XVI

| Field | Detail |
|-------|--------|
| **Berdasarkan** | PRD-SIMPEG-Fase1-Core.md v1.4 |
| **Tanggal** | 22 Juli 2026 |
| **Pembaruan status terakhir** | 7 Agustus 2026 |
| **Basis verifikasi status** | Branch `development` @ `9be633d` (setelah PR #123–#166) |
| **Total User Stories** | 53 |
| **Total Epics** | 9 |

> **Catatan sinkronisasi PRD 1.3:** Keycloak digunakan hanya untuk SSO/login. Role dan permission dikelola di database SIMPEG. Approval cuti memakai chain dinamis per pegawai/unit, status keputusan resmi (`Disetujui`, `Perubahan`, `Ditangguhkan`, `Tidak Disetujui`), cuti tahunan tidak boleh lintas tahun, EWS menambahkan Satyalancana, notifikasi harus channel-configurable, dan laporan mendukung export nominatif Excel custom.
>
> **Keputusan import Fase 1 (kanonis, disetujui pengguna 22 Juli 2026):** import massal hanya mengaktifkan template Data Utama. Import membuat record pegawai beserta field snapshot awal, tidak membuat riwayat kepangkatan/jabatan/KGB, dan tidak memanggil kalkulasi TMT. Riwayat resmi diinput per pegawai melalui CRUD append-only. Tanggal pensiun hasil import dipertahankan apa adanya. Kalkulasi TMT dipicu saat riwayat/sumber resmi disimpan, bukan saat import selesai. Template lanjutan multi-jenis tidak termasuk ruang lingkup saat ini.

---

## Pembaruan Status Acceptance Criteria — 7 Agustus 2026

Penyelarasan terhadap modul Cuti setelah peninjauan ulang kriteria US-4.2. Basis pemeriksaan adalah branch `development` @ `eba7f1c` setelah PR #171 masuk. Mengikuti aturan penyelarasan 5 Agustus 2026, centang diberikan karena perubahan sudah berada di `development` dan terbukti pada source beserta test.

### Kriteria yang dinaikkan menjadi selesai

| User Story | Kriteria | Bukti implementasi |
|---|---|---|
| US-4.2 | AC-4 | Lapisan query menerima nilai `YYYY` selain `YYYY-MM` melalui `App\Support\Cuti\CutiPeriodFilter`, penyaringan dijalankan di basis data sebagai rentang tanggal setengah terbuka, dan daftar opsi tahun dibentuk dari rentang data dalam scope pengguna. Dikunci `tests/Unit/Cuti/CutiPeriodFilterTest.php` dan `tests/Feature/CutiListPeriodFilterTest.php` |

Dengan masuknya AC-4, seluruh acceptance criteria US-4.2 berstatus selesai.

### Koreksi kriteria yang sudah bertanda selesai

| User Story | Kriteria | Koreksi |
|---|---|---|
| US-4.2 | AC-2 | Kriteria sudah bertanda selesai sejak sebelumnya, namun dua warna menyimpang dari ketetapan: Menunggu memakai biru dan Perubahan memakai merah, sehingga permintaan perubahan tampak identik dengan penolakan. Kelima warna kini mengikuti AC-2 dan dikunci test |

### Keputusan warna status cuti

Diputuskan 7 Agustus 2026 oleh Lead Backend sebagai pemilik modul Cuti, karena pokok ini bersifat teknis dan tidak mengubah aturan bisnis. Berlaku sampai ada keputusan yang lebih baru.

| Pokok | Keputusan | Alasan |
|---|---|---|
| `ditangguhkan_tugas_dinas` | Mengikuti warna Ditangguhkan, yaitu oranye | Keduanya sama-sama menahan pengajuan tanpa menjadi keputusan akhir, sehingga membedakan warnanya hanya menambah beban baca tanpa menyampaikan informasi baru |
| `dikembalikan_karena_rollover` | Warna netral, bukan salah satu dari lima warna keputusan | Status ini terbit dari proses rollover saldo, bukan dari keputusan approval, sehingga tidak boleh tampak seperti persetujuan maupun penolakan |
| Cakupan ketetapan warna AC-2 | Berlaku untuk seluruh permukaan yang menampilkan status cuti, bukan hanya daftar pengajuan pegawai | Satu status yang tampil dengan warna berbeda antar halaman membuat pengguna salah membaca keputusan |

Sisa pekerjaan penerapan cakupan di atas: `resources/views/admin/cuti/show.blade.php`, `resources/views/pegawai/dashboard.blade.php`, dan `resources/views/pimpinan/cuti/index.blade.php` masih memetakan status Perubahan ke warna merah sehingga tampak identik dengan Tidak Disetujui. Halaman detail pengajuan adalah kasus paling terasa karena pengguna membukanya langsung dari daftar dan melihat warna yang berbeda untuk status yang sama. Penerapan ditunda karena ketiga berkas tersebut sedang diubah oleh PR #163, #167, #169, dan #172; mengubahnya lebih dulu hanya akan menimbulkan konflik. Dikerjakan setelah PR tersebut masuk `development`.

### Butuh keputusan produk

| Pokok | Keadaan | Usul |
|---|---|---|
| Status `Draft` pada PRD | PRD §9 masih menyebut `Draft` sebagai status proses yang harus ditampilkan, sedangkan migrasi dukungan pengembalian rollover memetakan `Draft` menjadi `menunggu_approval` dan batasan kolom status hasil migrasi terbaru tidak lagi mengizinkan nilai tersebut | Hapus `Draft` dari daftar status proses pada PRD, atau kembalikan dukungannya pada skema bila fitur simpan sebagai draf memang masih diinginkan |

---

## Pembaruan Status Acceptance Criteria — 5 Agustus 2026

Penyelarasan status dilakukan terhadap kode aktual branch `development` @ `4839ab6`, mencakup hasil PR #123–#162. Centang hanya diberikan pada kriteria yang terbukti pada source dan/atau test, bukan berdasarkan status tracker.

### Kriteria yang dinaikkan menjadi selesai

| User Story | Kriteria | Bukti implementasi |
|---|---|---|
| US-1.2 | AC-2 | Logout memanggil `Auth::logout()`, invalidasi session, dan regenerasi token sebelum memutus sesi Keycloak |
| US-1.3 | AC-2, AC-4 | Batas idle dibaca dari `SIMPEG_SESSION_IDLE_TIMEOUT` melalui `config/session.php` dan ditegakkan middleware; pesan *"Sesi Anda telah berakhir. Silakan login kembali."* tersimpan saat sesi berakhir. AC-2 selesai setelah revisi kriteria oleh keputusan K-US-03 |
| US-4.1 | AC-3, AC-7 | Form membaca hasil kalkulasi hari kerja dari endpoint; `StoreLeaveRequestRequest` menolak pengajuan saat rantai approval tidak dapat diresolusi |
| US-4.2 | AC-5 | Daftar cuti memakai paginator server-side dengan pilihan jumlah baris dan mempertahankan query string |
| US-4.4 | AC-2 | Detail pengajuan bawahan menampilkan identitas pemohon, jenis, rentang tanggal, jumlah hari, alasan, dan lampiran |
| US-4.7 | AC-1, AC-3 | Detail pengajuan memuat informasi lengkap beserta lampiran; akses baca dibatasi pemohon, approver snapshot, dan pemantau ber-izin baca menyeluruh |
| US-4.10 | AC-1, AC-4, AC-5 | Halaman konfigurasi approval chain tersedia; rantai diresolusi lalu di-snapshot pada pengajuan baru; verifikator dapat menunjuk pegawai tertentu tanpa role baru |
| US-4.11 | AC-1, AC-5 | Detail pegawai menampilkan kepala bagian aktif; audit penetapan bersifat fail-closed di dalam transaksi |
| US-4.12 | AC-1, AC-2 | Kalkulator hari kerja mengecualikan akhir pekan dan membaca tanggal libur dari `ref_hari_libur` sesuai rentang tahun |
| US-5.1 | AC-1, AC-3 | Jadwal EWS membaca jam dari konfigurasi dengan proteksi overlap; kelayakan kenaikan pangkat dihitung dari masa kerja, hukuman disiplin aktif, dan flag kinerja |
| US-6.2 | AC-5 | Klik notifikasi menandai dibaca lalu mengarahkan ke halaman terkait |
| US-7.2 | AC-5, AC-6 | Urutan default terbaru di atas; akses digerbang role Super Admin/Admin Kepegawaian beserta izin baca audit |
| US-7.3 | AC-2, AC-4 | Halaman detail menampilkan operator, waktu, IP, user agent, event, modul, dan ID record, serta tombol menuju record terkait |
| US-8.1 | AC-1, AC-2, AC-3, AC-5, AC-6, AC-7, AC-8, AC-9 | Dashboard Admin memakai payload nyata kontrak K-3 dengan tata letak responsif |
| US-8.3 | AC-3 | EWS bawahan dibatasi pada bawahan langsung dan disajikan berpaginasi |
| US-9.1B | AC-4 | Export kustom mempertahankan urutan kolom sesuai pilihan pengguna |
| US-9.2 | AC-2, AC-3, AC-4, AC-5 | PDF daftar pegawai memuat identitas instansi, judul, tanggal cetak, tabel sesuai filter, penomoran halaman, dan orientasi lanskap |
| US-9.4 | AC-3, AC-4, AC-5 | Template PDF rekap cuti memuat tabel rekap per pegawai, area tanda tangan, dan footer bernomor halaman |

### Kriteria yang sengaja tetap terbuka

| User Story | Kriteria | Alasan |
|---|---|---|
| US-3.2, US-3.3 | AC-4, AC-5 (US-3.2); AC-5 (US-3.3) | Pemetaan kolom manual dan peringatan kolom tidak dikenal belum tersedia. Untuk US-3.3 AC-5, arah kerja sudah ditetapkan keputusan K-US-02 dan menunggu implementasi |
| US-4.5 | AC-2 | Verifikator belum melihat rincian saldo tahun berjalan beserta riwayat dua tahun sebelumnya pada layar keputusan |
| US-4.10 | AC-2 | Kriteria sudah direvisi oleh keputusan K-US-01 sehingga precedence tidak lagi menjadi pertanyaan terbuka; sisa pekerjaan adalah aksi penerapan template rantai ke seluruh anggota unit |
| US-5.4 | AC-2, AC-4 | Alert dan notifikasi kenaikan pangkat masih terbit meskipun flag kinerja bernilai negatif; teks bantuan belum memakai kalimat kriteria |
| US-5.5 | AC-4, AC-5 | Milestone Satyalancana belum disimpan sebagai hasil kalkulasi sehingga masih dihitung ulang saat penjadwalan |
| US-6.2 | AC-2 | Daftar notifikasi membedakan warna menurut jenis, belum menurut status sudah atau belum dibaca |
| US-7.1 | AC-1, AC-3 | Cakupan kejadian belum diverifikasi menyeluruh dan model audit belum menolak perubahan maupun penghapusan |
| US-7.2 | AC-3, AC-4 | Halaman masih memuat seluruh catatan sekaligus tanpa paginasi maupun pencarian sisi server |
| US-8.5 | AC-3 | Kedelapan tabel referensi sudah dapat dikelola setelah CRUD `ref_jabatan` masuk `development`. Sisa pekerjaan ada pada guard penghapusan `ref_status_pegawai` yang belum menghitung pemakaian pada `employee_status_histories`, sehingga status yang hanya dirujuk riwayat masih dapat dihapus permanen dan relasinya dikosongkan diam-diam |
| US-9.2 | AC-1 | Kriteria dipertahankan oleh keputusan K-US-04; sisa pekerjaan adalah menambahkan tombol Export PDF di halaman daftar pegawai |

### Keputusan produk 5 Agustus 2026

Empat kriteria yang sebelumnya ambigu telah diputuskan dan disetujui pengguna pada 5 Agustus 2026. Keputusan ini menjadi acuan sampai ada keputusan yang lebih baru.

| Kode | Kriteria | Keputusan | Alasan utama |
|---|---|---|---|
| K-US-01 | US-4.10 AC-2 | Rantai approval aktif pada runtime tetap tepat satu per pegawai. Konfigurasi dapat disalin ke seluruh anggota satu unit kerja sebagai template. Cakupan unit sebagai lapisan resolusi runtime ditunda ke Fase 2 | Kebutuhan sebenarnya adalah efisiensi konfigurasi, bukan resolusi berlapis. Menjaga satu rantai aktif per pegawai menghapus pertanyaan precedence secara permanen dan mempertahankan keutuhan snapshot per pengajuan |
| K-US-02 | US-3.3 AC-5 | Kriteria dipertahankan dan jalur baris terlewat dihidupkan. NIP yang sudah ada di database menjadi terlewat; NIP ganda dalam satu berkas tetap error; email pegawai yang sudah terdaftar tetap error | Import awal wajar dijalankan berulang atas berkas yang saling menimpa. Memisahkan kondisi wajar dari data bermasalah membuat Admin fokus pada baris yang benar-benar perlu diperbaiki. Pelaporan baris terlewat sudah tersedia |
| K-US-03 | US-1.3 AC-2 | Kriteria direvisi agar menyebut `SIMPEG_SESSION_IDLE_TIMEOUT` sebagai kunci batas idle. Desain dua lapis dipertahankan dengan `SESSION_LIFETIME` sebagai jaring pengaman sesi native | AC-5 pada user story yang sama mewajibkan peristiwa sesi berakhir tercatat di audit log. Kewajiban itu hanya terpenuhi bila penegakan terjadi di lapisan aplikasi sebelum sesi native dihapus, sehingga identitas pengguna masih tersedia saat audit ditulis |
| K-US-04 | US-9.2 AC-1 | Kriteria dipertahankan. Tombol Export PDF ditambahkan di halaman daftar pegawai memakai rute dan gerbang yang sudah ada | US-9.1 AC-1 sudah menempatkan Export Excel di halaman yang sama, sehingga pengguna wajar mencari PDF di sana. Rute PDF sudah menegakkan peran, izin, dan batas baris, sehingga penambahan pemicu tidak melemahkan kontrol |

Pekerjaan lanjutan yang timbul dari keputusan di atas:

| Kriteria | Sisa pekerjaan | Pelaksana |
|---|---|---|
| US-4.10 AC-2 | Aksi penerapan template rantai ke seluruh anggota unit, mengikuti pola backfill yang sudah ada | Jordan |
| US-3.3 AC-5 | Lepas aturan unik NIP pada validasi import agar cabang baris terlewat tercapai, pertahankan pemeriksaan ulang NIP saat penyisipan, lalu perbarui test yang mengunci perilaku lama | Grantly |
| US-1.3 AC-2 | Tambahkan keterangan pada `.env.example` bahwa `SIMPEG_SESSION_IDLE_TIMEOUT` adalah batas idle yang ditegakkan dan diaudit, sedangkan `SESSION_LIFETIME` adalah jaring pengaman yang nilainya harus lebih besar | Jordan |
| US-9.2 AC-1 | Tambahkan tombol Export PDF pada halaman daftar pegawai beserta test gerbang peran dan penerusan filter | Adriel |

---

## Pembaruan Status Acceptance Criteria — 7 Agustus 2026

Penyelarasan tambahan dilakukan terhadap branch `development` @ `9be633d` setelah PR #166 masuk.

### Kriteria yang dinaikkan menjadi selesai

| User Story | Kriteria | Bukti implementasi |
|---|---|---|
| US-8.4 | AC-1 sampai AC-5 | Halaman web hari libur membaca `ref_hari_libur` melalui Action halaman dengan filter tahun, filter tipe, pencarian nama, dan paginasi sisi server; tambah, ubah, dan hapus memakai FormRequest beserta Action yang menulis audit resmi. Diverifikasi `HariLiburWebPageTest` (23 test) beserta regresi PostgreSQL 17, dan smoke test browser Super Admin pada 7 Agustus 2026 |

---

## Panduan Membaca Dokumen Ini

### Prioritas

| Label | Arti |
|-------|------|
| 🔴 **P0 — Must Have** | Wajib ada saat go-live. Tanpa ini sistem tidak bisa dipakai. |
| 🟡 **P1 — Should Have** | Sangat diharapkan saat go-live. Bisa ditunda maks 1 sprint jika terpaksa. |
| 🟢 **P2 — Nice to Have** | Bisa ditunda ke iterasi setelah go-live tanpa mengganggu operasional. |

### Story Points (Estimasi Kompleksitas)

| SP | Estimasi |
|----|----------|
| 1 | Sangat sederhana (< 2 jam) |
| 2 | Sederhana (2-4 jam) |
| 3 | Sedang (4-8 jam / 1 hari) |
| 5 | Kompleks (1-2 hari) |
| 8 | Sangat kompleks (2-3 hari) |
| 13 | Sangat besar, pertimbangkan pecah menjadi beberapa story |

### Format User Story

Setiap story mengikuti format:

> **Sebagai** [role],
> **Saya ingin** [aksi],
> **Sehingga** [manfaat/tujuan].

---

## Ringkasan Epics & Stories

| # | Epic | Stories | Total SP |
|---|------|:-------:|:--------:|
| E1 | Autentikasi & SSO | 5 | 18 |
| E2 | Manajemen Data Pegawai | 10 | 47 |
| E3 | Import Data Excel/CSV | 4 | 19 |
| E4 | Manajemen Cuti | 12 | 56 |
| E5 | Early Warning System (EWS) | 5 | 26 |
| E6 | Notifikasi | 4 | 16 |
| E7 | Audit Log | 3 | 11 |
| E8 | Dashboard | 5 | 26 |
| E9 | Laporan & Export | 5 | 16 |
| | **Total** | **53** | **235** |

---

## E1 — Autentikasi & SSO

### US-1.1 · Login via Keycloak SSO

| Field | Detail |
|-------|--------|
| **ID** | US-1.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Autentikasi |
| **Dependensi** | Trait/fungsi Keycloak, Client ID, Client Secret, URL Keycloak, dan akun SSO testing dari LLDIKTI |

> **Sebagai** pegawai LLDIKTI XVI,
> **Saya ingin** login ke SIMPEG menggunakan akun Keycloak saya,
> **Sehingga** saya tidak perlu mengingat username dan password terpisah untuk sistem ini.

**Acceptance Criteria:**

- [x] AC-1: Saat mengakses URL SIMPEG tanpa session aktif, browser otomatis redirect ke halaman login Keycloak.
- [x] AC-2: Setelah login berhasil di Keycloak, browser redirect kembali ke SIMPEG dan session aktif terbentuk.
- [x] AC-3: Data email user dari Keycloak disimpan / di-cache ke tabel `employees` (kolom `keycloak_id` dan email) saat login pertama kali.
- [x] AC-4: Jika email Keycloak belum ter-mapping ke data pegawai manapun di SIMPEG, tampilkan halaman informasi: *"Akun Anda belum terdaftar di SIMPEG. Silakan hubungi Admin Kepegawaian."*
- [x] AC-5: Jika ini adalah user pertama yang berhasil login melalui SSO dan belum ada user lokal SIMPEG, sistem otomatis membuat user tersebut sebagai `Super Admin` untuk kebutuhan bootstrap awal.
- [x] AC-6: Jika mapping ditemukan tetapi role SIMPEG user masih kosong / belum diset / tidak valid, session login tetap terbentuk tetapi akses dashboard/fitur normal ditolak dengan HTTP `403 Access Forbidden` dan pesan: *"Akun Anda belum memiliki role SIMPEG. Hubungi Admin."*
- [x] AC-7: Jika mapping ditemukan dan role SIMPEG valid, user diarahkan ke halaman dashboard sesuai role-nya.
- [x] AC-8: Login yang berhasil, gagal mapping, dan penolakan karena role belum diset dicatat di audit log jika mekanisme audit sudah tersedia pada flow tersebut.

---

### US-1.2 · Logout

| Field | Detail |
|-------|--------|
| **ID** | US-1.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 2 |
| **Modul** | Autentikasi |
| **Dependensi** | US-1.1 |

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** bisa logout dari sistem dengan aman,
> **Sehingga** tidak ada orang lain yang bisa mengakses akun saya setelah saya selesai.

**Acceptance Criteria:**

- [x] AC-1: Tombol "Keluar" tersedia di navbar/header pada semua halaman.
- [x] AC-2: Klik "Keluar" menghapus session Laravel lokal.
- [x] AC-3: Logout juga memicu single logout di Keycloak (end session endpoint).
- [x] AC-4: Setelah logout, user diarahkan kembali ke halaman login Keycloak.
- [x] AC-5: Event logout dicatat di audit log.

---

### US-1.3 · Session Timeout Otomatis

| Field | Detail |
|-------|--------|
| **ID** | US-1.3 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Autentikasi |
| **Dependensi** | US-1.1 |

> **Sebagai** pengelola keamanan sistem,
> **Saya ingin** session user otomatis berakhir setelah 30 menit tidak ada aktivitas,
> **Sehingga** keamanan data terjaga jika user lupa logout.

**Acceptance Criteria:**

- [x] AC-1: Session Laravel expired setelah 30 menit tanpa aktivitas (request ke server).
- [x] AC-2: Durasi idle timeout bisa dikonfigurasi melalui file `.env` memakai kunci `SIMPEG_SESSION_IDLE_TIMEOUT` dengan nilai bawaan 30 menit. `SESSION_LIFETIME` tetap dipertahankan sebagai jaring pengaman sesi native dan nilainya harus lebih besar dari batas idle. *(Direvisi oleh keputusan K-US-03, 5 Agustus 2026.)*
- [x] AC-3: Saat session expired dan user mengakses halaman, redirect ke Keycloak untuk login ulang.
- [x] AC-4: Tampilkan flash message: *"Sesi Anda telah berakhir. Silakan login kembali."*
- [x] AC-5: Event session timeout dicatat di audit log.

---

### US-1.4 · Mapping User Keycloak ke Pegawai

| Field | Detail |
|-------|--------|
| **ID** | US-1.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Autentikasi / Admin |
| **Dependensi** | US-1.1, US-2.1 |

> **Sebagai** Super Admin,
> **Saya ingin** memetakan akun Keycloak ke data pegawai di SIMPEG dan menetapkan role internal aplikasi,
> **Sehingga** setiap user yang login mendapatkan hak akses yang sesuai.

**Acceptance Criteria:**

- [x] AC-1: Halaman "Kelola Akses User" menampilkan daftar pegawai beserta status mapping (Terhubung / Belum Terhubung).
- [x] AC-2: Super Admin bisa mengisi Keycloak ID atau email Keycloak untuk setiap pegawai.
- [x] AC-3: Super Admin bisa menetapkan satu role internal per pegawai: Super Admin, Admin Kepegawaian, Pimpinan, Kepala Bagian, atau Pegawai.
- [x] AC-4: Perubahan role langsung berlaku pada login berikutnya.
- [x] AC-5: Perubahan mapping dan role dicatat di audit log.
- [x] AC-6: Validasi: satu akun Keycloak hanya bisa di-mapping ke satu pegawai.
- [x] AC-7: Role dan permission aplikasi dibaca dari database SIMPEG, bukan dari data otorisasi Keycloak.
- [x] AC-8: User yang sudah berhasil login SSO tetapi belum memiliki role internal SIMPEG tetap tercatat sebagai user lokal dengan role kosong sampai Super Admin menetapkan role.
- [x] AC-9: Role dasar dari SSO tidak boleh otomatis memberi akses fitur; role tersebut hanya boleh diperlakukan sebagai informasi identitas eksternal, bukan sumber otorisasi SIMPEG.

---

### US-1.5 · Redirect Berdasarkan Role Setelah Login

| Field | Detail |
|-------|--------|
| **ID** | US-1.5 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Autentikasi |
| **Dependensi** | US-1.1, US-1.4 |

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** langsung diarahkan ke halaman yang relevan setelah login,
> **Sehingga** saya bisa langsung bekerja tanpa navigasi manual.

**Acceptance Criteria:**

- [x] AC-1: Super Admin dan Admin Kepegawaian → Dashboard Admin.
- [x] AC-2: Pimpinan → Dashboard Pimpinan.
- [x] AC-3: Kepala Bagian → Dashboard Kepala Bagian (daftar bawahan + pengajuan pending).
- [x] AC-4: Pegawai → Dashboard Pribadi (profil ringkas + saldo cuti).

---

## E2 — Manajemen Data Pegawai

### US-2.1 · Tambah Data Pegawai Baru

| Field | Detail |
|-------|--------|
| **ID** | US-2.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 8 |
| **Modul** | Data Pegawai |
| **Dependensi** | E1 selesai |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menambahkan data pegawai baru ke dalam sistem,
> **Sehingga** seluruh data kepegawaian tersimpan secara terpusat dan terstruktur.

**Acceptance Criteria:**

- [x] AC-1: Form input multi-step atau tabbed:
  - Tab 1: Data Utama (nama, email pegawai, NIP, status pegawai dari `ref_status_pegawai`, keterangan status, tanggal lahir, golongan/pangkat terkini dari riwayat, jabatan dari `ref_jabatan`, kelas jabatan dari riwayat, pendidikan terakhir, prodi pendidikan terakhir, tanggal pensiun)
  - Tab 2: Data Pelengkap (NIK, No. KK, tempat lahir, jenis kelamin, agama, status kawin, golongan darah, foto)
  - Tab 3: Data Kontak (alamat, no HP, no telepon rumah)
  - Tab 4: Data Pengangkatan (jenis pengangkatan, TMT, no SK, tanggal SK, upload file SK)
- [x] AC-2: Validasi NIP unik — tidak boleh duplikat dengan pegawai lain.
- [x] AC-3: Validasi NIK — format 16 digit.
- [x] AC-3a: Validasi No. KK — format 16 digit (opsional, boleh kosong).
- [x] AC-4: Upload foto: maks 10MB, format JPG/PNG, preview sebelum simpan.
- [x] AC-5: Semua field bertanda wajib harus terisi sebelum bisa disimpan.
- [x] AC-6: Setelah simpan, pegawai memakai status default `Aktif` dari `ref_status_pegawai` dan muncul di daftar pegawai.
- [x] AC-7: Audit log mencatat: user yang menambahkan, timestamp, dan seluruh data yang dimasukkan.
- [x] AC-8: Tampilkan notifikasi sukses: *"Data pegawai [Nama] berhasil ditambahkan."*

---

### US-2.2 · Edit Data Pegawai

| Field | Detail |
|-------|--------|
| **ID** | US-2.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengubah data pegawai yang sudah tersimpan,
> **Sehingga** data selalu akurat dan mencerminkan kondisi terbaru.

**Acceptance Criteria:**

- [x] AC-1: Semua field data pribadi dan kontak bisa diedit.
- [x] AC-2: Validasi tetap berlaku (NIP unik, NIK 16 digit, dll).
- [x] AC-3: Audit log mencatat nilai sebelum (`old_values`) dan sesudah (`new_values`) perubahan.
- [x] AC-4: Timestamp `updated_at` otomatis diperbarui.
- [x] AC-5: Foto bisa diganti dengan upload baru (foto lama di-replace).
- [x] AC-6: Tampilkan notifikasi sukses setelah simpan.

---

### US-2.3 · Daftar Pegawai (Admin View)

| Field | Detail |
|-------|--------|
| **ID** | US-2.3 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat daftar seluruh pegawai dalam bentuk tabel,
> **Sehingga** saya bisa mencari, memfilter, dan mengelola data dengan cepat.

**Acceptance Criteria:**

- [x] AC-1: Tabel daftar pegawai menampilkan kolom: Foto (thumbnail), Nama, NIP, Golongan, Jabatan, Unit Kerja, Jenis Pegawai, Status.
- [x] AC-2: Search bar — bisa mencari berdasarkan nama atau NIP.
- [x] AC-3: Filter dropdown: golongan, unit/tim kerja hierarkis, jenis pegawai (PNS/PPPK), dan status dari `ref_status_pegawai`.
- [x] AC-4: Sorting: klik header kolom untuk sort ascending/descending.
- [x] AC-5: Pagination: 10 / 25 / 50 baris per halaman (user bisa memilih).
- [x] AC-6: Klik nama pegawai membuka halaman detail pegawai.
- [x] AC-7: Tombol "Tambah Pegawai" mengarah ke form tambah (US-2.1).
- [x] AC-8: Daftar default hanya menampilkan pegawai aktif (yang belum di-soft-delete).

---

### US-2.4 · Detail Pegawai (Admin View)

| Field | Detail |
|-------|--------|
| **ID** | US-2.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat detail lengkap seorang pegawai dalam satu halaman,
> **Sehingga** saya bisa memeriksa dan memverifikasi data dengan mudah.

**Acceptance Criteria:**

- [x] AC-1: Halaman detail menampilkan semua data dalam layout tabbed atau accordion:
  - Profil & Kontak
  - Data Keluarga
  - Riwayat Kepangkatan (tabel, urut terbaru di atas)
  - Riwayat Jabatan
  - Riwayat KGB
  - Hukuman Disiplin
  - Riwayat Pendidikan
  - Dokumen & SK
  - Data Pengangkatan
- [x] AC-2: Di setiap tab riwayat ada tombol "Tambah Riwayat" untuk menambah record baru (append-only).
- [x] AC-3: Tombol "Edit" untuk mengedit data pribadi dan kontak.
- [x] AC-4: Menampilkan informasi kalkulasi otomatis: tanggal kenaikan pangkat berikutnya, tanggal KGB berikutnya, tanggal pensiun.
- [x] AC-5: Menampilkan flag "Kinerja Baik" (toggle, bisa diubah admin — US-5.4).
- [x] AC-6: Menampilkan kepala bagian yang di-assign.

---

### US-2.5 · Lihat Profil Sendiri (Pegawai)

| Field | Detail |
|-------|--------|
| **ID** | US-2.5 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.1, US-1.1 |

> **Sebagai** pegawai,
> **Saya ingin** melihat data kepegawaian saya sendiri,
> **Sehingga** saya bisa memastikan data saya lengkap dan benar.

**Acceptance Criteria:**

- [x] AC-1: Halaman "Profil Saya" menampilkan semua data milik pegawai yang login (layout sama dengan US-2.4).
- [x] AC-2: Semua data bersifat **read-only** (tidak ada tombol edit/tambah).
- [x] AC-3: Pegawai tidak bisa mengakses data pegawai lain.
- [x] AC-4: Menampilkan saldo cuti tahun berjalan.
- [x] AC-5: Menampilkan tanggal kenaikan pangkat & KGB berikutnya (hasil kalkulasi otomatis).

---

### US-2.6 · Tambah Riwayat Kepangkatan / Jabatan / KGB

| Field | Detail |
|-------|--------|
| **ID** | US-2.6 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Data Pegawai — Riwayat |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menambahkan riwayat kepangkatan, jabatan, atau KGB baru untuk pegawai,
> **Sehingga** data riwayat selalu lengkap dan TMT terbaru bisa digunakan untuk kalkulasi EWS.

**Acceptance Criteria:**

- [x] AC-1: Form input riwayat sesuai jenis:
  - Kepangkatan: Golongan (dropdown ref_golongan), TMT Pangkat, No SK, Tanggal SK, Upload SK.
  - Jabatan: Nama Jabatan, Jenis Jabatan (dropdown), Unit Kerja (dropdown), TMT Jabatan, No SK, Tanggal SK, Upload SK.
  - KGB: TMT KGB, Gaji Pokok (angka), No SK, Tanggal SK, Upload SK.
- [x] AC-2: Data bersifat **append-only** — record riwayat yang sudah ada tidak bisa diedit atau dihapus.
- [x] AC-3: Saat record baru ditambahkan, field `is_latest` pada record sebelumnya otomatis diubah menjadi `false`, dan record baru menjadi `true`.
- [x] AC-4: Kalkulasi TMT otomatis di-update setelah riwayat baru ditambahkan:
  - Pangkat baru → hitung ulang `tanggal_kenaikan_pangkat_berikutnya` (TMT + 4 tahun).
  - KGB baru → hitung ulang `tanggal_kgb_berikutnya` (TMT + 2 tahun).
- [x] AC-5: Audit log mencatat penambahan riwayat.

---

### US-2.7 · Tambah Riwayat Hukuman Disiplin

| Field | Detail |
|-------|--------|
| **ID** | US-2.7 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Data Pegawai — Disiplin |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mencatat hukuman disiplin pegawai,
> **Sehingga** data disiplin bisa digunakan untuk menentukan eligibility kenaikan pangkat di EWS.

**Acceptance Criteria:**

- [x] AC-1: Form input: jenis hukuman (Ringan/Sedang/Berat), deskripsi, tanggal mulai, tanggal berakhir (opsional — null berarti masih aktif), No SK, Tanggal SK, Upload SK.
- [x] AC-2: Data bersifat append-only.
- [x] AC-3: Field `is_active` otomatis `true` jika `tanggal_berakhir` null atau belum terlewati.
- [x] AC-4: Scheduler harian otomatis mengubah `is_active` menjadi `false` jika `tanggal_berakhir` sudah terlewati.
- [x] AC-5: Pegawai dengan hukuman disiplin aktif (`is_active = true`) **tidak** eligible untuk kenaikan pangkat di EWS.
- [x] AC-6: Audit log mencatat penambahan.

---

### US-2.8 · Kelola Data Keluarga

| Field | Detail |
|-------|--------|
| **ID** | US-2.8 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Data Pegawai — Keluarga |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menambah dan mengelola data keluarga pegawai (pasangan dan anak),
> **Sehingga** data tunjangan keluarga tercatat dengan lengkap.

**Acceptance Criteria:**

- [x] AC-1: Di halaman detail pegawai, tab "Keluarga" menampilkan daftar anggota keluarga.
- [x] AC-2: Tombol "Tambah Anggota Keluarga" dengan form: nama, hubungan (Suami/Istri/Anak), NIK, tempat lahir, tanggal lahir, jenis kelamin, status tunjangan (Ya/Tidak), pekerjaan.
- [x] AC-3: Admin bisa mengedit dan soft-delete data anggota keluarga.
- [x] AC-4: Audit log mencatat semua perubahan.

---

### US-2.9 · Soft Delete Pegawai

| Field | Detail |
|-------|--------|
| **ID** | US-2.9 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menonaktifkan pegawai yang sudah pensiun atau mutasi tanpa menghapus datanya,
> **Sehingga** data historis tetap tersimpan untuk arsip dan pelaporan.

**Acceptance Criteria:**

- [x] AC-1: Tombol "Nonaktifkan" di halaman detail pegawai.
- [x] AC-2: Konfirmasi dialog: *"Apakah Anda yakin ingin menonaktifkan pegawai [Nama]? Data tidak dihapus dan bisa diaktifkan kembali."*
- [x] AC-3: Pegawai yang dinonaktifkan tidak muncul di daftar pegawai aktif (default view).
- [x] AC-4: Filter "Tampilkan Pegawai Non-Aktif" di halaman daftar pegawai.
- [x] AC-5: Admin bisa melakukan "Aktifkan Kembali" (restore) dari daftar non-aktif.
- [x] AC-6: Pegawai yang dinonaktifkan tidak lagi diproses oleh EWS.
- [x] AC-7: Audit log mencatat soft delete dan restore.

---

### US-2.10 · Soft Delete Pegawai oleh Super Admin

| Field | Detail |
|-------|--------|
| **ID** | US-2.10 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 2 |
| **Modul** | Data Pegawai |
| **Dependensi** | US-2.9 |

> **Sebagai** Super Admin,
> **Saya ingin** menonaktifkan data pegawai tanpa menghapus permanen,
> **Sehingga** data yang sewaktu-waktu dibutuhkan masih bisa ditemukan dan dipulihkan.

**Acceptance Criteria:**

- [x] AC-1: Tidak ada tombol "Hapus Permanen" di aplikasi untuk role apa pun, termasuk Super Admin.
- [x] AC-2: Super Admin hanya bisa melakukan soft delete/nonaktifkan pegawai dengan konfirmasi.
- [x] AC-3: Data yang dinonaktifkan tetap tersimpan di database dan bisa ditemukan melalui filter pegawai non-aktif.
- [x] AC-4: Super Admin bisa melakukan restore jika data perlu dipakai kembali.
- [x] AC-5: Audit log mencatat soft delete dan restore.

---

## E3 — Import Data Excel/CSV

### US-3.1 · Download Template Import

| Field | Detail |
|-------|--------|
| **ID** | US-3.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 2 |
| **Modul** | Import Excel/CSV |
| **Dependensi** | — |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengunduh template import yang sudah sesuai format SIMPEG,
> **Sehingga** saya bisa menyiapkan data pegawai dengan format yang benar sebelum diimpor.

**Acceptance Criteria:**

- [x] AC-1: Tombol "Download Template Import" di halaman Import.
- [x] AC-2: Minimal tersedia template CSV berformat UTF-8; jika disediakan Excel, gunakan `.xlsx` dengan header yang sama.
- [x] AC-3: Header template utama mengikuti file `daftar_pegawai.xlsx`: `No`, `Nama Pegawai`, `Email Pegawai`, `Golongan`, `Jabatan`, `Kelas Jabatan`, `NIP`, `Nomor Telepon`, `Pangkat`, `Pendidikan Terakhir`, `Pensiun`, `Person`, `Person Formula`, `Prodi Pendidikan Terakhir`, `Status Kepegawaian`, `Tanggal Lahir`.
- [x] AC-4: Sertakan 2 baris contoh data (dummy) sebagai panduan pengisian.
- [x] AC-5: Hanya template Data Utama yang aktif di Fase 1 (keputusan pengguna 22 Juli 2026). Template lanjutan multi-jenis (Data Pelengkap, Riwayat Kepangkatan, Riwayat Jabatan, Riwayat KGB) tidak termasuk ruang lingkup saat ini dan tidak dipulihkan tanpa keputusan eksplisit baru.

---

### US-3.2 · Upload & Preview Excel/CSV

| Field | Detail |
|-------|--------|
| **ID** | US-3.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Import Excel/CSV |
| **Dependensi** | US-3.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengupload file Excel/CSV dan melihat preview datanya sebelum diimpor,
> **Sehingga** saya bisa memverifikasi bahwa data sudah benar sebelum masuk ke database.

**Acceptance Criteria:**

- [x] AC-1: Upload file Excel/CSV (maks 10MB).
- [x] AC-2: Sistem mendeteksi header kolom secara otomatis.
- [x] AC-3: Tampilkan preview 10 baris pertama dalam bentuk tabel.
- [ ] AC-4: Tampilkan mapping kolom: setiap kolom Excel/CSV dipasangkan ke field SIMPEG atau opsi "Tidak dipakai". Sistem mengisi mapping awal melalui nama header template `daftar_pegawai.xlsx` setelah normalisasi spasi dan huruf besar-kecil; Admin dapat mengubahnya melalui dropdown sebelum validasi. Mapping yang dipilih disimpan pada batch import yang sama dan dipakai kembali oleh preview, validasi, dan eksekusi. Satu field tujuan tidak boleh menerima dua kolom sumber; field wajib yang belum memiliki mapping harus ditolak dengan pesan yang menyebut field tersebut.
- [ ] AC-5: Jika ada kolom yang tidak cocok, tampilkan peringatan yang menyebut nama kolom dan menjelaskan bahwa nilainya tidak disimpan bila tetap dipetakan ke "Tidak dipakai". Peringatan tidak memblokir import selama seluruh field wajib sudah dipetakan. Sistem juga memperingatkan header wajib yang belum ditemukan. Kolom `Role` tidak dipakai oleh import dan harus diperingatkan sebagai kolom ekstra; penetapan role aplikasi tetap dilakukan melalui Kelola Akses User (US-1.4).
- [x] AC-6: Tombol "Lanjutkan ke Validasi" dan "Batal".

**Catatan pengerjaan AC-4 dan AC-5 (belum diimplementasikan):**

1. Tetapkan daftar header canonical berdasarkan US-3.1 AC-3. `No` hanya kolom tampilan; `NIK` dan `No KK` boleh tidak ada; `Tanggal Lahir` wajib untuk setiap baris import karena diperlukan oleh kalkulasi BUP/pensiun. Jangan menambah `Role` ke template, mapping, atau validasi import.
2. Ubah langkah unggah agar berkas dengan header yang belum dikenali tetap membentuk batch import dan masuk ke layar pemetaan, bukan langsung ditolak. Batch menyimpan header sumber, mapping otomatis, mapping pilihan Admin, peringatan kolom, dan batas waktu 30 menit.
3. Tambahkan endpoint pemetaan yang memakai FormRequest untuk otorisasi serta validasi mapping; controller hanya meneruskan request ke Action. Endpoint preview membatasi respons di server menjadi maksimal 10 baris dan memakai mapping aktif.
4. Pemeta baris membaca pilihan mapping manual sebagai sumber utama. Nilai kosong pada field wajib membuat hanya baris tersebut gagal; baris valid lain tetap dapat diproses. Heuristik kolom bergeser hanya boleh dipakai sebelum Admin menyimpan mapping manual dan hasilnya harus terlihat sebagai mapping awal yang dapat diubah.
5. Selaraskan validasi NIP dengan US-3.3 AC-5/K-US-02: NIP yang sudah ada di database menjadi baris terlewat, NIP ganda dalam satu berkas tetap gagal, dan keberadaan NIP diperiksa lagi saat penyisipan untuk mencegah kondisi balapan.
6. Pertahankan alur `Route → Middleware/RBAC → FormRequest → Controller → Action → Service/Model → Payload → Response`, audit `IMPORT`, pembatasan akses backend, masking data sensitif, serta test PostgreSQL 17 untuk mapping, peringatan, validasi, baris terlewat, dan batas preview.

---

### US-3.3 · Validasi Data Excel/CSV

| Field | Detail |
|-------|--------|
| **ID** | US-3.3 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Import Excel/CSV |
| **Dependensi** | US-3.2 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** sistem memvalidasi semua baris data Excel/CSV sebelum diimpor,
> **Sehingga** hanya data yang valid yang masuk ke database.

**Acceptance Criteria:**

- [x] AC-1: Validasi semua baris: NIP unik, email pegawai terisi, tanggal lahir valid, status kepegawaian valid (PNS/CPNS/PPPK), field wajib Excel terisi, golongan ada di reference table jika ref sudah tersedia.
- [x] AC-2: Tampilkan ringkasan validasi: jumlah baris total, baris valid (✅), baris error (❌).
- [x] AC-3: Untuk baris error, tampilkan detail: nomor baris, kolom yang bermasalah, jenis error.
- [x] AC-4: Admin bisa memilih: "Import Hanya yang Valid" atau "Batalkan Semua".
- [ ] AC-5: Baris yang sudah ada (NIP duplikat) ditandai sebagai "Sudah ada — akan di-skip" (bukan error). *(Kriteria dipertahankan dan jalur skip dihidupkan sesuai keputusan K-US-02, 5 Agustus 2026. Batasnya: NIP yang sudah ada di database menjadi baris terlewat; NIP ganda di dalam satu berkas tetap error; email pegawai yang sudah terdaftar tetap error.)*

---

### US-3.4 · Eksekusi Import & Laporan Hasil

| Field | Detail |
|-------|--------|
| **ID** | US-3.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 7 |
| **Modul** | Import Excel/CSV |
| **Dependensi** | US-3.3 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menjalankan import dan mendapatkan laporan hasilnya,
> **Sehingga** saya tahu berapa data yang berhasil diimpor dan jika ada yang gagal.

**Acceptance Criteria:**

- [x] AC-1: Proses import berjalan di background (queue) agar tidak timeout untuk file besar.
- [x] AC-2: Tampilkan progress bar atau loading indicator.
- [x] AC-3: Setelah selesai, tampilkan laporan: jumlah berhasil, jumlah gagal, jumlah di-skip.
- [x] AC-4: Laporan bisa di-download sebagai laporan CSV/Excel (berisi baris yang gagal + alasan gagal).
- [x] AC-5: Semua record yang berhasil diimpor langsung berstatus aktif dan muncul di daftar pegawai.
- [x] AC-6: Audit log mencatat: user, timestamp, nama file, jumlah record berhasil/gagal.
- [x] AC-7: Import hanya mempersistensikan record pegawai beserta field snapshot awal (golongan, pangkat, jabatan, kelas jabatan, pendidikan, prodi, dan tanggal pensiun bila tersedia). Import tidak membuat riwayat kepangkatan, riwayat jabatan, maupun riwayat KGB.
- [x] AC-8: Tanggal pensiun hasil import dipertahankan apa adanya; import tidak menghitung ulang atau menimpa tanggal pensiun.
- [x] AC-9: Import tidak memanggil kalkulasi TMT. Kalkulasi TMT hanya dipicu saat riwayat/sumber resmi disimpan per pegawai (lihat US-5.5).

---

## E4 — Manajemen Cuti

### US-4.1 · Ajukan Cuti

| Field | Detail |
|-------|--------|
| **ID** | US-4.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Cuti |
| **Dependensi** | US-2.5, US-2.6 (set supervisor) |

> **Sebagai** pegawai,
> **Saya ingin** mengajukan cuti secara digital melalui SIMPEG,
> **Sehingga** saya tidak perlu mengurus berkas fisik dan bisa memantau statusnya secara online.

**Acceptance Criteria:**

- [x] AC-1: Form pengajuan cuti dengan field:
  - Jenis cuti (dropdown — hanya tampilkan jenis yang sesuai status PNS/PPPK).
  - Tanggal mulai (date picker).
  - Tanggal selesai (date picker).
  - Alasan (textarea, wajib diisi).
  - Upload lampiran (opsional, maks 10MB, PDF/JPG/PNG).
- [x] AC-2: Sistem otomatis menghitung jumlah hari kerja (exclude Sabtu, Minggu, hari libur nasional, cuti bersama).
- [x] AC-3: Tampilkan jumlah hari kerja secara real-time saat tanggal dipilih.
- [x] AC-4: Validasi tanggal: satu pengajuan tidak boleh melewati tahun kalender; periode Desember–Januari harus dibuat sebagai dua pengajuan.
- [x] AC-5: Validasi saldo: jika jenis cuti = Cuti Tahunan dan saldo tidak cukup → tampilkan pesan error, form tidak bisa di-submit.
- [x] AC-6: Setelah submit:
  - Status = "Menunggu [step pertama approval chain]".
  - Notifikasi in-app + email terkirim ke pihak pertama pada chain.
- [x] AC-7: Pegawai tidak bisa mengajukan cuti jika belum memiliki approval chain aktif atau pihak pertama tidak valid.

---

### US-4.2 · Daftar Pengajuan Cuti (Pegawai)

| Field | Detail |
|-------|--------|
| **ID** | US-4.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Cuti |
| **Dependensi** | US-4.1 |

> **Sebagai** pegawai,
> **Saya ingin** melihat daftar semua pengajuan cuti saya beserta statusnya,
> **Sehingga** saya tahu posisi approval setiap pengajuan.

**Acceptance Criteria:**

- [x] AC-1: Tabel daftar pengajuan: Jenis Cuti, Tanggal Mulai, Tanggal Selesai, Jumlah Hari, Status, Tanggal Pengajuan.
- [x] AC-2: Status ditampilkan dengan warna badge:
  - Kuning: Menunggu step approval/verifikasi
  - Hijau: Disetujui
  - Biru: Perubahan
  - Oranye: Ditangguhkan
  - Merah: Tidak Disetujui
- [x] AC-3: Klik baris membuka detail pengajuan + timeline approval.
- [x] AC-4: Filter: tahun, status.
- [x] AC-5: Pagination.

---

### US-4.3 · Lihat Saldo Cuti

| Field | Detail |
|-------|--------|
| **ID** | US-4.3 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Cuti |
| **Dependensi** | US-4.1 |

> **Sebagai** pegawai,
> **Saya ingin** melihat saldo cuti tahunan saya,
> **Sehingga** saya tahu berapa hari cuti yang masih bisa saya gunakan.

**Acceptance Criteria:**

- [x] AC-1: Tampilkan informasi saldo cuti tahunan:
  - Jatah tahun ini: 12 hari
  - Carry-over N-1: maks 6 hari
  - Hak tambahan N-2/N-1 jika dua tahun berturut-turut tidak mengambil cuti
  - Total tersedia: Y hari
  - Sudah terpakai: Z hari
  - Sisa: (Y - Z) hari
- [x] AC-2: Tampilkan riwayat penggunaan cuti tahun berjalan, N-1, dan N-2 yang memengaruhi carry-over.
- [x] AC-3: Data saldo diperbarui secara real-time setelah cuti disetujui.

---

### US-4.4 · Approval/Verifikasi Step 1 — Kepala Bagian atau Pihak Pertama

| Field | Detail |
|-------|--------|
| **ID** | US-4.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Cuti — Approval |
| **Dependensi** | US-4.1 |

> **Sebagai** Kepala Bagian,
> **Saya ingin** melihat dan menindaklanjuti pengajuan cuti bawahan saya,
> **Sehingga** proses approval bisa berjalan tepat waktu.

**Acceptance Criteria:**

- [x] AC-1: Halaman "Pengajuan Cuti Bawahan" menampilkan daftar pengajuan yang menunggu tindakan saya.
- [x] AC-2: Detail pengajuan: nama pegawai, jenis cuti, tanggal mulai–selesai, jumlah hari, alasan, lampiran.
- [x] AC-3: Opsi aksi memakai label resmi: **"Disetujui"**, **"Perubahan"**, **"Ditangguhkan"**, dan **"Tidak Disetujui"**.
- [x] AC-4: Klik "Disetujui" → konfirmasi → status berubah ke step berikutnya yang dikonfigurasi → notifikasi terkirim ke approver/verifikator berikutnya.
- [x] AC-5: Klik "Perubahan", "Ditangguhkan", atau "Tidak Disetujui" → muncul textarea keterangan wajib → status dan keterangan tersimpan → notifikasi ke pegawai.
- [x] AC-6: Tidak ada tombol formal "Tolak"; keputusan negatif memakai label **"Tidak Disetujui"**.
- [x] AC-7: Audit log mencatat aksi approval/penundaan.
- [x] AC-8: Kepala Bagian hanya melihat pengajuan dari pegawai yang di-assign kepadanya (bukan semua pegawai).
- [x] AC-9: Jika kepala bagian juga menjadi approver pada step berikutnya, sistem otomatis skip step duplikat agar orang yang sama tidak menyetujui dua kali.
- [x] AC-10: Pengajuan berstatus "Perubahan" atau "Ditangguhkan" tetap terlihat di timeline sampai ada tindak lanjut sesuai keputusan.
- [x] AC-11: Perubahan rekomendasi/keputusan dicatat di audit log beserta timestamp dan komentar.
- [x] AC-12: Notifikasi terkirim ke step berikutnya atau pegawai sesuai status terbaru.

---

### US-4.5 · Verifikasi Kepegawaian / Step Lanjutan

| Field | Detail |
|-------|--------|
| **ID** | US-4.5 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Cuti — Approval |
| **Dependensi** | US-4.4 |

> **Sebagai** verifikator/Kepegawaian yang dikonfigurasi,
> **Saya ingin** menyetujui atau menunda pengajuan cuti yang sudah diketahui kepala bagian,
> **Sehingga** proses approval berlanjut ke tahap akhir.

**Acceptance Criteria:**

- [x] AC-1: Sama dengan US-4.4, tetapi hanya menampilkan pengajuan yang sudah sampai pada step saya.
- [ ] AC-2: Verifikator dapat melihat saldo tahun berjalan, carry-over N-1, riwayat N-2/N-1, dan lampiran.
- [x] AC-3: Opsi aksi: "Disetujui", "Perubahan", "Ditangguhkan", atau "Tidak Disetujui"; semua selain "Disetujui" wajib keterangan.
- [x] AC-4: Notifikasi terkirim sesuai aksi.
- [x] AC-5: Audit log tercatat.
- [x] AC-6: Chain mendukung lebih dari satu verifikator dan skip otomatis jika pegawai yang sama muncul berurutan.

---

### US-4.6 · Keputusan Final Pimpinan/PYBMC

| Field | Detail |
|-------|--------|
| **ID** | US-4.6 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Cuti — Approval |
| **Dependensi** | US-4.5 |

> **Sebagai** Pimpinan/PYBMC,
> **Saya ingin** memberikan persetujuan akhir pengajuan cuti,
> **Sehingga** cuti resmi berlaku dan saldo diperbarui otomatis.

**Acceptance Criteria:**

- [x] AC-1: Menampilkan pengajuan yang sudah melewati step sebelumnya, beserta catatan/komentar dari approver/verifikator sebelumnya.
- [x] AC-2: Opsi keputusan final: "Disetujui", "Perubahan", "Ditangguhkan", atau "Tidak Disetujui".
- [x] AC-3: Jika "Disetujui":
  - Status menjadi **"Disetujui"**.
  - Saldo cuti tahunan **dikurangi otomatis** sebesar jumlah hari kerja (hanya untuk cuti tahunan).
  - Notifikasi "Cuti Anda Disetujui" terkirim ke pegawai (in-app + email).
  - Sistem menghasilkan formulir cuti resmi dengan QR Code verifikasi.
- [x] AC-4: Jika "Perubahan", "Ditangguhkan", atau "Tidak Disetujui":
  - Keterangan wajib diisi.
  - Saldo cuti **tidak** dikurangi.
  - Notifikasi status + keterangan terkirim ke pegawai.
- [x] AC-5: Audit log mencatat aksi beserta komentar.
- [x] AC-6: Stage final mengikuti konfigurasi approval chain; default awal adalah Pimpinan / PYBMC.
- [x] AC-7: Untuk cuti Kepala Lembaga sendiri, Admin Kepegawaian dapat mencatat approval eksternal dan upload dokumen yang sudah disetujui.

---

### US-4.7 · Detail & Timeline Approval Cuti

| Field | Detail |
|-------|--------|
| **ID** | US-4.7 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Cuti |
| **Dependensi** | US-4.4, US-4.5, US-4.6 |

> **Sebagai** pegawai atau approver,
> **Saya ingin** melihat timeline detail proses approval cuti,
> **Sehingga** saya tahu di tahap mana pengajuan berada dan siapa yang sudah/belum bertindak.

**Acceptance Criteria:**

- [x] AC-1: Halaman detail pengajuan cuti menampilkan info lengkap (jenis, tanggal, alasan, lampiran).
- [x] AC-2: Timeline visual (vertikal) menampilkan setiap stage:
  - Step, nama approver/verifikator, aksi (`Disetujui`/`Perubahan`/`Ditangguhkan`/`Tidak Disetujui`), waktu aksi, keterangan.
  - Stage yang belum diproses ditampilkan sebagai "Menunggu".
- [x] AC-3: Akses: pegawai yang mengajukan + semua approver di chain + Admin.

---

### US-4.8 · Daftar Pengajuan Cuti (Admin View)

| Field | Detail |
|-------|--------|
| **ID** | US-4.8 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Cuti |
| **Dependensi** | US-4.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat semua pengajuan cuti dari seluruh pegawai,
> **Sehingga** saya bisa memonitor proses cuti secara keseluruhan.

**Acceptance Criteria:**

- [x] AC-1: Tabel semua pengajuan cuti dari seluruh pegawai (tidak terbatas pada bawahan).
- [x] AC-2: Filter: status, jenis cuti, unit kerja, periode.
- [x] AC-3: Search berdasarkan nama/NIP pegawai.
- [x] AC-4: Admin bisa melihat detail + timeline approval setiap pengajuan.
- [x] AC-5: Admin **tidak bisa** melakukan approval (hanya monitor).

---

### US-4.9 · Kelola Saldo Cuti (Admin)

| Field | Detail |
|-------|--------|
| **ID** | US-4.9 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 5 |
| **Modul** | Cuti |
| **Dependensi** | US-4.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat dan mengelola saldo cuti semua pegawai,
> **Sehingga** saya bisa melakukan koreksi jika ada kesalahan atau kasus khusus.

**Acceptance Criteria:**

- [x] AC-1: Halaman daftar saldo cuti: Nama, NIP, Jatah, Carry-Over, Terpakai, Sisa.
- [x] AC-2: Admin bisa melakukan **koreksi manual** saldo cuti (misal: menambah/mengurangi carry-over) dengan alasan wajib.
- [x] AC-3: Koreksi manual tercatat di audit log.
- [x] AC-4: Setiap awal tahun (1 Januari), sistem otomatis:
  - Menghitung carry-over N-1 maksimal 6 hari.
  - Menghitung hak tambahan jika pegawai tidak mengambil cuti tahunan pada N-2 dan N-1.
  - Membuat record `leave_balances` baru untuk tahun berjalan.
  - Mengisi total hak sesuai jatah dasar + carry-over/hak tambahan.

---

### US-4.10 · Konfigurasi Approval Chain Cuti

| Field | Detail |
|-------|--------|
| **ID** | US-4.10 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Cuti — Konfigurasi |
| **Dependensi** | US-1.4 |

> **Sebagai** Super Admin,
> **Saya ingin** mengatur urutan approver cuti,
> **Sehingga** alur cuti bisa mengikuti struktur LLDIKTI tanpa perubahan kode.

**Acceptance Criteria:**

- [x] AC-1: Halaman konfigurasi approval chain cuti.
- [ ] AC-2: Admin dapat mengatur chain per pegawai: kepala bagian, Ketua Tim Kerja, satu atau lebih verifikator, Kabag/Kepegawaian, dan Pimpinan/PYBMC. Konfigurasi tersebut dapat diterapkan sekaligus ke seluruh anggota satu unit kerja melalui penyalinan template. *(Direvisi oleh keputusan K-US-01, 5 Agustus 2026: rantai aktif pada runtime tetap tepat satu per pegawai sehingga tidak ada precedence global, unit, dan pegawai yang perlu ditetapkan. Cakupan unit sebagai lapisan resolusi runtime ditunda ke Fase 2. Sisa pekerjaan Fase 1 adalah aksi penerapan template ke unit.)*
- [x] AC-3: Perubahan konfigurasi tercatat di audit log.
- [x] AC-4: Konfigurasi langsung berlaku untuk pengajuan cuti baru.
- [x] AC-5: Ketua Tim Kerja dapat dipilih sebagai verifikator tanpa perlu role baru.
- [x] AC-6: Sistem melakukan skip otomatis jika approver pada dua step adalah orang yang sama.

---

### US-4.11 · Assign Kepala Bagian per Pegawai

| Field | Detail |
|-------|--------|
| **ID** | US-4.11 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Data Pegawai — Supervisor |
| **Dependensi** | US-2.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menetapkan kepala bagian untuk setiap pegawai,
> **Sehingga** pengajuan cuti bawahan otomatis diarahkan ke kepala bagian yang benar.

**Acceptance Criteria:**

- [x] AC-1: Di halaman detail pegawai, bagian "Kepala Bagian" menampilkan kepala bagian yang saat ini di-assign.
- [x] AC-2: Tombol "Ubah Kepala Bagian" membuka form: dropdown semua pegawai (kecuali diri sendiri), tanggal mulai berlaku.
- [x] AC-3: Satu pegawai hanya bisa memiliki satu kepala bagian aktif.
- [x] AC-4: Riwayat perubahan kepala bagian tersimpan (tanggal_mulai, tanggal_berakhir).
- [x] AC-5: Audit log mencatat perubahan.

---

### US-4.12 · Kalkulasi Hari Kerja Otomatis

| Field | Detail |
|-------|--------|
| **ID** | US-4.12 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Cuti — Engine |
| **Dependensi** | US-7.1 (hari libur) |

> **Sebagai** sistem,
> **Saya ingin** menghitung jumlah hari kerja secara otomatis saat pegawai memilih tanggal cuti,
> **Sehingga** saldo cuti yang dikurangi akurat dan sesuai dengan hari kerja sesungguhnya.

**Acceptance Criteria:**

- [x] AC-1: Kalkulasi menghitung hari kerja = total hari kalender dikurangi Sabtu, Minggu, hari libur nasional, dan cuti bersama.
- [x] AC-2: Referensi hari libur diambil dari tabel `ref_hari_libur` untuk tahun yang sesuai.
- [x] AC-3: Hasil kalkulasi ditampilkan real-time di form pengajuan cuti saat user memilih tanggal mulai dan selesai.
- [x] AC-4: Jika tanggal mulai atau selesai jatuh pada weekend/libur, tampilkan peringatan.
- [x] AC-5: Hasil kalkulasi disimpan di `jumlah_hari_kerja` saat submit.
- [x] AC-6: Jika tanggal mulai dan selesai berada pada tahun kalender berbeda, tampilkan error dan instruksi membuat dua pengajuan.

---

## E5 — Early Warning System (EWS)

### US-5.1 · Scheduler EWS Harian

| Field | Detail |
|-------|--------|
| **ID** | US-5.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 8 |
| **Modul** | EWS |
| **Dependensi** | US-2.6, US-2.7, E6 (Notifikasi) |

> **Sebagai** sistem,
> **Saya ingin** menjalankan pengecekan otomatis setiap hari terhadap semua pegawai aktif,
> **Sehingga** notifikasi kenaikan pangkat, KGB, pensiun, kontrak PPPK, dan Satyalancana terkirim tepat waktu.

**Acceptance Criteria:**

- [x] AC-1: Laravel scheduler berjalan setiap hari pukul 07:00 WITA (configurable). *(Jam dibaca dari konfigurasi EWS; setelah jam tersebut tercapai, pemeriksaan diulang tiap lima menit dengan proteksi overlap agar jadwal yang terlewat tetap terkejar.)*
- [x] AC-2: Cek semua pegawai aktif terhadap 5 trigger:
  - **Kenaikan Pangkat**: TMT pangkat terakhir + 4 tahun → cek H-90, H-60, H-30.
  - **KGB**: TMT KGB terakhir + 2 tahun → cek H-60, H-30, H-14.
  - **Pensiun (BUP)**: Tanggal lahir + BUP per jabatan → cek H-1thn, H-6bln, H-3bln.
  - **Kontrak PPPK**: Tanggal berakhir kontrak → cek H-6bln, H-3bln, H-1bln.
  - **Satyalancana**: TMT pengangkatan pertama + 10/20/30 tahun → cek H-180, H-90, H-30.
- [x] AC-3: Eligibility kenaikan pangkat: (1) 4 tahun terpenuhi, (2) `is_active` hukuman disiplin = false, (3) `is_kinerja_baik` = true. *(Status kelayakan dihitung dan disimpan pada alert; penekanan notifikasi saat flag bernilai false masih menjadi gap US-5.4 AC-2.)*
- [x] AC-4: Notifikasi **tidak duplikat**: jika notifikasi H-90 sudah dikirim hari ini, tidak kirim H-90 lagi besok. Gunakan tabel `ews_alerts` untuk tracking.
- [x] AC-5: Log eksekusi scheduler dicatat: waktu mulai, selesai, jumlah alert baru.
- [x] AC-6: Jika scheduler gagal (error), catat error di log dan kirim notifikasi ke Super Admin.
- [x] AC-7: Alert menyimpan status tindak lanjut (`aktif`, `ditangani`, `tidak_perlu`, `kedaluwarsa`).

---

### US-5.2 · Halaman Daftar EWS Aktif

| Field | Detail |
|-------|--------|
| **ID** | US-5.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | EWS |
| **Dependensi** | US-5.1 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat semua peringatan EWS yang aktif saat ini,
> **Sehingga** saya bisa menindaklanjuti sebelum deadline terlewat.

**Acceptance Criteria:**

- [x] AC-1: Tabel daftar EWS aktif, urut dari sisa hari terkecil (paling mendesak di atas).
- [x] AC-2: Kolom: Nama Pegawai, NIP, Jenis Event, Tanggal Target, Sisa Hari, Status Eligibility, Status Tindak Lanjut.
- [x] AC-3: Indikator warna baris:
  - 🔴 Merah: sisa < 30 hari.
  - 🟡 Kuning: sisa 30–90 hari.
  - 🟢 Hijau: sisa > 90 hari.
- [x] AC-4: Filter berdasarkan jenis event (Kenaikan Pangkat / KGB / Pensiun / Kontrak PPPK / Satyalancana) dan status tindak lanjut.
- [x] AC-5: Klik nama pegawai membuka halaman detail pegawai.
- [x] AC-6: Akses: Admin Kepegawaian, Super Admin, Pimpinan.
- [x] AC-7: Admin dapat menandai alert sebagai ditangani/tidak perlu dengan catatan.

---

### US-5.3 · EWS Pribadi (Pegawai)

| Field | Detail |
|-------|--------|
| **ID** | US-5.3 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | EWS |
| **Dependensi** | US-5.1 |

> **Sebagai** pegawai,
> **Saya ingin** melihat peringatan EWS yang relevan untuk diri saya,
> **Sehingga** saya tahu kapan kenaikan pangkat, KGB, pensiun, atau Satyalancana saya tiba.

**Acceptance Criteria:**

- [x] AC-1: Di dashboard pribadi atau halaman "Profil Saya", tampilkan section "Peringatan Penting".
- [x] AC-2: Menampilkan: jenis event, tanggal target, sisa hari, status eligibility.
- [x] AC-3: Hanya menampilkan EWS milik pegawai yang login.

---

### US-5.4 · Update Flag Kinerja Baik

| Field | Detail |
|-------|--------|
| **ID** | US-5.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 2 |
| **Modul** | EWS — Data Pendukung |
| **Dependensi** | US-2.4 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** menandai status kinerja pegawai secara manual,
> **Sehingga** EWS bisa menentukan apakah pegawai eligible untuk kenaikan pangkat.

**Acceptance Criteria:**

- [x] AC-1: Di halaman detail pegawai, toggle "Kinerja Baik" (default: Ya / `true`).
- [ ] AC-2: Jika diubah ke "Tidak" → pegawai **tidak eligible** kenaikan pangkat → EWS tidak mengirim notifikasi kenaikan pangkat untuk pegawai ini.
- [x] AC-3: Perubahan flag dicatat di audit log.
- [ ] AC-4: Tooltip penjelasan: *"Flag ini menggantikan penilaian SKP yang belum tersedia di Fase 1. Akan digantikan oleh modul Penilaian Kinerja di fase selanjutnya."*
- [x] AC-5: Untuk Satyalancana, Admin dapat mengisi flag/catatan kelayakan manual sampai data SKP terintegrasi.

---

### US-5.5 · Kalkulasi TMT Otomatis

| Field | Detail |
|-------|--------|
| **ID** | US-5.5 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | EWS — Engine |
| **Dependensi** | US-2.6 |

> **Sebagai** sistem,
> **Saya ingin** otomatis menghitung tanggal kenaikan pangkat, KGB, dan pensiun berikutnya setiap kali data riwayat/sumber resmi disimpan,
> **Sehingga** EWS selalu menggunakan data terbaru untuk trigger notifikasi.

**Acceptance Criteria:**

- [x] AC-1: Saat riwayat kepangkatan baru ditambahkan → hitung `tanggal_kenaikan_pangkat_berikutnya = tmt_pangkat + 4 tahun`.
- [x] AC-2: Saat riwayat KGB baru ditambahkan → hitung `tanggal_kgb_berikutnya = tmt_kgb + 2 tahun`.
- [x] AC-3: Saat jabatan baru ditambahkan → hitung ulang `tanggal_pensiun = tanggal_lahir + BUP_sesuai_jenis_jabatan_baru`.
- [ ] AC-4: Saat data pengangkatan pertama tersedia → hitung milestone Satyalancana 10/20/30 tahun.
- [ ] AC-5: Hasil kalkulasi disimpan di tabel `employees` (kolom computed atau tabel terpisah) agar scheduler EWS tidak perlu hitung ulang setiap hari.
- [x] AC-6: Kalkulasi TMT dipicu saat riwayat/sumber resmi disimpan per pegawai, bukan saat import massal selesai (keputusan pengguna 22 Juli 2026). Import Data Utama tidak memanggil kalkulasi ini dan tanggal pensiun hasil import dipertahankan apa adanya.

---

## E6 — Notifikasi

### US-6.1 · Notifikasi In-App

| Field | Detail |
|-------|--------|
| **ID** | US-6.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Notifikasi |
| **Dependensi** | — |

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** melihat notifikasi di dalam aplikasi,
> **Sehingga** saya segera tahu jika ada pengajuan cuti, persetujuan, atau peringatan EWS yang perlu perhatian saya.

**Acceptance Criteria:**

- [x] AC-1: Icon lonceng (🔔) di navbar dengan **badge angka** yang menunjukkan jumlah notifikasi belum dibaca.
- [x] AC-2: Klik icon lonceng membuka **dropdown** berisi 10 notifikasi terbaru.
- [x] AC-3: Setiap item notifikasi menampilkan: judul, waktu relatif ("5 menit lalu", "2 jam lalu"), indicator belum/sudah dibaca.
- [x] AC-4: Klik item notifikasi → tandai sebagai dibaca + redirect ke halaman terkait (misal: detail pengajuan cuti).
- [x] AC-5: Link "Lihat Semua Notifikasi" di bawah dropdown → halaman daftar notifikasi lengkap.
- [x] AC-6: Badge angka di-update tanpa perlu refresh halaman (polling setiap 30 detik atau SSE/WebSocket).

---

### US-6.2 · Halaman Semua Notifikasi

| Field | Detail |
|-------|--------|
| **ID** | US-6.2 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Notifikasi |
| **Dependensi** | US-6.1 |

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** melihat riwayat semua notifikasi saya,
> **Sehingga** saya bisa meninjau kembali notifikasi yang sudah lewat.

**Acceptance Criteria:**

- [x] AC-1: Halaman daftar seluruh notifikasi milik user yang login, urut terbaru di atas.
- [ ] AC-2: Indicator visual belum dibaca (bold/background highlight) vs sudah dibaca.
- [x] AC-3: Tombol "Tandai Semua Sudah Dibaca".
- [x] AC-4: Pagination.
- [x] AC-5: Klik notifikasi → redirect ke halaman terkait.

---

### US-6.3 · Notifikasi Email

| Field | Detail |
|-------|--------|
| **ID** | US-6.3 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Notifikasi |
| **Dependensi** | Mailpit untuk development; email operasional LLDIKTI untuk production |

> **Sebagai** pegawai,
> **Saya ingin** menerima notifikasi penting via email,
> **Sehingga** saya tetap informed meskipun tidak sedang membuka aplikasi SIMPEG.

**Acceptance Criteria:**

- [x] AC-1: Email dikirim via Laravel Mail + Queue (non-blocking).
- [x] AC-2: Template email Bahasa Indonesia, HTML formatted, responsive.
- [x] AC-3: Isi email: judul event, detail singkat, tombol/link "Lihat di SIMPEG" mengarah ke halaman terkait.
- [x] AC-4: Pengirim configurable via `.env`; nilai production memakai email operasional LLDIKTI atau Gmail resmi yang disediakan.
- [x] AC-5: Jika pengiriman gagal, catat di log dan retry otomatis (maks 3x).
- [x] AC-6: Email terkirim untuk semua jenis notifikasi yang berlabel ✅ Email di tabel notifikasi PRD.
- [x] AC-7: Pengiriman email dipanggil melalui notification dispatcher/channel config, bukan hardcoded langsung di domain cuti/EWS.

---

### US-6.4 · Tandai Notifikasi Sudah Dibaca

| Field | Detail |
|-------|--------|
| **ID** | US-6.4 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 2 |
| **Modul** | Notifikasi |
| **Dependensi** | US-6.1 |

> **Sebagai** pengguna SIMPEG,
> **Saya ingin** menandai notifikasi sebagai sudah dibaca,
> **Sehingga** badge counter berkurang dan saya bisa fokus pada notifikasi baru.

**Acceptance Criteria:**

- [x] AC-1: Klik notifikasi otomatis menandai sebagai dibaca.
- [x] AC-2: Tombol "Tandai sudah dibaca" per individual notifikasi (tanpa harus klik/redirect).
- [x] AC-3: Tombol "Tandai Semua Sudah Dibaca" di halaman daftar notifikasi.
- [x] AC-4: Badge angka di navbar langsung berkurang setelah aksi.

---

## E7 — Audit Log

### US-7.1 · Pencatatan Otomatis Semua Perubahan

| Field | Detail |
|-------|--------|
| **ID** | US-7.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Audit Log |
| **Dependensi** | — |

> **Sebagai** sistem,
> **Saya ingin** secara otomatis mencatat setiap operasi create, update, delete, verifikasi/keputusan cuti, dan login/logout,
> **Sehingga** tersedia audit trail lengkap yang tidak bisa dimanipulasi.

**Acceptance Criteria:**

- [ ] AC-1: Setiap operasi berikut otomatis menghasilkan record di `audit_logs`:
  - CREATE (tambah pegawai, tambah riwayat, tambah keluarga, dll)
  - UPDATE (edit data pegawai, koreksi saldo cuti, dll)
  - SOFT_DELETE (nonaktifkan pegawai)
  - RESTORE (aktifkan kembali)
  - VERIFY / DECIDE (verifikasi dan keputusan cuti)
  - CHANGE_REQUESTED / DEFER / NOT_APPROVED (Perubahan, Ditangguhkan, Tidak Disetujui)
  - LOGIN (login berhasil)
  - LOGOUT (logout manual atau session timeout)
  - IMPORT (import CSV)
- [x] AC-2: Setiap record audit log menyimpan: `user_id`, `user_name`, `event`, `auditable_type` (model), `auditable_id`, `old_values` (JSON), `new_values` (JSON), `ip_address`, `user_agent`, `created_at`.
- [ ] AC-3: Audit log **immutable** — tidak bisa diedit atau dihapus melalui aplikasi oleh siapa pun.
- [x] AC-4: Implementasi via Laravel Model Events atau package audit (misal: `owen-it/laravel-auditing`).

---

### US-7.2 · Halaman Daftar Audit Log

| Field | Detail |
|-------|--------|
| **ID** | US-7.2 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Audit Log |
| **Dependensi** | US-7.1 |

> **Sebagai** Admin Kepegawaian atau Super Admin,
> **Saya ingin** melihat daftar semua audit log,
> **Sehingga** saya bisa melacak siapa mengubah data apa dan kapan.

**Acceptance Criteria:**

- [x] AC-1: Tabel audit log: Waktu, User, Jenis Event, Modul/Tabel, Ringkasan Perubahan.
- [x] AC-2: Filter: jenis event (dropdown), user (dropdown), modul/tabel (dropdown), periode (date range picker).
- [ ] AC-3: Search berdasarkan nama user atau ID record.
- [ ] AC-4: Pagination (default 25 per halaman).
- [x] AC-5: Urut default: terbaru di atas.
- [x] AC-6: Akses: Super Admin dan Admin Kepegawaian saja.

---

### US-7.3 · Detail Audit Log (Diff View)

| Field | Detail |
|-------|--------|
| **ID** | US-7.3 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Audit Log |
| **Dependensi** | US-7.2 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** melihat detail perubahan data (sebelum vs sesudah),
> **Sehingga** saya tahu persis field mana yang berubah dan dari nilai apa ke nilai apa.

**Acceptance Criteria:**

- [x] AC-1: Klik baris audit log membuka halaman/modal detail.
- [x] AC-2: Tampilkan info: user, waktu, IP address, browser, jenis event, model, record ID.
- [x] AC-3: Tampilkan diff view:
  - Untuk UPDATE: tabel 2 kolom — "Sebelum" | "Sesudah", hanya field yang berubah (highlight).
  - Untuk CREATE: menampilkan semua `new_values`.
  - Untuk DELETE: menampilkan semua `old_values`.
- [x] AC-4: Tombol "Lihat Record" untuk navigasi ke record yang diubah (jika masih ada).

---

## E8 — Dashboard

### US-8.1 · Dashboard Admin & Pimpinan

| Field | Detail |
|-------|--------|
| **ID** | US-8.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 8 |
| **Modul** | Dashboard |
| **Dependensi** | US-2.3, US-4.8, US-5.2 |

> **Sebagai** Admin Kepegawaian atau Pimpinan,
> **Saya ingin** melihat ringkasan data kepegawaian dalam satu halaman dashboard,
> **Sehingga** saya bisa memantau kondisi terkini tanpa harus membuka banyak halaman.

**Acceptance Criteria:**

- [x] AC-1: **Widget W1 — Komposisi Pegawai**: KPI card jumlah total pegawai aktif + pie chart breakdown PNS vs PPPK.
- [x] AC-2: **Widget W2 — Kenaikan Pangkat**: KPI card + daftar pegawai yang naik pangkat bulan ini dan tahun ini.
- [x] AC-3: **Widget W3 — Status Cuti**: KPI card jumlah pengajuan pending, disetujui bulan ini, ditunda.
- [x] AC-4: **Widget W4 — EWS Aktif**: Tabel 5 EWS paling urgent, dengan indikator warna. Link ke halaman EWS lengkap.
- [x] AC-5: **Widget W5 — Distribusi Golongan**: Bar chart jumlah pegawai per golongan.
- [x] AC-6: **Widget W6 — Audit Terbaru**: List 5 perubahan data terakhir. Link ke audit log.
- [x] AC-7: **Widget W7 — Tren Pegawai**: Line chart jumlah pegawai aktif per bulan (12 bulan terakhir).
- [x] AC-8: Data dashboard diperbarui setiap kali halaman di-load (server-rendered).
- [x] AC-9: Layout responsive — tampil rapi di desktop, tablet, dan mobile.

> **Catatan status Dashboard Admin (5 Agustus 2026):** payload W1–W7 dipasok `BuildAdminDashboardAction` sesuai kontrak K-3, tanpa data contoh dan tanpa tautan mati. Setiap widget memiliki empty state jujur ketika data belum tersedia. Admin Kepegawaian tetap bukan approver cuti sehingga widget cuti hanya menautkan ke halaman pemantauan. Dashboard Pimpinan memakai `BuildPimpinanDashboardAction` terpisah. Penyempurnaan basis tren W7 tetap menjadi pekerjaan lanjutan yang tercatat di tracker Sprint 6.

---

### US-8.2 · Dashboard Pegawai (Pribadi)

| Field | Detail |
|-------|--------|
| **ID** | US-8.2 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 5 |
| **Modul** | Dashboard |
| **Dependensi** | US-2.5, US-4.3, US-5.3 |

> **Sebagai** pegawai,
> **Saya ingin** melihat dashboard pribadi saat login,
> **Sehingga** saya langsung tahu informasi penting tentang data kepegawaian saya.

**Acceptance Criteria:**

- [x] AC-1: **Profil Ringkas**: Foto, nama, NIP, golongan, jabatan, unit kerja.
- [x] AC-2: **Saldo Cuti**: Card menampilkan sisa cuti tahunan (jatah + carry-over - terpakai).
- [x] AC-3: **Pengajuan Cuti Aktif**: Daftar pengajuan cuti yang sedang berjalan + statusnya.
- [x] AC-4: **EWS Pribadi**: Peringatan yang relevan (kenaikan pangkat, KGB, pensiun).
- [x] AC-5: **Notifikasi Terbaru**: 5 notifikasi terakhir.
- [x] AC-6: Layout responsive.

---

### US-8.3 · Dashboard Kepala Bagian

| Field | Detail |
|-------|--------|
| **ID** | US-8.3 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 5 |
| **Modul** | Dashboard |
| **Dependensi** | US-4.4, US-4.11 |

> **Sebagai** Kepala Bagian,
> **Saya ingin** melihat ringkasan data bawahan langsung saya,
> **Sehingga** saya bisa memantau pengajuan cuti dan informasi penting bawahan.

> **Catatan keputusan 26 Juli 2026 (kanonis):** status `Dinas Luar` ditunda ke Fase 2 dan kelak diturunkan dari modul Surat Tugas/penugasan, bukan input manual. AC-1 Fase 1 direvisi menjadi status `aktif/cuti` saja. Lihat `Kickoff-Sprint-6-Kontrak-dan-Keputusan.md` (K-2).

**Acceptance Criteria:**

- [x] AC-1: **Daftar Bawahan**: Nama, jabatan, status (aktif/cuti; *dinas luar ditunda ke Fase 2 — keputusan 26 Juli 2026*).
- [x] AC-2: **Pengajuan Cuti Pending**: Daftar pengajuan cuti bawahan yang menunggu tindakan saya (quick action sesuai label resmi keputusan cuti).
- [x] AC-3: **EWS Bawahan**: Peringatan EWS yang relevan untuk bawahan langsung.
- [x] AC-4: Klik nama bawahan membuka detail ringkas (read-only).

---

### US-8.4 · Kelola Hari Libur Nasional & Cuti Bersama

| Field | Detail |
|-------|--------|
| **ID** | US-8.4 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Konfigurasi / Reference Table |
| **Dependensi** | — |

> **Sebagai** Super Admin,
> **Saya ingin** menginput daftar hari libur nasional dan cuti bersama setiap tahun,
> **Sehingga** kalkulasi hari kerja cuti dan EWS akurat.

**Acceptance Criteria:**

- [x] AC-1: Halaman daftar hari libur per tahun: tanggal, nama, tipe (Libur Nasional / Cuti Bersama).
- [x] AC-2: Form tambah: tanggal (date picker), nama hari libur, tipe.
- [x] AC-3: Bisa edit dan hapus hari libur.
- [x] AC-4: Filter per tahun.
- [x] AC-5: Audit log mencatat perubahan.

---

### US-8.5 · Kelola Reference Tables

| Field | Detail |
|-------|--------|
| **ID** | US-8.5 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 5 |
| **Modul** | Konfigurasi |
| **Dependensi** | — |

> **Sebagai** Super Admin,
> **Saya ingin** mengelola tabel referensi (golongan, jabatan, unit kerja, dll),
> **Sehingga** pilihan dropdown di seluruh aplikasi selalu up-to-date.

**Acceptance Criteria:**

- [x] AC-1: Halaman admin untuk mengelola setiap reference table: ref_golongan, ref_jenis_jabatan, ref_jabatan, ref_status_pegawai, ref_eselon, ref_unit_kerja hierarkis, ref_jenjang_pendidikan, dan ref_notification_channels — **8 tabel**. `ref_bup` dikeluarkan dari cakupan per K-4 (27 Juli 2026) karena tidak dibaca perhitungan BUP mana pun; sumber BUP resmi adalah `ref_jabatan.default_bup` dengan fallback `ref_jenis_jabatan.maks_usia_pensiun`. Tujuh tabel dikelola sebagai tab pada halaman `/data-master`, sedangkan `ref_notification_channels` dikelola pada halaman tersendiri `/data-master/channel-notifikasi` yang tertaut dari menu Super Admin.
- [x] AC-2: CRUD per table: lihat daftar, tambah, edit, hapus (soft delete jika sudah dipakai oleh data pegawai). Frasa *soft delete* di sini dibaca sebagai **nonaktif melalui kolom `is_active`** sesuai K-1 aturan 5 (26 Juli 2026); Fase 1 tidak menambahkan `SoftDeletes` maupun kolom `deleted_at` pada reference table. Item yang belum pernah dipakai boleh dihapus permanen, item yang sudah dipakai hanya boleh dinonaktifkan dan diaktifkan kembali.
- [ ] AC-3: Validasi: tidak bisa menghapus item reference table yang sedang dipakai oleh data pegawai.
- [x] AC-4: Perubahan tercatat di audit log.
- [x] AC-5: Data reference table yang sudah di-seed saat instalasi tidak boleh hilang.

---

## E9 — Laporan & Export

### US-9.1 · Export Daftar Pegawai ke Excel

| Field | Detail |
|-------|--------|
| **ID** | US-9.1 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Laporan |
| **Dependensi** | US-2.3 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport daftar pegawai ke file Excel,
> **Sehingga** saya bisa mengolah data lebih lanjut di spreadsheet.

**Acceptance Criteria:**

- [x] AC-1: Tombol "Export Excel" di halaman daftar pegawai.
- [x] AC-2: Export mengikuti filter yang sedang aktif (golongan, unit kerja, jenis pegawai, status).
- [x] AC-3: File Excel (.xlsx) berisi kolom: No, NIP, Nama, Golongan, Jabatan, Unit Kerja, Jenis Pegawai, Status.
- [x] AC-4: File otomatis ter-download di browser.
- [x] AC-5: Nama file: `Daftar_Pegawai_LLDIKTI_XVI_{tanggal}.xlsx`.

---

### US-9.1B · Export Daftar Pegawai Custom ke Excel

| Field | Detail |
|-------|--------|
| **ID** | US-9.1B |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Laporan |
| **Dependensi** | US-2.3 |

> **Sebagai** Admin Kepegawaian atau Pimpinan,
> **Saya ingin** memilih kolom dan filter sebelum export nominatif Excel,
> **Sehingga** saya bisa membuat laporan sesuai kebutuhan tanpa perubahan kode.

**Acceptance Criteria:**

- [x] AC-1: Halaman export custom menyediakan daftar kolom yang boleh dipilih.
- [x] AC-2: Filter baris mendukung status pegawai, unit/tim kerja, jenis pegawai, golongan, jabatan, dan periode pensiun.
- [x] AC-3: Output hanya Excel `.xlsx`.
- [x] AC-4: Urutan kolom di file mengikuti pilihan pengguna.
- [x] AC-5: Kolom sensitif yang tidak diizinkan tidak muncul di daftar pilihan.

---

### US-9.2 · Export Daftar Pegawai ke PDF

| Field | Detail |
|-------|--------|
| **ID** | US-9.2 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Laporan |
| **Dependensi** | US-2.3 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport daftar pegawai ke PDF berformat laporan resmi,
> **Sehingga** saya bisa mencetak laporan untuk kebutuhan administrasi.

**Acceptance Criteria:**

- [ ] AC-1: Tombol "Export PDF" di halaman daftar pegawai. *(Kriteria dipertahankan sesuai keputusan K-US-04, 5 Agustus 2026: tombol ditambahkan di halaman daftar pegawai agar sejajar dengan tombol Export Excel pada US-9.1 AC-1, meneruskan filter yang sedang aktif ke rute PDF yang sudah ada, memakai gerbang peran dan izin yang sama, serta menampilkan pesan penyempitan filter bila batas 500 baris terlampaui. Halaman Laporan tetap menjadi permukaan lanjutan untuk pilihan kolom, rentang baris, dan pratinjau. Perilaku baca-saja Pimpinan dipertahankan.)*
- [x] AC-2: PDF memiliki header: logo (jika ada), nama instansi "LLDIKTI Wilayah XVI", judul "Daftar Pegawai", tanggal cetak.
- [x] AC-3: Tabel data pegawai sesuai filter aktif.
- [x] AC-4: Footer: halaman X dari Y.
- [x] AC-5: Orientasi landscape untuk mengakomodasi banyak kolom.

---

### US-9.3 · Export Rekap Cuti ke Excel

| Field | Detail |
|-------|--------|
| **ID** | US-9.3 |
| **Prioritas** | 🔴 P0 |
| **Story Points** | 3 |
| **Modul** | Laporan |
| **Dependensi** | US-4.8 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport rekap penggunaan cuti ke Excel,
> **Sehingga** saya punya data cuti yang bisa diolah untuk pelaporan.

**Acceptance Criteria:**

- [x] AC-1: Halaman export rekap cuti dengan filter: periode (bulan/tahun), unit kerja, pegawai tertentu.
- [x] AC-2: File Excel berisi: No, NIP, Nama, Jenis Cuti, Tanggal Mulai, Tanggal Selesai, Jumlah Hari, Status.
- [x] AC-3: Sheet tambahan: ringkasan per pegawai (total per jenis cuti, sisa saldo).
- [x] AC-4: File otomatis ter-download.
- [x] AC-5: Nama file: `Rekap_Cuti_{periode}_{tanggal}.xlsx`.

---

### US-9.4 · Export Rekap Cuti ke PDF

| Field | Detail |
|-------|--------|
| **ID** | US-9.4 |
| **Prioritas** | 🟡 P1 |
| **Story Points** | 3 |
| **Modul** | Laporan |
| **Dependensi** | US-4.8 |

> **Sebagai** Admin Kepegawaian,
> **Saya ingin** mengexport rekap cuti ke PDF berformat laporan resmi,
> **Sehingga** bisa dicetak dan diarsipkan.

**Acceptance Criteria:**

- [x] AC-1: Filter periode (bulan/tahun) sebelum export.
- [x] AC-2: PDF memiliki header institusi, judul "Rekap Cuti Pegawai", periode laporan.
- [x] AC-3: Tabel rekap per pegawai.
- [x] AC-4: Bagian bawah: tempat tanda tangan (Pembuat Laporan, Mengetahui).
- [x] AC-5: Footer halaman.

---

## Dependency Map

```
E1 (Auth/SSO)
  ├── US-1.1 Login
  ├── US-1.2 Logout ←── US-1.1
  ├── US-1.3 Session Timeout ←── US-1.1
  ├── US-1.4 Mapping User ←── US-1.1
  └── US-1.5 Redirect ←── US-1.1, US-1.4

E2 (Data Pegawai) ←── E1
  ├── US-2.1 Tambah Pegawai
  ├── US-2.2 Edit Pegawai ←── US-2.1
  ├── US-2.3 Daftar Pegawai ←── US-2.1
  ├── US-2.4 Detail Pegawai ←── US-2.1
  ├── US-2.5 Profil Sendiri ←── US-2.1, US-1.1
  ├── US-2.6 Riwayat (Pangkat/Jabatan/KGB) ←── US-2.1
  ├── US-2.7 Hukuman Disiplin ←── US-2.1
  ├── US-2.8 Data Keluarga ←── US-2.1
  ├── US-2.9 Soft Delete ←── US-2.1
  └── US-2.10 Soft Delete Super Admin ←── US-2.9

E3 (Import Excel/CSV) ←── E2
  ├── US-3.1 Template Import
  ├── US-3.2 Upload & Preview ←── US-3.1
  ├── US-3.3 Validasi ←── US-3.2
  └── US-3.4 Eksekusi ←── US-3.3

E4 (Cuti) ←── E2
  ├── US-4.11 Assign Kepala Bagian ←── US-2.1
  ├── US-4.10 Konfigurasi Approval Chain ←── US-1.4
  ├── US-4.12 Kalkulasi Hari Kerja ←── US-8.4
  ├── US-4.1 Ajukan Cuti ←── US-4.11, US-4.12
  ├── US-4.2 Daftar Cuti Pegawai ←── US-4.1
  ├── US-4.3 Saldo Cuti ←── US-4.1
  ├── US-4.4 Approval/Verifikasi Step 1 ←── US-4.1
  ├── US-4.5 Verifikasi Kepegawaian ←── US-4.4
  ├── US-4.6 Keputusan Final PYBMC ←── US-4.5
  ├── US-4.7 Timeline Approval ←── US-4.4
  ├── US-4.8 Daftar Cuti Admin ←── US-4.1
  └── US-4.9 Kelola Saldo ←── US-4.1

E5 (EWS) ←── E2
  ├── US-5.5 Kalkulasi TMT ←── US-2.6
  ├── US-5.4 Flag Kinerja ←── US-2.4
  ├── US-5.1 Scheduler ←── US-5.5, US-5.4, E6
  ├── US-5.2 Daftar EWS ←── US-5.1
  └── US-5.3 EWS Pribadi ←── US-5.1

E6 (Notifikasi) — Independen
  ├── US-6.1 In-App
  ├── US-6.2 Halaman Notifikasi ←── US-6.1
  ├── US-6.3 Email
  └── US-6.4 Tandai Dibaca ←── US-6.1

E7 (Audit Log) — Independen
  ├── US-7.1 Pencatatan Otomatis
  ├── US-7.2 Halaman Daftar ←── US-7.1
  └── US-7.3 Detail Diff ←── US-7.2

E8 (Dashboard) ←── E2, E4, E5
  ├── US-8.1 Dashboard Admin ←── US-2.3, US-4.8, US-5.2
  ├── US-8.2 Dashboard Pegawai ←── US-2.5, US-4.3, US-5.3
  ├── US-8.3 Dashboard Kepala Bagian ←── US-4.4, US-4.11
  ├── US-8.4 Kelola Hari Libur
  └── US-8.5 Kelola Reference Tables

E9 (Laporan) ←── E2, E4
  ├── US-9.1 Export Pegawai Excel ←── US-2.3
  ├── US-9.1B Export Pegawai Custom Excel ←── US-2.3
  ├── US-9.2 Export Pegawai PDF ←── US-2.3
  ├── US-9.3 Export Cuti Excel ←── US-4.8
  └── US-9.4 Export Cuti PDF ←── US-4.8
```

---

## Rekomendasi Urutan Sprint

### Sprint 1 — Fondasi (Minggu 1–2)

Sprint 1 tetap menjadi fondasi teknis sebelum vertical slice dimulai.

| Story | SP |
|-------|:--:|
| US-7.1 Audit Log otomatis | 5 |
| US-6.1 Notifikasi in-app | 5 |
| US-1.1 Login SSO | 5 |
| US-1.2 Logout | 2 |
| US-1.4 Mapping User | 5 |
| US-8.4 Kelola hari libur | 3 |
| **Total** | **25** |

### Sprint 2 — Data Pegawai Core (Minggu 3–4)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| CRUD pegawai core | US-2.1 Tambah pegawai | 8 |
| CRUD pegawai core | US-2.2 Edit pegawai | 5 |
| CRUD pegawai core | US-2.3 Daftar pegawai | 5 |
| CRUD pegawai core | US-2.4 Detail pegawai | 5 |
| Riwayat pegawai | US-2.6 Riwayat kepangkatan/jabatan/KGB | 5 |
| Disiplin pegawai | US-2.7 Hukuman disiplin | 3 |
| **Total** | | **31** |

### Sprint 3 — Import & Pelengkap Data Pegawai (Minggu 5–6)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| Import Excel/CSV | US-3.1 Template Import | 2 |
| Import Excel/CSV | US-3.2 Upload & Preview | 5 |
| Import Excel/CSV | US-3.3 Validasi | 5 |
| Import Excel/CSV | US-3.4 Eksekusi Import | 7 |
| Profil & keluarga | US-2.5 Profil sendiri | 3 |
| Profil & keluarga | US-2.8 Data keluarga | 3 |
| Penghapusan aman | US-2.9 Soft delete | 3 |
| Penghapusan aman | US-2.10 Soft delete Super Admin | 2 |
| **Total** | | **30** |

### Sprint 4 — Cuti Core (Minggu 7–9)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| Setup aturan cuti | US-4.10 Konfigurasi approval chain | 3 |
| Setup aturan cuti | US-4.11 Assign kepala bagian | 3 |
| Setup aturan cuti | US-4.12 Kalkulasi hari kerja | 5 |
| Setup aturan cuti | US-4.3 Saldo cuti | 3 |
| Pengajuan cuti | US-4.1 Ajukan cuti | 5 |
| Pengajuan cuti | US-4.2 Daftar cuti pegawai | 3 |
| Approval & timeline | US-4.4 Approval/verifikasi step 1 | 5 |
| Approval & timeline | US-4.5 Verifikasi Kepegawaian | 3 |
| Approval & timeline | US-4.6 Keputusan final PYBMC | 5 |
| Approval & timeline | US-4.7 Timeline approval | 3 |
| **Total** | | **38** |

### Sprint 5 — EWS & Notifikasi (Minggu 10–11)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| Kalkulasi & scheduler EWS | US-5.1 Scheduler EWS | 8 |
| Kalkulasi & scheduler EWS | US-5.5 Kalkulasi TMT | 5 |
| Daftar EWS & flag | US-5.2 Daftar EWS | 5 |
| Daftar EWS & flag | US-5.3 EWS pribadi | 3 |
| Daftar EWS & flag | US-5.4 Flag kinerja | 2 |
| Notifikasi & keamanan session | US-6.3 Email notifikasi | 5 |
| Notifikasi & keamanan session | US-1.3 Session timeout | 3 |
| **Total** | | **31** |

### Sprint 6 — Dashboard & Laporan (Minggu 12–13)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| Dashboard admin/pegawai | US-8.1 Dashboard admin | 8 |
| Dashboard admin/pegawai | US-8.2 Dashboard pegawai | 5 |
| Dashboard Kepala Bagian/reference | US-8.3 Dashboard Kepala Bagian | 5 |
| Dashboard Kepala Bagian/reference | US-8.5 Kelola reference tables | 5 |
| Laporan & export | US-9.1 Export pegawai Excel | 3 |
| Laporan & export | US-9.1B Export pegawai custom Excel | 3 |
| Laporan & export | US-9.2 Export pegawai PDF | 3 |
| Laporan & export | US-9.3 Export cuti Excel | 3 |
| Laporan & export | US-9.4 Export cuti PDF | 3 |
| **Total** | | **38** |

### Sprint 7 — Stabilization, Regression, UAT, Go-Live Prep (Minggu 14–16)

| Vertical Slice | Story | SP |
|----------------|-------|:--:|
| Audit & redirect | US-7.2 Halaman audit log | 3 |
| Audit & redirect | US-7.3 Detail audit diff | 3 |
| Audit & redirect | US-1.5 Redirect per role | 3 |
| Notifikasi lanjutan | US-6.2 Halaman notifikasi | 3 |
| Notifikasi lanjutan | US-6.4 Tandai dibaca | 2 |
| Admin cuti & bugfix | US-4.8 Daftar cuti admin | 3 |
| Admin cuti & bugfix | US-4.9 Kelola saldo | 5 |
| Regression/UAT | Bugfix mayor, full regression, UAT, release candidate | — |
| **Total story point fitur** | | **22** |

---

> **Catatan:** Mulai Sprint 2, setiap vertical slice mengikuti alur: kickoff acceptance criteria, frontend mock/dummy data, backend real data, sinkronisasi, review PR oleh Adriel, bugfix oleh owner task, lalu QA/retest oleh Grantly. Estimasi story points dan sprint plan tetap indikatif dan bisa disesuaikan dengan kapasitas aktual tim.
