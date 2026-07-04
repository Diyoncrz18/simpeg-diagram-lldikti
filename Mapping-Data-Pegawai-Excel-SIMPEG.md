# Mapping Data Pegawai Excel ke SIMPEG

> Sumber: `C:\Users\Asus\Downloads\daftar_pegawai.xlsx`  
> Sheet: `Pegawai`  
> Jumlah data terbaca: 10 pegawai  
> Jumlah kolom: 16

Dokumen ini menjadi acuan import awal data pegawai dari file Excel yang diberikan. Format ini lebih ringkas dari struktur lengkap PRD, sehingga field yang belum tersedia di Excel tetap disimpan sebagai data manual/lanjutan setelah pegawai berhasil diimpor.

## Header Excel

| # | Header Excel |
|---|--------------|
| 1 | `No` |
| 2 | `Nama Pegawai` |
| 3 | `Email Pegawai` |
| 4 | `Golongan` |
| 5 | `Jabatan` |
| 6 | `Kelas Jabatan` |
| 7 | `NIP` |
| 8 | `Nomor Telepon` |
| 9 | `Pangkat` |
| 10 | `Pendidikan Terakhir` |
| 11 | `Pensiun` |
| 12 | `Person` |
| 13 | `Person Formula` |
| 14 | `Prodi Pendidikan Terakhir` |
| 15 | `Status Kepegawaian` |
| 16 | `Tanggal Lahir` |

## Mapping ke Field SIMPEG

| Header Excel | Field SIMPEG | Perlakuan Import |
|--------------|--------------|------------------|
| `No` | - | Nomor urut Excel, tidak disimpan sebagai data utama. Dipakai hanya untuk referensi baris/error import. |
| `Nama Pegawai` | `employees.nama_lengkap` | Wajib. |
| `Email Pegawai` | `employees.email_pribadi` | Wajib untuk data Excel ini. Bisa dipakai sebagai kandidat mapping awal ke email Keycloak. |
| `Golongan` | `employees.golongan_terakhir` atau relasi ke `ref_golongan` | Wajib. Validasi ke reference table jika data referensi sudah tersedia. |
| `Jabatan` | `employees.jabatan_terakhir` atau record awal `position_histories.nama_jabatan` | Wajib. |
| `Kelas Jabatan` | `employees.kelas_jabatan` | Wajib. Simpan sebagai angka/string pendek sesuai nilai sumber. |
| `NIP` | `employees.nip` | Wajib, unik. Simpan sebagai string agar angka awal/nol tidak hilang. |
| `Nomor Telepon` | `employees.no_hp` | Wajib pada data Excel ini. Normalisasi seperlunya, jangan ubah angka secara agresif. |
| `Pangkat` | `employees.pangkat_terakhir` | Opsional. Pada file saat ini ada baris yang kosong, terutama untuk PPPK. |
| `Pendidikan Terakhir` | `employees.pendidikan_terakhir` atau `employee_education.jenjang` | Wajib. Contoh nilai: S1, S2, D3. |
| `Pensiun` | `employees.tanggal_pensiun` | Opsional. Jika kosong, dapat dihitung ulang dari `tanggal_lahir` + aturan BUP setelah referensi jabatan/BUP final tersedia. |
| `Person` | `employees.person_label` atau metadata import | Opsional. Tampak sebagai nama pendek/display name dari sumber. |
| `Person Formula` | `employees.person_formula_label` atau metadata import | Opsional. Tampak duplikat/hasil formula dari `Person`; boleh diabaikan jika tidak dipakai aplikasi. |
| `Prodi Pendidikan Terakhir` | `employees.prodi_pendidikan_terakhir` atau `employee_education.jurusan` | Wajib pada data Excel ini. |
| `Status Kepegawaian` | `employees.jenis_pegawai` / `employees.status_kepegawaian` | Wajib. Nilai terdeteksi: PNS, CPNS, PPPK. |
| `Tanggal Lahir` | `employees.tanggal_lahir` | Wajib. Dipakai untuk kalkulasi BUP/pensiun. |

## Field PRD yang Belum Ada di Excel

Field berikut tetap ada di desain SIMPEG, tetapi tidak tersedia pada file Excel ini sehingga tidak boleh menjadi blocker import awal:

| Kelompok | Field Belum Tersedia |
|----------|----------------------|
| Identitas | `nik`, `no_kk`, `tempat_lahir`, `jenis_kelamin`, `agama_id`, `status_kawin_id`, `golongan_darah`, `foto_path` |
| Alamat/Kontak | `alamat`, `no_telepon_rumah` |
| Pengangkatan/SK | `jenis_pengangkatan`, `tmt_pengangkatan`, `no_sk_pengangkatan`, `tanggal_sk_pengangkatan`, `file_sk_pengangkatan` |
| Riwayat | TMT pangkat, No SK pangkat, TMT jabatan, No SK jabatan, TMT KGB, gaji pokok, No SK KGB |
| Keluarga | Data pasangan/anak dan status tunjangan |
| Organisasi | Unit kerja dan kepala bagian |

## Aturan Import Awal

1. Import awal boleh membuat pegawai dengan status profil `belum_lengkap` selama kolom wajib dari Excel valid.
2. Validasi wajib import awal: `Nama Pegawai`, `Email Pegawai`, `NIP`, `Golongan`, `Jabatan`, `Kelas Jabatan`, `Nomor Telepon`, `Pendidikan Terakhir`, `Prodi Pendidikan Terakhir`, `Status Kepegawaian`, dan `Tanggal Lahir`.
3. `Pangkat` dan `Pensiun` boleh kosong.
4. Tanggal dari Excel perlu dinormalisasi ke format database `YYYY-MM-DD`.
5. `NIP` dan nomor telepon harus diperlakukan sebagai string.
6. Field yang belum ada di Excel diisi manual melalui form edit pegawai atau import lanjutan.
