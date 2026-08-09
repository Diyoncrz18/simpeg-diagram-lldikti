# Tracking Sprint 5 — EWS & Notifikasi

| Field | Detail |
|---|---|
| Periode | 21 – 30 Juli 2026 |
| Cakupan issue | #33 – #38, #49 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 10 Agustus 2026 |
| Basis verifikasi | Branch `development` @ `54ab90d` setelah PR #176 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**4 ✅ · 3 ⚠️ · 0 ❌ pada source.** Issue #36 telah selesai melalui PR #173/#167. Perilaku canonical flag kinerja ialah alert kenaikan pangkat tetap disimpan dan tampil sebagai tidak eligible, sementara notifikasi in-app/email ditahan dan `notified_at` tetap kosong; rumusan lama “alert tidak dibuat” tidak lagi berlaku.

### Rekonsiliasi issue #36 — 10 Agustus 2026

- ✅ PR #173 (`7e831c1`) menutup perilaku flag kinerja dan penahanan pengingat untuk seluruh penerima tanpa menghilangkan alert dari EWS.
- ✅ Teks bantuan kinerja diselaraskan dan ikut diverifikasi pada rangkaian PR #173/#167.
- Baris audit 26 Juli di bawah dipertahankan sebagai kronologi; status aktif issue #36 adalah selesai pada source.

## Arsip Status per Issue — snapshot 26 Juli 2026

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #33 | Kalkulasi TMT otomatis | ⚠️ | Sentralisasi tercapai (**PR #120**): `TmtCalculatorService` dipanggil dari 5 titik saat riwayat/sumber resmi disimpan; import tidak memanggilnya (sesuai keputusan 22 Juli, dikunci test PR #121). **Gap tersisa:** (a) milestone Satyalancana masih dihitung ephemeral di loop scheduler, tidak disimpan konsisten; (b) perubahan referensi BUP (`ref_jabatan.default_bup` / `ref_jenis_jabatan.maks_usia_pensiun`) tidak memicu rekalkulasi — tidak ada observer, dan guard `if (tanggal_pensiun === null)` membuat nilai lama tidak pernah dikoreksi. |
| #34 | EWS scheduler harian | ⚠️ | Engine 5 trigger, threshold configurable, unique alert, log run, error notification, command, test tersedia. **Gap P0 terverifikasi:** `app:run-ews` terdaftar **ganda** — `bootstrap/app.php:20-26` DAN `routes/console.php:27-29` → berisiko jalan 2×/hari; test `EwsCommandScheduleTest` memakai `->first()` sehingga duplikat lolos regresi. Gap lain: kolom `is_eligible` & `tahun` (milestone) tidak ada di `ews_alerts` (eligibility dihitung ulang tiap render; `interval_days` ≈ `trigger_days` sudah oke). Penerima in-app Admin Kepegawaian kini **sudah** menerima (PR #122). |
| #35 | Halaman daftar EWS aktif | ✅ | Data nyata, sort urgensi, filter event/status, warna, detail pegawai, akses role, follow-up ber-catatan + audit, test role. |
| #36 | Flag kinerja & kelayakan Satyalancana | ⚠️ | Toggle AJAX, FormRequest, Action, RBAC route, audit, flag/catatan Satyalancana, test tersedia. **Gap P0 (konflik PRD) terverifikasi:** saat `is_kinerja_baik=false`, `EwsEngineService` **tetap membuat** alert pangkat dan menampilkannya "Tidak Eligible" di admin/kabag/pimpinan — PRD §10.3 menyatakan pegawai tidak muncul di EWS pangkat. Guard `isNonEligiblePromotion()` di resolver efektif dead code (engine tidak mengirim `is_eligible` di data). Minor: teks helper belum memakai frasa AC ("Flag ini menggantikan penilaian SKP yang belum tersedia di Fase 1…"). |
| #37 | Notifikasi email | ✅ | **Ditutup PR #122.** Tabel baru `notification_event_channels` (kebijakan per event × channel, seeded di migrasi) + `NotificationChannelResolver` fail-closed dua lapis (kebijakan event AND channel global); `emailEnabled()` kini murni delegasi DB — array hardcoded dihapus; `ews.satyalancana` terdaftar in-app **dan** email; Admin Kepegawaian menerima in-app EWS; queue 3× retry + template Bahasa Indonesia tetap. Catatan: UI admin untuk mengelola kebijakan ini belum ada — dicakup Issue #46 (Sprint 6). |
| #38 | Session timeout | ⚠️ | Middleware idle 30 menit, invalidasi sesi, redirect Keycloak, JSON 401, audit `SESSION_TIMEOUT`, test tersedia. **Gap kecil:** `.env.example` masih `SESSION_LIFETIME=120` (cookie hidup 120 menit vs idle 30) — samakan ke 30 atau dokumentasikan alasan dua nilai. |
| #49 | EWS pribadi (pegawai) | ✅ | Section EWS pribadi di dashboard pegawai memakai data real ber-scope pegawai login + `DashboardEwsTest`; ditata ulang PR #123. |

## Gap Terbuka (urutan prioritas)

1. **P0 — Hapus registrasi ganda `app:run-ews`** (#34): pertahankan satu registrasi (versi configurable `EwsConfig`), perkuat test agar menghitung jumlah event terdaftar.
2. **P1 — Kolom snapshot `ews_alerts`** (#34): verifikasi lanjutan konsistensi snapshot milestone Satyalancana.
3. **P1 — TMT lanjutan** (#33): simpan milestone Satyalancana konsisten; rekalkulasi saat referensi BUP berubah; tinjau guard skip `tanggal_pensiun` (hormati keputusan import 22 Juli: nilai import tidak ditimpa).
4. **P2 — dokumentasi `.env.example`** (#38): jelaskan `SIMPEG_SESSION_IDLE_TIMEOUT=30` sebagai batas idle yang ditegakkan/audited dan `SESSION_LIFETIME` yang lebih besar sebagai jaring pengaman.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 10 Agustus 2026 | #36 ditutup pada source oleh PR #173 (`7e831c1`) dan penyelarasan UI terkait PR #167: alert tetap ada, tetapi notifikasi ditahan saat tidak eligible. Rumusan lama yang meminta alert tidak dibuat dinyatakan superseded oleh keputusan canonical. |
| 22 Juli 2026 | Baseline audit: #35 ✅; #33/#34/#36/#37/#38 ⚠️. Reklasifikasi: kalkulasi TMT pasca-import sengaja tidak ada (keputusan import). |
| 23 Juli 2026 | PR #122: kebijakan channel per event berbasis DB → #37 ✅; in-app Admin untuk EWS tertutup (sebagian #34). PR #123: EWS pribadi dashboard pegawai ditata ulang → #49 dikonfirmasi ✅. |
| 26 Juli 2026 | Verifikasi HEAD `1b2e5b6`: double scheduler (#34) dan perilaku flag kinerja (#36) dikonfirmasi belum berubah — keduanya P0 tersisa sprint ini. |
