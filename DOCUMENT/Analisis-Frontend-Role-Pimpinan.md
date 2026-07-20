# Analisis Frontend Role Pimpinan

> Status: Audit read-only — belum sepenuhnya sesuai dokumen
>
> Tanggal audit: 21 Juli 2026
>
> Cakupan: frontend Blade, rute, controller/action pendukung, validasi request, dan test untuk role `pimpinan`. Dokumen ini tidak mengubah kebutuhan produk dan tidak menggantikan PRD.

## 1. Ringkasan Eksekutif

Frontend role **Pimpinan** telah mempunyai fondasi yang baik pada keputusan cuti, detail pegawai read-only, EWS, dan laporan cuti/kepangkatan. Pemisahan Controller → Action pada sebagian besar alur Pimpinan juga sudah mengikuti pola proyek.

Namun, frontend ini **belum dapat dinyatakan sepenuhnya sesuai** dengan PRD dan User Stories. Temuan paling penting adalah:

1. Halaman Data Pegawai Pimpinan masih menampilkan aksi tambah, impor, edit, dan ekspor Admin, padahal Pimpinan hanya memiliki akses read-only.
2. Menu laporan Pimpinan mengarah ke halaman laporan generik yang tidak konsisten dengan kontrak laporan Pimpinan. Halaman custom Pimpinan yang benar sudah ada, tetapi tidak terhubung dari sidebar.
3. Halaman laporan custom Pimpinan belum menyediakan filter unit kerja, jenis pegawai, dan jabatan.
4. Dashboard mempunyai ketidakakuratan pada distribusi golongan, tren historis pegawai, dan total EWS aktif.
5. Pencarian EWS tampil di antarmuka tetapi tidak diproses oleh controller/action.
6. Terdapat konflik dokumen terkait tautan audit log untuk Pimpinan yang memerlukan keputusan produk sebelum diimplementasikan.

## 2. Acuan Audit

Urutan sumber kebenaran yang digunakan:

1. [PRD-SIMPEG-Fase1-Core.md](PRD-SIMPEG-Fase1-Core.md) versi 1.2.
2. [Panduan-Penulisan-Kode-SIMPEG.md](Panduan-Penulisan-Kode-SIMPEG.md).
3. [User-Stories-SIMPEG-Fase1.md](User-Stories-SIMPEG-Fase1.md).

Hak Pimpinan menurut PRD:

| Area | Ketentuan |
|---|---|
| Dashboard | Mengakses dashboard dengan data seluruh pegawai. |
| Cuti | Menjadi pejabat final/PYBMC sesuai konfigurasi approval chain. |
| Data Pegawai | Read-only untuk semua data pegawai. |
| Laporan | Generate dan export laporan. |

Widget Dashboard Pimpinan dinilai terhadap W1 sampai W7. Laporan dinilai terhadap L1, L1b, L2, L3, dan US-9.1B. Keputusan cuti final dinilai terhadap US-4.6, sedangkan EWS dinilai terhadap US-5.2.

## 3. Status Kesesuaian per Area

| Area | Status | Kesimpulan |
|---|:---:|---|
| Navigasi Pimpinan | ⚠️ | Menu utama tersedia, tetapi landing page laporan khusus Pimpinan tidak terhubung dari sidebar. |
| Dashboard | ⚠️ | Semua kelompok widget tersedia, tetapi beberapa data/angka belum memenuhi definisi dokumen. |
| Data Pegawai | ❌ | Backend mengamankan akses, tetapi UI masih menampilkan aksi mutasi yang tidak boleh digunakan Pimpinan. |
| Persetujuan Cuti | ✅ | Alur keputusan, guard approver aktif, dokumen, dan notifikasi telah tersedia. |
| EWS | ⚠️ | Tabel dan filter utama ada; pencarian di UI tidak berfungsi dan markup memiliki cacat. |
| Laporan Cuti | ✅ | Pratinjau dan ekspor sesuai filter telah tersedia dan teruji. |
| Laporan Kepangkatan | ✅ | Pratinjau serta ekspor Excel/PDF fixed tersedia dan teruji. |
| Laporan Pegawai Standard/Custom | ❌ | Ada dua jalur yang tidak konsisten; custom Pimpinan tidak mudah ditemukan dan filternya belum lengkap. |
| Struktur kode | ⚠️ | Mayoritas alur sudah Controller → Action; laporan generik masih memakai closure route serta filtering browser. |
| Aksesibilitas/responsif | ⚠️ | Banyak komponen sudah baik, tetapi terdapat HTML invalid dan kontrol role yang salah. |

