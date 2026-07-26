# Tracking Sprint 7 — Stabilization, Regression & UAT

| Field | Detail |
|---|---|
| Periode | 10 – 20 Agustus 2026 (gate akhir: release candidate siap, go-live 20 Agustus) |
| Cakupan issue | #45, #47, #48, #50, #52 (`Issues-SIMPEG-Fase1.md`) + jendela bugfix seluruh sisa temuan Sprint 1–6 |
| Slice | 7.1 Audit view + redirect + polish P1 (hari 1–4) · 7.2 Bugfix backlog (hari 5–8) · 7.3 Full regression + UAT + RC (hari 9–15) |
| Pembaruan terakhir | 26 Juli 2026 (pra-sprint; status awal per verifikasi source HEAD `e82b527`) |
| Basis verifikasi | `development` pasca-merge PR #127–#131 |
| Acuan | User-Stories US-7.2/7.3, US-1.5, US-6.2/6.4, US-4.8/4.9 · Tracking-Sprint-Vertical-Slice §10 · Tim-dan-Pembagian-Tugas §4 Sprint 7 |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum. Status source ≠ tracker `Done` (tetap butuh review, QA/retest, evidence).

## Ringkasan Kondisi Awal (pra-sprint, 26 Juli)

Kabar baik: **3 dari 4 issue fitur Sprint 7 sudah jadi lebih awal** — #45 (daftar cuti admin + kelola saldo) dan #48 (halaman notifikasi + tandai dibaca) ✅ di source; #50 (redirect per role) berjalan tetapi belum dikunci test formal. Beban sprint bergeser ke tempat yang benar: **#47 (audit view masih ⚠️), jendela bugfix backlog, dan #52 regression/UAT** yang memang inti sprint ini.

## Status Awal per Issue

| Issue | Deliverable | Status awal | Bukti & sisa pekerjaan |
|---:|---|:---:|---|
| #45 | Daftar cuti admin view + kelola saldo (US-4.8, US-4.9) | ✅ | Monitoring cuti admin (scope `cuti.read_all`, admin bukan approver), rekap saldo + ledger, koreksi manual ber-alasan + audit, rollover `cuti:rollover` terjadwal — semuanya tersedia & teruji. Sisa: QA formal AC + evidence, lalu tracker `Done`. |
| #47 | Halaman audit log + diff view (US-7.2, US-7.3) | ⚠️ | Halaman + filter + drawer diff tersedia dengan data `audit_logs` nyata. **Gap:** seluruh record dimuat `->get()` lalu difilter/paginasi Alpine di browser (tidak skalabel), dan `old_values`/`new_values` dikirim mentah tanpa masking. Perbaikan bergantung task masking (7.2-1). |
| #48 | Halaman semua notifikasi + tandai dibaca (US-6.2, US-6.4) | ✅ | Route `notifications.index` + permission, paginator backend, tandai dibaca per item & semua, ownership per user, `NotificationInboxTest`. Sisa: verifikasi AC badge berkurang tanpa reload + QA formal. |
| #50 | Redirect berdasarkan role (US-1.5) | ⚠️ | Callback SSO mengarah ke `route('dashboard')` yang mendispatch view/redirect per role (admin/pimpinan/kabag/pegawai — kabag dikunci `KepalaBagianRouteGateTest`). Sisa: feature test eksplisit kelima role dari callback + samakan interpretasi AC (redirect vs dispatcher). |
| #52 | Full regression test + UAT | ❌ | Belum dimulai — inti Sprint 7. 14 skenario E2E + evidence + sign-off (rincian di Slice 7.3). |

---

## Daftar Task Sprint 7

### Slice 7.1 — Audit View, Redirect, Polish P1 (hari 1–4)

