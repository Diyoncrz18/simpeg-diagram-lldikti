# Laporan Bug — Hasil Testing Browser SIMPEG Fase 1

| Field | Detail |
|-------|--------|
| **Tanggal Testing** | 8 Agustus 2026 |
| **Penguji** | Jordan Sutarto (Lead Backend) |
| **Metode** | Browser testing langsung (Chrome, headed) + verifikasi API lewat session browser + verifikasi state database (PostgreSQL 17 via `podman exec`) |
| **Cakupan** | Babak 2 (Data Pegawai) dan Babak 3 (Import Excel/CSV) |
| **Environment** | SIMPEG Fase 1 — Laravel 12, Blade SSR, Podman containers (nginx `localhost:8000`, PostgreSQL 17, queue worker, scheduler, Mailpit) |
| **Akun Testing** | `demo-klabat-kepeg` (Admin Kepegawaian), `demo-klabat` (Super Admin) |
| **Basis Perbandingan** | PRD-SIMPEG-Fase1-Core.md v1.4, User-Stories-SIMPEG-Fase1.md, keputusan kanonis (K-US-02, K-1..K-4, keputusan 22 Juli 2026) |

---

## Ringkasan Eksekutif

Testing browser langsung terhadap modul Data Pegawai (Babak 2) dan Import Excel/CSV (Babak 3) menemukan **4 bug** yang perlu ditangani sebelum demo/UAT ke LLDIKTI:

| ID | Judul Singkat | Severity | Modul | Status |
|----|---------------|:--------:|-------|--------|
| BUG-01 | Tombol "Tambah Riwayat" (Kepangkatan/Jabatan/KGB) tidak ada di UI | 🔴 Critical | Data Pegawai | Open |
| BUG-02 | Soft delete hanya muncul untuk Super Admin (route mengizinkan Admin) | 🟠 Major | Data Pegawai | Open |
| BUG-03 | Auto-match pemetaan kolom salah petakan (substring) → validasi import terblokir | 🔴 Critical | Import Excel/CSV | Open |
| BUG-04 | NIP duplikat dari DB ditandai ERROR, bukan SKIP (K-US-02 belum terimplementasi) | 🟠 Major | Import Excel/CSV | Open |

**Yang terverifikasi BERFUNGSI BAIK (bukan bug):** login SSO Keycloak, dashboard Admin real data, daftar pegawai (render, search Enter-triggered, filter jenis, pagination, sorting), detail pegawai 8 tab, tambah riwayat via API (backend), soft delete + restore via API (backend), wizard import lengkap (download template, upload, preview, validasi, eksekusi queue), hasil import sesuai keputusan 22 Juli (snapshot tanpa riwayat, tanggal pensiun apa adanya), dan audit log mencatat import.

---

## BUG-01 — Tombol "Tambah Riwayat" Kepangkatan/Jabatan/KGB tidak ada di UI

| Field | Detail |
|-------|--------|
| **Severity** | 🔴 Critical |
| **Modul** | Data Pegawai — Riwayat (Babak 2) |
| **Role Terdampak** | Admin Kepegawaian (role yang bertugas menambah riwayat) |
| **Halaman / URL** | Detail Pegawai — `/pegawai/{id}`, tab **Kepangkatan**, **Jabatan**, **KGB** |
| **User Story Terkait** | US-2.4 AC-2 ("Di setiap tab riwayat ada tombol Tambah Riwayat"), US-2.6 |
| **Sprint/Slice** | Sprint 2 — Slice 2.2 (Riwayat Pegawai) |

**Deskripsi:**
Backend untuk menambah riwayat kepangkatan/jabatan/KGB **berfungsi** (terbukti saat testing: `POST /api/v1/pegawai/{id}/riwayat-kepangkatan` mengembalikan HTTP 201 dan record tersimpan dengan perhitungan TMT yang benar). Namun **tidak ada tombol UI** untuk membuka form-nya, sehingga Admin Kepegawaian tidak bisa menambah riwayat lewat antarmuka.

**Bukti teknis:**
- `resources/views/admin/pegawai/show.blade.php` — tab Kepangkatan (sekitar baris 1310–1340), Jabatan (1342–1374), dan KGB (1376–1406) **tidak memiliki tombol "Tambah"**. Sebagai pembanding, tab Keluarga (baris 1229), Hukuman Disiplin (1416), dan Pendidikan (1468) masing-masing punya tombol `openModal(...)`.
- `resources/views/admin/pegawai/index.blade.php` — modal "Tambah Riwayat (Pangkat/Jabatan/KGB)" lengkap ada (baris 969–1181) dan fungsi `openRiwayatModal()` didefinisikan (baris 227), **tetapi tidak ada satupun pemanggil** `openRiwayatModal(...)` di seluruh view (terverifikasi via grep).