## 4. Hal yang Sudah Sesuai

### 4.1 Persetujuan cuti final Pimpinan

Alur cuti Pimpinan telah mendukung kebutuhan utama US-4.6:

- Pengajuan hanya dapat diputus oleh approver pada step aktif.
- Keputusan memakai istilah resmi: `Disetujui`, `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui`.
- Catatan diwajibkan untuk semua keputusan selain `Disetujui`.
- Keputusan final menggunakan workflow cuti yang sama, termasuk audit, notifikasi, pengurangan saldo cuti tahunan, serta bukti cuti dengan QR.
- Lampiran dan dokumen cuti dibuka melalui rute yang terotorisasi.

Implementasi daftar cuti memakai Action khusus `App\Actions\Cuti\ListPimpinanLeavesAction`, sehingga filter dan transformasi data tidak diletakkan di Blade.

### 4.2 Detail pegawai Pimpinan

Halaman detail pegawai menggunakan rute Pimpinan sendiri dan bersifat read-only. Beberapa aspek yang sudah baik:

- NIK keluarga tidak ditampilkan pada tampilan Pimpinan.
- Tab informasi, keluarga, dan supervisor memakai atribut ARIA.
- Navigasi keyboard pada tab tersedia.
- Data supervisor aktif dan keluarga ditampilkan tanpa tombol mutasi.

### 4.3 EWS

Halaman EWS Pimpinan telah menyediakan:

- Data EWS aktif dengan urutan urgensi.
- Filter jenis event dan status tindak lanjut.
- Kolom pegawai, NIP, event, tanggal target, sisa hari, kelayakan, dan tindak lanjut.
- Link nama pegawai menuju halaman detail Pimpinan.
- Paginasi dan pilihan jumlah data per halaman.

### 4.4 Laporan cuti dan kepangkatan

Laporan cuti serta riwayat kepangkatan telah memiliki jalur khusus Pimpinan. Laporan kepangkatan mendukung format Excel dan PDF fixed, sehingga tidak melanggar batas Fase 1 yang melarang PDF custom.

## 5. Temuan Prioritas Tinggi

### P1 — Data Pegawai Pimpinan tidak read-only dari sisi UI

**Kebutuhan:** PRD menetapkan Data Pegawai Pimpinan sebagai read-only.

**Kondisi saat ini:** `PimpinanEmployeeController` merender view bersama `admin.pegawai.index` dengan flag `$isReadOnly`. Akan tetapi flag tersebut hanya mengubah sebagian tautan detail. Elemen berikut tetap dirender:

- Tombol `Tambah Pegawai`.
- Pilihan `Tambah Manual`.
- Pilihan `Import Pegawai`.
- Tombol `Edit` per pegawai.
- Checkbox pemilihan data dan bulk action.
- Tombol `Export Excel` yang mengarah ke endpoint khusus Admin.

Backend memang sudah membatasi rute tambah, impor, edit, serta ekspor untuk Super Admin/Admin Kepegawaian. Karena itu tidak terjadi eskalasi hak akses. Namun, Pimpinan masih akan melihat aksi yang ketika digunakan berakhir dengan `403 Forbidden`, sehingga frontend tidak sesuai peran dan menghasilkan pengalaman pengguna yang buruk.

**Rekomendasi:**

1. Gunakan `$isReadOnly` untuk menyembunyikan semua kontrol mutasi dan bulk action.
2. Sisakan detail, filter, refresh, dan tautan menuju laporan Pimpinan yang sah.
3. Tambahkan test respons halaman yang memastikan tidak ada label tambah/impor/edit dan tidak ada URL endpoint Admin untuk Pimpinan.

### P1 — Jalur laporan custom Pimpinan tidak terhubung dari sidebar

**Kondisi saat ini:** Sidebar Pimpinan mengarahkan `Laporan → Data Pegawai` ke rute generik `laporan.pegawai`. Di sisi lain, halaman khusus Pimpinan sudah tersedia pada rute `pimpinan.laporan.index` dan `pimpinan.laporan.pegawai`.

Struktur saat ini:

```text
Sidebar Pimpinan
  └── Laporan / Data Pegawai
        └── laporan.pegawai              (halaman generik)

Halaman laporan Pimpinan
  └── pimpinan.laporan.index             (tidak ada pada sidebar)
        └── pimpinan.laporan.pegawai     (custom Excel Pimpinan)
```

