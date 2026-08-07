# QA Testing Checklist
## SIMPEG Fase 1 - Grantly User Stories
## Tanggal: 7 Agustus 2026

---

## Pre-Testing Setup

### Environment
- [ ] Application running: `http://localhost:8000`
- [ ] Database seeded with test data
- [ ] All containers running (app, nginx, postgres, queue, mailpit)
- [ ] Clear browser cache and cookies

### Test Users
```
Admin Kepegawaian:
- Email: admin@simpeg.test
- Password: password
- Role: admin_kepegawaian

Kepala Bagian:
- Email: kepala.bagian@simpeg.test  
- Password: password
- Role: kepala_bagian

Pegawai:
- Email: pegawai@simpeg.test
- Password: password
- Role: pegawai
```

---

## US-3.2: Column Mapping UI (AC-4,5)

### Test Case 1: Column Mapping Info Panel Display

**Precondition:** Login sebagai `admin_kepegawaian`

**Steps:**
1. [ ] Navigate ke `/pegawai/import`
2. [ ] Download template utama (XLSX)
3. [ ] Upload template (tanpa modifikasi)
4. [ ] Klik "Upload & Lanjutkan ke Preview"
5. [ ] Pada Step 2 (Preview & Edit), cari section **"📋 Informasi Pemetaan Kolom"**

**Expected Result:**
- [ ] Card dengan title **"📋 Informasi Pemetaan Kolom"** tampil
- [ ] Badge menampilkan: **"18 Kolom Dikenali"** (success/hijau)
- [ ] Badge **"0 Tidak Dikenal"** tidak tampil (karena 0)
- [ ] Deskripsi: "Sistem secara otomatis memetakan kolom Excel ke field SIMPEG..."
- [ ] Button **"Lihat Detail"** tersedia dengan icon dropdown
- [ ] Collapsible panel tertutup (default)

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 2: Column Mapping Detail Table

**Steps:**
1. [ ] Dari Test Case 1, klik button **"Lihat Detail"**
2. [ ] Verifikasi tabel mapping muncul

**Expected Result:**
- [ ] Collapsible panel terbuka dengan smooth animation
- [ ] Tabel dengan 4 kolom tampil:
  - [ ] **Kolom Excel** — nama kolom dari file
  - [ ] **Sample Data** — data dari row pertama (max 30 char)
  - [ ] **Dipetakan ke Field SIMPEG** — nama field database
  - [ ] **Status** — badge auto-matched
- [ ] 18 rows tampil untuk semua kolom
- [ ] Semua rows memiliki badge **"✓ Auto-Matched"** (success/hijau)
- [ ] Mapping benar:
  - [ ] `Nama Pegawai` → `nama_dengan_gelar`
  - [ ] `Person` → `nama_lengkap`
  - [ ] `NIP` → `nip`
  - [ ] `NIK` → `nik`
  - [ ] `Email Pegawai` → `email_pribadi`
  - [ ] `Status Kepegawaian` → `jenis_pegawai`
  - [ ] `Role` → `role`
- [ ] Sample data tampil (truncated jika > 30 char)
- [ ] Button text berubah menjadi **"Sembunyikan"**
- [ ] Icon dropdown berputar 180° (rotated)

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 3: Unknown Column Detection

**Steps:**
1. [ ] Download template utama
2. [ ] Buka di Excel/LibreOffice
3. [ ] Tambahkan kolom baru: **"Alamat Rumah"** (tidak ada di template standar)
4. [ ] Isi data di kolom baru
5. [ ] Upload file
6. [ ] Lanjutkan ke Step 2 (Preview)
7. [ ] Lihat section **"📋 Informasi Pemetaan Kolom"**

**Expected Result:**
- [ ] Badge menampilkan: **"18 Kolom Dikenali"** (hijau)
- [ ] Badge menampilkan: **"1 Tidak Dikenal"** (warning/kuning)
- [ ] Panel **auto-expand** (showDetails = true by default)
- [ ] Tabel mapping tampil otomatis
- [ ] Row "Alamat Rumah" memiliki:
  - [ ] Background kuning muda (`bg-warning/5`)
  - [ ] Status badge: **"⚠ Unknown Column"** (warning)
  - [ ] Mapping: **"❌ Tidak dipetakan (akan diabaikan)"**
- [ ] Warning panel tampil di bawah tabel:
  - [ ] Title: **"Peringatan: Kolom Tidak Dikenal"**
  - [ ] Icon warning (segitiga)
  - [ ] Deskripsi: "Kolom berikut tidak cocok dengan template standar dan akan diabaikan saat impor:"
  - [ ] List kolom: **"• Alamat Rumah"**
  - [ ] Rekomendasi dengan link download template

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 4: Multiple Unknown Columns

