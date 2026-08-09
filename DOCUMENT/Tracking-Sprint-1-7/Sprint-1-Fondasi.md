# Tracking Sprint 1 — Fondasi

| Field | Detail |
|---|---|
| Periode | 8 – 20 Juni 2026 |
| Cakupan issue | #1 – #12 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 10 Agustus 2026 |
| Basis verifikasi | Branch `development` @ `54ab90d` setelah PR #176 |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` (dokumen monolitik lama, dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian (ada gap requirement/kualitas) · ❌ belum selesai/tidak sesuai. Ikon menyatakan status source; status tracker `Done` tetap membutuhkan review PR, QA/retest, dan evidence.

## Ringkasan

**7 ✅ · 5 ⚠️ · 0 ❌.** Fondasi (auth, notifikasi, design system, reference tables, testing framework) berdiri. Masking NIK/No. KK dan immutability audit telah ditutup PR #174. Issue #5 tetap ⚠️ hanya karena perluasan kebijakan audit fail-closed untuk CRUD pegawai/import belum selesai menyeluruh.

### Rekonsiliasi issue #5 — 10 Agustus 2026

- ✅ PR #174 (`037e137`) menyelesaikan masking NIK/No. KK pada tulis dan baca audit serta menegakkan immutability melalui guard model dan trigger PostgreSQL.
- ⚠️ PR #176 (`54ab90d`) menyelesaikan event wajib US-7.1 AC-1 dan memperkeras mutasi yang disentuh, tetapi belum menutup seluruh CRUD pegawai/import maupun dokumentasi kebijakan dan regression test rollback.
- Baris status issue di bawah dipertahankan sebagai kronologi audit sebelum PR #174/#176; rekonsiliasi ini yang berlaku untuk status terkini.

## Arsip Status per Issue — snapshot sebelum PR #174/#176

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #1 | Setup Laravel & environment | ⚠️ | Laravel 12, PostgreSQL 17, queue, compose, Pint, PHPStan, helper Podman tersedia. Sisa: PHP lokal 8.3 vs CI 8.4, evidence deployment Podman production belum ada. |
| #2 | Keycloak SSO & middleware | ⚠️ | Controller/Action callback-logout-redirect, middleware, config Socialite, test tersedia. Sisa: login IdP nyata belum diuji (credential belum diterima); keputusan bootstrap first-mapped employee perlu didokumentasikan. |
| #3 | Logout & session management | ⚠️ | Logout POST + invalidasi session + audit tersedia. **Gap terverifikasi:** route GET `/logout` (`routes/web.php:54-55`) tetap memutasi session tanpa CSRF — hapus route GET atau jadikan halaman konfirmasi POST. |
| #4 | Mapping user Keycloak & RBAC | ✅ | Ditutup commit #109; diperkuat besar-besaran PR #126 (`users.employee_id` canonical, strict audit rollback, uniqueness PostgreSQL `23505`). QA UA-40…47 lulus (`Bukti-QA-Kelola-Akses-User-Super-Admin.md`). |
| #5 | Audit log | ⚠️ | Migration, model, `AuditService`, audit auth & mutasi tersedia. **Gap terverifikasi (P0):** (a) NIK/No. KK bisa tersimpan plaintext — `UpdateEmployeeAction` memakai `toArray()` untuk old/new values, cast `encrypted` terdekripsi saat serialisasi, tanpa test masking; (b) model `AuditLog` tanpa guard `updating`/`deleting`; (c) `AuditService::log()` fail-open (varian strict `logOrFail()` baru ada sejak PR #124 dan baru dipakai 3 dari 36 call-site). |
| #6 | Notifikasi in-app backend | ✅ | Migration/model, service, action inbox/read/unread, controller API, test tersedia. |
| #7 | Bell icon notifikasi | ✅ | Komponen bell, endpoint, dan JavaScript aplikasi tersedia. |
| #8 | Design system & layout master | ✅ | Layout, komponen UI/form reusable, Tailwind/Vite, UI Bahasa Indonesia. |
| #9 | Reusable Blade components | ⚠️ | Komponen utama tersedia. **Gap terverifikasi:** `resources/views/components/README.md` tidak ada; `design-system.md:984-1000` stale (mendokumentasikan struktur folder yang tidak ada). |
| #10 | CRUD Hari Libur & cuti bersama | ✅ | API v1 + Action + FormRequest + audit + `HariLiburCrudTest` sudah benar. Ditutup PR #166 (`9be633d`): controller web menjadi adapter HTTP tipis, data statis dan audit session `dynamic_audit_logs` dihapus, mutasi web memakai FormRequest + Action DB dengan audit resmi, serta filter/pencarian/paginasi dijalankan di database. Evidence: `HariLiburWebPageTest` (23 test) termasuk regresi dampak pada kalkulasi hari kerja cuti, regresi PostgreSQL 17, dan smoke test browser Super Admin 7 Agustus 2026. |
| #11 | Migration/seeder reference tables | ✅ | PR #118: hierarchy `ref_unit_kerja`, lifecycle jabatan (`default_bup`, `is_active`), katalog 10 status pegawai, `ref_notification_channels`; seeder idempoten + test. |
| #12 | Setup testing framework | ✅ | PHPUnit + Dusk, smoke test browser, helper, `phpunit.dusk.xml`, dokumentasi; evidence PostgreSQL 17 dari test fokus + CI. |

## Gap Terbuka (urutan prioritas)

1. **P1 — Logout GET tanpa CSRF** (#3).
2. **P1 — Perluas audit fail-closed** (#5): tetapkan daftar mutation wajib `logOrFail()` untuk sisa CRUD pegawai/import, dokumentasikan kebijakan, dan tambah regression test rollback.
3. **P2 — README komponen** (#9) dan perapian `design-system.md`.
4. **Proses** — samakan PHP lokal/CI, uji login IdP nyata saat credential diterima, evidence Podman production.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 10 Agustus 2026 | PR #174 menutup masking identitas dan immutability audit. PR #176 menutup event wajib US-7.1 AC-1 serta memperluas strict audit secara parsial. Issue #5 tetap ⚠️ hanya untuk hardening fail-closed CRUD pegawai/import dan kebijakan/test rollback. |
| 22 Juli 2026 | Baseline audit (HEAD `9a27caa`): #4/#6/#7/#8/#11/#12 ✅, sisanya ⚠️. |
| 23–26 Juli 2026 | #4 diperkuat PR #126 + QA lulus penuh. `logOrFail()` (audit strict) tersedia sejak PR #124 — #5 tetap ⚠️ karena adopsi masih sempit dan masking NIK/No. KK belum ada. Verifikasi 26 Juli mengonfirmasi #3, #9, #10 belum berubah. |
| 7 Agustus 2026 | #10 ditutup PR #166 (`9be633d`): halaman web Hari Libur memakai `ref_hari_libur` dengan audit resmi dan query sisi server. Evidence test otomatis, regresi PostgreSQL 17, dan smoke test browser tersedia sehingga status naik menjadi ✅. Ringkasan sprint menjadi 7 ✅ · 5 ⚠️ · 0 ❌. |
