# Keputusan Tahap 0 — Saldo dan Batas Pengajuan Cuti

| Field | Detail |
|---|---|
| Tanggal keputusan | 27 Juli 2026 |
| Status | **Disetujui** — menjadi acuan implementasi saldo dan batas pengajuan cuti Fase 1 sampai ada keputusan yang lebih baru |
| Disetujui melalui | Persetujuan pengguna pada task Codex: “Setujui keputusan Tahap 0.” |
| Acuan utama | `PRD-SIMPEG-Fase1-Core.md` §9.2–§9.4; `User-Stories-SIMPEG-Fase1.md` US-4.1, US-4.3, dan US-4.6; `Panduan-Penulisan-Kode-SIMPEG.md`; `AGENTS.md` |
| Cakupan | Saldo cuti tahunan saat pengajuan aktif, pengajuan lintas tahun, serta batas jenis cuti yang eksplisit di PRD |
| Di luar cakupan | Integrasi SIASN/BKN, aturan operasional Dosen DPK, aktivasi WhatsApp Business produksi, dan penetapan batas numerik baru yang tidak ditulis di PRD |

---

## K-CUT-01 — Pengajuan aktif mencadangkan hak cuti tahunan tanpa memotong saldo final

### Keputusan

Setiap pengajuan **Cuti Tahunan** yang masih aktif mencadangkan hak cuti sebesar jumlah hari kerjanya. Reservasi **bukan** pemotongan saldo final.

### Aturan penerapan

1. Saldo ledger resmi (`sisa`) dan `terpakai` hanya berubah saat keputusan final **`Disetujui`**, sesuai PRD dan US-4.6.
2. Sistem menghitung `saldo_dapat_diajukan` dari saldo tersedia dikurangi total reservasi aktif milik pegawai pada tahun cuti yang sama.
3. Reservasi aktif berlaku untuk pengajuan dengan status berikut:
   - `menunggu_approval`;
   - `ditangguhkan`;
   - `perlu_perubahan`.
4. Pada `perlu_perubahan`, reservasi tetap terikat pada request yang sama dan dihitung ulang atomik saat Pegawai mengirim ulang pengajuan dengan tanggal/jumlah hari baru.
5. Saat keputusan final `disetujui`, reservasi dikonversi menjadi pemotongan saldo final yang sudah berlaku di ledger. Sistem tidak boleh melakukan pemotongan ganda.
6. Saat `tidak_disetujui`, reservasi dilepas. Jika pada masa depan tersedia pembatalan pengajuan Fase 1 yang sah, pembatalan juga wajib melepas reservasi secara atomik.
7. `ditangguhkan` tidak melepas reservasi sampai ada keputusan atau tindak lanjut yang mengubah statusnya.
8. Form Pegawai wajib membedakan dengan jelas: **Saldo tersedia aktual**, **Dialokasikan untuk pengajuan aktif**, **Masih dapat diajukan**, dan **Sudah terpakai setelah disetujui**.
9. Setiap pembuatan, penyesuaian, konversi, dan pelepasan reservasi harus dapat diaudit; reservasi tidak boleh ditulis sebagai pemakaian cuti final.

### Alasan

- Menutup risiko beberapa pengajuan pending yang totalnya melebihi hak Pegawai.
- Mempertahankan ketentuan bahwa hanya keputusan final `Disetujui` yang mengurangi saldo cuti tahunan.
- Memberi informasi yang jujur pada Pegawai tanpa mencampur hak dasar, koreksi administratif, pemakaian final, dan alokasi sementara.

---

## K-CUT-02 — Batas tahun kalender berlaku untuk semua jenis cuti

### Keputusan

Aturan PRD bahwa **satu pengajuan tidak boleh melewati tahun kalender** berlaku untuk seluruh jenis cuti, tanpa pengecualian, termasuk Cuti Melahirkan dan CLTN.

### Aturan penerapan

1. Rentang Desember–Januari harus dibuat sebagai pengajuan terpisah per tahun kalender.
2. Cuti Melahirkan dan CLTN yang secara keseluruhan dapat berlangsung lebih dari satu tahun tetap dibagi menjadi beberapa pengajuan tahunan.
3. Batas maksimum Cuti Melahirkan dan CLTN dihitung **kumulatif** atas seluruh pengajuan yang merupakan kelanjutan dari kebutuhan cuti yang sama; pemecahan per tahun tidak boleh menjadi cara untuk melewati batas maksimum.
4. Implementasi wajib menyimpan atau menentukan keterkaitan antarpengajuan tersebut secara eksplisit dan auditabel; sistem tidak boleh menyimpulkan keterkaitan dari teks alasan bebas semata.
5. Validasi lintas tahun di `Store` dan `Resubmit` tetap menjadi pengaman backend. Umpan balik di form hanya melengkapi pengalaman pengguna, bukan menggantikan validasi server.