Akibatnya, fitur custom yang telah dibuat dan memiliki test tidak mudah ditemukan oleh pengguna normal. Halaman tersebut hanya dapat dicapai jika pengguna mengetahui URL internal atau terlebih dahulu membuka landing page laporan yang tidak ada pada menu utama.

**Rekomendasi:**

1. Arahkan sidebar ke `pimpinan.laporan.index` atau langsung ke `pimpinan.laporan.pegawai`.
2. Konsolidasikan laporan standard dan custom agar tidak ada dua pengalaman yang berbeda untuk role yang sama.
3. Pastikan rute menu, breadcrumb, controller, dan test menggunakan jalur Pimpinan yang sama.

### P1 — Filter laporan custom Pimpinan belum lengkap

US-9.1B mensyaratkan filter status, unit/tim kerja, jenis pegawai, golongan, jabatan, dan periode pensiun.

Halaman `pimpinan/laporan/pegawai` saat ini hanya menyediakan:

- Nama/NIP.
- Golongan.
- Status.
- Pensiun dari.
- Pensiun sampai.

Filter **unit kerja**, **jenis pegawai**, dan **jabatan** tidak tersedia pada UI, meskipun kontrak request di backend telah mendukung field tersebut.

**Rekomendasi:**

1. Tambahkan select berbasis reference table untuk unit kerja, jenis pegawai, dan jabatan.
2. Pertahankan filter aktif ketika validasi gagal atau ketika pengguna kembali ke halaman pratinjau.
3. Tambahkan test filter kombinasi serta pastikan output Excel menggunakan filter yang sama.

### P1 — Laporan generik pada menu Pimpinan memuat data di luar kontrak L1

Rute `laporan.pegawai` yang digunakan sidebar masih berupa closure route. Closure tersebut mengambil seluruh pegawai dan mengirim data berikut ke browser:

- Data pegawai aktif dan tidak aktif.
- Email pribadi.
- Nomor telepon pribadi.

Padahal L1 pada PRD mendefinisikan daftar nominatif standard sebagai daftar semua **pegawai aktif**, dengan informasi inti NIP, nama, golongan, jabatan, unit kerja, dan jenis pegawai.

Selain itu, filtering dilakukan di browser. Halaman menyediakan kolom email/no. HP dan pengguna dapat memilih seluruh kolom untuk pratinjau/cetak browser. Walaupun endpoint custom server-side memakai whitelist yang aman dan menolak NIK/No. KK, data kontak sudah terlanjur masuk ke browser pada jalur laporan generik.

Test yang ada juga mengunci perilaku preview yang menampilkan pegawai pensiun, sehingga ketidaksesuaian ini tidak akan terdeteksi sebagai regresi.

**Rekomendasi:**

1. Ganti closure route dengan `LaporanController` → `ExportPegawaiPreviewAction`.
2. Gunakan `EmployeeExportDataService` untuk pratinjau dan ekspor agar keduanya konsisten dengan default pegawai aktif.
3. Jangan kirim email pribadi dan nomor telepon pribadi bila bukan bagian dari kontrak laporan Pimpinan.
4. Ubah test agar preview dan Excel standard sama-sama mengecualikan pegawai tidak aktif secara default.
5. Tambahkan test role Pimpinan pada laporan standard, tidak hanya Admin Kepegawaian.

### P1 — Modal custom pada halaman laporan generik tidak dapat dipakai

Halaman laporan generik memiliki markup modal custom, tetapi audit menemukan state/fungsi Alpine berikut hanya dipakai tanpa deklarasi implementasi:

- `showCustomExportModal`
- `customColumns`
- `customJabatan`
- `customPensiunDari`
- `customPensiunSampai`
- `submitCustomExport`
- `filterOptions`

Tidak ditemukan tombol yang membuka modal tersebut. Dengan demikian, modal custom pada halaman generik yang diakses dari menu Pimpinan tidak berfungsi sebagai jalur export custom.

**Rekomendasi:** hapus modal generik yang tidak selesai atau lengkapi semua state, trigger, dan test browser. Opsi yang lebih aman adalah mengarahkan Pimpinan ke halaman custom Pimpinan yang sudah memakai form eksplisit.

## 6. Temuan Dashboard

### P1 — W5 menggabungkan golongan dan tidak menampilkan I/a sampai IV/e

PRD meminta jumlah pegawai per golongan `I/a — IV/e`. Implementasi dashboard memotong nilai golongan menggunakan bagian sebelum karakter `/`, sehingga seluruh `III/a` sampai `III/d` digabung sebagai `III`.