**Langkah Reproduksi:**
1. Login sebagai Admin Kepegawaian.
2. Buka Detail Pegawai, misal `/pegawai/{id}`.
3. Klik tab "Kepangkatan" (atau "Jabatan" / "KGB").
4. Amati: tidak ada tombol "Tambah Riwayat" di tab tersebut.

**Expected result:** Setiap tab riwayat memiliki tombol "Tambah Riwayat" yang membuka form/modal (sesuai US-2.4 AC-2).

**Actual result:** Tidak ada tombol; riwayat hanya bisa ditambah lewat API langsung.

**Dampak demo:** Langkah "Tambah Riwayat" pada Babak 2 tidak dapat didemokan lewat UI. Ini juga memblokir alur EWS (riwayat baru memicu kalkulasi TMT) bila hanya mengandalkan UI.

**Rekomendasi perbaikan:** Tambahkan tombol pemanggil modal riwayat di setiap tab (Kepangkatan/Jabatan/KGB) pada halaman detail, yang memanggil `openModal('pangkat'|'jabatan'|'kgb', ...)`, mengikuti pola tombol yang sudah ada di tab Keluarga/Disiplin/Pendidikan. Atau sambungkan pemanggil `openRiwayatModal(...)` yang sudah ada di `index.blade.php`.

**Status retest:** Belum — menunggu perbaikan.

---

## BUG-02 — Soft delete pegawai hanya muncul untuk Super Admin (route mengizinkan Admin Kepegawaian)

| Field | Detail |
|-------|--------|
| **Severity** | 🟠 Major |
| **Modul** | Data Pegawai — Soft Delete/Restore (Babak 2) |
| **Role Terdampak** | Admin Kepegawaian (tidak bisa soft delete lewat UI); Super Admin (bisa) |
| **Halaman / URL** | Daftar Pegawai — `/pegawai` (tombol hapus di kolom Aksi) |
| **User Story Terkait** | US-2.9 (Soft Delete oleh Admin Kepegawaian), US-2.10 (Super Admin) |
| **Sprint/Slice** | Sprint 3 — Slice 3.3 (Soft Delete dan Restore) |

**Deskripsi:**
Fitur soft delete + restore **berfungsi di backend** (terbukti saat testing: `DELETE /api/v1/pegawai/{id}` lalu `POST /api/v1/pegawai/{id}/restore` berhasil; pegawai hilang dari daftar aktif lalu kembali). Namun tombol hapus di daftar pegawai **hanya dirender untuk Super Admin**, sementara route web untuk delete mengizinkan Admin Kepegawaian. Ada ketidakselarasan antara gate UI dan gate route.

**Bukti teknis:**
- `resources/views/admin/pegawai/index.blade.php` (sekitar baris 679): tombol hapus dibungkus `@if(auth()->user()->role === 'super_admin')`.
- `routes/api/v1/pegawai.php` (baris 47–50): `Route::delete('/{employee}', ...)` dengan middleware `permission:employees.deactivate` + `role:super_admin`.
- `routes/web.php` (baris 401–404): `Route::post('/pegawai/{id}/delete', ...)` dengan middleware `role:super_admin,admin_kepegawaian` + `permission:employees.deactivate` — mengizinkan Admin Kepegawaian.

**Langkah Reproduksi:**
1. Login sebagai **Admin Kepegawaian**.
2. Buka `/pegawai`.
3. Amati kolom Aksi: tombol hapus (ikon tong sampah) **tidak muncul**.
4. (Pembanding) Login sebagai **Super Admin** → tombol hapus muncul dan berfungsi.

**Expected result:** Sesuai PRD US-2.9, Admin Kepegawaian dapat menonaktifkan (soft delete) pegawai.

**Actual result:** Admin Kepegawaian tidak melihat kontrol soft delete; hanya Super Admin yang bisa.

**Catatan:** Temuan ini sejalan dengan catatan di Tracking-Role (inkonsistensi UI/route). Perlu keputusan: apakah memang dibatasi ke Super Admin (maka sesuaikan PRD/route) atau Admin Kepegawaian diberi akses (maka tampilkan tombol + selaraskan middleware API delete yang saat ini `role:super_admin`).

