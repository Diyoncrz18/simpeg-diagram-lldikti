# Keputusan Arsitektur — Skema Cuti Canonical

| Field | Detail |
|---|---|
| Tanggal keputusan | 28 Juli 2026 |
| Status | **Disetujui** — menjadi acuan penamaan tabel fisik cuti Fase 1 sampai ada keputusan yang lebih baru |
| Disetujui melalui | Persetujuan pengguna pada task Codex: “oke sekarang lanjutkan implementasi opsi 1 dan pastikan sesuai dengan document yang ada dan buatkan di branch berbeda” |
| Acuan utama | `PRD-SIMPEG-Fase1-Core.md` §15.2; `Panduan-Penulisan-Kode-SIMPEG.md`; `AGENTS.md`; migration dan model runtime SIMPEG |
| Cakupan | Penetapan nama tabel fisik canonical untuk snapshot approval pengajuan, ledger saldo cuti, dan bukti formulir/QR cuti |
| Di luar cakupan | Rename tabel, perubahan data yang sudah ada, perubahan alur approval chain per unit, serta desain penyimpanan dokumen eksternal baru |

---

## K-SCHEMA-01 — Nama tabel runtime menjadi canonical

### Keputusan

Nama tabel yang sudah digunakan oleh migration, model, relasi, dan test runtime SIMPEG ditetapkan sebagai **nama tabel fisik canonical** untuk Fase 1:

| Konsep data | Nama tabel canonical | Referensi lama yang tidak lagi dipakai sebagai nama fisik |
|---|---|---|
| Snapshot langkah approval per pengajuan | `leave_request_steps` | `leave_approval_steps` |
| Ledger mutasi saldo cuti tahunan yang append-only | `leave_balance_ledger` | `leave_balance_adjustments` |
| Bukti formulir cuti resmi dan token QR | `leave_proofs` | `leave_documents` |

Nama lama pada kolom ketiga adalah istilah dokumentasi historis, **bukan** alias runtime, view kompatibilitas, atau tabel yang boleh dipakai oleh migration dan query baru.

### Dasar keputusan

1. Migration fondasi runtime membuat `leave_request_steps`, `leave_balance_ledger`, dan `leave_proofs`; tidak ada migration yang membuat tiga nama tabel lama.
2. Model `LeaveRequestStep`, `LeaveBalanceLedger`, dan `LeaveProof`, beserta relasi `LeaveRequest`, memakai tiga tabel canonical tersebut.
3. `CutiFoundationSchemaTest` mengunci keberadaan kolom inti dan relasi tabel canonical sebagai regression contract.
4. Rename tabel yang sudah dipakai akan menambah risiko foreign key, riwayat migration, audit trail, data pengajuan, dan deployment tanpa memperbaiki kebutuhan bisnis.

### Konsekuensi

1. PRD §15.2, issue breakdown yang masih menyebut nama fisik lama, dan tracker Sprint 4 harus menggunakan nama canonical di atas.
2. Tidak ada perubahan migration, model, data, route, payload API, maupun perilaku aplikasi sebagai akibat keputusan ini.
3. `leave_request_steps` menyimpan snapshot approver dan keputusan per langkah untuk satu pengajuan; perubahan template chain tidak mengubah riwayat request lama.
4. `leave_balance_ledger` menyimpan seluruh event mutasi saldo yang diaudit, termasuk hak tahunan, carry-over, pemotongan final, dan koreksi manual. Ia lebih luas daripada konsep koreksi saldo manual semata.
5. `leave_proofs` menyimpan bukti formulir resmi yang telah diterbitkan, token QR, path/mime PDF, serta metadata snapshot. Ia bukan tabel arsip umum untuk semua dokumen kepegawaian.
6. Kebutuhan bisnis dokumen eksternal pada alur `external_approval` tidak dihapus oleh keputusan ini. Runtime saat ini tidak memiliki kolom `external_approval` atau `external_document_path` pada `leave_proofs`; bila kebutuhan tersebut akan diaktifkan, desain storage, migration, otorisasi, audit, dan retensi harus disetujui dalam keputusan terpisah sebelum implementasi.
7. Pada saat keputusan ini dibuat, approval chain per unit masih menjadi gap terpisah Sprint 4 #28. Status terbaru: K-US-01 menetapkan tepat satu chain runtime per pegawai dengan unit sebagai target penyalinan template, dan implementasinya selesai melalui PR #177 (`1fd99cb`) tanpa mengubah schema `leave_approval_chains` menjadi scoped per unit. Hardening invarian pada seluruh writer bersama kemudian selesai melalui PR #179 (`ff260a5`) tanpa mengubah keputusan schema tersebut.

### Batas penerapan

1. Semua migration atau kode baru yang merujuk snapshot approval, ledger saldo, atau bukti QR wajib memakai nama canonical.
2. Perubahan nama fisik pada masa depan wajib memiliki ADR/keputusan baru, inventaris foreign key dan data, strategi migrasi/rollback PostgreSQL, serta regression test.
3. Dokumentasi boleh memakai istilah bisnis “langkah approval”, “koreksi saldo”, atau “dokumen cuti”, tetapi bila menyebut tabel fisik harus memakai nama canonical dalam keputusan ini.

### Verifikasi keputusan

- Migration `2026_07_06_000000_add_cuti_revision_foundation.php` membuat tiga tabel canonical.
- `LeaveBalanceLedger` secara eksplisit menetapkan `$table = 'leave_balance_ledger'`.
- `CutiFoundationSchemaTest` memeriksa `leave_request_steps`, `leave_balance_ledger`, dan `leave_proofs` beserta kolom inti masing-masing.
- Pencarian source runtime SIMPEG tidak menemukan penggunaan untuk `leave_approval_steps`, `leave_balance_adjustments`, atau `leave_documents`.

---

## Dampak terhadap dokumen sumber

1. PRD §15.2 memakai nama tabel canonical dan menggambarkan QR/PDF pada `leave_proofs`, bukan pada `leave_requests`.
2. `Issues-SIMPEG-Fase1.md` tetap berfungsi sebagai breakdown pekerjaan, tetapi referensi nama tabel fisiknya diselaraskan agar tidak bertentangan dengan PRD dan runtime.
3. Tracker Sprint 4 #26 ditutup setelah keputusan ini dan sinkronisasi dokumen selesai.