Hasilnya, UI hanya menampilkan Golongan I, II, III, dan IV. Ini tidak memenuhi granularitas data yang diminta PRD.

**Rekomendasi:** gunakan kode golongan lengkap dari reference table atau nilai golongan penuh, tanpa `explode('/')`.

### P1 — W7 bukan tren historis pegawai yang sebenarnya

PRD mensyaratkan line chart jumlah pegawai aktif per bulan dari data historis selama 12 bulan terakhir.

Implementasi saat ini menggunakan kondisi status aktif **saat ini** serta `created_at <= akhir bulan`. Pegawai yang aktif pada masa lalu tetapi pensiun hari ini akan hilang dari titik masa lalu, sehingga grafik dapat berubah secara retrospektif dan tidak merepresentasikan sejarah sebenarnya.

**Rekomendasi:** gunakan riwayat status, snapshot bulanan, atau sumber histori lain yang disetujui. Jangan menyebut hasilnya tren historis sebelum status pegawai dapat direkonstruksi pada setiap bulan.

### P2 — KPI EWS Aktif maksimal hanya lima

Action mengambil seluruh EWS aktif, lalu menyimpan lima data pertama untuk tabel ringkas. Kartu KPI kemudian menggunakan jumlah data ringkas itu sebagai `EWS Aktif`.

Jika terdapat 23 alert aktif, kartu akan menampilkan angka 5 karena hanya lima baris yang dipratinjau.

**Rekomendasi:** kirim `totalEwsAktif` untuk KPI dan `ewsAktifTeratas` untuk tabel lima alert terdekat.

### P2 — W2 belum menampilkan golongan awal → golongan tujuan

PRD meminta informasi kenaikan pangkat berupa golongan saat ini menuju golongan tujuan. Dashboard saat ini hanya memetakan satu nilai golongan dari riwayat pangkat dan menampilkan satu kolom `Golongan`.

**Rekomendasi:** sediakan data pangkat sebelum dan sesudah kenaikan, misalnya `III/c → III/d`.

### P3 — KPI Total Pegawai Aktif memakai tautan dummy

Kartu W1 menggunakan `href="#"`. Kartu seperti ini membuat interaksi palsu dan tidak mengarahkan pengguna ke halaman yang bermanfaat.

**Rekomendasi:** hilangkan tautan bila kartu hanya informatif, atau arahkan ke daftar pegawai Pimpinan yang difilter aktif.

### Perlu keputusan produk — Link audit log Pimpinan

W6 pada PRD dan US-8.1 meminta link ke audit log detail. Implementasi menampilkan lima audit terbaru, tetapi sidebar secara eksplisit mengecualikan `audit-log` bagi Pimpinan.

Ini adalah konflik kebutuhan yang tidak boleh diselesaikan dengan asumsi. Diperlukan keputusan resmi:

1. Berikan audit log read-only kepada Pimpinan dengan masking data sensitif; atau
2. Tetapkan W6 hanya sebagai ringkasan lima aktivitas dan revisi kriteria dokumen agar tidak lagi mewajibkan link detail.

## 7. Temuan EWS dan Markup

### P2 — Pencarian EWS tidak berfungsi

Frontend EWS menampilkan input `Cari pegawai` dengan parameter `search`. Controller hanya memproses parameter `event` dan `status`, lalu meneruskan keduanya ke Action. Parameter pencarian tidak digunakan.

Akibatnya pengguna dapat mengisi kolom pencarian, tetapi hasil tabel tidak berubah.

**Rekomendasi:** validasi `search`, teruskan ke `ListActiveEwsAlertsAction`, filter nama/NIP di server, dan tambahkan test hasil pencarian.

### P2 — HTML EWS tidak valid

Terdapat paragraf `<p>` yang ditutup dengan `</span>` pada deskripsi threshold event dan alasan eligibility. Browser biasanya memperbaiki DOM secara otomatis, tetapi markup invalid dapat memengaruhi style, pembaca layar, dan perilaku antar-browser.

### P3 — `per_page` EWS tidak dibatasi

UI menawarkan nilai 10, 25, dan 50, tetapi controller menerima nilai query `per_page` apa pun tanpa whitelist. Nilai harus dibatasi ke pilihan yang tersedia dan halaman harus minimal 1.

## 8. Temuan Kualitas Kode dan Efisiensi

### 8.1 Query detail pegawai terlalu besar untuk tampilan yang dirender

