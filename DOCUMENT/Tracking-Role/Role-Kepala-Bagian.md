# Tracking Role: Kepala Bagian

| Field | Nilai |
|---|---|
| Role internal | `kepala_bagian` |
| Tanggal analisis ulang | 27 Juli 2026 |
| Basis verifikasi | Branch `development` @ `e82b527` (sesudah PR #121–#131 dan perbaikan Sprint 4–6 awal) |
| Dokumen asal (dikonsolidasi ke file ini) | `Analisis-Frontend-Role-Kepala-Bagian.md` — audit 21 Juli 2026 |
| Acuan produk | PRD v1.3 (Kabag = approver tahap 1, data bawahan langsung), US-4.4, US-8.3, Panduan Kode |
| Status keseluruhan | ✅ **Substansial sesuai** — sisa 2 temuan teknis (bersih-bersih Dinas Luar, paginasi EWS) |

## Ringkasan

Kabag adalah role dengan tingkat kesesuaian tertinggi saat ini. Audit 21 Juli mencatat dua gap
acceptance criteria (quick action cuti di dashboard, status `Dinas Luar`) dan tiga masalah kualitas
(pagination tidak konsisten, halaman cuti berorientasi riwayat, copy/aksesibilitas). Verifikasi ulang
27 Juli menunjukkan **hampir semuanya sudah ditutup**; status `Dinas Luar` berubah kategorinya karena
keputusan kickoff K-2 (26 Juli) menundanya ke Fase 2.

## ✅ Sudah Sesuai (terverifikasi 27 Juli)

| # | Area | Bukti |
|---|---|---|
| 1 | Menu & layout sesuai role — hanya Dashboard, Daftar Bawahan, Cuti Bawahan, Pengajuan Cuti pribadi, EWS Bawahan, Notifikasi; tanpa menu administrasi global | `resources/views/components/layouts/app.blade.php` |
| 2 | Scope bawahan langsung ditegakkan server-side (daftar, detail, cuti, lampiran, EWS) | `app/Services/Employees/KepalaBagianScopeService.php` dipakai semua Action Kabag |
| 3 | Detail bawahan read-only — tanpa kontrol edit/hapus | `resources/views/kabag/bawahan/show.blade.php` |
| 4 | Alur keputusan cuti US-4.4 — 4 label resmi, tanpa `Tolak`, catatan wajib min. 5 karakter, timeline, guard approver step aktif di service | `KepalaBagianLeaveDecisionRequest`, `LeaveApprovalService` |
| 5 | **Quick action keputusan cuti di dashboard** (gap P1 audit lama) — 4 tombol per baris + modal konfirmasi Alpine + catatan wajib untuk non-`Disetujui`, POST ke endpoint keputusan yang sudah ada | `resources/views/kabag/dashboard.blade.php:223-332` |
| 6 | Pagination Daftar Bawahan selaras end-to-end — UI dan whitelist Action sama-sama 10/25/50 (opsi 100 yang dulu silent-fallback sudah dihapus) | `kabag/bawahan/index.blade.php:174-176`, `ListKepalaBagianEmployeesAction.php:75` |
| 7 | Pagination Cuti Bawahan mengikuti pilihan pengguna — tidak lagi `paginate(20)` tetap | `ListKepalaBagianLeavesAction.php:16,35`, `KepalaBagianLeaveFilterRequest.php:23` |
| 8 | Default halaman Cuti Bawahan = **Menunggu Keputusan** (antrean tindakan), riwayat penuh via filter `all` eksplisit | `KepalaBagianLeaveController.php:21-25` |
| 9 | Opsi filter unit kerja/jenis pegawai di-scope ke bawahan (tidak lagi seluruh reference table) | `KepalaBagianEmployeeController.php:36-37` |
| 10 | Copy & aksesibilitas dirapikan — subtitle "Kelola dan pantau" dan atribut `Toggle Detail` sudah tidak ada di view kabag | grep `resources/views/kabag/**` = 0 match |
| 11 | Halaman notifikasi bersama — paginator, tandai dibaca, ownership per user | `notifications.index` (lihat tracking Sprint 5/7) |

## ⚠️ Belum Sesuai / Sisa Pekerjaan

| # | Prioritas | Temuan (terverifikasi 27 Juli) | Tindak lanjut |
|---|:---:|---|---|
| 1 | P1 | **Sisa implementasi `Dinas Luar` di backend** — keputusan K-2 (kickoff 26 Juli) menunda Dinas Luar ke Fase 2, tetapi masih ada 3 titik kode hidup: rule `Rule::in(['aktif','cuti','dinas_luar'])` di `KepalaBagianEmployeeFilterRequest.php:19`, cabang filter `dinas_luar` di `ListKepalaBagianEmployeesAction.php:63-67`, badge `Dinas Luar` di `kabag/bawahan/show.blade.php:44-45`. Dropdown UI sudah tidak menawarkannya, tapi jalur query string masih bisa memicunya. | Task tracker #23 — bersihkan sisa kode agar konsisten dengan K-2 |
| 2 | P2 | **EWS Bawahan tanpa paginasi server-side** — Action mengembalikan array penuh (`ListKepalaBagianEwsAlertsAction.php:18-25`), semua alert dirender sekaligus, pencarian client-side. Catatan positif: kontrol jumlah-data kosmetik yang menyesatkan sudah dihapus, jadi tidak ada lagi UI bohong — tinggal paginasi yang belum ada. | Task tracker #23 |

## Perubahan Kategori karena Keputusan Produk

| Temuan audit 21 Juli | Status sekarang |
|---|---|
| ❌ "Status `Dinas Luar` belum tersedia" (US-8.3 AC-1) | **Tidak berlaku untuk Fase 1** — keputusan K-2 pada `Kickoff-Sprint-6-Kontrak-dan-Keputusan.md`: Dinas Luar berasal dari modul Surat Tugas yang belum ada, ditunda ke Fase 2. Kewajiban Fase 1 tersisa: badge status hanya Aktif/Cuti + pembersihan sisa kode (baris ⚠️ #1 di atas). |

## Riwayat Temuan Audit 21 Juli → Kondisi Sekarang

| Temuan lama | Kondisi 27 Juli |
|---|---|
| ❌ Quick action cuti dashboard belum ada | ✅ Diperbaiki (tombol + modal + catatan wajib) |
| ❌ Status Dinas Luar belum ada | Direklasifikasi — ditunda Fase 2 (K-2); sisa kode perlu dibersihkan |
| ⚠️ Pagination bawahan (opsi 100 palsu) | ✅ Diperbaiki (UI = whitelist = 10/25/50) |
| ⚠️ Pagination cuti (`paginate(20)` tetap) | ✅ Diperbaiki (per_page tervalidasi) |
| ⚠️ Kontrol jumlah data EWS kosmetik | ✅ Kontrol menyesatkan dihapus; ⚠️ paginasi server-side belum ada |
| ⚠️ Cuti Bawahan default riwayat, bukan antrean | ✅ Diperbaiki (default `menunggu_approval`) |
| P3 copy "Kelola dan pantau", `Toggle Detail`, filter dari seluruh reference | ✅ Ketiganya sudah dirapikan |
