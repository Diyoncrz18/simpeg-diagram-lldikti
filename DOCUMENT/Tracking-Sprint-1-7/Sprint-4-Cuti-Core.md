# Tracking Sprint 4 — Cuti Core

| Field | Detail |
|---|---|
| Periode | 11 – 20 Juli 2026 |
| Cakupan issue | #26 – #32, #44 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 10 Agustus 2026 (penutupan hardening Issue #178 melalui PR #179) |
| Basis verifikasi | `development` @ `ff260a5` setelah PR #179 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**8 ✅ · 0 ⚠️ · 0 ❌ pada source.** Seluruh issue Sprint 4 telah selesai. Gap terakhir #28 ditutup PR #177 dengan penyalinan template rantai ke anggota unit tanpa menambah scope runtime per unit. Hardening invarian writer pada issue #178 selesai melalui PR #179. Constraint database penugasan Kepala Bagian tetap follow-up terpisah, bukan bagian acceptance criteria yang belum selesai.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #26 | Migration tabel cuti | ✅ | Tabel/model chain, snapshot step, balance ledger, dan QR proof tersedia serta diuji. **Ditutup 28 Juli:** [Keputusan Skema Cuti Canonical](../Keputusan-Skema-Cuti-Canonical.md) menetapkan `leave_request_steps` / `leave_balance_ledger` / `leave_proofs` sebagai nama fisik canonical. PRD §15.2 dan issue breakdown telah diselaraskan; `CutiFoundationSchemaTest` mengunci schema runtime. |
| #27 | Assign Kepala Bagian per pegawai | ✅ | **Ditutup PR #124 (+#128).** Route web+API kini `role:super_admin,admin_kepegawaian` + `permission:employees.update`; `effective_date` wajib & bisa diisi pengguna; overlap ditolak + interval lama ditutup H-1 dalam transaksi + `lockForUpdate`; 15 test regresi (`SupervisorAssignmentTest`). Catatan: enforcement satu kabag aktif di level aplikasi — belum ada unique/exclusion constraint DB. |
| #28 | Konfigurasi approval chain | ✅ | Keputusan K-US-01 mempertahankan tepat satu chain runtime per pegawai dan menjadikan unit sebagai target penyalinan template, bukan scope resolver baru. PR #177 (`1fd99cb`) menerapkan template ke seluruh anggota unit dengan validasi eligibility, atasan efektif per pegawai, audit fail-closed, perlindungan concurrent run per unit, dan jaminan snapshot pengajuan berjalan tidak berubah. Hardening issue #178 selesai melalui PR #179 (`ff260a5`): invarian dipusatkan pada writer bersama, writer konfigurasi diserialkan, dan audit menggunakan aktor eksplisit. |
| #29 | Kalkulasi hari kerja otomatis | ✅ | **Ditutup PR #116.** Blade kini membaca `result.data?.jumlah_hari_kerja` + merender `warnings` (`aria-live`), plus guard race-condition `workdayRequestId`. `WorkdayCalculator`, endpoint API, test unit/feature tersedia. |
| #30 | Form pengajuan cuti | ✅ | Form, lampiran tervalidasi, hitung server-side, saldo, snapshot chain, notifikasi, audit, dan test tersedia. Dropdown jenis cuti menyaring PNS/PPPK dari metadata `khusus_pns` (**ditutup PR #104**) dengan validasi server sebagai lapis kedua. Validasi tahun kalender kini mandiri pada Store dan Resubmit, dijalankan sebelum gate saldo dan cek TMT, sehingga berlaku untuk **semua** jenis cuti; Cuti Sakit/Melahirkan/Alasan Penting/Besar/CLTN lintas Desember–Januari ditolak dengan pesan generik. Test regresi mencakup seluruh jenis non-tahunan dan resubmit. Verifikasi ulang menunjukkan perbaikan telah tersedia di `development` melalui `952f723` (`fix(cuti): tegakkan batas tahun kalender untuk semua jenis cuti`, PR #140). |
| #31 | Approval engine cuti dinamis | ✅ | Snapshot per request, data-scope approver, notifikasi, audit, pengurangan saldo final, QR proof, timeline, dokumen eksternal Kepala Lembaga, test E2E. Vocabulary legacy tuntas: `DeclineLeaveAction` + `NOT_APPROVED` + label `Tidak Disetujui`; arm `'rejected'` di `LeaveProofService` **dihapus PR #128** — grep `rejected/REJECT/Ditolak` sebagai status di `app/` = nol. |
| #32 | Saldo cuti & daftar cuti pegawai | ✅ | Balance ledger, bucket N-2/N-1/berjalan, seeder 2026, koreksi ber-audit, rollover scheduler, halaman/API saldo, test; jenis mutasi ledger dikunci test. |
| #44 | Daftar cuti pegawai + timeline approval | ✅ | Daftar + badge status + timeline vertikal tersedia. Label langkah aktif `Menunggu [nama step]` dari snapshot chain (**PR #119**, disempurnakan **PR #125**); kontrak dikunci `CutiListDisplayTest`. |

