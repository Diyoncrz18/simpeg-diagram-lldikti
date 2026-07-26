# Tracking Role: Pimpinan

| Field | Nilai |
|---|---|
| Role internal | `pimpinan` |
| Tanggal analisis ulang | 27 Juli 2026 |
| Basis verifikasi | Branch `development` @ `e82b527` (sesudah PR #121–#131, termasuk perbaikan dashboard PR #125) |
| Dokumen asal (dikonsolidasi ke file ini) | `Analisis-Frontend-Role-Pimpinan.md` — audit 21 Juli 2026 |
| Acuan produk | PRD v1.3 (dashboard seluruh pegawai, PYBMC/approver final, data pegawai read-only, laporan), US-4.6, US-5.2, US-8.1 W1–W7, US-9.1B, L1/L1b/L2/L3 |
| Status keseluruhan | ⚠️ **Sebagian besar sesuai** — inti operasional lengkap; sisa: akurasi tren W7, konsistensi label, dan 1 keputusan produk |

## Ringkasan

Audit 21 Juli menemukan 6 kelompok masalah besar (UI tidak read-only, jalur laporan ganda, filter
kurang, dashboard tidak akurat, search EWS mati, konflik dokumen audit log). Verifikasi ulang
27 Juli menunjukkan **mayoritas sudah ditutup** — terutama oleh perbaikan dashboard (PR #125) dan
konsolidasi laporan Pimpinan. Yang tersisa bersifat terlokalisir.

## ✅ Sudah Sesuai (terverifikasi 27 Juli)

| # | Area | Bukti |
|---|---|---|
| 1 | Keputusan cuti final US-4.6 — hanya approver step aktif, 4 label resmi tanpa `Tolak`, catatan wajib untuk non-`Disetujui`, audit + notifikasi + pengurangan saldo + bukti QR | `ListPimpinanLeavesAction`, `LeaveApprovalService` |
| 2 | Detail pegawai Pimpinan read-only — NIK keluarga tidak tampil, ARIA + navigasi keyboard pada tab | `pimpinan.pegawai.show` |
| 3 | **Data Pegawai benar-benar read-only dari UI** (dulu ❌ P1) — Tambah/Import/Edit/checkbox bulk/Export Excel dibungkus `@if(!$isReadOnly)`; Pimpinan mendapat link "Laporan Pegawai" sebagai gantinya | `admin/pegawai/index.blade.php:364-413,495,619-631,695`; flag di `PimpinanEmployeeController.php:75` |
| 4 | **Sidebar laporan menunjuk jalur Pimpinan** (dulu ❌ P1 — dua jalur membingungkan) — menu kini ke `pimpinan.laporan.pegawai`, `pimpinan.laporan.cuti`, `pimpinan.laporan.kepangkatan` | `components/layouts/app.blade.php:210-216` |
| 5 | **Filter laporan custom lengkap sesuai US-9.1B** (dulu ❌ P1) — Unit Kerja, Jenis Pegawai, Jabatan kini tersedia di samping Nama/NIP, Golongan, Status, periode pensiun | `pimpinan/laporan/pegawai.blade.php:108-186` |
| 6 | Dashboard W5 distribusi golongan penuh `I/a–IV/e` (dulu ❌ digabung per angka romawi) | `BuildPimpinanDashboardAction.php:129-133` |
| 7 | Dashboard W2 kenaikan pangkat "Golongan (Asal → Tujuan)" (dulu satu kolom) | `BuildPimpinanDashboardAction.php:87-97`, `pimpinan/dashboard.blade.php:190,210-216` |
| 8 | KPI EWS Aktif memakai total sebenarnya, preview 5 baris terpisah (dulu ter-cap 5) | `BuildPimpinanDashboardAction.php:117-118` |
| 9 | Kartu W1 menaut ke daftar pegawai Pimpinan (dulu `href="#"`) | `pimpinan/dashboard.blade.php:125` |
| 10 | Pencarian EWS berfungsi server-side — nama/NIP difilter sebelum paginasi (dulu param diabaikan) | `PimpinanEwsController.php:14,19-26` |
| 11 | `per_page` EWS di-whitelist 10/25/50; markup `<p>…</span>` invalid sudah hilang | `PimpinanEwsController.php:29-30`, `pimpinan/ews/index.blade.php:62-66,82` |
| 12 | Laporan cuti & laporan kepangkatan (Excel/PDF fixed) jalur Pimpinan, teruji | `PimpinanReportController`, `ExportRankHistoryPdfAction` |

## ⚠️ Belum Sesuai / Sisa Pekerjaan

| # | Prioritas | Temuan (terverifikasi 27 Juli) | Tindak lanjut |
|---|:---:|---|---|
| 1 | P1 | **W7 belum tren historis sebenarnya** — filter status kini lewat `tanggal_pensiun` (perbaikan parsial), tetapi basis titik bulanan masih `whereDate('created_at','<=',akhir bulan)` (`BuildPimpinanDashboardAction.php:26-39`); angka bulan lampau terdistorsi oleh kapan record diinput, bukan kondisi kepegawaian saat itu | Task tracker #24 |
| 2 | P2 | **Label `Perlu Perubahan`** masih dipakai di `pimpinan/cuti/index.blade.php:15` (juga `admin/cuti/index.blade.php:17`, `admin/cuti/show.blade.php:305`); istilah resmi keputusan adalah `Perubahan` | Task tracker #25 |
| 3 | Keputusan produk | **Link audit log dari W6** — PRD/US-8.1 meminta link ke audit detail, sedangkan sidebar mengecualikan `audit-log` bagi Pimpinan. Belum diputuskan: beri akses read-only ter-masking, atau revisi kriteria W6 jadi ringkasan saja. Belum masuk keputusan kickoff K-1/K-3 | Eskalasi ke pemilik produk (bersama task #15) |

## Catatan Kualitas Kode (P2–P3, belum diverifikasi ulang 27 Juli)

Temuan audit 21 Juli berikut bersifat non-blocking dan belum dicek ulang; validasi saat menyentuh file terkait:

- Eager loading detail pegawai Pimpinan memuat relasi/reference yang tidak dirender view.
- Pratinjau laporan cuti/kepangkatan memuat seluruh Collection lalu `forPage()` di memori.
- Kalkulasi SVG tren W7 masih di Blade — layak dipindah ke view model saat W7 diperbaiki (gabungkan dengan task #24).

## Riwayat Temuan Audit 21 Juli → Kondisi Sekarang

| Temuan lama | Kondisi 27 Juli |
|---|---|
| ❌ P1 UI Data Pegawai tidak read-only | ✅ Diperbaiki |
| ❌ P1 Sidebar → laporan generik; halaman custom tak terjangkau | ✅ Diperbaiki (sidebar → `pimpinan.laporan.*`) |
| ❌ P1 Filter custom kurang (unit/jenis/jabatan) | ✅ Diperbaiki |
| ❌ P1 Laporan generik bocor kontak & pegawai nonaktif | Tidak lagi ditautkan untuk Pimpinan; closure `laporan.pegawai` sendiri masih hidup di area Admin — lihat `Role-Admin-Kepegawaian.md` (task #18) |
| ❌ P1 Modal custom di halaman generik mati | Berpindah cakupan ke Admin (task #20) — jalur Pimpinan kini memakai form custom sendiri |
| ❌ W5 golongan digabung, W7 tren, KPI EWS cap 5, W2 satu kolom, W1 `#` | ✅ Semua diperbaiki kecuali **W7** (parsial — baris ⚠️ #1) |
| ⚠️ Search EWS mati, HTML invalid, per_page bebas | ✅ Ketiganya diperbaiki |
| ⚠️ Istilah `Perlu Perubahan` | ⚠️ Masih (baris ⚠️ #2) |
| Konflik dokumen link audit W6 | ⚠️ Masih menunggu keputusan produk (baris ⚠️ #3) |