**Dampak demo:** Untuk mendemokan soft delete/restore, gunakan akun **Super Admin**.

**Status retest:** Belum — menunggu keputusan produk + perbaikan.

---

## BUG-03 — Auto-match pemetaan kolom salah petakan (substring) sehingga validasi import terblokir

| Field | Detail |
|-------|--------|
| **Severity** | 🔴 Critical |
| **Modul** | Import Excel/CSV — Pemetaan Kolom (Babak 3) |
| **Role Terdampak** | Admin Kepegawaian (role yang melakukan import) |
| **Halaman / URL** | Import Data Pegawai — `/pegawai/import-data`, **Step 2 (Preview & Edit / Pemetaan Kolom)** |
| **User Story Terkait** | US-3.2 (Upload & Preview + mapping kolom) |
| **Sprint/Slice** | Sprint 3 — Slice 3.1 (Import Excel/CSV) |

**Deskripsi:**
Fungsi `autoMatchHeaders()` memetakan header file ke field SIMPEG memakai pencocokan substring (`cleanHeader.includes(cleanFieldKey)`). Karena itu, dengan template asli yang memang memuat kolom-kolom berikut, terjadi salah petakan ganda:
- Kolom `Kelas Jabatan` → salah ter-match ke field **Jabatan** (karena "kelasjabatan" mengandung "jabatan").
- Kolom `Prodi Pendidikan Terakhir` → salah ter-match ke field **Pendidikan Terakhir** (karena "prodipendidikanterakhir" mengandung "pendidikanterakhir").
- Kolom `Person Formula` → salah ter-match ke field **Person** (karena "personformula" mengandung "person").

Akibatnya muncul **3 pemetaan ganda** (field `Jabatan`, `Pendidikan Terakhir`, dan `Person` masing-masing dipilih oleh 2 kolom). Guard `hasDuplicateMapping` lalu **menonaktifkan tombol "Lanjutkan ke Validasi"**, sehingga proses import **terhenti sebelum validasi**.

**Bukti teknis:**
- `resources/views/admin/pegawai/import.blade.php` (baris 72–84): `autoMatchHeaders()` menggunakan `cleanHeader.includes(cleanFieldKey)`.
- Saat testing, diagnostik komponen menunjukkan `hasDuplicateMapping = true` dengan `dupFields = ["Jabatan", "Person", "Pendidikan Terakhir"]` tepat setelah upload template asli. Setelah pemetaan dikoreksi manual (`Kelas Jabatan→Kelas Jabatan`, `Prodi Pendidikan Terakhir→Prodi Pendidikan Terakhir`, `Person Formula→ignore`), guard terbuka (`hasDuplicateMapping = false`) dan validasi berjalan.

**Langkah Reproduksi:**
1. Login sebagai Admin Kepegawaian.
2. Buka `/pegawai/import-data`.
3. Upload file dengan kolom template asli (termasuk `Kelas Jabatan`, `Prodi Pendidikan Terakhir`, `Person Formula`).
4. Pada Step 2, amati panel "Pemetaan Kolom": muncul peringatan konflik pemetaan ganda dan tombol "Lanjutkan ke Validasi" nonaktif.

**Expected result:** Auto-match memetakan kolom secara benar (exact-match dulu sebelum substring), sehingga tidak ada pemetaan ganda dan validasi bisa langsung dijalankan.

**Actual result:** Terjadi pemetaan ganda; validasi terblokir sampai Admin mengoreksi pemetaan secara manual.

**Dampak demo:** Ini **blocker nyata** — Admin akan terhenti tepat sebelum validasi saat mengimport file asli `daftar_pegawai.xlsx`.

**Rekomendasi perbaikan:** Perbaiki `autoMatchHeaders()` agar mengutamakan exact-match (cocok penuh key/label) sebelum substring, dan/atau urutkan pencocokan dari nama field terpanjang ke terpendek; kolom alias `Person Formula` sebaiknya default `ignore`.

**Status retest:** Belum — menunggu perbaikan.

---

## BUG-04 — NIP duplikat dari database ditandai ERROR, bukan SKIP (K-US-02 belum terimplementasi)

| Field | Detail |
|-------|--------|
| **Severity** | 🟠 Major |
| **Modul** | Import Excel/CSV — Validasi (Babak 3) |
| **Role Terdampak** | Admin Kepegawaian |
| **Halaman / URL** | Import Data Pegawai — `/pegawai/import-data`, **Step 3 (Hasil Validasi)** |
| **User Story Terkait** | US-3.3 AC-5; keputusan kanonis **K-US-02** (5 Agustus 2026) |
| **Sprint/Slice** | Sprint 3 — Slice 3.1 (Import Excel/CSV) |

