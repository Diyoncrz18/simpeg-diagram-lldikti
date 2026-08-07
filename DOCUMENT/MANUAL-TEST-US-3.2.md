# Manual Testing Guide: US-3.2 Column Mapping UI
## Tanggal: 7 Agustus 2026

---

## Prerequisites

### 1. Clear Browser Cache
```
Chrome/Edge: Ctrl+Shift+Delete → Clear browsing data
Firefox: Ctrl+Shift+Delete → Clear Now
Safari: Cmd+Option+E
```

### 2. Clear Laravel View Cache
```bash
cd c:\Users\Victus\Desktop\MY-PROJECT\LLDIKTI-SIMPEG\SIMPEG
podman compose exec app php artisan view:clear
```

### 3. Login Credentials
```
URL: http://localhost:8000
Email: admin@simpeg.test
Password: password
Role: admin_kepegawaian
```

---

## Test Scenario 1: Standard Template (All Columns Recognized)

### Steps:
1. Navigate to `http://localhost:8000/pegawai/import`
2. Click **"Unduh Template Utama Pegawai"** (XLSX)
3. Do NOT modify the template (keep all columns as-is)
4. Upload the template file
5. Click **"Upload & Lanjutkan ke Preview"**
6. Wait for Step 2 (Preview & Edit) to load
7. Scroll down to find the **"📋 Informasi Pemetaan Kolom"** card

### Expected Results:

#### Card Header:
```
📋 Informasi Pemetaan Kolom
[18 Kolom Dikenali]  ← Green badge
```

#### Description:
```
Sistem secara otomatis memetakan kolom Excel ke field SIMPEG. 
Kolom yang tidak dikenal akan diabaikan saat impor.
```

#### Button:
```
[Lihat Detail ▼]  ← Clickable button, icon points down
```

#### State:
- ✅ Panel collapsed by default
- ✅ No warning panel visible (because 0 unknown columns)
- ✅ Badge "Tidak Dikenal" NOT visible

---

## Test Scenario 2: View Mapping Details

### Steps:
1. From Scenario 1, click **"Lihat Detail"** button
2. Observe the collapsible panel animation
3. Verify table contents

### Expected Results:

#### Animation:
- ✅ Panel expands smoothly (Alpine.js x-collapse)
- ✅ Button text changes to **"Sembunyikan"**
- ✅ Icon rotates 180° (points up: ▲)

#### Table Structure:
```
┌──────────────────────────────┬──────────────────┬─────────────────────────────┬─────────────────────┐
│ Kolom Excel                  │ Sample Data      │ Dipetakan ke Field SIMPEG   │ Status              │
├──────────────────────────────┼──────────────────┼─────────────────────────────┼─────────────────────┤
│ Nama Pegawai                 │ (data from row1) │ nama_dengan_gelar           │ ✓ Auto-Matched      │
│ Person                       │ (data from row1) │ nama_lengkap                │ ✓ Auto-Matched      │
│ NIP                          │ (data from row1) │ nip                         │ ✓ Auto-Matched      │
│ NIK                          │ (data from row1) │ nik                         │ ✓ Auto-Matched      │
│ Email Pegawai                │ (data from row1) │ email_pribadi               │ ✓ Auto-Matched      │
│ ...                          │ ...              │ ...                         │ ...                 │
└──────────────────────────────┴──────────────────┴─────────────────────────────┴─────────────────────┘
```

#### Verify All 18 Mappings:
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

#### Badge Verification:
- ✅ All rows have **green badge** "✓ Auto-Matched"
- ✅ No rows with yellow badge "⚠ Unknown Column"

#### Sample Data:
- ✅ Shows data from first row of uploaded file
- ✅ If data > 30 characters, truncated with "..."
- ✅ If cell empty, shows "-" (dash) in italic

---

## Test Scenario 3: Collapse Panel

### Steps:
1. From Scenario 2 (panel expanded), click **"Sembunyikan"** button
2. Observe animation

### Expected Results:
- ✅ Panel collapses smoothly
- ✅ Button text changes to **"Lihat Detail"**
- ✅ Icon rotates 180° back (points down: ▼)

---

## Test Scenario 4: Unknown Columns Detection