**Steps:**
1. [ ] Download template
2. [ ] Tambahkan 3 kolom baru: **"Alamat"**, **"Nomor BPJS"**, **"Status Nikah"**
3. [ ] Upload file & preview

**Expected Result:**
- [ ] Badge: **"18 Kolom Dikenali"**, **"3 Tidak Dikenal"**
- [ ] Panel auto-expand
- [ ] 3 rows dengan background warning
- [ ] Warning panel menampilkan list 3 kolom:
  - [ ] • Alamat
  - [ ] • Nomor BPJS
  - [ ] • Status Nikah
- [ ] Import tetap bisa dilanjutkan (kolom unknown diabaikan)

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 5: Collapsible Panel Interaction

**Steps:**
1. [ ] Upload file dengan semua kolom valid
2. [ ] Klik **"Lihat Detail"** → Panel expand
3. [ ] Klik **"Sembunyikan"** → Panel collapse
4. [ ] Repeat 2x

**Expected Result:**
- [ ] Panel toggle smooth (Alpine.js x-collapse)
- [ ] Icon arrow rotates 180° setiap toggle
- [ ] Button text toggle: "Lihat Detail" ↔ "Sembunyikan"
- [ ] No console errors
- [ ] Animation smooth (tidak laggy)

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 6: Sample Data Display

**Steps:**
1. [ ] Upload file dengan data lengkap di row pertama
2. [ ] Lihat detail mapping
3. [ ] Verifikasi kolom **"Sample Data"**

**Expected Result:**
- [ ] Sample data tampil dari first row (`allRows[0]`)
- [ ] Data truncated jika > 30 karakter (dengan "...")
- [ ] Jika cell kosong → tampil **"-"** (dash) dengan italic
- [ ] Special characters displayed correctly (no HTML escape issues)

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 7: Mapping Accuracy (All Fields)

**Steps:**
1. [ ] Upload template standar
2. [ ] Lihat detail mapping
3. [ ] Verifikasi mapping untuk semua 18 kolom

**Expected Result (complete mapping):**
- [ ] `Nama Pegawai` → `nama_dengan_gelar`
- [ ] `Person` → `nama_lengkap`
- [ ] `Email Pegawai` → `email_pribadi`
- [ ] `Golongan` → `golongan_terakhir`
- [ ] `Jabatan` → `jabatan_terakhir`
- [ ] `Kelas Jabatan` → `kelas_jabatan_terakhir`
- [ ] `NIP` → `nip`
- [ ] `NIK` → `nik`
- [ ] `No KK` → `no_kk`
- [ ] `Nomor Telepon` → `no_hp`
- [ ] `Pangkat` → `pangkat_terakhir`
- [ ] `Pendidikan Terakhir` → `pendidikan_terakhir`
- [ ] `Pensiun` → `tanggal_pensiun`
- [ ] `Prodi Pendidikan Terakhir` → `prodi_pendidikan_terakhir`
- [ ] `Status Kepegawaian` → `jenis_pegawai`
- [ ] `Tanggal Lahir` → `tanggal_lahir`
- [ ] `Role` → `role`
- [ ] `Person Formula` → `(internal check)`

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 8: Responsive Design

**Steps:**
1. [ ] Test mapping panel pada berbagai screen sizes:
   - [ ] Desktop (1920px)
   - [ ] Laptop (1366px)
   - [ ] Tablet (768px)
   - [ ] Mobile (375px)

**Expected Result:**
- [ ] Card layout responsive
- [ ] Badges wrap on small screens
- [ ] Tabel scrollable horizontal (overflow-x-auto)
- [ ] Button "Lihat Detail" tidak terpotong
- [ ] Warning panel readable di mobile
- [ ] No text overlap

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

## US-3.3: Skip NIP Duplikat dari Database

### Test Case 1: Import dengan NIP Duplikat dari Database
**Precondition:** Database sudah memiliki pegawai dengan NIP `199001012020011001`

**Steps:**
1. [ ] Login sebagai `admin_kepegawaian`
2. [ ] Navigate ke `/pegawai/import`
3. [ ] Download template utama (XLSX)
4. [ ] Isi template dengan data:
   - Row 2: NIP `199001012020011001` (sudah ada di DB)
   - Row 3: NIP `199001012020011002` (baru)
5. [ ] Upload file
6. [ ] Klik "Lanjutkan ke Validasi"

