# Tracking Sprint 3 — Import & Pelengkap Data Pegawai

| Field | Detail |
|---|---|
| Periode | 1 – 10 Juli 2026 |
| Cakupan issue | #20 – #25, #51 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 11 Agustus 2026 |
| Basis verifikasi | Branch `development` @ `4f3f2c3` setelah PR #183 merge; baseline historis verifikasi 26 Juli 2026 |
| Keputusan kanonis terkait | Import Fase 1 hanya template **Data Utama** (keputusan pengguna 22 Juli 2026): membuat record + snapshot saja, tanpa riwayat, tanpa kalkulasi TMT, tanggal pensiun dipertahankan apa adanya. Dikunci test regresi PR #121. |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**7 ✅ · 0 ⚠️ · 0 ❌** pada level source. Profil mandiri, keluarga, dan soft delete tuntas sejak awal. Gap manual column mapping dan warning pada #21 ditutup melalui **PR #183** yang telah merge ke `development`. Dusk/manual browser regression dan UAT formal tetap dicatat sebagai gate QA terpisah.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #20 | Download template import | ✅ | **Ditutup 26 Juli 2026 — PR #130, commit `db3c57b` (menunggu review/merge).** Template Data Utama kini 2 baris contoh: PNS lengkap + PPPK dengan Pangkat/Pensiun kosong (panduan kolom opsional non-PNS); kedua baris ber-marker sehingga tetap di-skip importer. Kontrak `example`→`examples` di action/writer/controller; 2 test pengunci bentuk lama diubah + assertion baru. Ketiadaan template multi-jenis tetap **sengaja tidak ada** (keputusan 22 Juli). |
| #21 | Upload, preview, validasi | ✅ | PR #183 (`4f3f2c3`, exact head `cbf907b`) menutup gap manual mapping: dropdown source→target/`Tidak dipakai`, state mapping batch sebagai sumber kebenaran, penyimpanan sebelum validasi, pencegahan target ganda dan missing required, warning header asing, klasifikasi canonical/ignored, pencocokan error eksak, serta normalisasi UI/backend. Source `Role` dijaga pada UI, Action, dan domain helper agar tidak dapat masuk ke field import. Review exact head menyatakan US-3.2 AC-4/AC-5 PASS dan CI hijau. |
| #22 | Eksekusi import & laporan hasil | ✅ | **Ditutup 26 Juli 2026 — PR #131, commit `e359248` (menunggu review/merge).** Tabel/model baru `import_batches` (id = batch id wizard, pemilik, counter, `row_issues` JSON, status, waktu; retensi permanen); eksekusi merekam batch saat mulai/sukses/gagal **sebelum** file sumber dibersihkan; endpoint `GET /pegawai/import/{batchId}/laporan` (gate role + kepemilikan) menghasilkan CSV ringkasan + rincian baris bermasalah dari database; tombol laporan wizard dialihkan ke server dan `downloadErrorReport()` browser dihapus. Test membuktikan laporan tetap bisa diunduh setelah `Cache::flush()`. Batasan Data Utama tetap dikunci test PR #121. |
| #23 | Profil sendiri read-only | ✅ | Ditutup PR #117: keluarga/pendidikan mandiri hanya GET ber-scope sesi; endpoint & UI mutasi dihapus; test mengunci scope + ketiadaan route mutasi. |
| #24 | CRUD data keluarga | ✅ | FormRequest, Action, soft delete, audit masking NIK keluarga, data scope, API v1, test lengkap. |
| #25 | Soft delete & restore pegawai | ✅ | Soft delete, restore, audit, daftar nonaktif, EWS exclusion. Seluruh mekanisme hard delete (action, command, scheduler purge 02:00) sudah dihapus; Data Backup menjadi filter permanen. |
| #51 | Kebijakan soft delete Super Admin | ✅ | Tidak ada tombol/jalur hapus permanen untuk role mana pun; restore tersedia; audit tercatat. |

## Gap dan Tindak Lanjut

1. **US-3.3 AC-5 / K-US-02:** cabang "skip NIP duplikat" di `ValidateImportBatchAction` masih perlu diselaraskan bila belum ditutup PR lain. NIP yang sudah ada di database harus menjadi baris terlewat, NIP ganda dalam satu berkas tetap error, dan email terdaftar tetap error. Pokok ini terpisah dari penyelesaian US-3.2 oleh PR #183.
2. **QA browser:** Laravel Dusk regression untuk mapping sudah tersedia, tetapi belum menjadi quality gate CI dan bukti eksekusi manual penuh belum tercatat. Jalankan Dusk/manual browser regression beserta UAT formal sebelum release candidate.

Rambu: pertahankan batasan keputusan 22 Juli (jangan memulihkan template multi-jenis, jangan membuat riwayat/kalkulasi TMT dari import) — test regresi PR #121 wajib tetap hijau.

## Langkah Proses Tersisa (#20 & #22)

1. Review PR #130 & #131 (Adriel) dan QA/retest (Grantly) sesuai skenario di badan masing-masing PR.
2. Verifikasi PostgreSQL 17 khusus PR #131 (migration + JSON column; test lokal memakai SQLite).
3. Setelah merge, ubah catatan #20/#22 dari "menunggu review/merge" menjadi terkonfirmasi merged.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 20 Juli 2026 | #23 naik ke ✅ (PR #117); #25 tuntas setelah penghapusan hard delete. |
| 22 Juli 2026 | Keputusan import Data Utama menjadi kanonis; temuan "template lanjutan/snapshot riwayat/TMT pasca-import" direklasifikasi **sengaja tidak ada**. PR #121 menambah test regresi batasnya. |
| 26 Juli 2026 (siang) | Verifikasi HEAD `1b2e5b6`: #20–#22 tidak berubah (tak tersentuh PR #122–#128). |
| 26 Juli 2026 (malam) | #20 ditutup via **PR #130** (`db3c57b`) dan #22 via **PR #131** (`e359248`) — keduanya menunggu review/QA. Bebas konflik terhadap development terbaru (termasuk PR #127) via simulasi merge; uji gabungan 38 test lulus. Ditemukan dead code jalur skip NIP duplikat (butuh keputusan kecil). Sisa gap sprint: #21. |
| 11 Agustus 2026 | #21 ditutup pada level source melalui **PR #183** (`4f3f2c3`, exact head `cbf907b`). US-3.2 AC-4/AC-5 dikonfirmasi PASS; enforcement source `Role` tersedia pada UI, backend, dan domain helper. Dusk/manual browser regression serta UAT formal tetap menjadi tindak lanjut QA. |
