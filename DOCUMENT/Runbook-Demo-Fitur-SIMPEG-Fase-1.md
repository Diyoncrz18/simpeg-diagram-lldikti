# Runbook Demo Fitur SIMPEG Fase 1

**Jenis dokumen:** Panduan Operasional Demo  
**Versi:** 1.0  
**Tanggal:** 10 Agustus 2026  
**Target pembaca:** Presenter, tim pengembang, QA, dan pendamping demo LLDIKTI Wilayah XVI  
**Status:** Siap digunakan setelah checklist preflight dinyatakan lulus  
**Kedudukan dokumen:** Panduan ini menjelaskan urutan dan cara mendemonstrasikan fitur. Dokumen ini tidak menggantikan PRD, Panduan Penulisan Kode, User Stories, atau keputusan produk yang telah disetujui.

---

## 1. Tujuan Dokumen

Dokumen ini menjadi panduan lengkap untuk melakukan demonstrasi SIMPEG Fase 1 dari awal sampai akhir. Setiap bagian menjelaskan:

1. Role dan akun yang harus digunakan.
2. Menu, halaman, dan URL yang harus dibuka.
3. Data prasyarat yang harus tersedia.
4. Tombol atau tindakan yang harus dilakukan.
5. Hasil yang harus muncul.
6. Waktu yang tepat untuk berpindah role.
7. Hal yang harus dilakukan oleh role berikutnya.
8. Bukti yang dapat ditunjukkan melalui notifikasi, saldo, laporan, dan Audit Log.
9. Penanganan jika hasil demo tidak sesuai harapan.

Demo sebaiknya disampaikan sebagai satu cerita lintas role, bukan sebagai kumpulan halaman terpisah:

```text
Super Admin menyiapkan akses dan konfigurasi
    ↓
Admin Kepegawaian mengelola data pegawai, riwayat, saldo, dan import
    ↓
Pegawai melihat profil dan mengajukan cuti
    ↓
Kepala Bagian memeriksa dan menyetujui tahap pertama
    ↓
Pimpinan memberikan keputusan akhir
    ↓
Pegawai melihat hasil, saldo, notifikasi, dan PDF
    ↓
Admin menindaklanjuti EWS dan membuat laporan
    ↓
Super Admin memeriksa Audit Log dan pemulihan data
```

---

## 2. Akun dan Role Demo

Akun berikut hanya untuk environment `local` atau `testing`. Jangan memperkenalkan akun dan login demo sebagai jalur login produksi.

| Role | Username | Password | Fungsi pada demo |
|---|---|---|---|
| Super Admin | `demo-klabat` | `demo-klabat` | Kelola akses, RBAC, data master, approval chain, EWS config, audit, dan backup |
| Admin Kepegawaian | `demo-klabat-kepeg` | `demo-klabat-kepeg` | Data pegawai, import, riwayat, saldo cuti, EWS, dan laporan |
| Kepala Bagian | `demo-klabat-kabag` | `demo-klabat-kabag` | Daftar bawahan, cuti bawahan, dan EWS bawahan |
| Pimpinan | `demo-klabat-pimpinan` | `demo-klabat-pimpinan` | Persetujuan akhir cuti, dashboard pimpinan, EWS, dan laporan |
| Pegawai | `demo-klabat-pegawai` | `demo-klabat-pegawai` | Profil sendiri, pengajuan cuti, saldo, notifikasi, dan EWS pribadi |

### 2.1 Cara login

- Jalur normal/SSO: buka `/login`, lalu masuk melalui Keycloak.
- Environment lokal yang mengaktifkan login demo: buka `/dev-login`.
- Produksi wajib menggunakan Keycloak SSO. Jangan menyatakan bahwa `/dev-login` merupakan fitur produksi.

### 2.2 Cara aman berpindah role

Gunakan profil browser atau jendela terpisah:

- `SA — Super Admin`
- `ADM — Admin Kepegawaian`
- `PEG — Pegawai`
- `KABAG — Kepala Bagian`
- `PIM — Pimpinan`

Jika hanya menggunakan satu browser, selalu logout sepenuhnya sebelum login dengan akun lain. Jangan mengandalkan tombol Back karena halaman sebelumnya dapat masih menampilkan state sesi lama.

---

## 3. Persiapan Wajib Sebelum Demo

### 3.1 Environment

Gunakan staging atau salinan database demo. Pastikan:

- Aplikasi, PostgreSQL, Keycloak, queue, scheduler, dan storage berjalan.
- Mailpit atau inbox testing tersedia jika email akan didemonstrasikan.
- Halaman utama dapat dibuka tanpa error browser console.
- Semua data dummy diberi nama yang mudah dikenali.
- NIK, No. KK, token, kredensial, atau dokumen asli pegawai tidak ditampilkan.

### 3.2 Pegawai utama untuk cerita demo

Employee yang terhubung ke `demo-klabat-pegawai` harus:

- berstatus aktif;
- memiliki role internal `pegawai`;
- memiliki Kepala Bagian yang merupakan employee dari `demo-klabat-kabag`;
- memiliki saldo cuti tahunan, misalnya 12 hari;
- memiliki email testing;
- memiliki unit kerja yang berada dalam cakupan Kepala Bagian demo; dan
- mempunyai profil yang cukup lengkap untuk mengajukan cuti.

Jangan memakai akun Super Admin, Admin Kepegawaian, Kepala Bagian, atau Pimpinan sebagai data yang akan dihapus, dipensiunkan, atau dimutasi secara berisiko.

### 3.3 Approval chain wajib dikonfigurasi sebelum pengajuan

Role `pimpinan` tidak otomatis dapat memutus semua pengajuan. Employee akun Pimpinan harus tercatat sebagai approver aktif pada snapshot chain pengajuan.

Login sebagai Super Admin:

