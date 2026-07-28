# Bukti QA — Saldo dan Batas Pengajuan Cuti Tahap 6

> Tanggal retest: 28 Juli 2026.
>
> Branch implementasi: `feature/cuti-balance-policy-stage-0`.
>
> Status: **Escalated untuk UAT runtime/release gate.** Seluruh acceptance criterion Tahap 0–5 yang dapat dibuktikan pada source, test suite Laravel, dan validasi migrasi PostgreSQL bersih lulus. Suite Laravel saat ini memang dikonfigurasi memakai SQLite in-memory; validasi PostgreSQL dicatat terpisah dan tidak disamakan dengan test suite. Penutupan UAT browser terautentikasi dan release gate repository masih memerlukan environment yang siap, sehingga Tahap 6 tidak dinyatakan accepted/release-ready secara penuh.

## Acuan dan batas QA

- `Keputusan-Cuti-Saldo-Tahap-0.md`: K-CUT-01, K-CUT-02, dan K-CUT-03.
- `PRD-SIMPEG-Fase1-Core.md` §9.2–§9.4.
- `User-Stories-SIMPEG-Fase1.md`: US-4.1, US-4.3, US-4.6, dan US-4.9.
- `Panduan-Penulisan-Kode-SIMPEG.md`: quality gate, PostgreSQL, audit, RBAC, dan test backend non-trivial.
- `Tracking-Sprint-1-7/Sprint-7-Stabilization-Regression-UAT.md`: evidence QA formal sebelum tracker dapat ditutup.

Cakupan bukti ini adalah perbaikan saldo pada form pengajuan, reservasi saldo cuti tahunan, validasi batas seluruh jenis cuti, dan rangkaian eksplisit Cuti Melahirkan/CLTN. Bukti ini tidak mengubah keputusan produk atau memigrasikan database development bersama.

## Matriks acceptance criteria

| ID | Kriteria penerimaan | Bukti | Hasil |
|---|---|---|---|
| QA-CUT-01 | Form Pegawai memakai saldo nyata milik pegawai login; pemuatan/preview tidak menulis saldo atau ledger. | `LeaveBalancePreviewTest`: ownership, virtual entitlement read-only, guest/role/validasi tanggal. | Lulus. |
| QA-CUT-02 | Pengajuan aktif Cuti Tahunan mencadangkan hak; final `Disetujui` mengonversi reservasi lalu memotong saldo sekali; `Tidak Disetujui` melepas reservasi; `Perubahan`/`Ditangguhkan` mempertahankannya. | `LeaveBalanceReservationTest` dan `LeaveApprovalEngineTest`. | Lulus. |
| QA-CUT-03 | Alur nyata Pegawai → Kepala Bagian → PYBMC menghasilkan status final, ledger, saldo, bukti QR, dan notifikasi. | `CutiEndToEndApprovalTest`. | Lulus. |
| QA-CUT-04 | Saldo tidak cukup, hitungan hari kerja, hari libur, lampiran, approval-chain, dan RBAC submit divalidasi server-side. | `SubmitLeaveRequestTest`, `CalculateWorkdaysTest`, `CutiRbacTest`. | Lulus. |
| QA-CUT-05 | Satu pengajuan tidak dapat melewati tahun kalender untuk setiap jenis cuti; resubmit juga tidak dapat mengakali aturan tersebut. | `SubmitLeaveRequestTest` mencakup Tahunan, Sakit, Melahirkan, Alasan Penting, Besar, CLTN, dan resubmit non-tahunan. | Lulus. |
| QA-CUT-06 | Cuti Besar PNS baru boleh mulai tepat lima tahun kalender sejak TMT; CLTN hanya PNS. | `LeaveEligibilityPolicyTest`. | Lulus. |
| QA-CUT-07 | Cuti Melahirkan maksimum tiga bulan dan CLTN maksimum tiga tahun kalender secara kumulatif; potongan antar-tahun dihubungkan lewat relasi eksplisit, bukan alasan bebas. | `LeaveEligibilityPolicyTest`: batas, kelanjutan antar-tahun, case milik sendiri, kecocokan jenis, dan resubmit. | Lulus. |
| QA-CUT-08 | UI hanya menyajikan rangkaian kelanjutan milik pemohon; ID case dari pegawai lain atau jenis lain ditolak backend. | Kontrak form dan validasi JSON pada `LeaveEligibilityPolicyTest`. | Lulus. |
| QA-CUT-09 | Admin memantau saldo tanpa menjadi approver; koreksi saldo beralasan dan ter-audit. | `LeaveBalanceAdjustmentTest` dan `CutiRbacTest`. | Lulus. |
| QA-CUT-10 | Mutation penting memiliki audit; kegagalan audit/pemotongan tidak meninggalkan state setengah jadi. | `CutiAuditCoverageTest`, `LeaveApprovalEngineTest`, `LeaveBalanceAdjustmentTest`, dan `LeaveEligibilityPolicyTest`. | Lulus. |