Controller detail Pimpinan memuat riwayat pangkat, jabatan, KGB, disiplin, pendidikan, dokumen, keluarga, dan supervisor. Namun Blade detail hanya merender tab informasi, keluarga, dan supervisor.

Controller juga mengambil beberapa reference table, tetapi hanya mengirim variabel pegawai ke view. Hal ini menambah query dan memori tanpa nilai tampilan.

**Rekomendasi:** hapus eager loading/reference query yang tidak digunakan, atau tambahkan tab read-only yang memang membutuhkan data tersebut bila frasa “semua data pegawai” dimaksudkan sampai ke seluruh riwayat individual.

### 8.2 Paginasi laporan dilakukan setelah semua data dimuat

Pratinjau laporan cuti dan kepangkatan memuat seluruh Collection terlebih dahulu, lalu memotongnya dengan `forPage()` di memori. Ini dapat menjadi mahal pada data besar.

**Rekomendasi:** sediakan query pagination dari database untuk pratinjau; ekspor tetap dapat memakai streaming/chunking bila diperlukan.

### 8.3 Kalkulasi SVG tren panjang berada di Blade

Dashboard menghitung dimensi, titik, dan path SVG secara langsung di Blade. Perhitungan tersebut bersifat presentasi, bukan domain utama, tetapi ukurannya cukup besar sehingga menyulitkan pemeliharaan view.

**Rekomendasi:** pindahkan pembentukan data chart ke presenter/view model atau Blade component kecil.

### 8.4 Istilah status belum konsisten

Daftar cuti menggunakan label `Perlu Perubahan`, sedangkan dokumen menetapkan label keputusan resmi `Perubahan`. Alur tidak rusak, tetapi istilah perlu diseragamkan pada daftar, detail, timeline, notifikasi, dan audit.

## 9. Bukti Pengujian

Pengujian lokal yang berhasil dijalankan:

| Perintah/cakupan | Hasil |
|---|---|
| Semua 11 berkas test Pimpinan | 32 test lulus, tanpa kegagalan. |
| `EmployeeReportExportTest` | 3 test lulus, 37 assertion. |
| `git diff --check` | Tidak menemukan masalah whitespace. |

Test Pimpinan meliputi dashboard, daftar/detail pegawai, EWS, keputusan cuti, guard approval, dokumen cuti, laporan cuti, laporan kepangkatan, dan laporan custom.

### Batasan bukti test

- PHPUnit saat ini menggunakan SQLite in-memory, bukan PostgreSQL 17.
- Test frontend mayoritas berupa HTTP/Blade assertion, bukan test browser nyata.
- Karena belum ada E2E browser test, masalah tombol read-only, search EWS, dan modal Alpine yang tidak selesai tidak tertangkap otomatis.

### Catatan format worktree

`composer format:check` tidak lulus pada worktree saat audit dilakukan. Kegagalan hanya dilaporkan pada perubahan lokal yang bukan bagian analisis Pimpinan:

- `SIMPEG/bootstrap/app.php`
- `SIMPEG/test_show.php`

Tidak ada perubahan otomatis yang dilakukan terhadap file tersebut.

## 10. Urutan Perbaikan yang Disarankan

1. Jadikan Data Pegawai Pimpinan benar-benar read-only dari sisi UI.
2. Hubungkan sidebar Pimpinan ke jalur laporan Pimpinan dan satukan pengalaman laporan standard/custom.
3. Lengkapi filter laporan custom: unit kerja, jenis pegawai, jabatan, serta periode pensiun.
4. Hilangkan data kontak pribadi dan pegawai tidak aktif dari preview laporan standard Pimpinan.
5. Perbaiki metrik Dashboard W5, W7, dan total EWS W4.
6. Perbaiki search EWS, markup HTML invalid, serta validasi pagination.
7. Putuskan secara resmi akses audit detail Pimpinan sebelum mengubah W6.
8. Tambahkan regression test untuk setiap temuan, lalu jalankan pada PostgreSQL dan browser E2E.

## 11. Kesimpulan Akhir

Role Pimpinan sudah memiliki alur operasional inti yang baik, terutama pada keputusan cuti, otorisasi step approval, dokumen cuti, EWS dasar, dan laporan cuti/kepangkatan. Namun status keseluruhan frontend masih **⚠️ belum sepenuhnya sesuai dokumen**.

Prioritas utama adalah memperbaiki read-only UI pada Data Pegawai dan menyatukan jalur Laporan Pegawai Pimpinan. Setelah itu, akurasi dashboard dan fungsi EWS perlu diperbaiki agar data yang disajikan tidak menyesatkan pengguna.