1. Buka **Cuti → Konfigurasi Approval Cuti** atau `/cuti/konfigurasi-approval`.
2. Pada **PYBMC Global**, pilih employee milik `demo-klabat-pimpinan`.
3. Isi alasan perubahan, misalnya `Konfigurasi approval untuk demo LLDIKTI`.
4. Klik **Simpan PYBMC Global**.
5. Pada **Chain Approval Pegawai**, cari employee `demo-klabat-pegawai`.
6. Pastikan Kepala Bagian adalah employee milik `demo-klabat-kabag`.
7. Pastikan urutan chain:
   - Tahap 1: Kepala Bagian — employee `demo-klabat-kabag`.
   - Tahap final: PYBMC/Pimpinan — employee `demo-klabat-pimpinan`.
8. Isi tanggal efektif yang tidak lebih baru dari tanggal demo.
9. Isi alasan perubahan.
10. Klik **Simpan Chain Pegawai**.
11. Muat kembali pegawai tersebut dan periksa ulang urutannya.

Approval chain disalin menjadi snapshot saat pengajuan dibuat. Memperbaiki konfigurasi setelah pengajuan dibuat tidak menjamin pengajuan lama ikut berubah. Jika snapshot lama salah, perbaiki konfigurasi lalu buat pengajuan baru.

### 3.4 Saldo cuti

Login sebagai Admin Kepegawaian:

1. Buka **Cuti → Administrasi Saldo Cuti** atau `/cuti/administrasi-saldo`.
2. Cari employee `demo-klabat-pegawai`.
3. Jika saldo tahun berjalan belum ada, buka **Pendaftaran Saldo Awal**.
4. Contoh data:
   - Sisa N-2: `0`.
   - Sisa N-1: `0`.
   - Sisa Tahun Berjalan: `12`.
   - Alasan: `Pendaftaran saldo awal untuk demo LLDIKTI`.
5. Simpan dan pastikan angka tampil pada ringkasan dan ledger.

Jika saldo sudah tersedia, jangan mendaftarkan ulang. Cukup tunjukkan nilai awalnya.

### 3.5 Tanggal cuti

Pilih dua hari kerja berurutan yang:

- berada setelah tanggal demo;
- tidak melewati tahun kalender;
- bukan Sabtu/Minggu;
- tidak terdaftar sebagai hari libur; dan
- tidak tumpang tindih dengan pengajuan lain.

Contoh untuk demo tahun 2026: 9–10 November 2026. Sistem tetap menjadi sumber kebenaran jumlah hari kerja.

### 3.6 File contoh

Siapkan:

- `lampiran-cuti-demo.pdf`, maksimal 10 MB;
- `sk-pangkat-demo.pdf`, maksimal 10 MB;
- `import-pegawai-demo.xlsx`, dibuat dari template resmi aplikasi;
- `import-pegawai-duplikat.xlsx`, untuk demonstrasi validasi negatif; dan
- foto dummy jika ingin menunjukkan unggah foto.

---

## 4. Rundown dan Durasi

| Bagian | Durasi |
|---|---:|
| Login, SSO, dan pemisahan role | 5 menit |
| Super Admin: akses dan konfigurasi | 8 menit |
| Admin: data pegawai dan riwayat | 10 menit |
| Import pegawai | 7 menit |
| Pegawai: profil dan pengajuan cuti | 8 menit |
| Kepala Bagian: approval tahap pertama | 6 menit |
| Pimpinan: keputusan akhir | 6 menit |
| Pegawai: hasil, notifikasi, saldo, PDF | 6 menit |
| EWS lintas role | 10 menit |
| Dashboard dan laporan | 8 menit |
| Audit dan restore | 6 menit |

Jika waktu terbatas, prioritaskan login/role, data pegawai, cuti end-to-end, notifikasi, EWS, dashboard Pimpinan, laporan, dan Audit Log.

---

## 5. Demo Login, SSO, dan Role Internal

### 5.1 Super Admin

**Role:** Super Admin  
**Akun:** `demo-klabat`  
**Halaman:** `/login` atau `/dev-login` pada local/testing

Langkah:

1. Login.
2. Pastikan diarahkan ke `/dashboard`.
3. Tunjukkan nama dan role **Super Admin**.
4. Tunjukkan menu administrasi: Kelola Akses User, Role & Permission, Data Master, Hari Libur, Pengaturan Sistem, dan Audit Log.

Hasil yang benar:

- akun masuk ke dashboard admin;
- menu sensitif terlihat sesuai permission; dan
- tidak ada pemilihan role bebas.

Narasi:

> Keycloak memastikan identitas pengguna. Setelah identitas berhasil diautentikasi, SIMPEG membaca role dan permission dari database internal untuk menentukan halaman dan aksi yang boleh digunakan.

### 5.2 Pembuktian role Pimpinan

Pindah ke jendela Pimpinan dan login sebagai `demo-klabat-pimpinan`.

1. Pastikan diarahkan ke `/pimpinan/dashboard`.
2. Tunjukkan menu Data Pegawai, Persetujuan Cuti, EWS, Laporan, dan Notifikasi.
3. Pastikan menu Kelola Akses User, RBAC, dan Data Master tidak terlihat.
4. Opsional: buka `/user-management`. Hasil yang benar adalah penolakan `403`.

Jika Pimpinan masuk ke halaman Super Admin, periksa username, `keycloak_username`, role internal, session browser, lalu logout dan login kembali.

---

## 6. Demo Kelola Akses User dan RBAC

### 6.1 Kelola Akses User

**Role:** Super Admin  
**Menu:** Administrasi Sistem → Kelola Akses User  
**URL:** `/user-management`

1. Cari `demo-klabat-pimpinan`.
2. Buka aksi edit.
3. Pada modal **Edit Otorisasi & SSO**, tunjukkan username Keycloak, employee terhubung, role internal, dan status pemetaan.
4. Jangan mengubah role akun utama saat demo.
5. Jelaskan bahwa perubahan role berlaku setelah login berikutnya.