| # | Task | Owner | Detail & kriteria selesai |
|---|---|---|---|
| 7.1-1 | **#47: Audit log server-side** | Adriel (UI) + Jordan (query) | Ganti `->get()` + filter Alpine dengan Action/query berpaginasi server-side (filter event/user/modul/periode via query param); payload `old_values`/`new_values` dirender **setelah masking** (bergantung 7.2-1). Test: filter benar, pagination, data sensitif tidak tampil. |
| 7.1-2 | **#50: Kunci redirect per role** | Adriel | Feature test: login callback tiap 5 role berakhir di dashboard yang benar (Super Admin/Admin → Dashboard Admin; Pimpinan → `pimpinan.dashboard`; Kabag → `kepala-bagian.dashboard`; Pegawai → dashboard pribadi). Bila implementasi dispatcher `/dashboard` dipertahankan, catat sebagai interpretasi resmi AC US-1.5. |
| 7.1-3 | **#48: Verifikasi formal AC notifikasi** | Adithian + Adriel | Ceklis AC US-6.2/6.4: badge berkurang tanpa reload penuh, tandai per item tanpa redirect, tandai semua, pagination. Tambal test yang kurang; QA evidence. |
| 7.1-4 | **#45: QA formal cuti admin + saldo** | Jordan (owner) + Grantly (QA) | Ceklis AC US-4.8 (admin monitor-only, tanpa tombol keputusan) & US-4.9 (koreksi ber-alasan ter-audit; rollover 1 Jan: carry-over maks 6, hak 24 hari bila N-2/N-1 kosong). Evidence QA → tracker `Done`. |

### Slice 7.2 — Jendela Bugfix Backlog Sprint 1–6 (hari 5–8)

Prioritas P0 — **wajib tuntas sebelum UAT dimulai**:

| # | Task | Asal temuan | Owner usulan |
|---|---|---|---|
| 7.2-1 | Masking NIK/No. KK pada payload audit pegawai (tulis & baca; `UpdateEmployeeAction` masih `toArray()`), + test regresi "NIK tidak pernah muncul di audit" | Sprint 1 #5 (P0) | Jordan |
| 7.2-2 | Guard immutable model `AuditLog` (tolak `update`/`delete` via booted() + test) | Sprint 1 #5 (P0) | Jordan |
| 7.2-3 | **Verifikasi dulu, jangan kerja dobel:** dampak PR #127 terhadap (a) registrasi ganda scheduler `app:run-ews` — `withSchedule` di bootstrap sudah dihapus, pastikan `routes/console.php` kini satu-satunya registrasi; (b) kolom `is_eligible` + `satyalancana_years` di `ews_alerts`; (c) perilaku flag kinerja false vs PRD §10.3; (d) milestone Satyalancana/rekalkulasi BUP. Hasil verifikasi menentukan sisa pekerjaan EWS yang sesungguhnya | Sprint 5 #33/#34/#36 | Grantly |
| 7.2-4 | Selaraskan flag kinerja false dengan PRD (alert pangkat tidak dibuat) — **bila** verifikasi 7.2-3 menyatakan masih gap | Sprint 5 #36 (P0) | Grantly |

Prioritas P1:

| # | Task | Asal temuan | Owner usulan |
|---|---|---|---|
| 7.2-5 | Hubungkan Hari Libur web ke database (controller web masih array statis + audit session; reuse Action/FormRequest API yang sudah benar) | Sprint 1 #10 | Jordan |
| 7.2-6 | Validasi lintas tahun kalender untuk SEMUA jenis cuti (kini hanya cuti pemotong saldo; angkat keluar dari `validateSaldoTahunan()` di Store & Resubmit request) | Sprint 4 #30 | Jordan |
| 7.2-7 | Amankan logout: hapus route GET `/logout` yang memutasi session tanpa CSRF | Sprint 1 #3 | Adriel |
| 7.2-8 | Import manual column mapping + warning kolom tak dikenal + FormRequest validate/execute (sisa terakhir Sprint 3 #21; rencana rinci sudah ada). Sekalian putuskan nasib jalur "skip NIP duplikat" yang dead code (rule `unique` menangkap lebih dulu) — hidupkan atau revisi AC US-3.3 | Sprint 3 #21 | Grantly (BE) + Adithian (UI) |
| 7.2-9 | Perluas audit fail-closed (`logOrFail()` baru 3/36 call-site → terapkan pada mutation wajib: keputusan cuti, CRUD pegawai, import) + dokumentasikan kebijakan | Sprint 1 #5 | Jordan |
| 7.2-10 | Carry-over Sprint 6 yang belum `Done` per 9 Agustus (cek `Sprint-6-Dashboard-dan-Laporan.md`): prioritas dashboard Admin real-data, PDF pegawai, penyelarasan export cuti, CRUD reference tables | Sprint 6 #39–#46 | sesuai pembagian Sprint 6 |

Prioritas P2 & dokumen:

| # | Task | Owner usulan |
|---|---|---|
| 7.2-11 | Item kecil: `SESSION_LIFETIME` .env.example → 30 (atau dokumentasikan), `resources/views/components/README.md`, teks helper flag kinerja sesuai AC | Adriel |
| 7.2-12 | ADR skema cuti canonical (`leave_request_steps`/`leave_balance_ledger`/`leave_proofs`) + selaraskan PRD §15.2 | Dion |
| 7.2-13 | Eskalasi keputusan produk: approval chain per unit (precedence pegawai > unit > global) — keputusan saja, implementasi di luar Fase 1 bila waktu tidak cukup | Dion |
| 7.2-14 | Bereskan hygiene lokal yang mengganjal `composer qa`: perubahan menggantung `bootstrap/app.php` (formatting) & `test_show.php` — commit/buang agar gate QA lokal hijau | Dion |

### Slice 7.3 — Full Regression, UAT, Release Candidate (hari 9–15)

| # | Task | Owner | Detail |
|---|---|---|---|
| 7.3-1 | **Full regression 14 skenario** (Issue #52): Auth SSO→mapping→redirect→logout→timeout · CRUD pegawai · riwayat append-only + `is_latest` + TMT · import template→upload→validasi→eksekusi→laporan persisten · cuti E2E sampai QR + saldo berkurang · cuti edge cases (saldo habis, weekend, lintas tahun, PPPK, `Perubahan`/`Ditangguhkan`/`Tidak Disetujui`) · EWS scheduler manual + no-duplicate + follow-up + Satyalancana · dashboard 4 role · export Excel/PDF pegawai & cuti · audit log + diff · notifikasi in-app + email (Mailpit) · responsive Chrome/Firefox/Edge desktop+tablet · RBAC per role · soft delete & restore | Grantly | Setiap skenario dicatat Pass/Fail + evidence; bug masuk board dengan severity. |
| 7.3-2 | Verifikasi PostgreSQL 17 menyeluruh (migration baru `import_batches` + EWS #127, UUID, JSON, FK, transaksi) — SQLite lokal bukan bukti cukup | Grantly | Jalankan suite penuh pada container PostgreSQL 17; catat hasil. |
| 7.3-3 | Bugfix hasil regression/UAT — owner sesuai area, PR kecil per bug, retest oleh Grantly | Semua (routing: Adriel) | Tidak ada bug critical/major terbuka sebelum RC. |
| 7.3-4 | Koordinasi UAT dengan LLDIKTI: jadwal, skenario demo, data demo, pencatatan feedback | Dion | Daftar issue UAT terprioritaskan. |
| 7.3-5 | Release gate: tahan merge tanpa evidence, bersihkan branch `develop`, siapkan RC | Adriel | Branch siap RC. |
| 7.3-6 | Handover: runbook setup, daftar known issues, QA summary, release note; finalisasi seluruh file `Tracking-Sprint-1-6/` (semua status terkonfirmasi merged/Done) | Dion + Grantly | Dokumen final siap. |

## Definition of Done Sprint 7 (dari tracker vertical slice)

- [ ] Semua P0 dan P1 kritis sudah merge ke `development`.
- [ ] Tidak ada bug critical/major terbuka.
- [ ] Full regression selesai dengan evidence (termasuk PostgreSQL 17).
- [ ] UAT selesai atau daftar issue UAT sudah diprioritaskan.
- [ ] Release candidate siap untuk deployment preparation (menunggu server/domain/credential dari LLDIKTI).

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 26 Juli 2026 | File dibuat pra-sprint. Status awal: #45 ✅, #48 ✅, #50 ⚠️ (perlu test formal), #47 ⚠️ (pagination server + masking), #52 ❌. Daftar task disusun dari Issues #45–#52 + backlog terverifikasi Sprint 1–6; item EWS ditandai "verifikasi dulu" karena PR #127 kemungkinan sudah menutup sebagian. |