## Gap Terbuka

1. **P2 — Constraint DB penugasan kabag** (#27, opsional): unique/exclusion constraint agar tulis-langsung-DB tidak bisa membuat overlap.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 10 Agustus 2026 | Hardening issue #178 ditutup melalui PR #179 (`ff260a5`): invarian bentuk/kelayakan rantai dipusatkan pada writer bersama, konfigurasi diserialkan dengan lock deterministik, dan audit kritis mencatat aktor eksplisit. Status Sprint 4 tetap 8 ✅ · 0 ⚠️ · 0 ❌ karena follow-up ini bukan issue Sprint 4 baru. |
| 10 Agustus 2026 | #28/US-4.10 AC-2 ditutup PR #177 (`1fd99cb`): template rantai dapat diterapkan ke anggota unit sambil mempertahankan satu chain runtime per pegawai dan snapshot pengajuan berjalan. Sprint 4 menjadi 8 ✅ · 0 ⚠️ · 0 ❌ pada source. Issue #178 tetap follow-up hardening terpisah. |
| 22 Juli 2026 | Baseline audit: hanya #32 ✅; #26–#31 ⚠️. (Catatan: perbaikan #29 dan filter PPPK #30 sebenarnya sudah masuk sebelum audit — PR #116/#104 — tetapi belum tercermin di audit lama.) |
| 23 Juli 2026 | PR #119: label `Tidak Disetujui` + label langkah aktif → #44 dan sebagian #31 tertutup. |
| 24–25 Juli 2026 | PR #124: penugasan Kepala Bagian bertanggal efektif → #27 ✅. |
| 26 Juli 2026 | PR #128: kompatibilitas `rejected` dihapus → #31 ✅. Verifikasi HEAD `1b2e5b6` menetapkan status di atas. |
| 27 Juli 2026 | Perbaikan A dieksekusi: validasi lintas tahun seluruh jenis cuti (#30) + test E2E alur pengajuan→approval→potong saldo. Pendukung: penetapan Kepala Bagian inline + penataan ulang halaman konfigurasi approval + pencarian PYBMC mandiri (memperhalus alur #27/#28), serta penyamaan dokumen bukti tersimpan dengan formulir resmi (penyempurnaan #31). |
| 28 Juli 2026 | Verifikasi ulang terhadap `development` HEAD `478424f`: perbaikan lintas tahun seluruh jenis cuti telah terintegrasi melalui `952f723` (PR #140), termasuk validasi Store/Resubmit sebelum gate saldo/TMT dan regression test untuk jenis non-tahunan. **#30 ditutup menjadi ✅; status "menunggu review" dihapus.** Pada tanggal yang sama, keputusan K-SCHEMA-01 menetapkan `leave_request_steps`/`leave_balance_ledger`/`leave_proofs` sebagai schema cuti canonical; PRD §15.2 dan issue breakdown diselaraskan. **#26 ditutup menjadi ✅.** |
| 5 Agustus 2026 | Konflik penggabungan dokumen pada bagian Ringkasan, Gap Terbuka, dan Riwayat Perubahan diselesaikan. Resolusi mengikuti [Keputusan Skema Cuti Canonical](../Keputusan-Skema-Cuti-Canonical.md) yang sudah disetujui dan sudah tercermin pada PRD butir 18 serta §15.2, dan pada tugas 7.2-12 tracker Sprint 7. Konsekuensinya: hitungan resmi menjadi **7 ✅ · 1 ⚠️ · 0 ❌**, dan butir gap "ADR skema cuti" dihapus karena keputusannya sudah terbit. Verifikasi ulang terhadap `development` @ `4839ab6`: nama tabel canonical dipakai runtime dan tidak ada sisa nama lama pada `app/`, `database/`, maupun `tests/`; #28 tetap ⚠️ karena `leave_approval_chains` masih hanya bercakup `employee_id` tanpa `scope_type`/`unit_kerja_id` dan resolver masih memilih chain per pegawai. |