## Hasil command yang dapat direproduksi

| Command | Hasil |
|---|---|
| `podman exec simpeg_app php artisan test tests/Feature/CutiEndToEndApprovalTest.php tests/Feature/LeaveEligibilityPolicyTest.php tests/Feature/LeaveBalanceReservationTest.php tests/Feature/LeaveBalancePreviewTest.php tests/Feature/LeaveApprovalEngineTest.php tests/Feature/LeaveBalanceAdjustmentTest.php tests/Feature/SubmitLeaveRequestTest.php --testdox` | **112 test, 582 assertion lulus.** |
| `podman exec simpeg_app composer test` | **1 skipped, 1.092 test lulus, 5.351 assertion** dalam 500,14 detik. |
| Konfigurasi `phpunit.xml` | Suite Laravel memakai `DB_CONNECTION=sqlite` dan `DB_DATABASE=:memory:`; hasil test tidak diklaim sebagai bukti PostgreSQL. |
| `podman exec simpeg_app php vendor/bin/phpstan analyse --memory-limit=1G --no-progress` | **Tidak ada error.** |
| `npm run build` | **Lulus**; Vite membangun 56 modul. |
| `podman exec simpeg_app php artisan view:cache` lalu `view:clear` | **Lulus**; kompilasi Blade dan pembersihan cache selesai. |
| `podman exec simpeg_app php artisan route:list --path=api/v1/cuti --json` | Kontrak endpoint preview saldo dan kalkulator hari kerja tercatat dengan middleware autentikasi, role, dan permission `cuti.create`. |
| `Invoke-WebRequest http://127.0.0.1:8000/up` | **HTTP 200**; container aplikasi lokal aktif. |
| `git -C SIMPEG diff --check` | **Lulus**; tidak ada whitespace error pada worktree SIMPEG. |
| Database sementara `simpeg_stage6_verify_20260728` | Semua migrasi, termasuk empat migrasi pending, lulus pada PostgreSQL. Kolom UUID dan tiga FK restricted untuk `leave_request_cases`/`leave_requests.leave_request_case_id` diverifikasi melalui katalog PostgreSQL; database sementara kemudian dihapus. |

## Retest klarifikasi jenis cuti dan saldo — 28 Juli 2026

Pengguna menguji pengajuan milik Pegawai dengan durasi **23 hari kerja** ketika informasi saldo tahunan menunjukkan **21 hari**. Detail request membuktikan jenis yang dipilih adalah **Cuti Sakit**, bukan Cuti Tahunan. Hasil tersebut ditinjau terhadap sumber produk dan kode, dengan kesimpulan berikut.

| Pemeriksaan | Bukti | Hasil |
|---|---|---|
| Saldo tahunan hanya memblokir Cuti Tahunan | PRD §9.3, `US-CUT-01` AC-5 menyebut eksplisit “untuk cuti tahunan”; User Stories US-4.1 AC-5 menyatakan hal yang sama. | Lulus / sesuai dokumen. |
| Cuti Sakit tidak memperoleh batas angka asumsi di Fase 1 | `Keputusan-Cuti-Saldo-Tahap-0.md` K-CUT-03: “Tidak ditambah batas numerik baru pada Fase 1”; aturan rinci/surat dokter menunggu sumber peraturan yang disahkan. | Bukan defect saldo. Pengajuan 23 hari kerja tidak dibandingkan dengan saldo tahunan 21 hari. |
| Saldo Cuti Tahunan tidak cukup ditolak sebelum penyimpanan | `StoreLeaveRequestRequest` menghitung hari kerja di server dan mengembalikan error `tanggal_selesai`; form mengunci tombol kirim; `LeaveBalanceReservationService` mengulang pemeriksaan di dalam transaksi. | Lulus. Tidak ada request/reservasi baru bila kebutuhan Cuti Tahunan melebihi `saldo_dapat_diajukan`. |
| Retest regresi jalur submit dan reservasi | `php artisan test tests/Feature/SubmitLeaveRequestTest.php tests/Feature/LeaveBalanceReservationTest.php` | **65 test, 318 assertion lulus** dalam 8,06 detik. |

Catatan kepatuhan: PRD §9.2 memang merujuk “Sesuai PP, surat dokter diperlukan (sesuai ketentuan)” untuk Cuti Sakit. Namun durasi pemicu, format, dan validasi surat dokter belum dipasok sebagai peraturan operasional yang disahkan. Sesuai K-CUT-03, implementasi tidak boleh mengarang durasi atau syarat baru. Jika ketentuan tersebut tersedia atau kebijakan baru disetujui, perubahan harus melalui keputusan produk dan test terpisah.

## Verifikasi data dan migrasi