Hasil yang dijelaskan:

- Keycloak menentukan identitas;
- SIMPEG menentukan kewenangan;
- akun harus terhubung ke employee untuk fitur personal; dan
- akun tanpa role valid harus ditolak secara fail-closed.

### 6.2 Role & Permission

**Menu:** Administrasi Sistem → Role & Permission  
**URL:** `/rbac`

1. Tunjukkan matriks role dan permission.
2. Jelaskan contoh permission Data Pegawai, Riwayat, EWS Aktif, Audit Log, Konfigurasi EWS, dan Penyesuaian Saldo Cuti.
3. Jelaskan bahwa dua role dapat melihat jenis halaman serupa tetapi memiliki aksi berbeda.
4. Jangan mengubah permission role utama pada demo live.

---

## 7. Demo Data Master dan Hari Libur

### 7.1 Data Master

**Role:** Super Admin  
**Menu:** Administrasi Sistem → Data Master  
**URL:** `/data-master`

1. Tunjukkan tab Golongan Pangkat, Jenis Jabatan, Jabatan, Unit Kerja, Status Pegawai, dan Jenjang Pendidikan.
2. Buka tab **Jabatan**.
3. Tunjukkan nama, jenis, status aktif, dan aksi tambah/edit/nonaktifkan.
4. Jelaskan bahwa referensi yang sudah dipakai tidak boleh dihapus sembarangan; data tersebut dinonaktifkan agar histori tetap valid.

Jika ingin menambah data, gunakan nama `[DEMO] Jabatan Uji LLDIKTI`, simpan, buktikan di tabel, lalu bersihkan sesuai prosedur staging setelah demo.

### 7.2 Hari Libur

**Menu:** Administrasi Sistem → Hari Libur  
**URL:** `/hari-libur`

1. Tunjukkan daftar hari libur.
2. Jelaskan dampaknya pada kalkulasi hari kerja cuti.
3. Buka **Tambah Hari Libur Baru**.
4. Tunjukkan field Tanggal, Nama Hari Libur, dan Jenis Libur.
5. Jika melakukan simulasi, gunakan nama `[DEMO] Hari Libur Uji` dan tanggal yang tidak dipakai pengajuan utama.
6. Simpan dan tunjukkan perubahan.
7. Nanti buktikan pada Audit Log.

---

## 8. Demo Data Pegawai dan Riwayat

### 8.1 Daftar pegawai

**Role:** Admin Kepegawaian  
**Menu:** Kepegawaian → Data Pegawai  
**URL:** `/pegawai`

1. Login sebagai `demo-klabat-kepeg`.
2. Tunjukkan pencarian, filter, sorting, pagination, status, jabatan/unit, dan golongan/jenis pegawai.
3. Cari employee `demo-klabat-pegawai`.
4. Buka detail.

Jelaskan bahwa daftar menggunakan real data backend dan cakupan aksi dibatasi permission.

### 8.2 Detail pegawai

Pada **Detail Pegawai**, tunjukkan tab:

- Profil/Data Utama;
- Keluarga;
- Kepangkatan;
- Jabatan;
- KGB;
- Hukuman Disiplin;
- Pendidikan; dan
- Dokumen/SK.

Narasi:

> Data utama menggambarkan keadaan pegawai sekarang. Riwayat pangkat, jabatan, dan KGB bersifat append-only: sistem menambah riwayat baru dan memperbarui snapshot terbaru tanpa menghapus histori lama.

Jangan membacakan NIK atau No. KK.

### 8.3 Edit data utama

1. Klik **Edit Pegawai**.
2. Tunjukkan bagian identitas, status kepegawaian, jabatan/unit, kontak, data pensiun, Kepala Bagian, dan dokumen.
3. Jika menyimpan, ubah hanya data dummy non-sensitif.
4. Simpan dan periksa kembali detail.

### 8.4 Tambah riwayat pangkat

Gunakan employee staging khusus:

1. Buka **Tambah Riwayat Kepangkatan**.
2. Pilih golongan baru.
3. Nomor SK: `SK-DEMO-PANGKAT-001`.
4. Isi Tanggal SK.
5. Jika demo 10 Agustus 2026 dan masa pangkat 4 tahun, gunakan TMT `30 Agustus 2022` agar target berikutnya `30 Agustus 2026`.
6. Unggah `sk-pangkat-demo.pdf`.
7. Simpan.

Hasil:

- riwayat lama tetap ada;
- riwayat baru ditambahkan;
- snapshot pangkat diperbarui;
- target EWS dihitung ulang;
- SK masuk ke arsip; dan
- audit CREATE tercatat.

### 8.5 Riwayat lainnya

Tunjukkan tanpa harus memutasi seluruhnya:

- Riwayat Jabatan: penugasan berdasarkan SK.
- Riwayat KGB: TMT, gaji pokok, nomor/tanggal SK, dan file.
- Hukuman Disiplin: memengaruhi kelayakan EWS pangkat.
- Keluarga: anggota keluarga pegawai.
- Pendidikan: histori pendidikan formal.
- Dokumen & SK: arsip lihat/unduh sesuai kewenangan.

---

## 9. Demo Import Pegawai

**Role:** Admin Kepegawaian atau Super Admin dengan permission import  
**URL:** `/pegawai/import-data`

### 9.1 Data uji

Gunakan template resmi aplikasi. File positif berisi satu pegawai baru dengan NIP dan email unik. File negatif terpisah berisi NIP yang sudah ada atau dua baris dengan NIP sama.

### 9.2 Langkah

