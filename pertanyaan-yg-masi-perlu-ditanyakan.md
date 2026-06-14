# Pertanyaan dan Status Konfirmasi ke Pihak LLDIKTI XVI

> Disusun berdasarkan cross-reference antara:
> - PRD-SIMPEG-Fase1-Core.md v1.1
> - Rekap PDF SIMPEG (rekap_SIMPEG_LLDikti_Wilayah_XVI.md)
> - Transkrip paparan (traskrip.md)
> - Diagram alur v0.4 (SIMPEG-v0.4-diagrams.md)
>
> **Prioritas:** 🔴 Blocker (harus dijawab sebelum development) · 🟡 Penting (perlu dijawab sebelum modul terkait dikerjakan) · 🟢 Nice-to-know (bisa diputuskan nanti)
>
> **Catatan sinkronisasi PRD 1.1:** beberapa pertanyaan awal sudah terjawab melalui meeting teknis dan sudah dimasukkan ke PRD. Pertanyaan yang sudah terjawab tidak perlu ditanyakan ulang, kecuali untuk meminta data final seperti credential, file referensi, atau nilai konfigurasi production.

---

## A. Keycloak SSO & Autentikasi

### A1. 🔴 Status Keycloak Saat Ini
**Status PRD 1.1:** Sebagian terjawab. Keycloak digunakan sebagai SSO, dan LLDIKTI akan membagikan trait/fungsi integrasi.

Yang masih perlu diminta sebagai data final:
- URL Keycloak.
- Realm yang digunakan.
- Client ID dan Client Secret.
- Akun SSO testing.

### A2. 🔴 Data User di Keycloak
**Status PRD 1.1:** Sebagian terjawab. Mapping dilakukan melalui email atau Keycloak User ID dan disimpan di SIMPEG.

- Apakah **semua 46 pegawai sudah terdaftar** di Keycloak?
- Field apa saja yang tersimpan di Keycloak per user? (email, nama, NIP?)
- Field mana yang paling stabil untuk mapping awal: email atau Keycloak User ID?

### A3. 🟡 Aplikasi Lain yang Pakai Keycloak
- Sistem internal apa saja yang sudah menggunakan Keycloak SSO ini? (untuk memahami konfigurasi yang sudah ada)
- Apakah SIMPEG perlu membuat **client baru di Keycloak** atau menumpang ke client yang sudah ada?

### A4. 🟡 Role Management
**Status PRD 1.1: Terjawab.**

Role dan permission dikelola di database SIMPEG, bukan di Keycloak. Keycloak hanya menjadi gerbang autentikasi / SSO. Super Admin mengatur role internal melalui aplikasi SIMPEG.

---

## B. Data Pegawai & Struktur Organisasi

### B1. 🔴 Struktur Organisasi LLDIKTI XVI
Berdasarkan penelusuran website LLDIKTI XVI, kami menemukan struktur organisasi berupa Kepala Lembaga (Bapak Munawir Sadzali Razak), Kabag Umum (Bapak Irwan Halid), 4 Tim Kerja, dan 7 Urusan.
- Apakah 11 unit kerja ini (4 Tim Kerja + 7 Urusan) adalah daftar unit kerja resmi yang akan dimasukkan ke SIMPEG?
- Siapa saja pejabat Ketua Tim Kerja dan Kepala Urusan saat ini?
- Siapa saja yang berperan sebagai **Atasan Langsung**? Apakah Ketua Tim Kerja dan Kepala Urusan tersebut, atau Kabag Umum untuk semua pegawai?

### B2. 🔴 Data Pegawai Existing
**Status PRD 1.1:** Sebagian terjawab. Sample Excel awal sudah tersedia dan digunakan sebagai acuan import awal.

Yang masih perlu diminta:
- Sample/format final CSV atau Excel yang akan dipakai saat import production.
- **Kolom/field lengkap** yang akan dipakai, karena sample awal belum mencakup semua field PRD.
- Apakah data sudah **bersih dan konsisten**, atau masih perlu pembersihan?
- Berapa banyak **riwayat kepangkatan/jabatan** rata-rata per pegawai?

