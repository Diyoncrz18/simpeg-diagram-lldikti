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

**4 ✅ · 3 ⚠️ · 0 ❌.** Profil mandiri, keluarga, dan soft delete tuntas. Seluruh gap tersisa terkonsentrasi di modul import (#20–#22) dan tidak berubah sejak audit 22 Juli — PR #122–#128 tidak menyentuh file import sama sekali.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #20 | Download template import | ⚠️ | Download CSV UTF-8/XLSX via `GenerateImportTemplateAction` + `ImportTemplateWriter` tersedia. Ketiadaan template multi-jenis = **sengaja tidak ada** (keputusan 22 Juli). **Gap tersisa:** contoh data hanya **1 baris** (AC minta 2) — kontrak `example` masih map tunggal dan test justru mengunci bentuk 1-baris (`EmployeeImportTemplateTest`, `GenerateImportTemplateActionTest` ikut disesuaikan saat diperbaiki). |
| #21 | Upload, preview, validasi | ⚠️ | Upload ≤10MB, parser XLS/XLSX/CSV, preview, validasi per baris, skip NIP duplikat, wizard, test tersedia. **Gap terverifikasi:** auto-match all-or-nothing pada header canonical (`UploadImportBatchAction:104-112` melempar error bila header tak dikenali); **tidak ada** dropdown mapping kolom manual di view (858 baris tanpa satu pun select mapping) dan **tidak ada** warning kolom ekstra — kolom tak dikenal dibuang diam-diam oleh `EmployeeRowMapper`. |
| #22 | Eksekusi import & laporan hasil | ⚠️ | Queue job, progress polling, hasil inserted/skipped/failed, audit, notifikasi tersedia. Batasan Data Utama **sesuai keputusan** dan dikunci test `test_import_wizard_persists_data_utama_snapshots_without_histories_or_tmt_calculation` (PR #121). **Gap terverifikasi:** tidak ada persistence hasil import (state hanya cache TTL 30/10 menit, tanpa tabel/model batch); tombol unduh laporan error dibangun dari state browser **tahap validasi** (bukan hasil eksekusi); file sumber dihapus setelah sukses (`cleanupBatch()`), jejak durable hanya counter agregat di audit. |
| #23 | Profil sendiri read-only | ✅ | Ditutup PR #117: keluarga/pendidikan mandiri hanya GET ber-scope sesi; endpoint & UI mutasi dihapus; test mengunci scope + ketiadaan route mutasi. |
| #24 | CRUD data keluarga | ✅ | FormRequest, Action, soft delete, audit masking NIK keluarga, data scope, API v1, test lengkap. |
| #25 | Soft delete & restore pegawai | ✅ | Soft delete, restore, audit, daftar nonaktif, EWS exclusion. Seluruh mekanisme hard delete (action, command, scheduler purge 02:00) sudah dihapus; Data Backup menjadi filter permanen. |
| #51 | Kebijakan soft delete Super Admin | ✅ | Tidak ada tombol/jalur hapus permanen untuk role mana pun; restore tersedia; audit tercatat. |

## Gap Terbuka (semuanya modul import — prioritas P1)

1. **Template 2 baris contoh** (#20): ubah kontrak `example` menjadi daftar baris di `GenerateImportTemplateAction` + `ImportTemplateWriter`, sesuaikan dua test yang mengunci bentuk lama.
2. **Manual column mapping + warning** (#21): kontrak mapping kolom→field pada upload/validate, dropdown mapping di wizard, warning header ekstra/tak cocok, test API + UI.
3. **Laporan hasil import persisten** (#22): tabel/model batch import (hasil per-baris gagal + alasan), endpoint download laporan server-side dari hasil **eksekusi**, jangan hapus informasi sebelum dapat diunduh.

Rambu: pertahankan batasan keputusan 22 Juli (jangan memulihkan template multi-jenis, jangan membuat riwayat/kalkulasi TMT dari import) — test regresi PR #121 wajib tetap hijau.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 20 Juli 2026 | #23 naik ke ✅ (PR #117); #25 tuntas setelah penghapusan hard delete. |
| 22 Juli 2026 | Keputusan import Data Utama menjadi kanonis; temuan "template lanjutan/snapshot riwayat/TMT pasca-import" direklasifikasi **sengaja tidak ada**. PR #121 menambah test regresi batasnya. |
| 26 Juli 2026 | Verifikasi HEAD `1b2e5b6`: #20–#22 tidak berubah (tak tersentuh PR #122–#128). |