### Steps:
1. Download template (XLSX)
2. Open in Excel or LibreOffice Calc
3. Add a NEW column: **"Alamat Rumah"** (not in standard template)
4. Fill some data in the new column (e.g., "Jl. Veteran No. 123")
5. Save and upload the modified file
6. Click "Upload & Lanjutkan ke Preview"
7. Observe the **"📋 Informasi Pemetaan Kolom"** card

### Expected Results:

#### Badge Update:
```
📋 Informasi Pemetaan Kolom
[18 Kolom Dikenali]  [1 Tidak Dikenal]  ← Yellow/warning badge appears
```

#### Auto-Expand:
- ✅ Panel **auto-expands** (showDetails = true by default)
- ✅ Table visible immediately (no need to click "Lihat Detail")

#### Table Row for Unknown Column:
```
┌──────────────────────────────┬──────────────────┬─────────────────────────────────────────┬─────────────────────┐
│ Alamat Rumah                 │ Jl. Veteran...   │ ❌ Tidak dipetakan (akan diabaikan)     │ ⚠ Unknown Column    │
└──────────────────────────────┴──────────────────┴─────────────────────────────────────────┴─────────────────────┘
```

#### Row Styling:
- ✅ Row has **yellow background** (`bg-warning/5`)
- ✅ Status badge is **yellow** "⚠ Unknown Column"

#### Warning Panel:
```
┌────────────────────────────────────────────────────────────────────────────┐
│ ⚠  Peringatan: Kolom Tidak Dikenal                                         │
│                                                                            │
│ Kolom berikut tidak cocok dengan template standar dan akan diabaikan      │
│ saat impor:                                                                │
│                                                                            │
│ • Alamat Rumah                                                             │
│                                                                            │
│ Rekomendasi: Gunakan template standar atau pastikan nama kolom Excel      │
│ sesuai dengan template. [Download template standar] untuk referensi.      │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Verification:
- ✅ Warning panel appears below the table
- ✅ Warning icon (triangle) visible
- ✅ Unknown column listed: **"• Alamat Rumah"**
- ✅ Link to download template is clickable

---

## Test Scenario 5: Multiple Unknown Columns

### Steps:
1. Download template
2. Add 3 new columns:
   - **"Alamat"**
   - **"Nomor BPJS"**
   - **"Status Nikah"**
3. Fill data in these columns
4. Upload modified file
5. Preview Step 2

### Expected Results:

#### Badge:
```
[18 Kolom Dikenali]  [3 Tidak Dikenal]  ← Badge shows count 3
```

#### Table:
- ✅ 3 rows with yellow background
- ✅ All 3 have badge "⚠ Unknown Column"

#### Warning Panel List:
```
• Alamat
• Nomor BPJS
• Status Nikah
```

#### Import Flow:
- ✅ Can still proceed to validation (button enabled)
- ✅ Unknown columns will be ignored during import
- ✅ Only 18 recognized columns imported

---

## Test Scenario 6: Responsive Design

### Steps:
1. Test on different screen sizes
2. Resize browser window to simulate:
   - Desktop: 1920px
   - Laptop: 1366px
   - Tablet: 768px (use DevTools device emulation)
   - Mobile: 375px (use DevTools device emulation)

### Expected Results:

#### Desktop (1920px):
- ✅ Card layout full width
- ✅ Badges inline with title
- ✅ Table fully visible (no horizontal scroll needed)

#### Laptop (1366px):
- ✅ Card layout responsive
- ✅ Badges may wrap below title
- ✅ Table visible (might need horizontal scroll)

#### Tablet (768px):
- ✅ Badges wrap below title
- ✅ Table scrollable horizontally (`overflow-x-auto`)
- ✅ Warning panel readable
- ✅ Button "Lihat Detail" not cut off

#### Mobile (375px):
- ✅ Badges stack vertically
- ✅ Table scrollable horizontally
- ✅ Text not overlapping
- ✅ Warning panel readable (text wraps)
- ✅ Button full width (if needed)

---

## Test Scenario 7: Sample Data Display

### Steps:
1. Download template
2. Fill first row with complete data:
   - Nama Pegawai: **"Grantly Antonio Edward Sorongan, S.Kom., M.Kom."** (long name)
   - NIP: **199001012020011001**
   - Email: **grantly.sorongan@example.com**
   - (etc. for all columns)
3. Upload and preview
4. Click "Lihat Detail"
5. Verify **"Sample Data"** column

### Expected Results:

#### Long Text Truncation:
```
Nama Pegawai:
  Sample Data: "Grantly Antonio Edward Sor..."  ← Truncated at 30 chars