### B3. 🟡 NIP dan Identitas Pegawai
- Apakah semua pegawai sudah punya **NIP**? (termasuk PPPK?)
- Format NIP yang digunakan? (18 digit standar BKN?)
- Untuk PPPK, apakah ada **nomor identitas khusus** selain NIP?

### B4. 🟡 Data Keluarga
Rekap (Bagian 6.4) menyebutkan data keluarga sebagai salah satu jenis data yang dikelola.
- Data keluarga ini untuk keperluan apa? **Tunjangan keluarga**, **ahli waris**, atau keduanya?
- Field apa saja yang perlu dicatat? (Nama pasangan, NIK pasangan, jumlah anak, nama anak, tanggal lahir anak?)
- Apakah ada **batas jumlah anak** yang dicatat untuk tunjangan?

### B5. 🟡 Golongan dan Jabatan
- Apakah bisa diberikan **daftar lengkap jabatan** yang ada di LLDIKTI XVI?
- Masing-masing jabatan masuk **jenis jabatan** apa? (Struktural, Fungsional Tertentu, Fungsional Umum/Pelaksana)
- Apakah ada jabatan **Fungsional Tertentu** di LLDIKTI XVI? Jika ya, apa saja?

### B6. 🟡 Pegawai PPPK
- Apakah saat ini **sudah ada pegawai PPPK** di LLDIKTI XVI?
- Jika ada, berapa jumlahnya?
- Data kontrak PPPK apa saja yang perlu disimpan? (Tanggal mulai, tanggal berakhir, nomor kontrak, perpanjangan ke berapa?)

---

## C. Manajemen Cuti

### C1. 🔴 Alur Approval — Peran Spesifik & Jumlah Tahap
**Status PRD 1.1: Terjawab untuk desain Fase 1.**

Alur awal dibuat seragam: Atasan Langsung → Kabag/verifikator → Pimpinan/PYBMC. Sistem tetap didesain agar approval bisa dikonfigurasi dinamis per pegawai/unit dan melakukan skip jika approver sama.

Yang masih perlu diminta sebagai data final:
- Daftar atasan langsung per pegawai/unit.
- Nama pejabat Kabag/verifikator dan Pimpinan/PYBMC yang aktif saat go-live.
- Bagaimana jika **approver sedang cuti/dinas luar**? Apakah ada mekanisme **delegasi approval**?

### C2. 🟡 Aturan Cuti Spesifik
Rekap (Bagian 7.1) menyebutkan aturan per jenis cuti, tapi beberapa detail belum lengkap.
- **Cuti Sakit**: Mulai dari berapa hari perlu surat dokter? (1 hari, 3 hari, atau lainnya?)
- **Cuti Sakit**: Apakah ada **batas maksimum** hari cuti sakit? Jika lebih dari batas, apa yang terjadi?
- **Cuti Melahirkan**: Rekap menyebutkan 3 bulan untuk anak ke-1 s.d. ke-3. Apakah anak ke-4 dst tidak dapat cuti melahirkan?
- **Cuti Alasan Penting**: Apa saja **alasan yang termasuk "penting"**? Apakah perlu dokumen pendukung?
- **Cuti Besar**: Rekap menyebutkan minimal 5 tahun masa kerja. Berapa lama durasi cuti besar? 3 bulan?
- **CLTN**: Rekap menyebutkan maks 3 tahun. Apakah selama CLTN pegawai tetap tercatat sebagai pegawai aktif?

### C3. 🟡 Carry-Over Cuti Tahunan
Diagram v0.4 menyebutkan "Carry-over Dihitung".
- Berapa **maksimum hari** cuti yang bisa di-carry-over ke tahun berikutnya?
- Apakah cuti carry-over **hangus** di bulan tertentu? (misal: hangus setelah 30 Juni?)
- Atau carry-over **ditambahkan** ke saldo tahun berikutnya tanpa batas waktu?

### C4. 🟡 Cuti Bersama
- Apakah **cuti bersama otomatis mengurangi** saldo cuti tahunan pegawai?
- Atau cuti bersama dihitung terpisah dari saldo cuti tahunan?

