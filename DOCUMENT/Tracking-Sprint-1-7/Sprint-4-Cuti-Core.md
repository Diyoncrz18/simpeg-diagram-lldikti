# Tracking Sprint 4 — Cuti Core

| Field | Detail |
|---|---|
| Periode | 11 – 20 Juli 2026 |
| Cakupan issue | #26 – #32, #44 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 26 Juli 2026 |
| Basis verifikasi | Source HEAD `1b2e5b6` + verifikasi kode 26 Juli 2026 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**5 ✅ · 3 ⚠️ · 0 ❌.** Sprint ini paling banyak membaik sejak audit 22 Juli: penugasan Kepala Bagian (PR #124/#128), kontrak kalkulasi hari kerja (PR #116), filter jenis cuti PPPK (PR #104), label langkah aktif (PR #119/#125), dan penghapusan vocabulary legacy `rejected` (PR #128) semuanya sudah tertutup. Sisa gap: chain per unit, validasi lintas tahun non-tahunan, dan ADR skema cuti.

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #26 | Migration tabel cuti | ⚠️ | Tabel/model chain, snapshot step, balance ledger, QR proof tersedia & diuji. **Gap dokumentasi (P0 dok):** runtime memakai `leave_request_steps` / `leave_balance_ledger` / `leave_proofs`, PRD §15.2 menyebut `leave_approval_steps` / `leave_balance_adjustments` / `leave_documents`. Kode 100% konsisten & dikunci `CutiFoundationSchemaTest`; nama alternatif nol hit di repo — yang belum ada hanyalah **ADR/keputusan tertulis** yang mengesahkan penamaan. |
| #27 | Assign Kepala Bagian per pegawai | ✅ | **Ditutup PR #124 (+#128).** Route web+API kini `role:super_admin,admin_kepegawaian` + `permission:employees.update`; `effective_date` wajib & bisa diisi pengguna; overlap ditolak + interval lama ditutup H-1 dalam transaksi + `lockForUpdate`; 15 test regresi (`SupervisorAssignmentTest`). Catatan: enforcement satu kabag aktif di level aplikasi — belum ada unique/exclusion constraint DB. |
| #28 | Konfigurasi approval chain | ⚠️ | Chain per pegawai, snapshot step, PYBMC global, backfill, audit, skip duplikat tersedia; step type kini `kepala_bagian`/`verifier`/`pybmc`. **Gap:** scope **per unit** belum ada (tanpa `scope_type`/`unit_kerja_id`; resolver hanya `employee_id`). Butuh keputusan produk precedence global–unit–pegawai sebelum implementasi (`AGENTS.md`: escalate, do not guess). |
| #29 | Kalkulasi hari kerja otomatis | ✅ | **Ditutup PR #116.** Blade kini membaca `result.data?.jumlah_hari_kerja` + merender `warnings` (`aria-live`), plus guard race-condition `workdayRequestId`. `WorkdayCalculator`, endpoint API, test unit/feature tersedia. |
| #30 | Form pengajuan cuti | ⚠️ | Form, lampiran tervalidasi, hitung server-side, saldo, snapshot chain, notifikasi, audit, test tersedia. Dropdown jenis cuti kini menyaring PNS/PPPK dari metadata `khusus_pns` (**ditutup PR #104**) dengan validasi server sebagai lapis kedua. **Gap tersisa:** validasi lintas tahun kalender hanya berlaku untuk cuti yang memotong saldo tahunan (cek bersarang di `validateSaldoTahunan()` pada Store & Resubmit request) — Cuti Sakit/Melahirkan/Alasan Penting/Besar/CLTN masih bisa diajukan Des–Jan, padahal PRD §9.4 melarang untuk semua jenis. |
| #31 | Approval engine cuti dinamis | ✅ | Snapshot per request, data-scope approver, notifikasi, audit, pengurangan saldo final, QR proof, timeline, dokumen eksternal Kepala Lembaga, test E2E. Vocabulary legacy tuntas: `DeclineLeaveAction` + `NOT_APPROVED` + label `Tidak Disetujui`; arm `'rejected'` di `LeaveProofService` **dihapus PR #128** — grep `rejected/REJECT/Ditolak` sebagai status di `app/` = nol. |
| #32 | Saldo cuti & daftar cuti pegawai | ✅ | Balance ledger, bucket N-2/N-1/berjalan, seeder 2026, koreksi ber-audit, rollover scheduler, halaman/API saldo, test; jenis mutasi ledger dikunci test. |
| #44 | Daftar cuti pegawai + timeline approval | ✅ | Daftar + badge status + timeline vertikal tersedia. Label langkah aktif `Menunggu [nama step]` dari snapshot chain (**PR #119**, disempurnakan **PR #125**); kontrak dikunci `CutiListDisplayTest`. |

## Gap Terbuka

1. **P1 — Validasi lintas tahun untuk semua jenis cuti** (#30): angkat cek tahun keluar dari `validateSaldoTahunan()` di `StoreLeaveRequestRequest` & `ResubmitLeaveRequestRequest`; test lintas tahun untuk jenis non-tahunan.
2. **P1 — Approval chain per unit** (#28): **eskalasi keputusan produk dulu** (model precedence pegawai > unit > global, perilaku hierarki unit), catat di dokumen, baru migration + resolver + UI + test.
3. **Dokumentasi — ADR skema cuti canonical** (#26): sahkan penamaan `leave_request_steps`/`leave_balance_ledger`/`leave_proofs` dan selaraskan PRD §15.2.
4. **P2 — Constraint DB penugasan kabag** (#27, opsional): unique/exclusion constraint agar tulis-langsung-DB tidak bisa membuat overlap.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 22 Juli 2026 | Baseline audit: hanya #32 ✅; #26–#31 ⚠️. (Catatan: perbaikan #29 dan filter PPPK #30 sebenarnya sudah masuk sebelum audit — PR #116/#104 — tetapi belum tercermin di audit lama.) |
| 23 Juli 2026 | PR #119: label `Tidak Disetujui` + label langkah aktif → #44 dan sebagian #31 tertutup. |
| 24–25 Juli 2026 | PR #124: penugasan Kepala Bagian bertanggal efektif → #27 ✅. |
| 26 Juli 2026 | PR #128: kompatibilitas `rejected` dihapus → #31 ✅. Verifikasi HEAD `1b2e5b6` menetapkan status di atas. |
