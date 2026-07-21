# Halaman dan Hak Akses Role Pegawai — SIMPEG Fase 1

| Field | Nilai |
|---|---|
| Status | Dokumen turunan kebutuhan produk |
| Role utama | `pegawai` |
| Acuan utama | PRD SIMPEG Fase 1 Core v1.3 dan User Stories SIMPEG Fase 1 |
| Bahasa antarmuka | Bahasa Indonesia |
| Batasan utama | Pegawai melihat data sendiri secara read-only, dapat mengajukan serta memantau cuti, dan menerima notifikasi. |

## Tujuan

Dokumen ini merangkum halaman, informasi, aksi yang diizinkan, dan batas akses untuk role utama `pegawai`. Dokumen ini adalah target produk Fase 1, bukan daftar route atau status implementasi saat ini.

## Prinsip Akses

Role `pegawai` memiliki akses dasar berikut:

- melihat data kepegawaian miliknya sendiri secara read-only;
- mengajukan cuti, melihat status, detail/timeline approval, dan saldo cuti sendiri;
- melihat EWS pribadi dan menerima notifikasi terkait EWS maupun cuti;
- tidak dapat melihat atau mengubah data pegawai lain;
- tidak dapat melakukan perubahan mandiri atas profil, keluarga, pendidikan, atau riwayat kepegawaian dalam Fase 1.

Kewenangan sebagai verifikator cuti bukan role baru. Pegawai yang ditunjuk pada approval chain dapat memperoleh akses kondisional untuk menindaklanjuti pengajuan yang tiba pada step dirinya.

## Struktur Navigasi

```text
Pegawai
├── Dashboard Pribadi
├── Profil Saya
├── Cuti
│   ├── Ajukan Cuti
│   ├── Daftar Cuti Saya
│   ├── Detail Cuti & Timeline Approval
│   └── Saldo Cuti
├── EWS Pribadi
├── Notifikasi
│   ├── Bell / dropdown notifikasi
│   └── Semua Notifikasi
├── Dokumen Cuti & Verifikasi QR
└── Pengajuan Menunggu Tindakan (kondisional sebagai verifikator)
```

## 1. Dashboard Pribadi

### Tujuan

Menjadi halaman pertama setelah pegawai login agar informasi kepegawaian penting milik sendiri dapat dilihat dengan cepat.

### Informasi yang ditampilkan

- Foto, nama, NIP, golongan, jabatan, dan unit kerja.
- Card saldo cuti tahunan.
- Daftar pengajuan cuti aktif beserta statusnya.
- EWS pribadi yang relevan.
- Lima notifikasi terakhir.
- Tampilan responsif untuk desktop, tablet, dan mobile.

### Aksi yang dapat dilakukan

- Membuka Profil Saya.
- Membuka saldo cuti.
- Membuka daftar atau detail pengajuan cuti aktif.
- Membuka peringatan EWS pribadi.
- Membuka notifikasi terkait.

### Batas akses

Data yang muncul hanya data pegawai yang sedang login.

## 2. Profil Saya

### Tujuan

Memungkinkan pegawai memastikan data kepegawaiannya sendiri lengkap dan benar tanpa mengubah data tersebut secara langsung.

### Informasi/tab yang ditampilkan

1. Profil dan kontak.
2. Data keluarga.
3. Riwayat kepangkatan.
4. Riwayat jabatan.
5. Riwayat KGB.
6. Riwayat hukuman disiplin.
7. Riwayat pendidikan.
8. Dokumen dan SK.
9. Data pengangkatan.
10. Tanggal kenaikan pangkat berikutnya, KGB berikutnya, dan tanggal pensiun.
11. Kepala bagian yang ditetapkan.
12. Saldo cuti tahun berjalan.

### Aksi yang dapat dilakukan

- Melihat seluruh data dan riwayat milik sendiri.
- Melihat saldo cuti serta tanggal kalkulasi kepegawaian.
- Melihat dokumen atau SK yang diizinkan untuk dirinya.

### Aksi yang tidak diizinkan

- Edit profil atau kontak.
- Tambah, edit, atau hapus keluarga dan pendidikan.
- Tambah, edit, atau hapus pangkat, jabatan, KGB, disiplin, atau dokumen.
- Mengubah flag kinerja baik, kepala bagian, atau approval chain.
- Mengakses profil pegawai lain.

## 3. Ajukan Cuti

### Tujuan

Memungkinkan pegawai membuat pengajuan cuti digital dan mengirimkannya ke approval chain yang berlaku.

### Field form

- Jenis cuti; pilihan disaring sesuai status PNS atau PPPK.
- Tanggal mulai.
- Tanggal selesai.
- Alasan cuti, wajib diisi.
- Lampiran opsional, maksimal 10 MB, format PDF/JPG/PNG.

### Perilaku dan validasi