**Expected Result:**
- [ ] Step 3 (Validasi) muncul
- [ ] Ringkasan menampilkan:
  - Total: 2 baris
  - Valid: 1 baris
  - Terlewat (Skip): 1 baris
  - Error: 0 baris
- [ ] Row dengan NIP duplikat memiliki badge **TERLEWAT** (kuning/warning)
- [ ] Skip reason: "NIP sudah terdaftar di database — baris akan dilewati."
- [ ] Tombol "Import Hanya yang Valid" tersedia
7. [ ] Klik "Import Hanya yang Valid"
8. [ ] Verifikasi hanya 1 pegawai baru ditambahkan

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 2: Import dengan NIP Duplikat dalam File

**Steps:**
1. [ ] Login sebagai `admin_kepegawaian`
2. [ ] Navigate ke `/pegawai/import`
3. [ ] Isi template dengan data:
   - Row 2: NIP `199001012020011003`
   - Row 3: NIP `199001012020011003` (duplicate)
4. [ ] Upload file
5. [ ] Klik "Lanjutkan ke Validasi"

**Expected Result:**
- [ ] Step 3 (Validasi) muncul
- [ ] Ringkasan menampilkan:
  - Total: 2 baris
  - Valid: 0 baris
  - Error: 2 baris (kedua row error)
- [ ] Error message: "NIP 199001012020011003 muncul lebih dari sekali dalam berkas. Perbaiki duplikasi sebelum mengimpor."
- [ ] Badge **ERROR** (merah) pada kedua row

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 3: Import dengan Email Duplikat

**Steps:**
1. [ ] Login sebagai `admin_kepegawaian`
2. [ ] Isi template dengan email yang sudah terdaftar: `existing@example.com`
3. [ ] Upload & validasi

**Expected Result:**
- [ ] Badge **ERROR** (merah)
- [ ] Error message: "Email pegawai sudah terdaftar di database."
- [ ] Row tidak dapat diimpor

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

## US-5.4: Filter Notifikasi Berdasarkan Kinerja

### Test Case 1: Pegawai dengan Kinerja Buruk Tidak Dapat Notifikasi

**Precondition:** 
- Pegawai dengan `is_kinerja_baik = false`
- `tanggal_kenaikan_pangkat_berikutnya` dalam 90 hari

**Steps:**
1. [ ] Login sebagai `super_admin`
2. [ ] Navigate ke `/ews/scheduler` (jika ada UI) atau run:
   ```bash
   podman compose exec app php artisan ews:process
   ```
3. [ ] Check dashboard EWS
4. [ ] Check notifikasi pegawai

**Expected Result:**
- [ ] Alert **dibuat** di database (`ews_alerts` table)
- [ ] Field `is_eligible = false`
- [ ] Field `notified_at = null` (tidak ada notifikasi)
- [ ] Pegawai **TIDAK** menerima email/notifikasi
- [ ] Admin dapat lihat alert di dashboard

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 2: Pegawai dengan Kinerja Baik Dapat Notifikasi

**Precondition:**
- Pegawai dengan `is_kinerja_baik = true`
- Tidak ada `discipline_records` aktif
- `tanggal_kenaikan_pangkat_berikutnya` dalam 90 hari

**Steps:**
1. [ ] Run scheduler: `podman compose exec app php artisan ews:process`
2. [ ] Check mailpit: `http://localhost:8025`
3. [ ] Check notifikasi pegawai

**Expected Result:**
- [ ] Alert **dibuat** dengan `is_eligible = true`
- [ ] Field `notified_at` terisi dengan timestamp
- [ ] Email notification terkirim (cek Mailpit)
- [ ] Pegawai melihat notifikasi di dashboard

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 3: KGB/Pensiun Tidak Terpengaruh Flag Kinerja

**Precondition:**
- Pegawai dengan `is_kinerja_baik = false`
- `tanggal_kgb_berikutnya` atau `tanggal_pensiun` dalam range

**Steps:**
1. [ ] Run scheduler
2. [ ] Check notifikasi KGB/Pensiun

**Expected Result:**
- [ ] Notifikasi KGB/Pensiun **tetap dikirim**
- [ ] Tidak terpengaruh oleh flag `is_kinerja_baik`

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

## US-4.5: Saldo Cuti di Halaman Approval

### Test Case 1: Approver Melihat Saldo Cuti Pemohon

**Precondition:**
- Login sebagai `kepala_bagian`
- Ada pengajuan cuti tahunan yang menunggu approval

**Steps:**
1. [ ] Navigate ke `/cuti/approval`
2. [ ] Klik detail pada pengajuan cuti tahunan
3. [ ] Scroll ke section "Informasi Saldo Cuti Pemohon"