### C5. 🟢 Pembatalan & Perubahan Cuti
- Apakah pegawai bisa **membatalkan** pengajuan cuti yang sudah diajukan tapi belum disetujui?
- Apakah pegawai bisa **mengubah tanggal** cuti yang sudah disetujui?
- Jika cuti yang sudah disetujui dibatalkan, apakah **saldo dikembalikan otomatis**?

---

## D. Early Warning System (EWS)

### D1. 🔴 Syarat Kenaikan Pangkat — "Kinerja Baik"
**Status PRD 1.1:** Sebagian terjawab. Fase 1 memakai field manual `is_kinerja_baik` yang diisi Admin Kepegawaian.

Yang masih perlu dikonfirmasi:
- Atau ada **dokumen SKP di luar sistem** yang bisa dijadikan referensi oleh admin?
- Jika pegawai punya hukuman disiplin aktif, apakah EWS tetap mengirim notifikasi (dengan catatan "tidak eligible") atau **tidak mengirim sama sekali**?

### D2. 🟡 BUP (Batas Usia Pensiun) per Jabatan
**Status PRD 1.1:** Terjawab untuk desain sistem, masih perlu data final.

Desain sistem tidak meng-hardcode BUP. Usia pensiun disimpan di `ref_jenis_jabatan.maks_usia_pensiun` atau tabel detail `ref_bup`.

Yang masih perlu diminta:
- Daftar jabatan final beserta maksimal usia pensiun yang berlaku di LLDIKTI XVI.
- Konfirmasi apakah seluruh nilai mengikuti regulasi umum atau ada aturan internal tertentu.

### D3. 🟡 Kontrak PPPK
- Dari data kontrak PPPK, field mana yang digunakan EWS? **Tanggal berakhir kontrak**?
- Apakah ada **mekanisme perpanjangan kontrak** yang perlu didukung sistem? (misal: admin update tanggal kontrak baru setelah diperpanjang)

### D4. 🟢 EWS — Eskalasi
- Jika notifikasi EWS sudah dikirim di H-90 tapi **tidak ada tindak lanjut**, apakah ada mekanisme eskalasi? (misal: di H-60 notifikasi juga dikirim ke Pimpinan?)
- Atau cukup mengirim ulang notifikasi di interval berikutnya (H-60, H-30) tanpa eskalasi?

---

## E. Audit Log

### E1. 🟡 Retensi Audit Log
Rekap (Bagian 6.3) menyebutkan "audit log tidak dapat dihapus oleh siapa pun".
- Apakah ini benar **tidak bisa dihapus sama sekali**, termasuk oleh Super Admin?
- Atau Super Admin boleh menghapus audit log tertentu?
- Apakah ada **kebijakan retensi** tertentu? (misal: simpan 5 tahun, lalu arsipkan)

### E2. 🟢 Akses Audit Log
- Siapa saja yang boleh **melihat** audit log?
  - Hanya Super Admin dan Admin Kepegawaian?
  - Atau Pimpinan juga perlu akses lihat audit log?
- Apakah perlu fitur **filter/search** di audit log? (misal: filter by pegawai, by tanggal, by jenis perubahan)

---

## F. Dashboard & Laporan

### F1. 🟡 Akses Dashboard per Role
Rekap (Bagian 11) menyebutkan dashboard untuk admin dan pimpinan, tapi tidak detail per role.
- Apakah **semua role** bisa mengakses dashboard, atau hanya role tertentu?
- Apakah **data yang ditampilkan** di dashboard berbeda per role?
  - Contoh: Pimpinan melihat semua pegawai, Atasan Langsung hanya melihat bawahannya?
- Apakah Pegawai juga punya **dashboard pribadi**? (misal: saldo cuti, status pengajuan, EWS pribadi)