**Deskripsi:**
Per keputusan K-US-02, baris dengan NIP yang sudah ada di database seharusnya ditandai **SKIP** ("Sudah ada — akan di-skip"), bukan error. Namun saat testing, baris NIP duplikat dari database ditandai **ERROR** dengan pesan "NIP tersebut sudah digunakan/terdaftar."

**Bukti teknis:**
- `app/Support/EmployeeValidationRules.php` (baris 140): rule `nip` masih memuat `'unique:employees,nip'`, sehingga validator Laravel menandai NIP duplikat sebagai error **sebelum** jalur skip berjalan.
- `app/Actions/Employees/ValidateImportBatchAction.php` (baris 118–120): pemeriksaan skip (`Employee::where('nip', ...)->exists()` → status `skip`) berada **setelah** `$validator->validated()`, sehingga tidak pernah tercapai untuk NIP duplikat (jalur skip menjadi dead code).
- Hasil testing aktual: ringkasan validasi `total=6 valid=2 skip=0 error=4`; baris duplikat NIP muncul sebagai `error` dengan pesan "NIP tersebut sudah digunakan/terdaftar."

**Langkah Reproduksi:**
1. Login sebagai Admin Kepegawaian.
2. Buka `/pegawai/import-data`, upload CSV yang berisi NIP yang sudah terdaftar di database (contoh: NIP pegawai existing).
3. Perbaiki pemetaan kolom (lihat BUG-03), lalu jalankan validasi.
4. Amati: baris NIP duplikat ditandai **ERROR**, bukan **SKIP**.

**Expected result (per K-US-02):** NIP yang sudah ada di database → status **SKIP**; NIP ganda dalam satu file → tetap **ERROR**; email yang sudah terdaftar → tetap **ERROR**.

**Actual result:** NIP duplikat dari database → **ERROR**.

**Catatan:** Dokumen User Stories (pembaruan 7 Agustus 2026) masih menandai US-3.3 AC-5 sebagai belum selesai `[ ]`, konsisten dengan temuan ini. Terdapat laporan terpisah yang mengklaim sudah selesai — tidak cocok dengan kondisi kode aktual.

**Dampak demo:** Minor secara fungsional (baris duplikat tetap tidak ter-import), tetapi menyimpang dari spec K-US-02 dan berpotensi membingungkan Admin (duplikat tampak sebagai "kesalahan data" padahal "sudah ada").

**Rekomendasi perbaikan:** Lepas rule `unique:employees,nip` dari `EmployeeValidationRules::import()` agar jalur skip di `ValidateImportBatchAction` hidup, pertahankan pemeriksaan ulang NIP saat penyisipan (mencegah kondisi balapan), lalu perbarui test yang mengunci perilaku lama.

**Status retest:** Belum — menunggu perbaikan.

---

## Catatan Tambahan (bukan bug)

- **Search daftar pegawai bersifat Enter-triggered** (bukan live-typing). Terverifikasi berfungsi: menekan Enter setelah mengetik menembakkan fetch terfilter (`/api/v1/pegawai?...&search=...` → HTTP 200). Ini sesuai desain (`@keydown.enter`), hanya perlu dijelaskan saat demo agar tidak disangka live search.
- **Session idle timeout** terpicu sesuai konfigurasi (US-1.3) saat browser dibiarkan diam — perilaku yang diharapkan, bukan bug.
- Data uji testing sudah **dibersihkan** dari database (pegawai hasil import test dihapus; fixture demo yang sempat ter-nonaktif sudah di-restore).

---

## Referensi Bukti

- Screenshot tiap langkah tersimpan selama sesi testing (folder kerja sementara Playwright).
- State database diverifikasi langsung via `podman exec simpeg_postgres psql` (record pegawai hasil import, `import_batches`, `audit_logs`, dan transisi `deleted_at` untuk soft delete/restore).
- Ringkasan hasil mesin tersimpan di `results-v3.json` (Babak 2) dan `results-babak3.json` (Babak 3) selama sesi testing.

---

*Dokumen ini disusun sebagai catatan bug hasil testing browser langsung. Setiap bug mencantumkan role terdampak, halaman/URL, file terkait, langkah reproduksi, hasil yang diharapkan vs aktual, dan rekomendasi perbaikan, agar bisa langsung ditriase dan ditindaklanjuti sebelum UAT/demo ke LLDIKTI.*

