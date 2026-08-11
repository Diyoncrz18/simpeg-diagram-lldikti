# Tracking Sprint 2 — Data Pegawai Core

| Field | Detail |
|---|---|
| Periode | 21 – 30 Juni 2026 |
| Cakupan issue | #13 – #19 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 11 Agustus 2026 |
| Basis verifikasi | Branch `development` @ `71d2dae` setelah PR #180 merge; baseline append-only historis PR #129 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**7 ✅ · 0 ⚠️ · 0 ❌** — Sprint 2 tuntas pada level source. Baseline append-only riwayat diselesaikan melalui PR #129, sedangkan gap UI BUG-01 yang ditemukan kemudian ditutup oleh **PR #180** dan telah merge ke `development`. Full browser/E2E submit multipart tetap dicatat sebagai tindak lanjut QA non-blocking, bukan gap requirement pada source.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #13 | Migration tabel utama pegawai | ✅ | Model/relasi, UUID, index, cast enkripsi (`nik`, `no_kk`), migration sinkron PRD. |
| #14 | Form tambah pegawai multi-tab | ✅ | `StoreEmployeeRequest`, `CreateEmployeeAction`, view multi-tab, validasi upload, storage, audit, test. |
| #15 | Form edit pegawai | ✅ | FormRequest/Action, penggantian berkas/foto, audit, test. (Catatan lintas-sprint: payload audit `toArray()` membawa risiko NIK/No. KK plaintext — ditangani sebagai gap Issue #5 Sprint 1, bukan gap issue ini.) |
| #16 | Halaman daftar pegawai | ✅ | Search, filter, sort, pagination, eager loading, default aktif, RBAC, test. Status kelengkapan dokumen kini 4-nilai (`kosong/tersedia/tidak_lengkap/lengkap`) + 8 test tambahan. |
| #17 | Halaman detail pegawai bertab | ✅ | Data relasi lengkap, tab, informasi kalkulasi EWS, dan test detail tersedia. BUG-01 terkait kontrol tambah riwayat ditutup PR #180 (`71d2dae`): ketiga tab riwayat memiliki tombol tambah yang digate permission `employee_histories.create`, modal menyediakan Upload SK, dan tanggal riwayat Jabatan diformat untuk tampilan. |
| #18 | Riwayat pangkat/jabatan/KGB append-only | ✅ | Baseline append-only ditutup PR #129 (`2e58c8c`): request menolak id riwayat lama, Action selalu membuat record baru, dan form edit tidak memutasi riwayat existing. PR #180 (`71d2dae`, exact head `9c1c1e9`) menutup gap UI dengan kontrol tambah Kepangkatan/Jabatan/KGB, multipart Upload SK, dan integrasi ke backend existing yang mempertahankan `is_latest`, kalkulasi TMT, audit CREATE, serta dokumen SK. Review exact head menyatakan US-2.4 AC-2 dan US-2.6 AC-1 sampai AC-5 PASS. |
| #19 | Riwayat hukuman disiplin | ✅ | Ditutup PR #117 (`32ada6b`): route DELETE/action/permission/tombol hapus dihilangkan; `DeleteDocumentAction` menolak dokumen yang dipakai riwayat disiplin; test mengunci route lama 404/405 + record tetap tersimpan. |

## Gap Terbuka

Tidak ada gap requirement yang teridentifikasi pada source Sprint 2 setelah PR #180 merge. Tindak lanjut proses yang masih perlu dijaga:

1. Tambahkan regression browser/E2E untuk alur lengkap `buka modal → pilih file SK → submit multipart → riwayat tersimpan → dokumen SK terbentuk → baris baru tampil`.
2. Jalankan regression/UAT formal Sprint 7 terhadap penambahan riwayat, `is_latest`, kalkulasi TMT, audit, akses permission, dan keterhubungan dokumen SK.
3. Pertahankan verifikasi PostgreSQL 17 untuk transaksi, UUID, relasi dokumen, dan audit sebelum release candidate.

Keputusan produk terkait (bila dibutuhkan kelak): mekanisme "koreksi resmi" riwayat (edit ber-alasan + audit) adalah perubahan PRD dan harus diputuskan eksplisit — cara koreksi saat ini adalah menambah record baru yang benar.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 20 Juli 2026 | #19 naik ke ✅ (PR #117). |
| 22 Juli 2026 | Baseline audit: #13–#17, #19 ✅; #18 ⚠️ (append-only tidak absolut). |
| 26 Juli 2026 | #18 ditutup (commit `2e58c8c`) dan diajukan sebagai **PR #129** ke `development` — Sprint 2 tuntas di level source, menunggu review/QA. |
| 11 Agustus 2026 | BUG-01 ditutup melalui **PR #180** (`71d2dae`): kontrol tambah riwayat, Upload SK, multipart submission, label baris baru, dan format tanggal Jabatan masuk ke `development`. US-2.4 AC-2 serta US-2.6 AC-1 sampai AC-5 dikonfirmasi PASS pada source; full E2E submit dicatat sebagai follow-up QA non-blocking. |