- Menghitung hari kerja secara real-time.
- Tidak menghitung Sabtu, Minggu, hari libur nasional, dan cuti bersama.
- Menolak satu pengajuan yang melewati tahun kalender; cuti Desember–Januari harus dibuat sebagai dua pengajuan.
- Memvalidasi saldo untuk Cuti Tahunan; form tidak dapat disubmit jika saldo tidak cukup.
- Menolak pengajuan bila pegawai belum mempunyai approval chain aktif atau pihak pertama pada chain tidak valid.

### Hasil submit

- Status menjadi `Menunggu [nama step pertama]`.
- Notifikasi dikirim kepada pihak pertama pada approval chain melalui kanal yang dikonfigurasi.

## 4. Daftar Cuti Saya

### Tujuan

Memungkinkan pegawai melihat seluruh pengajuan cutinya, baik yang masih aktif maupun riwayat.

### Kolom minimal

- Jenis cuti.
- Tanggal mulai dan selesai.
- Jumlah hari kerja.
- Status.
- Tanggal pengajuan.

### Aksi yang dapat dilakukan

- Filter berdasarkan tahun dan status.
- Menggunakan pagination.
- Membuka baris pengajuan untuk melihat detail dan timeline approval.

### Label status resmi

- `Menunggu [step approval/verifikasi]`.
- `Disetujui`.
- `Perubahan`.
- `Ditangguhkan`.
- `Tidak Disetujui`.

Istilah formal `Ditolak` tidak digunakan untuk keputusan cuti.

## 5. Detail Cuti dan Timeline Approval

### Tujuan

Menunjukkan posisi pengajuan pada approval chain dan riwayat tindakan setiap approver/verifikator.

### Informasi yang ditampilkan

- Jenis cuti.
- Periode cuti dan jumlah hari kerja.
- Alasan dan lampiran.
- Status saat ini.
- Timeline vertikal berisi urutan step, nama approver/verifikator, peran dalam chain, aksi, waktu tindakan, dan keterangan.
- Step yang belum diproses ditampilkan sebagai `Menunggu`.

### Aksi yang dapat dilakukan

- Melihat proses approval milik sendiri.
- Melihat lampiran atau dokumen yang diizinkan.
- Melihat keputusan serta alasan/keterangan dari approver.
- Mengakses hasil formulir cuti atau verifikasi QR setelah proses selesai, bila tautan tersedia pada detail.

Pegawai tidak dapat mengambil keputusan atas pengajuan cutinya sendiri.

## 6. Saldo Cuti

### Tujuan

Menampilkan hak dan pemakaian cuti pribadi secara transparan.

### Informasi yang ditampilkan

- Jatah dasar tahun berjalan: 12 hari.
- Carry-over dari N-1, maksimal 6 hari.
- Hak tambahan bila memenuhi aturan tidak mengambil cuti dua tahun berturut-turut.
- Total tersedia.
- Jumlah hari terpakai.
- Sisa cuti.
- Riwayat penggunaan cuti tahun berjalan, N-1, dan N-2 yang memengaruhi carry-over.

### Perilaku

- Saldo diperbarui setelah keputusan final `Disetujui` untuk Cuti Tahunan.
- Keputusan `Perubahan`, `Ditangguhkan`, atau `Tidak Disetujui` tidak mengurangi saldo.
- Pegawai hanya melihat saldo sendiri dan tidak dapat mengoreksi saldo.

## 7. EWS Pribadi / Peringatan Penting

### Bentuk permukaan

EWS pribadi dapat ditampilkan sebagai section di Dashboard Pribadi atau Profil Saya; dokumen tidak mewajibkan menu sidebar terpisah.

### Informasi yang ditampilkan

- Jenis event.
- Tanggal target.
- Sisa hari.
- Status eligibility.
- Hanya alert milik pegawai yang sedang login.

### Contoh event

- Kenaikan pangkat.
- KGB.
- Pensiun.
- Satyalancana.
- Kontrak PPPK jika relevan dengan status pegawai.

### Batas akses

Pegawai melihat status dan tenggat dirinya; pegawai tidak menandai alert sebagai `ditangani` atau `tidak perlu`.

## 8. Bell Notifikasi di Navbar

### Bentuk permukaan

Bell notifikasi tersedia pada navbar, bukan halaman mandiri.

### Informasi dan aksi

- Badge jumlah notifikasi belum dibaca.
- Dropdown berisi 10 notifikasi terbaru.
- Judul, waktu relatif, dan indikator sudah/belum dibaca pada tiap item.
- Klik item menandai notifikasi dibaca lalu mengarahkan ke halaman terkait.
- Tautan menuju halaman Semua Notifikasi.
- Badge diperbarui tanpa refresh penuh halaman.

Notifikasi yang relevan untuk pegawai mencakup EWS pribadi dan perubahan status cuti.

## 9. Semua Notifikasi

### Tujuan

Menjadi riwayat seluruh notifikasi milik user yang sedang login.

### Informasi dan aksi