### F2. 🟡 Laporan yang Diekspor
Rekap (Bagian 11.5) menyebutkan beberapa laporan: daftar nominatif, rekap cuti, riwayat kepangkatan, pemenuhan 20 JP, data SIASN.
- Untuk Fase 1, laporan mana saja yang **benar-benar dibutuhkan**?
  - **Daftar nominatif pegawai** — apakah ada format standar yang diikuti? Bisa minta contoh?
  - **Rekap cuti** — per pegawai atau per unit kerja? Periode bulanan atau tahunan?
  - **Riwayat kepangkatan** — per individu atau keseluruhan?
- Apakah ada **template laporan resmi** dari Kemendiktisaintek atau BKN yang harus diikuti?

### F3. 🟢 Kalender Tim di Dashboard
Rekap (Bagian 11.4) menyebutkan "Kalender Tim" yang menampilkan siapa yang tidak masuk. Namun Kalender Virtual ada di Fase 2.
- Apakah **kalender tim sederhana** (hanya menampilkan pegawai yang cuti hari ini) sudah cukup untuk Fase 1?
- Atau fitur kalender seluruhnya ditunda ke Fase 2?

---

## G. Infrastruktur & Teknis

### G1. 🔴 Server & Hosting
**Status PRD 1.1: Terjawab untuk waktu penyediaan.**

Server production akan disiapkan oleh LLDIKTI saat sistem mendekati deployment. Production diprioritaskan menggunakan Podman.

Yang masih perlu diminta mendekati deployment:
- SIMPEG akan di-deploy di mana? **Server on-premise**, **VPS**, atau **cloud**?
- Spesifikasi server? (RAM, CPU, storage)
- Apakah server yang sama dengan Keycloak, atau **server terpisah**?

### G2. 🔴 Domain & SSL
**Status PRD 1.1: Terjawab untuk waktu penyediaan.**

Domain dan SSL production akan disiapkan oleh LLDIKTI saat tahap deployment. PRD tidak mengunci domain final.

Yang masih perlu diminta mendekati deployment:
- Domain final yang akan digunakan.
- Apakah sudah ada **sertifikat SSL** untuk domain tersebut?
- Siapa yang mengelola DNS? (untuk pointing domain ke server SIMPEG)

### G3. 🟡 Email Server / SMTP
**Status PRD 1.1: Terjawab untuk arah implementasi.**

Development dapat memakai Mailpit. Production memakai email operasional LLDIKTI.

Yang masih perlu diminta mendekati production:
- User/password email operasional.
- Host, port, encryption, dan alamat pengirim email.

### G4. 🟡 Backup & Recovery
- Apakah sudah ada **strategi backup** yang berjalan untuk sistem-sistem di LLDIKTI?
- Seberapa sering backup harus dilakukan? (harian, mingguan?)
- Apakah perlu backup ke **lokasi berbeda** (off-site backup)?

### G5. 🟢 Environment Development
- Apakah tersedia **server staging/testing** terpisah dari production?
- Atau development dan testing dilakukan di **lokal** oleh tim magang?

---

## H. Tim & Proses Development

### H1. 🟡 Komposisi Tim Magang
- Berapa jumlah **mahasiswa magang** yang akan mengerjakan SIMPEG?
- Apa **latar belakang teknis** mereka? (sudah familiar Laravel? PostgreSQL? Git?)
- Apakah ada **supervisor teknis** yang mendampingi? Siapa?
- Apakah tim magang juga mengerjakan **sistem lain** (misal: Sistem Penilaian Kinerja) secara paralel?

### H2. 🟡 Timeline & Milestone
Rekap (Bagian 13.1) menyebutkan Fase 1 harus selesai sebelum 1 September 2026.
- Apakah deadline **1 September 2026** masih berlaku?
- Apakah ada **milestone interim** yang diharapkan? (misal: demo di bulan ke-3?)
- Kapan **User Acceptance Testing (UAT)** direncanakan?
- Siapa yang akan melakukan **UAT** dari pihak LLDIKTI?

### H3. 🟢 Akses & Koordinasi
- Siapa **PIC (Person in Charge)** dari pihak LLDIKTI untuk koordinasi teknis?
- Bagaimana **mekanisme komunikasi** selama development? (WhatsApp, email, meeting rutin?)
- Apakah ada **jadwal review/demo** berkala yang diharapkan?

