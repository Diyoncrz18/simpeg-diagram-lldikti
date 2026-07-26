# Tracking Sprint 2 — Data Pegawai Core

| Field | Detail |
|---|---|
| Periode | 21 – 30 Juni 2026 |
| Cakupan issue | #13 – #19 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 26 Juli 2026 |
| Basis verifikasi | Source HEAD `1b2e5b6` + branch `fix/append-only-riwayat` (commit `76da2ec`, 26 Juli 2026) |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**7 ✅ · 0 ⚠️ · 0 ❌** — sprint pertama yang tuntas penuh di level source. Issue terakhir (#18 append-only riwayat) ditutup 26 Juli 2026 pada branch `fix/append-only-riwayat` dan tinggal menunggu review PR + QA sebelum merge ke `development`.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #13 | Migration tabel utama pegawai | ✅ | Model/relasi, UUID, index, cast enkripsi (`nik`, `no_kk`), migration sinkron PRD. |
| #14 | Form tambah pegawai multi-tab | ✅ | `StoreEmployeeRequest`, `CreateEmployeeAction`, view multi-tab, validasi upload, storage, audit, test. |
| #15 | Form edit pegawai | ✅ | FormRequest/Action, penggantian berkas/foto, audit, test. (Catatan lintas-sprint: payload audit `toArray()` membawa risiko NIK/No. KK plaintext — ditangani sebagai gap Issue #5 Sprint 1, bukan gap issue ini.) |
| #16 | Halaman daftar pegawai | ✅ | Search, filter, sort, pagination, eager loading, default aktif, RBAC, test. Status kelengkapan dokumen kini 4-nilai (`kosong/tersedia/tidak_lengkap/lengkap`) + 8 test tambahan. |
| #17 | Halaman detail pegawai bertab | ✅ | Data relasi lengkap, tab, informasi kalkulasi EWS, test detail. |
| #18 | Riwayat pangkat/jabatan/KGB append-only | ✅ | **Ditutup 26 Juli 2026 — commit `76da2ec` di branch `fix/append-only-riwayat` (menunggu review/merge).** Sebelumnya `UpdateEmployeeAction` bisa memutasi riwayat existing via `*_history_id` (default form bahkan mode edit riwayat terbaru, tanpa audit per record). Perbaikan: (a) `UpdateEmployeeRequest` menolak id riwayat lama (`in:new` + pesan append-only); (b) action selalu `create` (3 cabang update dihapus); (c) dropdown "Edit Riwayat" di form edit diganti banner append-only + nilai awal form dikosongkan agar sub-tab yang tidak disentuh tidak membuat duplikat; (d) 4 test regresi baru — total `EmployeeUpdateTest` 24 passed/99 assertion, Pint & PHPStan hijau. |
| #19 | Riwayat hukuman disiplin | ✅ | Ditutup PR #117 (`32ada6b`): route DELETE/action/permission/tombol hapus dihilangkan; `DeleteDocumentAction` menolak dokumen yang dipakai riwayat disiplin; test mengunci route lama 404/405 + record tetap tersimpan. |

## Gap Terbuka

Tidak ada gap requirement tersisa di sprint ini. Langkah proses yang tersisa untuk #18:

1. Push branch `fix/append-only-riwayat` + buka PR ke `development`.
2. Review PR (Adriel) dan QA/retest (Grantly): coba edit riwayat → ditolak validasi; tambah riwayat baru → `is_latest` & kalkulasi TMT benar; simpan form tanpa menyentuh tab riwayat → tidak ada duplikat.
3. Verifikasi PostgreSQL 17 (perubahan menyentuh transaksi/UUID; test lokal memakai SQLite).

Keputusan produk terkait (bila dibutuhkan kelak): mekanisme "koreksi resmi" riwayat (edit ber-alasan + audit) adalah perubahan PRD dan harus diputuskan eksplisit — cara koreksi saat ini adalah menambah record baru yang benar.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 20 Juli 2026 | #19 naik ke ✅ (PR #117). |
| 22 Juli 2026 | Baseline audit: #13–#17, #19 ✅; #18 ⚠️ (append-only tidak absolut). |
| 26 Juli 2026 | #18 ditutup pada branch `fix/append-only-riwayat` (commit `76da2ec`) — Sprint 2 tuntas di level source. |