### Alasan

- Mengikuti acceptance criteria PRD dan User Stories secara langsung.
- Menghindari ambiguitas reset kuota, carry-over, serta audit saldo antar tahun.
- Batas durasi maksimum tetap terlindungi meskipun pengajuan harus dipisahkan per tahun.

---

## K-CUT-03 — Batas jenis cuti memakai durasi kalender dan aturan eksplisit PRD

### Keputusan

Batas yang eksplisit dalam PRD diterapkan sebagai aturan server-side yang berbasis tanggal kalender, bukan hitungan hari tetap atau pencocokan nama tampilan jenis cuti.

### Aturan penerapan

| Jenis cuti | Aturan yang disetujui |
|---|---|
| Cuti Tahunan | Maksimum sebesar `saldo_dapat_diajukan` berbasis hari kerja. |
| Cuti Melahirkan | Maksimum 3 bulan kalender secara kumulatif untuk kebutuhan cuti yang sama. |
| Cuti Besar | Hanya PNS dan membutuhkan masa kerja minimal 5 tahun kalender sejak TMT pengangkatan. |
| CLTN | Hanya PNS dan maksimum 3 tahun kalender secara kumulatif untuk kebutuhan cuti yang sama. |
| Cuti Sakit | Tidak ditambah batas numerik baru pada Fase 1. Ketentuan surat dokter/aturan rinci menunggu sumber peraturan yang disahkan. |
| Cuti Karena Alasan Penting | Tidak ditambah batas numerik baru pada Fase 1. Ketentuan rinci menunggu sumber peraturan yang disahkan. |

1. Tanggal mulai dan tanggal selesai diperlakukan sebagai rentang tanggal inklusif.
2. Perhitungan bulan/tahun menggunakan aritmetika kalender; implementasi tidak boleh mengganti 3 bulan menjadi 90 hari atau 3 tahun menjadi 1.095 hari.
3. Implementasi menggunakan metadata kode/reference jenis cuti, bukan `nama` jenis cuti, agar perubahan label UI tidak mengubah penegakan aturan.
4. Aturan yang belum memiliki batas numerik eksplisit di PRD tidak boleh diisi dengan nilai asumsi. Bila peraturan resmi tersedia, keputusan ini harus diperbarui terlebih dahulu sebelum kode diberi batas baru.

### Alasan

- Menerapkan batas yang benar-benar tertulis di PRD tanpa menambah kebijakan legal yang belum disetujui.
- Menghindari kesalahan durasi di bulan dengan panjang berbeda dan tahun kabisat.
- Menjaga reference table sebagai sumber konfigurasi jenis cuti serta validasi server sebagai sumber kebenaran.

---

## Konsekuensi implementasi

1. Tahap 2–3 membangun payload saldo tunggal dan preview saldo berdasarkan tahun `tanggal_mulai`; payload membedakan saldo aktual, reservasi aktif, saldo yang masih dapat diajukan, pemakaian final, serta koreksi administratif.
2. Tahap 4–5 menambahkan model/ledger atau mekanisme reservasi yang append-only dan auditabel, dengan transaksi serta penguncian yang aman terhadap submit paralel.
3. Tahap 4–5 menambahkan keterkaitan eksplisit antarpengajuan untuk penerapan maksimum kumulatif Cuti Melahirkan dan CLTN yang dipisah per tahun.
4. Semua perubahan wajib memiliki Feature/Unit test untuk guest denial, RBAC/data scope, submit valid/tidak valid, perubahan status, audit, atomicity, dan regresi saldo final.
5. Perubahan route/API harus mengikuti pola: `Route -> Middleware/RBAC -> FormRequest -> Controller -> Action -> Service/Model -> Resource/Payload -> Response`.

## Batas yang tetap berlaku

- Label keputusan cuti tetap persis: `Disetujui`, `Perubahan`, `Ditangguhkan`, dan `Tidak Disetujui`.
- Kode tidak boleh memperkenalkan login manual produksi, mutasi saldo tanpa audit, atau akses saldo Pegawai lain dari endpoint Pegawai.
- Pengurangan saldo tahunan hanya terjadi setelah final `Disetujui`; keputusan lain tidak mengurangi saldo final.
- Keputusan ini melengkapi PRD/User Stories untuk area yang sebelumnya ambigu. Jika ada konflik baru, keputusan produk berikutnya harus dicatat sebelum implementasi dilanjutkan.