**Expected Result:**
- [ ] Card dengan title **"📊 Informasi Saldo Cuti Pemohon"** muncul
- [ ] Badge **"Tahun 2026"** (atau tahun berjalan) tampil
- [ ] 4 grid boxes visible:
  - [ ] **Jatah 2026:** [angka] hari
  - [ ] **Carry-Over:** [angka] hari
  - [ ] **Sudah Terpakai:** [angka] hari (warna warning)
  - [ ] **Sisa Saldo:** [angka] hari (warna success/hijau)
- [ ] Detail row tampil:
  - [ ] Total Tersedia: [angka] hari
  - [ ] Dialokasikan: [angka] hari
  - [ ] Dilindungi: [angka] hari
- [ ] Section **"Riwayat Penggunaan"** tampil:
  - [ ] Tahun 2024 (N-2): [used]/[total] hari dengan badge **N-2**
  - [ ] Tahun 2025 (N-1): [used]/[total] hari dengan badge **N-1**

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 2: Saldo Tidak Tampil untuk Cuti Non-Tahunan

**Steps:**
1. [ ] Login sebagai `kepala_bagian`
2. [ ] Buka detail pengajuan **cuti sakit** atau **cuti besar**

**Expected Result:**
- [ ] Card "Informasi Saldo Cuti Pemohon" **TIDAK muncul**
- [ ] Hanya detail pengajuan cuti yang tampil

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 3: Saldo Akurat dengan Data Database

**Steps:**
1. [ ] Query database untuk pegawai tertentu:
   ```sql
   SELECT * FROM leave_balances 
   WHERE employee_id = '[employee_id]' 
   AND tahun = 2026;
   ```
2. [ ] Bandingkan dengan data di UI

**Expected Result:**
- [ ] Angka di UI sama dengan database
- [ ] Jatah = `jatah_awal`
- [ ] Carry-Over = `n1`
- [ ] Terpakai = `terpakai`
- [ ] Sisa = `sisa`

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

## US-5.5: Milestone Calculations & Storage

### Test Case 1: Milestone Satyalancana Tersimpan

**Precondition:**
- Pegawai dengan TMT pengangkatan `2006-01-01`

**Steps:**
1. [ ] Login sebagai `admin_kepegawaian`
2. [ ] Update riwayat kepangkatan pegawai (trigger kalkulasi)
3. [ ] Query database:
   ```sql
   SELECT * FROM employee_milestones 
   WHERE employee_id = '[employee_id]' 
   AND type = 'satyalancana'
   ORDER BY milestone_date;
   ```

**Expected Result:**
- [ ] 3 records ditemukan:
  - [ ] Milestone 1: `milestone_date = 2016-01-01` (10 tahun)
  - [ ] Milestone 2: `milestone_date = 2026-01-01` (20 tahun)
  - [ ] Milestone 3: `milestone_date = 2036-01-01` (30 tahun)
- [ ] Setiap record memiliki:
  - [ ] `is_active = true`
  - [ ] `metadata` berisi: `tmt_pengangkatan`, `years_of_service`
  - [ ] `calculated_at` = hari ini

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 2: All Milestone Types Created

**Precondition:**
- Pegawai dengan data lengkap (appointment, rank, salary, position)

**Steps:**
1. [ ] Update riwayat pegawai (trigger kalkulasi)
2. [ ] Query database:
   ```sql
   SELECT type, milestone_date, is_active 
   FROM employee_milestones 
   WHERE employee_id = '[employee_id]'
   ORDER BY type, milestone_date;
   ```

**Expected Result:**
- [ ] Milestone types found:
  - [ ] `kenaikan_pangkat` — 1 record
  - [ ] `kgb` — 1 record
  - [ ] `pensiun` — 1 record
  - [ ] `satyalancana` — 3 records (10/20/30 years)
- [ ] All `is_active = true`
- [ ] Dates calculated correctly

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

### Test Case 3: Milestone Update (Not Duplicate)

**Steps:**
1. [ ] Create milestone for employee (via history update)
2. [ ] Update history again with new date
3. [ ] Query milestones

**Expected Result:**
- [ ] Only 1 record per type (no duplicates)
- [ ] `milestone_date` updated to new value
- [ ] `calculated_at` updated to today

**Actual Result:** _________________

**Status:** [ ] PASS [ ] FAIL

---

## Cross-Feature Integration Tests

### Test 1: Import → EWS → Notification