1. Buka **Import Data Pegawai**.
2. Pada **1. Download Template Import Pegawai**, klik **Unduh Template Utama Pegawai**.
3. Pada **2. Unggah Berkas Pegawai**, unggah file Excel/CSV.
4. Periksa **Pemetaan Kolom (Excel/CSV → SIMPEG)**.
5. Petakan kolom yang sah; abaikan kolom yang memang tidak diperlukan.
6. Buka preview dan periksa nilai per baris.
7. Jika tersedia, edit satu cell dummy untuk menunjukkan koreksi sebelum import.
8. Klik **Validasi Ulang**.
9. Tunjukkan **Hasil Validasi**: valid, error, skipped, dan alasannya.
10. Eksekusi file positif.
11. Tunggu **Proses Impor Selesai!**.
12. Buka **Laporan Hasil Import**.
13. Kembali ke `/pegawai` dan cari pegawai yang diimport.

Hasil:

- data valid masuk;
- NIP existing tidak membuat duplikat;
- duplikat dalam file dan email duplikat menghasilkan error;
- laporan membedakan imported, skipped, dan failed; dan
- aktivitas tercatat pada Audit Log.

Batasan wajib disampaikan:

> Import hanya untuk Data Utama Pegawai. Import tidak membuat histori pangkat, jabatan, atau KGB dan tidak memicu TMT/EWS. Histori resmi ditambahkan melalui fitur riwayat berdasarkan dokumen SK.

---

## 10. Demo Profil Pribadi Pegawai

**Role:** Pegawai  
**Akun:** `demo-klabat-pegawai`

### 10.1 Dashboard

**URL:** `/dashboard`

Tunjukkan identitas ringkas, saldo cuti, pengajuan aktif, EWS pribadi, dan shortcut profil/cuti.

### 10.2 Profil Saya

**URL:** `/dashboard/profil`

Tunjukkan data utama, jabatan/unit, informasi kepegawaian, keluarga, histori yang diizinkan, saldo, dan EWS pribadi. Fokuskan pada kemampuan melihat data sendiri; jangan menjanjikan employee self-service editing tanpa workflow persetujuan.

Hasil yang dijelaskan:

- Pegawai hanya melihat data sendiri;
- data sensitif dibatasi; dan
- data berasal dari pengelolaan Admin Kepegawaian.

---

## 11. Demo Administrasi Saldo Cuti

**Role:** Admin Kepegawaian  
**Menu:** Cuti → Administrasi Saldo Cuti  
**URL:** `/cuti/administrasi-saldo`

1. Cari employee demo.
2. Tunjukkan saldo aktual, terpakai, dialokasikan, dapat diajukan, N-2/N-1, ledger, dan status rollover.
3. Jika saldo belum ada, gunakan **Pendaftaran Saldo Awal** dengan tahun berjalan 12.

Untuk demonstrasi koreksi di staging:

1. Buka **Koreksi Saldo**.
2. Pilih bucket.
3. Isi `+1` atau `-1`.
4. Isi alasan `Koreksi administrasi untuk demonstrasi`.
5. Simpan.
6. Tunjukkan transaksi baru pada ledger.

Koreksi harus selalu memiliki alasan dan dapat ditelusuri melalui audit.

---

## 12. Demo Utama: Cuti End-to-End

Alur utama:

```text
Pegawai mengajukan
→ Kepala Bagian menyetujui tahap pertama
→ Pimpinan memberikan keputusan akhir
→ Pegawai melihat hasil
→ Admin membuktikan perubahan saldo, rekap, laporan, dan audit
```

### 12.1 Pegawai mengajukan

**Role:** Pegawai  
**Daftar:** `/dashboard/cuti`  
**Form:** `/dashboard/cuti/create`

1. Buka **Pengajuan Cuti** dan buat pengajuan baru.
2. Pastikan halaman **Form Pengajuan Cuti**.
3. Tunjukkan Tahun Saldo, Saldo Tersedia Aktual, Masih Dapat Diajukan, dan tahapan persetujuan.
4. Pastikan chain menampilkan Kepala Bagian dan PYBMC/Pimpinan.
5. Isi:
   - Jenis Cuti: **Cuti Tahunan**.
   - Mulai: `9 November 2026`.
   - Selesai: `10 November 2026`.
   - Alasan: `Keperluan keluarga`.
   - Alamat: `Manado, Sulawesi Utara`.
   - Telepon: `081234567890`.
   - Lampiran: opsional.
6. Tunggu kalkulasi **Jumlah Hari Kerja**.
7. Pastikan saldo mencukupi.
8. Klik **Kirim Pengajuan**.

Hasil:

- status **Menunggu Approval**;
- tahap Kepala Bagian aktif;
- saldo belum terpakai final;
- jumlah hari dialokasikan agar tidak dipakai ganda;
- Kepala Bagian menerima notifikasi; dan
- audit pengajuan dibuat.

Contoh saldo untuk pengajuan dua hari:

| Kondisi | Saldo aktual | Terpakai final | Dialokasikan | Dapat diajukan |
|---|---:|---:|---:|---:|
| Sebelum pengajuan | 12 | 0 | 0 | 12 |
| Menunggu approval | 12 | 0 | 2 | 10 |
| Final Disetujui | 10 | 2 | 0 | 10 |

### 12.2 Kepala Bagian memeriksa

**Pindah role:** Kepala Bagian  
**Akun:** `demo-klabat-kabag`  
**Menu:** Cuti → Cuti Bawahan  
**URL:** `/kepala-bagian/cuti`

1. Cari pengajuan berdasarkan nama, periode, status, atau alasan.
2. Buka **Detail Pengajuan Cuti**.
3. Tunjukkan data pegawai, jenis, periode, hari kerja, alasan, lampiran, saldo, alur persetujuan, dan riwayat tindakan resmi.
4. Pastikan Kepala Bagian yang login adalah approver aktif.
5. Pilih **Disetujui**.
6. Pada modal konfirmasi, klik **Ya, Setujui**.

Hasil:

- tahap Kepala Bagian disetujui;
- pengajuan belum final;
- tahap aktif berpindah ke Pimpinan;
- Pimpinan menerima notifikasi;
- saldo belum dipotong final; dan
- audit keputusan dibuat.

### 12.3 Pimpinan memberikan keputusan akhir

**Pindah role:** Pimpinan  
**Akun:** `demo-klabat-pimpinan`  
**Menu:** Cuti → Persetujuan Cuti  
**URL:** `/pimpinan/cuti`

1. Buka **Persetujuan Cuti**.
2. Periksa ringkasan **Menunggu Keputusan Anda**.
3. Cari dan buka pengajuan demo.
4. Tunjukkan data, periode, saldo, approval Kepala Bagian, timeline, riwayat resmi, dan lampiran.
5. Pastikan tombol keputusan tersedia.
6. Pilih **Disetujui** dan klik **Ya, Setujui**.

Hasil:

- status akhir **Disetujui**;
- seluruh tahap selesai;
- saldo dipotong sesuai hari kerja;
- formulir/PDF resmi tersedia;
- Pegawai menerima notifikasi; dan
- audit keputusan final dibuat.

### 12.4 Pegawai melihat hasil

**Pindah role:** Pegawai

#### Notifikasi

**URL:** `/notifications`

1. Cari notifikasi hasil persetujuan.
2. Klik dan pastikan diarahkan ke pengajuan yang benar.
3. Tunjukkan status sudah dibaca dan penurunan badge unread.

#### Riwayat cuti

**URL:** `/dashboard/cuti`

1. Cari pengajuan.
2. Pastikan status **Disetujui**.
3. Buka detail dan tunjukkan timeline kedua approver serta riwayat resmi.
4. Klik **Unduh Formulir Cuti (PDF)**.

#### Saldo Cuti Saya

**URL:** `/dashboard/cuti/saldo`

Tunjukkan saldo awal 12, penggunaan 2, dan sisa 10.

#### Verifikasi PDF

Jika PDF memiliki QR/token, buka `/cuti/verifikasi/{token}` di incognito. Tunjukkan bahwa dokumen valid tanpa mengekspos data sensitif yang tidak diperlukan.

### 12.5 Admin membuktikan perubahan

**Pindah role:** Admin Kepegawaian

1. Buka **Rekap Cuti** di `/cuti/rekap`, cari pengajuan, dan cocokkan jumlah hari/status.
2. Buka `/cuti/administrasi-saldo`, cari Pegawai, dan tunjukkan ledger pemakaian dua hari serta sisa 10.
3. Buka `/cuti/laporan`, gunakan periode November 2026, dan pastikan pengajuan muncul.

---

## 13. Cabang Keputusan Cuti

Gunakan pengajuan tambahan. Jangan mengubah pengajuan utama yang sudah final.

Label keputusan resmi harus tepat: **Disetujui**, **Perubahan**, **Ditangguhkan**, dan **Tidak Disetujui**. Jangan menggunakan istilah `Ditolak`.

### 13.1 Perubahan

1. Approver aktif membuka detail.
2. Pilih **Perubahan**.
3. Isi alasan wajib: `Tanggal cuti perlu disesuaikan dengan jadwal layanan unit.`
4. Simpan.

Hasil: status Perubahan, saldo tidak dipotong, Pegawai menerima notifikasi.

Pindah ke Pegawai:

1. Buka detail.
2. Gunakan **Kirim Ulang Perubahan** atau **Perbaiki dan Ajukan Kembali**.
3. Ubah tanggal/alasan/kontak/lampiran.
4. Klik **Kirim Ulang Pengajuan**.
5. Sistem menghitung ulang hari kerja dan saldo.

### 13.2 Ditangguhkan

1. Approver memilih **Ditangguhkan**.
2. Isi alasan: `Pelaksanaan cuti ditunda sampai kegiatan layanan selesai.`
3. Simpan.

Hasil: saldo tidak dipotong final, Pegawai diberi tahu, dan catatan tercatat.

### 13.3 Ditangguhkan karena Tugas Dinas

Gunakan hanya pada skenario Cuti Tahunan yang disiapkan. Tunjukkan bahwa label status eksplisit, alasan dicatat, hak terkait dapat dilindungi untuk rollover, dan saldo tidak dianggap terpakai tanpa keputusan final. Lebih aman memakai fixture karena skenario ini berdampak pada rollover.

### 13.4 Tidak Disetujui

1. Pilih **Tidak Disetujui**.
2. Isi alasan: `Periode tersebut bertepatan dengan kebutuhan layanan unit.`
3. Simpan.

Hasil: status final Tidak Disetujui, tahap berikutnya dilewati, saldo tidak dipotong, alokasi dilepas, Pegawai menerima alasan, dan audit tercatat.

---

## 14. Demo Notifikasi

**URL semua role:** `/notifications`

### 14.1 Alur notifikasi cuti

1. Pegawai mengirim pengajuan → Kepala Bagian menerima notifikasi.
2. Kepala Bagian menyetujui → Pimpinan menerima notifikasi.
3. Pimpinan memutus → Pegawai menerima notifikasi hasil.

Pada setiap role:

1. Buka Notifikasi.
2. Cari event terkait.
3. Klik notifikasi.
4. Pastikan redirect ke record yang benar.
5. Tandai dibaca dan periksa badge unread.

### 14.2 Email

Jika didemonstrasikan:

1. Pastikan channel email dan kebijakan event aktif.
2. Pastikan queue worker berjalan.
3. Buka Mailpit/inbox testing.
4. Tunjukkan subjek dan penerima tanpa memperlihatkan kredensial SMTP.

### 14.3 Channel Notifikasi

**Role:** Super Admin  
**URL:** `/data-master/channel-notifikasi`