---

## Babak 5 — EWS & Notifikasi (9 Agustus 2026)

### Ringkasan
Seluruh halaman EWS multi-role berfungsi. Scheduler berjalan, dedup bekerja. 2 alert PENSIUN sudah terbuat di DB.

| Fitur | Hasil | Detail |
|---|---|---|
| Admin `/ews` — daftar EWS | ✅ | 2 alert PENSIUN (H-90, H-180) tampil dengan status, nama pegawai, followup |
| Super Admin `/konfigurasi` | ✅ | Halaman konfigurasi EWS termuat, form update tersedia |
| Pimpinan `/pimpinan/ews` | ✅ | Halaman EWS Pimpinan termuat dengan data |
| Kabag `/kepala-bagian/ews` | ✅ | Halaman EWS Kabag termuat |
| Pegawai `/dashboard` — EWS pribadi | ✅ | Section EWS Pribadi tampil di dashboard |
| Pegawai `/dashboard/ews-saya` | ✅ | Halaman terpisah EWS pegawai termuat |
| Scheduler `app:run-ews` | ✅ | Scan 13 pegawai, dedup berfungsi, status `berhasil` |
| Notifikasi in-app | ⚠️ | Konfigurasi channel benar (`ews.pensiun` → in-app + email), tapi tidak ada notifikasi terbuat karena pegawai target (Akub_Busura, Rivai Hamzah) tidak punya akun user — **bukan bug, ini data issue** |
| Followup alert | ⚠️ | Belum diuji end-to-end via UI (form SK upload untuk PENSIUN) |

### Catatan
- Tidak ada bug baru ditemukan di Babak 5.
- Notifikasi untuk Admin Kepegawaian tidak terkirim karena `NotificationRecipientResolver` memerlukan pegawai target punya `user_id` — untuk demo, pastikan pegawai uji EWS punya akun user yang ter-link.
- Gap P0 Sprint 5 (double scheduler registration) tidak berdampak pada testing — scheduler hanya jalan sekali.

---

## Babak 6 — Dashboard & Laporan (9 Agustus 2026)

### Ringkasan
Podman networking mati setelah `podman machine restart` — port forwarding WSL ke Windows host putus. DB di-reseed ulang (migrasi + seeder OK). Testing terbatas pada halaman yang sempat termuat sebelum network putus + verifikasi route Laravel.

| Fitur | Hasil | Detail |
|---|---|---|
| Dashboard Admin `/dashboard` | ✅ | Halaman termuat, widget tampil (sebagian dummy — tracking Sprint 6) |
| Dashboard Pimpinan `/pimpinan/dashboard` | ✅ | Termuat dengan data real (statistik, EWS, distribusi golongan) |
| Dashboard Pegawai `/dashboard` | ✅ | Profil ringkas, card saldo cuti, daftar cuti, EWS pribadi |
| Dashboard Kabag `/kepala-bagian/dashboard` | ⚠️ | Route terdaftar, halaman seharusnya berfungsi (quick action + antrean cuti) |
| Export Pegawai `/laporan/export-pegawai` | ⚠️ | Route + controller terdaftar (Excel + PDF + custom), belum dicek konten |
| Export Cuti `/cuti/laporan` | ⚠️ | Route + controller terdaftar (Excel + PDF), belum dicek konten |
| Audit Log `/dashboard/audit` | ⚠️ | Route + controller terdaftar, belum dicek isi tabel |
| Pimpinan Report `/pimpinan/laporan` | ⚠️ | Route + controller terdaftar (nominatif, kepangkatan), belum dicek |
| Data Master `/admin/data-master` | ✅ | Termuat (CRUD reference tables) |
| RBAC `/rbac` | ⚠️ | Route terdaftar (Super Admin only), belum dicek |

### Catatan
- Tidak ada bug baru ditemukan di Babak 6.
- Podman networking on Windows gagal setelah machine restart — perlu reboot Windows. Semua container up, migrasi/seeder OK, nginx sehat.
- Dashboard Admin MASIH DUMMY (6 dari 7 widget) per tracking Sprint 6 — `BuildAdminDashboardAction` belum dibuat.
- Export Pegawai punya gap P0 PII (`email_pribadi` + `no_hp` dikirim ke DOM di route preview).

### Rekomendasi
- Reboot Windows → `podman-compose up -d` → smoke test manual 5 menit untuk halaman ⚠️ di atas.
- Prioritas: Dashboard Admin (#39), Export Pegawai (#42), Audit Log (#47).