**Steps:**
1. [ ] Import pegawai baru dengan data lengkap
2. [ ] Set `is_kinerja_baik = true`
3. [ ] Set `tanggal_kenaikan_pangkat_berikutnya` = 90 hari dari sekarang
4. [ ] Run scheduler
5. [ ] Check notification

**Expected Result:**
- [ ] Pegawai berhasil diimpor
- [ ] Milestone tersimpan di `employee_milestones`
- [ ] Alert EWS dibuat
- [ ] Notifikasi terkirim (cek Mailpit)

**Status:** [ ] PASS [ ] FAIL

---

### Test 2: Leave Request → Approval → Balance Display

**Steps:**
1. [ ] Login sebagai `pegawai`
2. [ ] Submit cuti tahunan 5 hari
3. [ ] Login sebagai `kepala_bagian`
4. [ ] Buka approval detail
5. [ ] Verifikasi saldo cuti tampil

**Expected Result:**
- [ ] Saldo cuti pemohon tampil dengan lengkap
- [ ] Saldo accurate (reflect submitted request as "reserved")
- [ ] Approver dapat decision dengan informasi lengkap

**Status:** [ ] PASS [ ] FAIL

---

## Performance Tests

### Test 1: Import Large File (1000 rows)

**Steps:**
1. [ ] Generate Excel dengan 1000 rows data pegawai
2. [ ] Upload & validate
3. [ ] Execute import
4. [ ] Measure time

**Expected Result:**
- [ ] Upload completes in < 10 seconds
- [ ] Validation completes in < 30 seconds
- [ ] Import completes in < 2 minutes
- [ ] No timeout errors

**Actual Time:**
- Upload: _______ seconds
- Validation: _______ seconds
- Import: _______ minutes

**Status:** [ ] PASS [ ] FAIL

---

### Test 2: EWS Scheduler Performance

**Steps:**
1. [ ] Ensure 100+ employees in database
2. [ ] Run scheduler with timing:
   ```bash
   time podman compose exec app php artisan ews:process
   ```

**Expected Result:**
- [ ] Completes in < 30 seconds
- [ ] No N+1 query issues
- [ ] All employees processed

**Actual Time:** _______ seconds

**Status:** [ ] PASS [ ] FAIL

---

## Security Tests

### Test 1: Authorization - Import Access

**Steps:**
1. [ ] Login sebagai `pegawai` (tidak punya permission)
2. [ ] Navigate to `/pegawai/import`

**Expected Result:**
- [ ] 403 Forbidden atau redirect

**Status:** [ ] PASS [ ] FAIL

---

### Test 2: Authorization - Approval Access

**Steps:**
1. [ ] Login sebagai `pegawai` A
2. [ ] Try to access approval detail of pegawai B's leave request

**Expected Result:**
- [ ] 403 Forbidden (unless pegawai A is approver)

**Status:** [ ] PASS [ ] FAIL

---

### Test 3: CSRF Protection

**Steps:**
1. [ ] Try to submit form without CSRF token
2. [ ] Using curl/Postman

**Expected Result:**
- [ ] 419 CSRF Token Mismatch

**Status:** [ ] PASS [ ] FAIL

---

## Browser Compatibility

### Desktop Browsers
- [ ] Chrome/Edge (v120+)
- [ ] Firefox (v120+)
- [ ] Safari (v17+)

### Mobile Browsers
- [ ] Chrome Mobile (Android)
- [ ] Safari Mobile (iOS)

### Test on Each Browser:
- [ ] Import wizard UI renders correctly
- [ ] Table pagination works
- [ ] Buttons clickable
- [ ] Forms submittable
- [ ] No console errors

---

## Bug Report Template

If any test fails, report using this format:

```
**Bug ID:** BUG-001
**Severity:** [ ] Critical [ ] High [ ] Medium [ ] Low
**User Story:** US-X.X
**Test Case:** [Test Case Name]

**Steps to Reproduce:**
1. ...
2. ...
3. ...

**Expected Result:**
...

**Actual Result:**
...

**Screenshots:**
[Attach screenshots]

**Environment:**
- Browser: ...
- OS: ...
- User Role: ...

**Logs:**
[Attach relevant logs from storage/logs/laravel.log]
```

---

## QA Sign-off

**Tested By:** _________________  
**Date:** _________________  
**Overall Status:** [ ] PASS [ ] FAIL  

**Summary:**
- Total Test Cases: _____
- Passed: _____
- Failed: _____
- Blocked: _____

**Recommendation:** [ ] Ready for Production [ ] Needs Bug Fixes

**Notes:**
_________________________________________________________________________________
_________________________________________________________________________________
_________________________________________________________________________________

---

**End of QA Checklist**