```

#### Normal Text:
```
NIP:
  Sample Data: "199001012020011001"  ← Full display (< 30 chars)
```

#### Empty Cell:
```
No KK:
  Sample Data: "-"  ← Dash in italic (if empty)
```

#### Special Characters:
- ✅ Gelar (S.Kom., M.Kom.) displayed correctly
- ✅ Email with @ symbol displayed correctly
- ✅ No HTML escape issues (e.g., &lt; or &gt;)

---

## Test Scenario 8: Continue Import Flow

### Steps:
1. Upload file with 1 unknown column
2. Verify warning panel appears
3. Click **"Lanjutkan ke Validasi"** button
4. Observe Step 3 (Validasi)

### Expected Results:
- ✅ Validation runs successfully
- ✅ Only 18 recognized columns validated
- ✅ Unknown column ignored (not causing error)
- ✅ Import can proceed to Step 4 (Execute)

---

## Browser Console Verification

### Steps:
1. Open Developer Tools (F12)
2. Go to **Console** tab
3. Perform all test scenarios above
4. Monitor console for errors

### Expected Results:
- ✅ No JavaScript errors
- ✅ No Alpine.js errors
- ✅ No 404 or 500 HTTP errors
- ✅ No CORS errors

### Common Errors to Check:
```
❌ Uncaught ReferenceError: mainHeaders is not defined
❌ Alpine expression error: knownHeaders.includes is not a function
❌ Cannot read property 'data' of undefined
❌ Failed to load resource: net::ERR_BLOCKED_BY_CORS
```

If any error appears, report immediately.

---

## Performance Check

### Steps:
1. Upload large file (100+ rows)
2. Click "Lihat Detail" to expand mapping panel
3. Observe response time

### Expected Results:
- ✅ Panel expands in < 500ms (smooth animation)
- ✅ Table renders completely in < 1 second
- ✅ No lag or freeze
- ✅ Scrolling smooth

---

## Checklist Summary

**Before Testing:**
- [ ] Clear browser cache
- [ ] Clear Laravel view cache (`php artisan view:clear`)
- [ ] Login as `admin_kepegawaian`

**Test Scenarios:**
- [ ] Scenario 1: Standard template (all recognized)
- [ ] Scenario 2: View mapping details
- [ ] Scenario 3: Collapse panel
- [ ] Scenario 4: 1 unknown column
- [ ] Scenario 5: 3 unknown columns
- [ ] Scenario 6: Responsive design (4 screen sizes)
- [ ] Scenario 7: Sample data display
- [ ] Scenario 8: Continue import flow

**Technical Checks:**
- [ ] No console errors
- [ ] Alpine.js working (collapsible animation)
- [ ] Badge counts accurate
- [ ] Mapping table accurate (all 18 fields)
- [ ] Warning panel appears when needed
- [ ] Download template link works

**Cross-Browser Testing (optional):**
- [ ] Chrome/Edge
- [ ] Firefox
- [ ] Safari (if available)

---

## Bug Report Template

If you find any issues, use this template:

```markdown
**Bug ID:** BUG-US-3.2-001
**Severity:** [ ] Critical [ ] High [ ] Medium [ ] Low
**Scenario:** [Scenario number]

**Steps to Reproduce:**
1. ...
2. ...
3. ...

**Expected Result:**
...

**Actual Result:**
...

**Screenshots:**
[Attach screenshot]

**Browser:** Chrome 131 / Firefox 132 / etc.
**Console Errors:** [Copy-paste console output]
```

---

## Sign-off

**Tested By:** _________________  
**Date:** _________________  
**Result:** [ ] PASS [ ] FAIL  

**Notes:**
_________________________________________________________________________________
_________________________________________________________________________________

---

**End of Manual Test Guide**
