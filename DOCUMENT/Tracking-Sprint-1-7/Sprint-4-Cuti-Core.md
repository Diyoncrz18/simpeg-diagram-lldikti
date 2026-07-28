# Tracking Sprint 4 — Cuti Core

| Field | Detail |
|---|---|
| Periode | 11 – 20 Juli 2026 |
| Cakupan issue | #26 – #32, #44 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 28 Juli 2026 (verifikasi ulang terhadap `development`) |
| Basis verifikasi | `development` HEAD `478424f` + verifikasi kode dan cakupan test regresi cuti 28 Juli 2026 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

<<<<<<< HEAD
**6 ✅ · 2 ⚠️ · 0 ❌.** Sprint ini paling banyak membaik sejak audit 22 Juli: penugasan Kepala Bagian (PR #124/#128), kontrak kalkulasi hari kerja (PR #116), filter jenis cuti PPPK (PR #104), label langkah aktif (PR #119/#125), penghapusan vocabulary legacy `rejected` (PR #128), serta validasi lintas tahun seluruh jenis cuti semuanya sudah tertutup. Sisa gap utama: chain per unit dan ADR skema cuti. Hardening constraint database penugasan Kepala Bagian tetap dicatat sebagai pekerjaan P2 terpisah.
=======
**7 ✅ · 1 ⚠️ · 0 ❌.** Sprint ini paling banyak membaik sejak audit 22 Juli: penugasan Kepala Bagian (PR #124/#128), kontrak kalkulasi hari kerja (PR #116), filter jenis cuti PPPK (PR #104), label langkah aktif (PR #119/#125), penghapusan vocabulary legacy `rejected` (PR #128), validasi lintas tahun seluruh jenis cuti, serta penetapan schema cuti canonical semuanya sudah tertutup. Sisa gap utama hanya chain per unit. Hardening constraint database penugasan Kepala Bagian tetap dicatat sebagai pekerjaan P2 terpisah.
>>>>>>> 9a7f966a8a1612375e12cdd2055689c9de664105

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #26 | Migration tabel cuti | ✅ | Tabel/model chain, snapshot step, balance ledger, dan QR proof tersedia serta diuji. **Ditutup 28 Juli:** [Keputusan Skema Cuti Canonical](../Keputusan-Skema-Cuti-Canonical.md) menetapkan `leave_request_steps` / `leave_balance_ledger` / `leave_proofs` sebagai nama fisik canonical. PRD §15.2 dan issue breakdown telah diselaraskan; `CutiFoundationSchemaTest` mengunci schema runtime. |
| #27 | Assign Kepala Bagian per pegawai | ✅ | **Ditutup PR #124 (+#128).** Route web+API kini `role:super_admin,admin_kepegawaian` + `permission:employees.update`; `effective_date` wajib & bisa diisi pengguna; overlap ditolak + interval lama ditutup H-1 dalam transaksi + `lockForUpdate`; 15 test regresi (`SupervisorAssignmentTest`). Catatan: enforcement satu kabag aktif di level aplikasi — belum ada unique/exclusion constraint DB. |
| #28 | Konfigurasi approval chain | ⚠️ | Chain per pegawai, snapshot step, PYBMC global, backfill, audit, skip duplikat tersedia; step type kini `kepala_bagian`/`verifier`/`pybmc`. **Gap:** scope **per unit** belum ada (tanpa `scope_type`/`unit_kerja_id`; resolver hanya `employee_id`). Butuh keputusan produk precedence global–unit–pegawai sebelum implementasi (`AGENTS.md`: escalate, do not guess). |
| #29 | Kalkulasi hari kerja otomatis | ✅ | **Ditutup PR #116.** Blade kini membaca `result.data?.jumlah_hari_kerja` + merender `warnings` (`aria-live`), plus guard race-condition `workdayRequestId`. `WorkdayCalculator`, endpoint API, test unit/feature tersedia. |
| #30 | Form pengajuan cuti | ✅ | Form, lampiran tervalidasi, hitung server-side, saldo, snapshot chain, notifikasi, audit, dan test tersedia. Dropdown jenis cuti menyaring PNS/PPPK dari metadata `khusus_pns` (**ditutup PR #104**) dengan validasi server sebagai lapis kedua. Validasi tahun kalender kini mandiri pada Store dan Resubmit, dijalankan sebelum gate saldo dan cek TMT, sehingga berlaku untuk **semua** jenis cuti; Cuti Sakit/Melahirkan/Alasan Penting/Besar/CLTN lintas Desember–Januari ditolak dengan pesan generik. Test regresi mencakup seluruh jenis non-tahunan dan resubmit. Verifikasi ulang menunjukkan perbaikan telah tersedia di `development` melalui `952f723` (`fix(cuti): tegakkan batas tahun kalender untuk semua jenis cuti`, PR #140). |
| #31 | Approval engine cuti dinamis | ✅ | Snapshot per request, data-scope approver, notifikasi, audit, pengurangan saldo final, QR proof, timeline, dokumen eksternal Kepala Lembaga, test E2E. Vocabulary legacy tuntas: `DeclineLeaveAction` + `NOT_APPROVED` + label `Tidak Disetujui`; arm `'rejected'` di `LeaveProofService` **dihapus PR #128** — grep `rejected/REJECT/Ditolak` sebagai status di `app/` = nol. |
| #32 | Saldo cuti & daftar cuti pegawai | ✅ | Balance ledger, bucket N-2/N-1/berjalan, seeder 2026, koreksi ber-audit, rollover scheduler, halaman/API saldo, test; jenis mutasi ledger dikunci test. |
| #44 | Daftar cuti pegawai + timeline approval | ✅ | Daftar + badge status + timeline vertikal tersedia. Label langkah aktif `Menunggu [nama step]` dari snapshot chain (**PR #119**, disempurnakan **PR #125**); kontrak dikunci `CutiListDisplayTest`. |

## Gap Terbuka

1. **P1 — Approval chain per unit** (#28): **eskalasi keputusan produk dulu** (model precedence pegawai > unit > global, perilaku hierarki unit), catat di dokumen, baru migration + resolver + UI + test.
<<<<<<< HEAD
2. **Dokumentasi — ADR skema cuti canonical** (#26): sahkan penamaan `leave_request_steps`/`leave_balance_ledger`/`leave_proofs` dan selaraskan PRD §15.2.
3. **P2 — Constraint DB penugasan kabag** (#27, opsional): unique/exclusion constraint agar tulis-langsung-DB tidak bisa membuat overlap.
=======
2. **P2 — Constraint DB penugasan kabag** (#27, opsional): unique/exclusion constraint agar tulis-langsung-DB tidak bisa membuat overlap.
>>>>>>> 9a7f966a8a1612375e12cdd2055689c9de664105

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 22 Juli 2026 | Baseline audit: hanya #32 ✅; #26–#31 ⚠️. (Catatan: perbaikan #29 dan filter PPPK #30 sebenarnya sudah masuk sebelum audit — PR #116/#104 — tetapi belum tercermin di audit lama.) |
| 23 Juli 2026 | PR #119: label `Tidak Disetujui` + label langkah aktif → #44 dan sebagian #31 tertutup. |
| 24–25 Juli 2026 | PR #124: penugasan Kepala Bagian bertanggal efektif → #27 ✅. |
| 26 Juli 2026 | PR #128: kompatibilitas `rejected` dihapus → #31 ✅. Verifikasi HEAD `1b2e5b6` menetapkan status di atas. |
| 27 Juli 2026 | Perbaikan A dieksekusi: validasi lintas tahun seluruh jenis cuti (#30) + test E2E alur pengajuan→approval→potong saldo. Pendukung: penetapan Kepala Bagian inline + penataan ulang halaman konfigurasi approval + pencarian PYBMC mandiri (memperhalus alur #27/#28), serta penyamaan dokumen bukti tersimpan dengan formulir resmi (penyempurnaan #31). |
<<<<<<< HEAD
| 28 Juli 2026 | Verifikasi ulang terhadap `development` HEAD `478424f`: perbaikan lintas tahun seluruh jenis cuti telah terintegrasi melalui `952f723` (PR #140), termasuk validasi Store/Resubmit sebelum gate saldo/TMT dan regression test untuk jenis non-tahunan. **#30 ditutup menjadi ✅; status “menunggu review” dihapus.** |
=======
| 28 Juli 2026 | Verifikasi ulang terhadap `development` HEAD `478424f`: perbaikan lintas tahun seluruh jenis cuti telah terintegrasi melalui `952f723` (PR #140), termasuk validasi Store/Resubmit sebelum gate saldo/TMT dan regression test untuk jenis non-tahunan. **#30 ditutup menjadi ✅.** Pada tanggal yang sama, keputusan K-SCHEMA-01 menetapkan `leave_request_steps`/`leave_balance_ledger`/`leave_proofs` sebagai schema cuti canonical; PRD §15.2 dan issue breakdown diselaraskan. **#26 ditutup menjadi ✅.** |
>>>>>>> 9a7f966a8a1612375e12cdd2055689c9de664105
