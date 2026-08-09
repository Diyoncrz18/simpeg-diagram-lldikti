# Tracking Role: Pegawai

| Field | Nilai |
|---|---|
| Role internal | `pegawai` |
| Tanggal analisis | 10 Agustus 2026 — rekonsiliasi dari audit implementasi 27 Juli |
| Basis verifikasi | Branch `development` @ `1fd99cb` setelah PR #177 |
| Dokumen asal (dikonsolidasi ke file ini) | `Halaman-dan-Hak-Akses-Role-Pegawai.md` — dokumen target produk 22 Juli (bukan audit); daftar halaman targetnya menjadi kerangka penilaian di bawah |
| Acuan produk | PRD v1.4 §4.2 (Pegawai & Verifikator Cuti), §9, §13.2–13.4; US-2.5, US-4.1–4.3, US-4.7, US-5.3, US-6.1/6.2/6.4, US-8.2 |
| Status keseluruhan | ⚠️ **Sebagian besar terbangun** — 11 halaman target hampir semua ada dan self-scoped; gap utama: 1 bug data (widget notifikasi) dan 2 halaman tanpa tautan navigasi |

## Catatan struktural

Berbeda dari Pimpinan dan Kabag, role pegawai **tidak punya grup route sendiri** — ia memakai ulang
route/view area admin yang membatasi diri ke data milik sendiri (self-scoped). Folder
`resources/views/pegawai/` hanya berisi `dashboard.blade.php`. Pola ini berfungsi, tetapi membuat
batas akses bergantung pada scoping per-controller, bukan prefix + middleware role seperti dua role
lainnya.

## Rekonsiliasi Temuan — 10 Agustus 2026

- Bug widget notifikasi dashboard tetap tertutup sejak PR #132; empty state saldo diselaraskan PR #167.
- Validasi lintas tahun seluruh jenis cuti sudah selesai dan merge melalui PR #140 (`952f723`), bukan lagi “PR #136 menunggu review”.
- Dua gap navigasi yang dicatat audit 27 Juli (halaman saldo dan antrean verifikator) tetap terbuka sampai ada bukti implementasi/QA baru.

Baris audit lama di bawah dipertahankan sebagai kronologi; rekonsiliasi ini yang berlaku bila ada status yang bertentangan.

## Status per Halaman Target