Tunjukkan channel In-App, Email, status aktif, dan kebijakan per event. Jangan menonaktifkan In-App pada demo utama. Jangan menyatakan WhatsApp Business sudah operasional kecuali layanan dan kredensial resmi tersedia.

---

## 15. Demo Early Warning System

Alur:

```text
Admin menyimpan riwayat resmi/TMT
→ Sistem menghitung target
→ Scheduler menjalankan engine
→ Alert muncul
→ Setiap role melihat scope-nya
→ Admin menindaklanjuti
→ Riwayat, notifikasi, dan audit diperbarui
```

### 15.1 Konfigurasi

**Role:** Super Admin  
**URL:** `/konfigurasi`

Tunjukkan nilai konfigurasi yang tersimpan, antara lain scheduler, masa pangkat, tahap peringatan pangkat, masa KGB, tahap KGB, pensiun, PPPK, dan Satyalancana. Jelaskan bahwa konfigurasi menentukan ambang, sedangkan tanggal sumber berasal dari data resmi pegawai.

Jangan mengubah threshold live karena dapat memengaruhi banyak alert.

### 15.2 Sumber alert pangkat

**Role:** Admin Kepegawaian  
**Halaman:** Data Pegawai → Detail → Riwayat Kepangkatan

Untuk demo 10 Agustus 2026 dengan masa empat tahun, TMT 30 Agustus 2022 menghasilkan target 30 Agustus 2026, yaitu 20 hari lagi dan masuk tahap kurang dari 30 hari.

### 15.3 Menjalankan engine

Operator menjalankan dari folder SIMPEG:

```powershell
php artisan app:run-ews
```

Hasil terminal yang diharapkan: `EWS scheduler selesai dijalankan.` Dalam produksi, scheduler menjalankan proses otomatis.

### 15.4 Admin melihat EWS Aktif

**Role:** Admin Kepegawaian  
**URL:** `/ews`

1. Muat ulang halaman.
2. Tunjukkan ringkasan total/aktif, kurang dari 30 hari, 30–90 hari, dan lebih dari 90 hari.
3. Cari Pegawai demo.
4. Tunjukkan jenis event, target, sisa hari, kelayakan, dan status tindak lanjut.

### 15.5 Kepala Bagian melihat EWS bawahan

**Role:** Kepala Bagian  
**URL:** `/kepala-bagian/ews`

1. Cari Pegawai demo.
2. Tunjukkan bahwa hanya bawahan langsung yang terlihat.
3. Jelaskan bahwa Kepala Bagian memantau, sedangkan tindak lanjut administratif dilakukan Admin Kepegawaian.

### 15.6 Pimpinan melihat EWS organisasi

**Role:** Pimpinan  
**URL:** `/pimpinan/ews`

Tunjukkan ringkasan lintas unit dan filter. Pimpinan memantau kebutuhan organisasi, bukan mengubah histori.

### 15.7 Pegawai melihat EWS pribadi

**Role:** Pegawai  
**URL:** `/dashboard/ews-saya`

Tunjukkan **Peringatan EWS Pribadi** dan pastikan tidak ada alert pegawai lain.

### 15.8 Tindak lanjut pangkat

Kembali sebagai Admin Kepegawaian:

1. Cari alert pangkat.
2. Klik **Ditangani**.
3. Isi Golongan Baru, TMT Pangkat, Tanggal SK, Nomor SK, File SK, dan Catatan.
4. Contoh catatan: `SK kenaikan pangkat telah diterima dan diarsipkan.`
5. Klik **Simpan Tindak Lanjut**.

Hasil:

- status Ditangani;
- riwayat pangkat baru ditambahkan;
- snapshot dan target dihitung ulang;
- file SK tersimpan;
- notifikasi sesuai kebijakan dibuat; dan
- audit tercatat.

Alternatif **Tidak Perlu** memerlukan catatan, tetapi tidak memerlukan SK.

### 15.9 Tindak lanjut KGB dan Pensiun

Untuk KGB Ditangani, isi TMT KGB, Gaji Pokok, Tanggal SK, Nomor SK, File SK, dan Catatan.

Untuk Pensiun Ditangani, isi Tanggal SK, Nomor SK, File SK, dan Catatan. Status pegawai dapat berubah menjadi Pensiun; karena itu gunakan employee dummy khusus, bukan akun demo utama.

### 15.10 Kelayakan pangkat

Skenario opsional: tandai kinerja tidak baik atau tambahkan hukuman disiplin aktif, kemudian jalankan engine. Alert pangkat tetap dapat terlihat sebagai informasi, tetapi ditandai tidak eligible dan notifikasi promosi ditahan. Jenis EWS lain tidak otomatis mengikuti aturan yang sama.

Batasan:

- Import Data Utama tidak memicu TMT.
- EWS memakai histori resmi per employee.
- EWS adalah peringatan, bukan keputusan otomatis.
- Deduplication mencegah alert/notifikasi sama dibuat berulang.

---

## 16. Demo Dashboard per Role

### 16.1 Super Admin/Admin Kepegawaian

**URL:** `/dashboard`

Tunjukkan jumlah pegawai, distribusi/status, cuti, EWS, aktivitas terbaru, dan shortcut. Super Admin memiliki konfigurasi lebih luas; Admin Kepegawaian berfokus pada operasional.

### 16.2 Pegawai

**URL:** `/dashboard`

Tunjukkan profil, saldo, pengajuan cuti, EWS pribadi, dan notifikasi milik sendiri.

### 16.3 Kepala Bagian

**URL:** `/kepala-bagian/dashboard`

Tunjukkan bawahan, cuti bawahan menunggu, dan EWS bawahan. Buka `/kepala-bagian/bawahan`, lalu detail Pegawai demo untuk membuktikan scope bawahan.

### 16.4 Pimpinan

**URL:** `/pimpinan/dashboard`

Tunjukkan ringkasan organisasi, cuti menunggu keputusan, EWS, statistik, aktivitas, dan shortcut laporan.

