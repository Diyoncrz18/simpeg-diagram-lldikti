# Tracking Sprint 3 — Import & Pelengkap Data Pegawai

| Field | Detail |
|---|---|
| Periode | 1 – 10 Juli 2026 |
| Cakupan issue | #20 – #25, #51 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 26 Juli 2026 |
| Basis verifikasi | Source HEAD `1b2e5b6` + verifikasi kode 26 Juli 2026 |
| Keputusan kanonis terkait | Import Fase 1 hanya template **Data Utama** (keputusan pengguna 22 Juli 2026): membuat record + snapshot saja, tanpa riwayat, tanpa kalkulasi TMT, tanggal pensiun dipertahankan apa adanya. Dikunci test regresi PR #121. |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**6 ✅ · 1 ⚠️ · 0 ❌** (pembaruan 26 Juli 2026 malam). Profil mandiri, keluarga, dan soft delete tuntas sejak awal. Dua dari tiga gap import ditutup hari ini: #20 via **PR #130** dan #22 via **PR #131** (keduanya menunggu review/QA). Sisa satu gap: #21 manual column mapping.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #20 | Download template import | ✅ | **Ditutup 26 Juli 2026 — PR #130, commit `db3c57b` (menunggu review/merge).** Template Data Utama kini 2 baris contoh: PNS lengkap + PPPK dengan Pangkat/Pensiun kosong (panduan kolom opsional non-PNS); kedua baris ber-marker sehingga tetap di-skip importer. Kontrak `example`→`examples` di action/writer/controller; 2 test pengunci bentuk lama diubah + assertion baru. Ketiadaan template multi-jenis tetap **sengaja tidak ada** (keputusan 22 Juli). |
| #21 | Upload, preview, validasi | ⚠️ | Upload ≤10MB, parser XLS/XLSX/CSV, preview, validasi per baris, skip NIP duplikat, wizard, test tersedia. **Gap terverifikasi:** auto-match all-or-nothing pada header canonical (`UploadImportBatchAction:104-112` melempar error bila header tak dikenali); **tidak ada** dropdown mapping kolom manual di view (858 baris tanpa satu pun select mapping) dan **tidak ada** warning kolom ekstra — kolom tak dikenal dibuang diam-diam oleh `EmployeeRowMapper`. |
| #22 | Eksekusi import & laporan hasil | ✅ | **Ditutup 26 Juli 2026 — PR #131, commit `e359248` (menunggu review/merge).** Tabel/model baru `import_batches` (id = batch id wizard, pemilik, counter, `row_issues` JSON, status, waktu; retensi permanen); eksekusi merekam batch saat mulai/sukses/gagal **sebelum** file sumber dibersihkan; endpoint `GET /pegawai/import/{batchId}/laporan` (gate role + kepemilikan) menghasilkan CSV ringkasan + rincian baris bermasalah dari database; tombol laporan wizard dialihkan ke server dan `downloadErrorReport()` browser dihapus. Test membuktikan laporan tetap bisa diunduh setelah `Cache::flush()`. Batasan Data Utama tetap dikunci test PR #121. |
| #23 | Profil sendiri read-only | ✅ | Ditutup PR #117: keluarga/pendidikan mandiri hanya GET ber-scope sesi; endpoint & UI mutasi dihapus; test mengunci scope + ketiadaan route mutasi. |
| #24 | CRUD data keluarga | ✅ | FormRequest, Action, soft delete, audit masking NIK keluarga, data scope, API v1, test lengkap. |
| #25 | Soft delete & restore pegawai | ✅ | Soft delete, restore, audit, daftar nonaktif, EWS exclusion. Seluruh mekanisme hard delete (action, command, scheduler purge 02:00) sudah dihapus; Data Backup menjadi filter permanen. |
| #51 | Kebijakan soft delete Super Admin | ✅ | Tidak ada tombol/jalur hapus permanen untuk role mana pun; restore tersedia; audit tercatat. |

## Gap Terbuka (prioritas P1)

1. **Manual column mapping + warning** (#21): kontrak mapping kolom→field pada upload/validate, dropdown mapping di wizard, warning header ekstra/tak cocok (saat ini kolom tak dikenal dibuang diam-diam oleh `EmployeeRowMapper`), FormRequest untuk validate/execute, test API + UI. Rencana rinci sudah disusun (PR-3 dari analisis 26 Juli).
2. **Keputusan kecil ikutan** (ditemukan saat PR #131): cabang "skip NIP duplikat" di `ValidateImportBatchAction` adalah dead code — rule `unique:employees,nip` menangkap duplikat database lebih dulu sebagai *error*, sehingga AC US-3.3 "duplikat ditandai skip" tidak pernah terjadi. Putuskan: hidupkan jalur skip (hapus rule unique) atau revisi AC.

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
