# Tracking Sprint 6 — Dashboard & Laporan

| Field | Detail |
|---|---|
| Periode | 31 Juli – 9 Agustus 2026 |
| Cakupan issue | #39 – #43, #46 (`Issues-SIMPEG-Fase1.md`) |
| Pembaruan terakhir | 26 Juli 2026 (pra-sprint; sebagian source sudah tersedia lebih awal) |
| Basis verifikasi | Source HEAD `1b2e5b6` + verifikasi kode 26 Juli 2026 |
| Keputusan kickoff (kanonis) | `Kickoff-Sprint-6-Kontrak-dan-Keputusan.md` — **K-1** hapus reference table = hybrid `is_active` (tanpa SoftDeletes); **K-2** status "Dinas Luar" ditunda ke Fase 2; **K-3** kontrak payload Dashboard Admin W1–W7 |
| Pembagian tugas | Lihat pembagian BE/FE final 26 Juli 2026 (Jordan: #17/#21/#22-BE; Grantly: #16/#18/#19/#20/#23-BE + QA; Adithian & Adriel: sisi FE) |
| Menggantikan | `Analisis-Kesesuaian-Sprint-1-5.md` bagian Sprint 6 (dihapus 26 Juli 2026) |

Legend: ✅ selesai pada source · ⚠️ sebagian · ❌ belum selesai. Status source, bukan status tracker `Done`.

## Ringkasan

**0 ✅ · 5 ⚠️ · 1 ❌.** Banyak fondasi sudah terbangun lebih awal (dashboard Pimpinan + Kabag + Pegawai, Excel export) berkat PR #123/#125, tetapi belum ada issue yang layak ✅ utuh. Fokus sprint: dashboard **Admin** real-data, PDF pegawai, penyelarasan export cuti, dan CRUD reference tables (#46 satu-satunya ❌ total).

## Status per Issue

| Issue | Deliverable | Status | Bukti & catatan |
|---:|---|:---:|---|
| #39 | Dashboard Admin & Pimpinan (7 widget) | ⚠️ | **Pimpinan: real & diperbaiki PR #125** — W5 golongan granular I/a–IV/e, KPI `totalEwsAktif` terpisah dari 5 teratas, W2 golongan asal→tujuan, tren W7 metode baru (perbandingan historis `tanggal_pensiun`), link dummy dihapus. **Admin: MASIH DUMMY 6 dari 7 widget** — `DashboardController@index` hanya memasok EWS; view berisi angka "228", "Ahmad Fauzi", `$cutiList`/`$distribusiGolongan`/`$auditLog` array, SVG tren statis, tombol `alert('Ini adalah data dummy')`; tombol Setuju/Tunda hanya manipulasi innerHTML. Tidak ada `BuildAdminDashboardAction`. Kerjakan sesuai kontrak **K-3** (tanpa tombol keputusan cuti — Admin bukan approver). Sisa gap Pimpinan (minor): W7 masih berbasis `created_at` (kurva step pada data import) dan Non-Aktif/Mutasi tidak dikecualikan historis; fallback `golongan_awal` tampil `- → III/c`; perbaikan #125 belum punya test regresi. |
| #40 | Dashboard Pegawai (pribadi) | ⚠️ | **PR #123 menutup** profil ringkas (40.1), card saldo (40.2), daftar cuti aktif (40.3), EWS pribadi (40.4), responsive (40.6). **Sisa:** (a) **BUG — widget 5 notifikasi permanen kosong**: `DashboardController.php:64` memfilter `user_id` dengan `$user->id` padahal FK kolom itu menunjuk `employees` → harus `$user->employee_id` (fix satu baris); (b) fallback literal "12" saat pegawai tanpa baris saldo; (c) tanpa `BuildPegawaiDashboardAction` (query inline, `latestPosition.unitKerja` tidak di-eager-load); (d) tanpa feature test saldo/cuti/notifikasi — penyebab bug (a) lolos. |
| #41 | Dashboard Kepala Bagian | ⚠️ | **PR #125 menutup:** quick action keputusan 4 tombol + modal catatan wajib langsung di dashboard (reuse endpoint keputusan resmi), default halaman cuti "Menunggu Keputusan" (+escape `all`, dikunci test), pagination bawahan & cuti (whitelist 10/25/50). **Sisa:** (a) per keputusan **K-2**, bersihkan kode "Dinas Luar" setengah jadi yang kini dead code (opsi filter di FilterRequest, cabang action, badge kondisional di detail bawahan) — TIDAK ada implementasi Dinas Luar di Fase 1; (b) EWS bawahan masih array tanpa paginator — kontrol jumlah data/pagination kosmetik; (c) minor: filter "Menunggu Keputusan" berbasis status pengajuan, bukan step-assignee. |
| #42 | Export daftar pegawai (Excel + PDF) | ⚠️ | Excel standar & custom (whitelist kolom aman, anti-formula) tersedia; sisi Pimpinan tuntas via PR #125 (sidebar → route pimpinan, filter custom lengkap, UI read-only). **Sisa (tak tersentuh sejak audit):** (a) **P0 PII** — route preview masih closure `web.php:241` yang mengirim `email_pribadi` + `no_hp` seluruh pegawai (termasuk nonaktif) ke DOM; `ExportPegawaiPreviewAction`/`LaporanController` dead code; scope preview ≠ Excel; test justru mengunci perilaku salah; (b) **PDF pegawai belum ada sama sekali** (42.9–42.14) — hanya `window.print()`; (c) modal custom Admin mengikat 7 state Alpine yang tidak didefinisikan + tanpa tombol pembuka; (d) urutan kolom custom Admin dipaksa canonical (versi Pimpinan sudah mengikuti input user — samakan). |
| #43 | Export rekap cuti (Excel + PDF) | ⚠️ | Excel 2 sheet + PDF dengan filter/role-gate/anti-formula tersedia. **Sisa (file tak tersentuh sejak PR #104):** (a) sheet 2 "Saldo Cuti" berisi bucket saldo, bukan ringkasan per pegawai — implementasi benar sudah ada di jalur Pimpinan (`ExportLeaveReportAction`, sheet "Ringkasan Cuti") → konsolidasikan; (b) nama file `Laporan_Cuti_…` ≠ `Rekap_Cuti_{periode}_{tanggal}.xlsx` (test mengunci nama salah); (c) tabel PDF per pengajuan, bukan rekap per pegawai; (d) footer PDF tanpa nomor halaman (pola benar ada di `ExportRankHistoryPdfAction`). |
| #46 | Kelola reference tables | ❌ | **Belum ada sama sekali** (satu-satunya ❌): route `/data-master` closure render view statis; nol controller/FormRequest/Action CRUD untuk 9 reference table; form modal tanpa `name`/`action`/`@csrf` (klaim "RESTRIKSI INTEGRITAS" hanya teks); tidak ada UI kelola `notification_event_channels` (kebijakan PR #122 hanya bisa diubah via DB); model referensi tanpa lifecycle hapus. **Kerjakan sesuai keputusan K-1** (hybrid `is_active`, tanpa SoftDeletes): migrasi `is_active` untuk tabel yang belum punya → CRUD per vertical slice dengan validasi item terpakai → audit → gate `role:super_admin` → UI Data Master data DB + tree unit hierarkis + halaman channel & event notifikasi. |

## Urutan Pengerjaan (selaras pembagian tugas 26 Juli)

1. **#40 fix bug notifikasi** (satu baris + test) dan **#42a preview PII** — dampak besar, mulai duluan.
2. **#39 Dashboard Admin** (kontrak K-3) dan **#46** (keputusan K-1) — jalan paralel, FE mulai dari dummy sesuai kontrak.
3. **#42b PDF pegawai + #43 export cuti** — berpasangan (pola action + template PDF, banyak reuse).
4. **#42c/d custom Admin, #41 sisa Kabag** — menyusul.
5. **Kunci regresi perbaikan PR #125** (test `totalEwsAktif`, distribusi granular, golongan asal→tujuan, EWS search, per_page, quick action) + sempurnakan W7 serempak Admin & Pimpinan.

## Riwayat Perubahan Status

| Tanggal | Perubahan |
|---|---|
| 22 Juli 2026 | Baseline audit (61 task audit: 15 selesai / 22 sebagian / 24 belum; #46 ❌). |
| 23 Juli 2026 | PR #123: dashboard pegawai 40.1/40.2/40.3/40.6 tertutup. |
| 24–26 Juli 2026 | PR #125: quick action + default antrean + pagination Kabag; seluruh perbaikan Pimpinan (#39 sisi Pimpinan, #42 sisi Pimpinan). Ditemukan bug baru FK notifikasi dashboard pegawai (40.5). |
| 26 Juli 2026 | Kickoff selesai: keputusan K-1/K-2/K-3 dikunci; pembagian tugas BE/FE final; status di atas ditetapkan per HEAD `1b2e5b6`. |