---

## 17. Demo Laporan dan Export

### 17.1 Export Pegawai

**Role:** Admin Kepegawaian, Super Admin, atau Pimpinan  
**URL:** `/laporan/export-pegawai`

1. Pilih kolom aman: Nama, NIP bila diizinkan, Jenis Pegawai, Golongan, Jabatan, Unit, dan Status.
2. Jangan memilih NIK/No. KK.
3. Atur urutan kolom.
4. Gunakan filter unit, golongan, jenis, status, jabatan, pensiun, dan sorting.
5. Periksa **Pratinjau Data Export**.
6. Klik **Unduh Excel Kustom** dan cocokkan hasil.
7. Klik **Export PDF** untuk format baku.

Excel dapat dikustomisasi; PDF tetap fixed-format. Jangan menjanjikan custom PDF bebas.

### 17.2 Laporan Cuti

**URL:** `/cuti/laporan`

1. Pilih periode pengajuan demo.
2. Filter unit/status/jenis/pegawai.
3. Pastikan pengajuan muncul.
4. Unduh Excel dan PDF.
5. Cocokkan dengan preview.

### 17.3 Laporan Pimpinan

**Nominatif:** `/pimpinan/laporan/nominatif`  
**Kepangkatan:** `/pimpinan/laporan/kepangkatan`

Pada nominatif, gunakan filter, periksa preview, lalu unduh PDF/Excel tanpa informasi kontak pribadi. Pada laporan kepangkatan, cari riwayat yang telah ditambahkan dan cocokkan hasil export.

---

## 18. Demo Audit Log

**Role:** Super Admin atau Admin Kepegawaian dengan permission audit  
**URL:** `/dashboard/audit`

Lakukan setelah mutasi data, import, cuti, keputusan, saldo, atau EWS.

1. Buka **Audit Log**.
2. Cari aktivitas berdasarkan Pegawai atau actor demo.
3. Buka **Detail Log Aktivitas**.
4. Tunjukkan actor, waktu, aksi, modul/model, record, nilai sebelum/sesudah, IP, dan user agent.
5. Tunjukkan bahwa tidak ada tombol edit/hapus audit.
6. Pastikan token, kredensial, NIK, dan No. KK tidak tampil mentah.

Narasi:

> Setiap mutasi penting dapat ditelusuri: siapa melakukan apa, kapan, dari mana, dan nilai apa yang berubah. Audit bersifat immutable melalui aplikasi.

---

## 19. Demo Soft Delete dan Restore

Gunakan employee dummy khusus, bukan salah satu akun demo utama.

### 19.1 Pindahkan ke backup

**Role:** Super Admin  
**URL:** `/pegawai`

1. Cari `[DEMO] Pegawai Restore`.
2. Pastikan bukan approver, Kepala Bagian, Pimpinan, atau employee pengajuan utama.
3. Buka aksi hapus.
4. Pada **Hapus Pegawai ke Backup**, klik **Ya, Hapus ke Backup**.
5. Pastikan data hilang dari daftar aktif.

### 19.2 Restore

**URL:** `/pegawai/data-backup`

1. Cari employee dummy.
2. Tunjukkan nama, NIP, jabatan/unit terakhir, waktu dinonaktifkan, dan tombol **Pulihkan**.
3. Klik **Pulihkan** dan konfirmasi.
4. Kembali ke `/pegawai`.
5. Pastikan employee dan historinya kembali.
6. Tunjukkan audit deactivate dan restore.

### 19.3 Catatan blocker narasi

Terdapat ketidakkonsistenan UI:

- modal daftar pegawai menyebut data akan dihapus permanen otomatis setelah 30 hari;
- halaman Data Backup menyebut data tidak dihapus otomatis; dan
- ketentuan produk Fase 1 menetapkan soft delete/restore tanpa hard delete permanen.

Sebelum demo stakeholder, teks penghapusan permanen 30 hari harus diperbaiki. Narasi resmi:

> Pegawai dipindahkan dari daftar aktif ke Data Backup dan dapat dipulihkan. SIMPEG Fase 1 tidak menyediakan penghapusan permanen pegawai.

---

## 20. Fitur yang Tidak Boleh Diklaim Siap Produksi

Jangan mendemonstrasikan sebagai fitur produksi:

- login manual produksi;
- pemberian akses dari role claim Keycloak;
- WhatsApp Business aktif tanpa service/kredensial resmi;
- custom PDF bebas;
- hard delete pegawai;
- import multi-template untuk histori;
- import yang otomatis membuat histori atau EWS;
- EWS sebagai keputusan otomatis;
- self-service editing tanpa approval;
- SIASN/BKN integration;
- absensi, SKP/RHK, daily log, dan fitur di luar Fase 1; atau
- tanda tangan elektronik tersertifikasi. Fase 1 menggunakan QR/token verifikasi cuti.

---

## 21. Troubleshooting

### 21.1 Pengajuan tidak muncul pada Kepala Bagian

- Periksa `kepala_bagian_id` employee pemohon.
- Periksa snapshot chain.
- Periksa employee yang terhubung ke akun Kepala Bagian.
- Pastikan pengajuan dibuat setelah chain dikonfigurasi.

### 21.2 Pengajuan tidak muncul pada Pimpinan

- Pastikan Kepala Bagian sudah menyetujui.
- Pastikan employee Pimpinan menjadi PYBMC/final approver.
- Pastikan current step berada pada Pimpinan.
- Periksa snapshot pengajuan, bukan hanya konfigurasi terbaru.
- Jika snapshot salah, buat pengajuan baru setelah konfigurasi diperbaiki.

### 21.3 Form menyatakan chain belum tersedia

Buka `/cuti/konfigurasi-approval` sebagai Super Admin, atur Kepala Bagian dan PYBMC, simpan chain, lalu muat ulang form Pegawai.