| # | Halaman target (dok. 22 Juli) | Status | Bukti |
|---|---|:---:|---|
| 1 | Dashboard Pribadi | ✅ | `BuildPegawaiDashboardAction` (PR #132, merged 27 Juli) → `pegawai/dashboard.blade.php`. Widget nyata: profil ringkas, 3 kartu saldo, cuti aktif (5), EWS pribadi, notifikasi (5). Bug FK widget notifikasi + ekstraksi Action ditutup PR #132 dengan 6 feature test |
| 2 | Profil Saya | ✅ | `/dashboard/profil` → `ShowProfilePageAction`; 10 tab (Profil, Cuti, Keluarga, Kepangkatan, Jabatan, KGB, Disiplin, Pendidikan, Pengangkatan, Dokumen SK) — read-only, tanpa kebocoran kontrol edit; pintasan admin dipagari role |
| 3 | Ajukan Cuti | ✅ | `cuti.create/store` + `StoreLeaveRequestRequest`; validasi saldo real-time (submit di-disable), hitung hari kerja server-side (`WorkdayCalculator`), tolak lintas tahun, jenis cuti disaring PNS/PPPK, chain approval wajib resolve |
| 4 | Daftar Cuti Saya | ✅ | `CutiController::index` self-scoped (kecuali ber-permission `cuti.read_all`); label resmi tanpa `Ditolak` (grep = 0) |
| 5 | Detail Cuti & Timeline Approval | ✅ | `cuti.show` — timeline per step, keputusan + keterangan, lampiran terotorisasi |
| 6 | Saldo Cuti | ⚠️ | Halaman ada (`cuti.saldo` → `personal-saldo.blade.php`: jatah, carry-over N-1, terpakai, sisa, riwayat) tapi **tidak ada satu pun tautan menuju ke sana** (lihat Belum Sesuai #2) |
| 7 | EWS Pribadi | ✅ | Route khusus `ews.saya` (`role:pegawai`), menu sidebar "EWS Saya", section di dashboard dan profil; difilter per employee, `abort_unless` |
| 8 | Bell Notifikasi navbar | ✅ | `<x-layouts.notification-bell />` — badge unread, dropdown, polling 30 detik, tandai dibaca/semua via API |
| 9 | Semua Notifikasi | ✅ | `notifications.index` — paginator server-side, indikator belum dibaca, tandai per item + semua, ownership per user |
| 10 | Dokumen Cuti & Verifikasi QR | ✅ | `cuti.formulir-pdf` — otorisasi di Action (pemohon/approver/`cuti.read_all`), wajib status `disetujui`, header `no-store`; verifikasi QR publik `cuti.verify` di luar auth + `throttle:60,1` |
| 11 | Pengajuan Menunggu Tindakan (verifikator kondisional) | ⚠️ | Backend lengkap: `cuti.approval` + 4 route keputusan ber-allowlist `pegawai`, antrean difilter step aktif milik aktor (`ListPendingLeaveApprovalsAction`), guard aktor di controller + service. **Tapi tidak ada tautan navigasi** (lihat Belum Sesuai #3) |

## ✅ Batas Akses (terverifikasi — sesuai)

- Seluruh area terlarang diblokir di **route middleware**, bukan hanya disembunyikan sidebar:
  data-pegawai & CRUD (`routes/web.php:293-347`), audit-log (:485-491), laporan (:172-175, :283),
  rekap/laporan cuti (:204-218), EWS global (:161-163), dokumen (:450-475), data-backup (:200-202),
  seluruh menu administrasi sistem (:177-244, :364-378, :504-509).
- `$lockedMenus['pegawai']` di layout hanyalah lapis kosmetik di atas gerbang route — pola yang benar.
- Keputusan cuti milik sendiri tidak bisa diambil sendiri (guard aktor per step di service).

## ⚠️ Belum Sesuai / Sisa Pekerjaan

| # | Prioritas | Temuan (terverifikasi 27 Juli) | Tindak lanjut |
|---|:---:|---|---|
| 1 | P1 | **Halaman Saldo Cuti tak terjangkau dari UI** — grep `route('cuti.saldo')` di seluruh views = 0; hanya bisa diakses ketik URL. Dokumen target menempatkannya sebagai halaman inti pegawai | Gabungkan dengan task #17 (finalisasi dashboard/navigasi pegawai) |
| 2 | P1 | **Antrean verifikator tak terjangkau dari UI** — grep `route('cuti.approval')` di seluruh views = 0; pegawai yang ditunjuk di approval chain hanya bisa menindaklanjuti lewat link notifikasi per-pengajuan. AC "Pengajuan Menunggu Tindakan" belum selesai dari sisi navigasi | Perlu diangkat ke backlog (kandidat gabung Issue #40/#44) |
| ~~3~~ | — | **Selesai PR #140 (`952f723`)** — validasi lintas tahun berlaku untuk semua jenis cuti pada submit dan resubmit | Ditutup; regression test seluruh jenis non-tahunan tersedia |
| 4 | P2 | Label `Perlu Perubahan` masih tampil di daftar/detail cuti yang dipakai pegawai (`admin/cuti/index.blade.php:17`, `show.blade.php:305`) — istilah resmi `Perubahan` | Task tracker #25 |
| 5 | P2 | Route `cuti.saldo` tanpa middleware `permission:` (hanya allowlist role grup) — aman karena controller resolve employee dari sesi, tetapi berbeda pola dari route lain yang bergerbang ganda | Catatan konsistensi — rapikan saat menyentuh route cuti |

> **Ditutup 27 Juli 2026 (PR #132, merged):** bug FK widget notifikasi (`$user->id` → `$user->employee_id`) dan ekstraksi `BuildPegawaiDashboardAction` — dua temuan sebelumnya di tabel ini (P0 bug + P2 arsitektur), dikunci 6 feature test.

## Kesimpulan

Fondasi role pegawai lebih sehat daripada yang terlihat: semua alur inti (profil, cuti end-to-end,
EWS pribadi, notifikasi, QR) ada dan pembatasan datanya ditegakkan server-side. Bug widget
notifikasi dan ekstraksi `BuildPegawaiDashboardAction` sudah ditutup PR #132 (merged 27 Juli);
validasi lintas tahun semua jenis selesai melalui PR #140. Sisa yang membuat
US-8.2/4.7 belum selesai penuh: dua halaman fungsional tanpa tautan navigasi (saldo & antrean
verifikator) — kandidat slice dashboard pegawai (Issue #40) di Sprint 6.