---

## I. Data Migration & Go-Live

### I1. 🔴 Data Awal yang Akan Diimpor
**Status PRD 1.1:** Sebagian terjawab. Sample Excel awal sudah tersedia.

Yang masih perlu diminta:
- Data final yang akan diimpor saat production.
- Selain data pegawai, apakah ada **data riwayat** (kepangkatan, jabatan, KGB) yang juga perlu diimpor?
- Apakah data riwayat sudah **lengkap dan terstruktur**, atau perlu dikumpulkan dari berbagai sumber?

### I2. 🟡 Validasi Data
- Siapa yang akan **memvalidasi** data setelah diimpor ke SIMPEG? Admin Kepegawaian?
- Apakah perlu fitur **preview sebelum import** (admin bisa review data yang akan dimasukkan sebelum confirm)?

### I3. 🟡 Skenario Go-Live
- Apakah akan ada **periode paralel** (sistem manual dan SIMPEG berjalan bersamaan)?
- Atau langsung **cut-over** ke SIMPEG setelah data diimpor?
- Apakah perlu **pelatihan/training** untuk pegawai sebelum go-live?

---

## J. Regulasi & Compliance

### J1. 🟡 Regulasi Terbaru
- Apakah ada **peraturan terbaru** (setelah 2021) yang mempengaruhi aturan cuti, kenaikan pangkat, atau KGB yang belum tercakup di dokumen?
- Apakah ada **aturan internal LLDIKTI XVI** (diluar PP/PermenPANRB) yang perlu diakomodasi?

### J2. 🟢 Kepatuhan Data Pribadi
- Apakah ada kebijakan **perlindungan data pribadi** yang harus diikuti? (UU PDP No. 27/2022)
- Apakah perlu **consent form** untuk pegawai terkait penyimpanan data pribadi mereka di sistem?

---

## K. Fitur Edge Cases

### K1. 🟡 Pegawai Pindah/Mutasi
- Apakah ada kemungkinan pegawai **mutasi masuk** ke LLDIKTI XVI dari instansi lain?
- Jika ya, bagaimana data riwayat mereka diinput? (Import dari instansi sebelumnya atau input manual?)
- Apakah ada pegawai yang **mutasi keluar**? Bagaimana statusnya di sistem? (non-aktifkan akun? soft delete?)

### K2. 🟡 Pegawai Non-Aktif
- Selain pensiun, status pegawai apa saja yang membuat seseorang **tidak aktif**?
  - Contoh: Tugas Belajar, Cuti di Luar Tanggungan Negara, Diberhentikan Sementara?
- Apakah pegawai non-aktif masih bisa **login dan lihat data** di SIMPEG?

### K3. 🟢 Cuti Mendadak / Darurat
- Apakah ada skenario cuti yang bisa **langsung disetujui** tanpa mengikuti approval chain normal? (misal: keluarga meninggal)
- Atau tetap mengikuti approval chain, tetapi diberi status **prioritas/fast-track**?

---

## Ringkasan Prioritas

| Prioritas | Jumlah | Kategori |
|-----------|--------|----------|
| 🔴 Blocker | 4 | Credential/akses Keycloak final, struktur atasan langsung, format data final import, keputusan kinerja/EWS yang belum eksplisit |
| 🟡 Penting | 14 | NIP, Data Keluarga, Golongan/Jabatan, PPPK, Aturan Cuti, Carry-over, daftar BUP final, Kontrak PPPK, Audit Log, Dashboard, Laporan, Backup, Timeline/UAT, Go-Live |
| 🟢 Nice-to-know | 8 | Pembatalan Cuti, Eskalasi EWS, Akses Audit Log, Kalender Tim, Environment Dev, Akses Koordinasi, Data Pribadi/PDP, Cuti Darurat |

> **Rekomendasi setelah PRD 1.1**: Development dapat berjalan sambil menunggu data final non-production. Yang paling perlu dikejar segera adalah credential Keycloak, daftar atasan langsung, file data final untuk import, dan referensi jabatan/BUP final.