- Daftar notifikasi pribadi, urutan terbaru di atas.
- Indikator visual notifikasi belum dibaca.
- Pagination.
- Klik notifikasi untuk membuka halaman terkait.
- Tombol per-item `Tandai sudah dibaca`.
- Tombol `Tandai Semua Sudah Dibaca`.
- Badge navbar langsung berkurang setelah tindakan.

## 10. Dokumen Cuti dan Verifikasi QR

### Tujuan

Setelah cuti selesai diproses, sistem menghasilkan formulir cuti resmi sesuai format LLDIKTI dengan QR Code verifikasi.

### Halaman verifikasi QR minimal menampilkan

- Nama pegawai.
- Jenis dan tanggal cuti.
- Status keputusan.
- Tanggal keputusan.
- Pejabat final/approver.
- Identitas LLDIKTI Wilayah XVI.

### Catatan navigasi

Dokumen mewajibkan hasil formulir dan halaman verifikasi QR, tetapi tidak secara eksplisit mewajibkan menu sidebar khusus bernama `Dokumen Cuti`. Tautan lihat/unduh dapat ditempatkan di Detail Cuti.

## 11. Pengajuan Menunggu Tindakan — Akses Kondisional Verifikator

### Kapan halaman ini tersedia

Halaman ini hanya muncul bila pegawai ditunjuk sebagai approver atau verifikator pada approval chain. Role utama tetap dapat `pegawai`; kewenangan muncul dari konfigurasi chain, bukan dari role baru.

### Informasi yang dapat dilihat

- Pengajuan yang tiba pada step dirinya.
- Detail cuti, lampiran, dan data yang diperlukan untuk verifikasi.
- Saldo serta riwayat cuti pemohon dalam batas yang diperlukan untuk verifikasi.

### Aksi yang dapat dilakukan

- `Disetujui`.
- `Perubahan`.
- `Ditangguhkan`.
- `Tidak Disetujui`.

Semua keputusan selain `Disetujui` wajib menyertakan keterangan. Sistem mengirim notifikasi sesuai aksi dan meneruskan pengajuan ke step berikutnya sesuai chain.

### Batas akses

- Hanya melihat pengajuan pada step dirinya.
- Tidak dapat melihat seluruh pengajuan instansi.
- Tidak dapat mengelola approval chain.
- Tidak ada tombol atau label formal `Tolak`.

## Halaman Umum, Bukan Menu Khusus Pegawai

| Permukaan | Ketentuan |
|---|---|
| Login Keycloak SSO | Semua user login melalui Keycloak. Tidak ada login manual SIMPEG. |
| Logout | Tersedia dari topbar setelah login. |
| Session timeout | Sesi yang tidak aktif diakhiri secara aman dan user diarahkan ke alur login. |
| Akses ditolak | User tanpa role SIMPEG valid menerima `403 Access Forbidden`. |
| Email notifikasi | Kanal pengiriman, bukan halaman. Email mengarah ke halaman terkait di SIMPEG. |

## Halaman dan Aksi yang Tidak Tersedia untuk Pegawai Biasa

Pegawai biasa tidak boleh memiliki menu atau aksi untuk:

- daftar seluruh pegawai atau detail pegawai lain;
- tambah, edit, nonaktifkan, restore, atau hapus pegawai;
- mengubah profil, keluarga, pendidikan, pangkat, jabatan, KGB, disiplin, dokumen, status, atau unit kerja sendiri;
- import Excel/CSV;
- koreksi atau mengelola saldo cuti seluruh pegawai;
- konfigurasi approval chain, supervisor, hari libur, EWS, atau reference table;
- audit log;
- laporan dan export pegawai/cuti;
- EWS seluruh instansi;
- keputusan cuti pegawai lain, kecuali bila ditunjuk pada approval chain.

## Referensi Dokumen

- `PRD-SIMPEG-Fase1-Core.md` §4.2 — Definisi Role dan Hak Akses, khususnya role Pegawai dan Verifikator Cuti.
- `PRD-SIMPEG-Fase1-Core.md` §9 — Manajemen Cuti, saldo, status, dokumen, dan QR verification.
- `PRD-SIMPEG-Fase1-Core.md` §13.2–13.4 — Dashboard per role dan Dashboard Pegawai.
- `User-Stories-SIMPEG-Fase1.md` US-2.5 — Lihat Profil Sendiri.
- `User-Stories-SIMPEG-Fase1.md` US-4.1, US-4.2, US-4.3, dan US-4.7 — Pengajuan, daftar, saldo, serta timeline cuti.
- `User-Stories-SIMPEG-Fase1.md` US-5.3 — EWS Pribadi.
- `User-Stories-SIMPEG-Fase1.md` US-6.1, US-6.2, dan US-6.4 — Notifikasi in-app serta tandai dibaca.
- `User-Stories-SIMPEG-Fase1.md` US-8.2 — Dashboard Pegawai.