### 21.4 Hari kerja salah

Periksa weekend, Hari Libur, urutan tanggal, rentang lintas tahun, dan hari libur dummy.

### 21.5 Saldo tidak cukup

Periksa saldo awal, pengajuan lain yang masih mengalokasikan saldo, dan ledger. Jangan mengoreksi tanpa alasan.

### 21.6 Tombol keputusan tidak muncul

User mungkin bukan approver aktif, employee mapping salah, tahap belum sampai, pengajuan sudah selesai, atau role/permission tidak sesuai.

### 21.7 PDF belum tersedia

Pastikan status final Disetujui, seluruh tahap selesai, storage dapat diakses, dan token verifikasi tersedia.

### 21.8 Notifikasi/email tidak muncul

Periksa recipient, channel, event policy, employee mapping, unread refresh, queue worker, email penerima, Mailpit, dan failed job.

### 21.9 EWS tidak muncul

- Periksa histori resmi dan TMT.
- Periksa target berada dalam threshold.
- Pastikan employee aktif.
- Jalankan `php artisan app:run-ews`.
- Periksa filter, status tindak lanjut, dan scope role.

### 21.10 Import gagal

Gunakan template terbaru; periksa NIP 18 digit, email unik, tanggal lahir, jenis pegawai, kolom wajib, mapping, dan hasil validasi.

### 21.11 Role salah setelah login

Periksa username, `keycloak_username`, role internal, employee mapping, dan session. Logout penuh lalu login kembali.

---

## 22. Checklist Preflight

- [ ] Lima akun demo dapat login.
- [ ] Pimpinan masuk ke `/pimpinan/dashboard`, bukan Super Admin.
- [ ] Pegawai terhubung ke employee aktif.
- [ ] Kepala Bagian Pegawai benar.
- [ ] PYBMC/final approver adalah employee Pimpinan demo.
- [ ] Approval chain dan tanggal efektif benar.
- [ ] Saldo awal tersedia.
- [ ] Tanggal cuti bukan weekend/libur dan tidak lintas tahun.
- [ ] Form menghitung hari kerja.
- [ ] Kepala Bagian dapat melihat pengajuan.
- [ ] Pimpinan dapat melihat pengajuan setelah tahap pertama.
- [ ] Saldo dipotong hanya setelah final Disetujui.
- [ ] PDF dan token verifikasi dapat dibuka.
- [ ] Notifikasi In-App muncul.
- [ ] Queue worker berjalan jika email didemokan.
- [ ] Template import dapat diunduh.
- [ ] File positif lolos; file negatif gagal dengan pesan yang benar.
- [ ] Alert EWS tersedia atau dapat dibentuk oleh scheduler.
- [ ] Export Excel/PDF dapat dibuka.
- [ ] Audit Log memuat tindakan demo.
- [ ] Employee dummy restore tersedia.
- [ ] Tidak ada NIK/No. KK nyata pada layar.
- [ ] Lima profil browser sudah diberi label.
- [ ] Zoom/proyektor nyaman dilihat.
- [ ] Tidak ada error browser console.

---

## 23. Checklist Setelah Demo

- [ ] Restore employee dummy yang masih berada di backup.
- [ ] Bersihkan data master/hari libur dummy sesuai prosedur staging.
- [ ] Kembalikan konfigurasi EWS jika sempat diubah.
- [ ] Pastikan tidak ada role/permission utama yang berubah.
- [ ] Pastikan approval chain demo tetap sesuai atau dikembalikan ke snapshot awal.
- [ ] Periksa failed queue jobs.
- [ ] Simpan bukti screenshot/hasil export jika diperlukan untuk laporan demo.
- [ ] Jangan memindahkan akun atau data demo ke produksi.

---

## 24. Contoh Narasi Presenter

### Pembuka

> Hari ini kami akan menunjukkan alur SIMPEG dari beberapa sisi pengguna. Kita mulai dari pengaturan akses oleh Super Admin, pengelolaan data oleh Admin Kepegawaian, pengajuan oleh Pegawai, persetujuan oleh Kepala Bagian dan Pimpinan, kemudian melihat bagaimana hasilnya masuk ke saldo, notifikasi, EWS, laporan, dan Audit Log.

### Perpindahan Pegawai ke Kepala Bagian

> Sekarang kita berpindah dari role Pegawai ke Kepala Bagian. Pegawai hanya dapat mengajukan dan melihat data miliknya, sedangkan Kepala Bagian menerima pengajuan dari bawahan yang memang berada dalam cakupannya.

### Perpindahan Kepala Bagian ke Pimpinan

> Setelah Kepala Bagian menyetujui, pengajuan belum selesai karena masih ada tahap Pimpinan. Sekarang kita login sebagai Pimpinan untuk memberikan keputusan final.

### Penutup

> Dari alur ini terlihat bahwa satu transaksi tidak berdiri sendiri. Data pegawai menjadi dasar pengajuan, approval mengikuti chain yang dikonfigurasi, notifikasi mengikuti perubahan tahap, saldo baru dipotong setelah keputusan final, hasil masuk ke laporan, dan seluruh tindakan penting tercatat pada Audit Log.

---

## 25. Referensi

Dokumen ini disusun dengan mengacu pada:

1. `PRD-DLL/PRD-SIMPEG-Fase1-Core.md`.
2. `PRD-DLL/Panduan-Penulisan-Kode-SIMPEG.md`.
3. `PRD-DLL/User-Stories-SIMPEG-Fase1.md`.
4. `PRD-DLL/Issues-SIMPEG-Fase1.md`.
5. `Tracking-Sprint-Vertical-Slice-SIMPEG.md`.
6. `Tim-dan-Pembagian-Tugas-SIMPEG.md`.
7. Route, controller, request validation, dan Blade view pada source SIMPEG saat dokumen dibuat.