1. Migrasi Tahap 5 `2026_07_27_000003_create_leave_request_cases_table` diverifikasi kembali pada database PostgreSQL sementara yang bersih. Semua migrasi selesai; kolom `leave_request_case_id` bertipe UUID dan FK `leave_request_cases.employee_id`, `leave_request_cases.jenis_cuti_id`, serta `leave_requests.leave_request_case_id` semuanya `ON DELETE RESTRICT`. Database sementara kemudian dihapus kembali.
2. Suite feature/unit Laravel memakai SQLite in-memory sesuai `phpunit.xml`. Suite tersebut membuktikan perilaku aplikasi, validasi, audit, dan atomicity pada level aplikasi; bukti migrasi PostgreSQL di butir 1 menjaga pemeriksaan tipe UUID dan constraint produksi agar tidak bergantung pada SQLite saja.
3. Pada retest runtime 28 Juli 2026, pengguna menemukan `SQLSTATE[42P01]` saat membuka form pengajuan karena kode preview saldo sudah memakai `leave_balance_reservation_events` sedangkan empat migrasi tanggal 27 Juli masih pending di database development. Setelah preflight dan simulasi SQL, seluruh batch diterapkan bersama-sama (batch 3), bukan hanya migration tabel yang hilang. Cache aplikasi dibersihkan setelah migrasi.
4. Retest runtime pada pegawai yang memicu error berhasil menjalankan `PrepareLeaveRequestFormAction`: saldo aktual `12`, alokasi aktif `0`, dan saldo dapat diajukan `12`. Tidak ada lagi query ke tabel yang tidak tersedia. Data existing tidak memerlukan backfill event reservasi atau rangkaian Melahirkan/CLTN pada saat migrasi.

## Release gate dan risiko terbuka

### 1. Gate kualitas repository: blocked

`podman exec simpeg_app composer qa` berhenti pada Pint sebelum PHPStan/test dipanggil karena dua file yang telah ada di worktree, di luar source Tahap 0–6:

- `bootstrap/app.php`: `concat_space`.
- `test_show.php`: `method_argument_space`, `fully_qualified_strict_types`, `concat_space`, `statement_indentation`, `single_line_after_imports`, dan `no_whitespace_in_blank_line`.

File source dan test Tahap 5 telah lolos Pint pada scope-nya. Owner perubahan dua file di atas perlu memformat atau memisahkan perubahan tersebut, kemudian `composer qa` harus diulang sebagai gate merge/release.

Retest 28 Juli 2026 menghasilkan warning yang sama: `composer qa` keluar dengan kode 1 pada tahap Pint dan tidak melanjutkan PHPStan/test. Tidak ada warning tambahan dari retest submit/reservasi Cuti Tahunan; dua warning format di atas tetap menjadi satu-satunya blocker gate kualitas repository yang teridentifikasi.

### 2. UAT browser autentikasi: blocked

Perintah browser test dihentikan sebelum assertion karena image `simpeg_app` tidak memiliki binary ChromeDriver Linux yang cocok:

```text
Invalid path to Chromedriver [.../vendor/laravel/dusk/bin/chromedriver-linux].
```

Pemeriksaan lanjutan juga menunjukkan `google-chrome`, `chromium`, dan `chromium-browser` tidak tersedia di image. Folder Dusk hanya memuat driver Windows (`chromedriver-win.exe` dan `chromedriver-win32`), yang tidak dapat dipakai oleh container Linux. Database development kini telah menerima batch migrasi tersebut, tetapi autentikasi produksi tetap menggunakan Keycloak. Karena itu, QA ini tidak memalsukan login hanya demi screenshot browser.

## Input manusia minimum untuk menutup Tahap 6

1. Pemilik workspace merapikan atau memisahkan perubahan `bootstrap/app.php` dan `test_show.php`, lalu menjalankan ulang `composer qa`.
2. Pemilik release menyiapkan database UAT yang bersih atau menerapkan **seluruh** batch migrasi Tahap 5 lewat prosedur deploy yang disetujui. Retest development membuktikan batch harus diterapkan bersama-sama, bukan hanya membuat tabel reservasi secara manual.
3. Pemilik environment menyediakan Chrome/Chromium Linux beserta ChromeDriver yang kompatibel, atau browser runner yang setara, dan akun UAT Keycloak ber-role Pegawai, Kepala Bagian, serta Pimpinan/PYBMC.
4. Setelah tiga prasyarat tersedia, lakukan retest browser desktop dan mobile untuk: pemilihan tanggal, preview saldo, submit Tahunan, kelanjutan Melahirkan/CLTN, keputusan Kabag/PYBMC, pembaruan saldo setelah final `Disetujui`, dan audit/notification timeline.

Tidak ada defect P0/P1 baru yang ditemukan dari source-level, validasi migrasi PostgreSQL bersih, atau regresi otomatis Tahap 6. Status tetap **Escalated**, bukan accepted, sampai gate di atas ditutup dengan bukti runtime UAT baru.
