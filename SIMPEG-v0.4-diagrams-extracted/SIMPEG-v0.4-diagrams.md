# Ekstraksi Diagram SIMPEG v0.4

Sumber: https://stalwart-tiramisu-848810.netlify.app/

> Catatan sinkronisasi PRD v1.1: dokumen ini adalah referensi legacy dari diagram v0.4. Untuk implementasi, ikuti `PRD-SIMPEG-Fase1-Core.md` v1.1: Keycloak hanya SSO, RBAC internal SIMPEG, domain/server disiapkan saat deployment, approval cuti mendukung skip approver duplikat, dan BUP berbasis referensi jabatan.

Konteks: Sistem Kepegawaian LLDIKTI XVI; ~46 pegawai internal; Laravel 12 + Blade + PostgreSQL 17 + Keycloak SSO; domain final disiapkan saat deployment; v0.4 — Mei 2026.

## 1 · Alur Input Data Pegawai

3 jalur akses: admin-only (append-only) · self-service langsung · pending changes (butuh persetujuan admin)

```mermaid
%%{init:{"flowchart":{"nodeSpacing":85,"rankSpacing":80,"padding":28},"theme":"dark"}}%%
flowchart LR
    subgraph ADMIN["Admin Kepegawaian Only"]
        A1["Data Pribadi\n(Nama, TTL, Agama, Foto)"]
        A2["Riwayat Kepangkatan\n(Append-Only)"]
        A3["Riwayat Jabatan\n(Append-Only)"]
        A4["Riwayat KGB\n(Append-Only)"]
        A5["Hukuman Disiplin\n(PP 94/2021)"]
        A6["Set Supervisor\n(per Konteks Cuti / Kinerja)"]
    end
    subgraph SELF["Self-Service Pegawai"]
        S1["Edit Kontak dan Data Pendukung\n(Langsung tersimpan)"]
        S2["Ajukan Pending Changes\n(NIK, KK, Keluarga, Pendidikan)"]
    end
    S2 --> APV{Admin\nReview}
    APV -->|Approve| OK["Data Diperbarui\n+ Notifikasi Pegawai"]
    APV -->|Tolak| NOK["Notifikasi Penolakan\n+ Alasan"]
```

## 2 · Workflow Pengajuan Cuti

Catatan PRD v1.1: alur cuti memakai approval chain yang dapat dikonfigurasi, dengan dukungan skip jika approver sama. Diagram ini tetap referensi legacy v0.4.

```mermaid
%%{init:{"flowchart":{"nodeSpacing":52,"rankSpacing":62,"padding":18},"theme":"dark"}}%%
flowchart TD
    PG([Pegawai]) --> A[Pilih Jenis Cuti]
    A --> B{Status\nKepegawaian}
    B -->|"PNS: Semua Jenis"| C
    B -->|"PPPK: Cuti Besar\ndan CLTN Disembunyikan"| C
    C{Cek Saldo\nCuti} -->|"Tidak Cukup"| D(["Ditolak\nOtomatis"])
    C -->|"Cukup"| E["Stage 1: Atasan Langsung\nMengetahui"]
    E --> F["Stage 2: Kabag/Verifikator\nMenyetujui"]
    F --> G["Stage 3: Pimpinan/PYBMC\nFinal"]
    G -->|"Disetujui"| H["Saldo Dikurangi Otomatis\nCarry-over Dihitung"]
    G -->|"Ditunda (bukan Ditolak)"| I["Notifikasi\nPenundaan"]
    H --> J[Kalender Virtual Update]
```

## 3 · Early Warning System (EWS)

PP 99/2000 · PP 11/2017 · PP 49/2018 — berjalan otomatis setiap hari. Urgen: Akub Zainal pensiun 1 Sep 2026

```mermaid
%%{init:{"flowchart":{"nodeSpacing":95,"rankSpacing":85,"padding":30},"theme":"dark"}}%%
flowchart LR
    SCH(["Scheduler\nHarian"]) --> DB[("Baca Data\nPegawai")]
    DB --> A{Evaluasi\nSemua Trigger}
    A --> B["Kenaikan Pangkat Reguler\n4 thn sejak TMT + SKP Baik\nH-90 · H-60 · H-30"]:::kp
    A --> C["Kenaikan Gaji Berkala\n2 thn sejak TMT KGB terakhir\nH-60 · H-30 · H-14"]:::kgb
    A --> D["Batas Usia Pensiun\nBUP beda per jenis jabatan\nH-1thn · H-6bln · H-3bln"]:::bup
    A --> E["Kontrak PPPK Berakhir\nKhusus pegawai PPPK\nH-6bln · H-3bln · H-1bln"]:::pppk
    B --> NT["Buat\nNotifikasi"]
    C --> NT
    D --> NT
    E --> NT
    NT --> N1["In-App\nReal-time"]
    NT --> N2["Email\nvia Queue"]
    N1 --> RCP(["Pegawai\nBersangkutan"])
    N1 --> ADM(["Admin\nKepegawaian"])
    N2 --> RCP
    N2 --> ADM
    classDef kp fill:#1a2d50,stroke:#3b82f6,color:#bfdbfe
    classDef kgb fill:#1a3020,stroke:#10b981,color:#a7f3d0
    classDef bup fill:#3a1a1a,stroke:#ef4444,color:#fca5a5
    classDef pppk fill:#2d1f3a,stroke:#8b5cf6,color:#c4b5fd
```

## 4 · Klaim Kehadiran (Advanced)

Pengajuan retroaktif absensi — batas 3 hari setelah kejadian, kuota gabungan 2x/bulan

```mermaid
%%{init:{"flowchart":{"nodeSpacing":90,"rankSpacing":80,"padding":28},"theme":"dark"}}%%
flowchart LR
    PG([Pegawai]) --> CK1{Cek\nHari}
    CK1 -->|"Hari Libur Nasional\nWeekend, Cuti, Dinas Luar"| BLOK(["Tidak Bisa\nDiajukan"])
    CK1 -->|"Hari Kerja"| CK2{Cek Batas\nWaktu}
    CK2 -->|"Lebih dari 3 hari lalu"| BLOK
    CK2 -->|"Dalam 3 hari"| CK3{Cek Kuota\nBulanan}
    CK3 -->|"Lupa Absen Gabungan\nsudah 2x bulan ini"| BLOK
    CK3 -->|"Izin Terlambat atau Cepat\nsudah 2x bulan ini"| BLOK
    CK3 -->|"Masih ada kuota"| FORM["Isi Form Klaim\nJenis, Tanggal, Alasan"]
    FORM --> APV{Atasan\nLangsung}
    APV -->|"Approve"| OK(["Kehadiran\nTercatat"])
    APV -->|"Tolak"| NOK(["Notifikasi\nPenolakan"])
```

## 5 · Surat Tugas & Kalender Virtual

Kalender terintegrasi dari semua sumber — otomatis blokir log harian pada hari dinas luar

```mermaid
%%{init:{"flowchart":{"nodeSpacing":85,"rankSpacing":80,"padding":28},"theme":"dark"}}%%
flowchart LR
    subgraph SRC["Sumber Data Kalender"]
        LN["Hari Libur Nasional\n(Input manual per tahun)"]
        CB["Cuti Bersama\n(Input manual, terpisah dari libur)"]
        CT["Cuti Disetujui\n(dari Modul Cuti)"]
        ST_IN["Surat Tugas\n(Input admin atau pegawai)"]
        KM["Klaim Kehadiran\n(Retroaktif disetujui)"]
        LH["Status Log Harian\n(Terisi atau belum)"]
    end
    ST_IN --> APV{Approval\nSurat Tugas}
    APV -->|"Disetujui"| DL["Tandai Dinas Luar\npada Tanggal Tercakup"]
    LN --> KV["Kalender Virtual\nper Pegawai"]
    CB --> KV
    CT --> KV
    DL --> KV
    KM --> KV
    LH --> KV
    KV --> BLK["Blokir Log Harian\npada Hari Dinas Luar"]
    KV --> VIEW(["Tampilan Kalender\nper Pegawai atau Tim"])
```

## 6 · Alur SKP & Penilaian Kinerja

PermenPANRB 6/2022 · PP 30/2019 — berbasis hasil kerja (RHK), bukan daftar aktivitas

```mermaid
%%{init:{"flowchart":{"nodeSpacing":52,"rankSpacing":60,"padding":18},"theme":"dark"}}%%
flowchart TD
    A(["Admin / Atasan"]) --> B["Buat SKP Periode Baru\nper Pegawai"]
    B --> C["Input RHK\n(Rencana Hasil Kerja)"]
    C --> D["Set Target, Indikator\ndan Satuan per RHK"]
    D --> E[Sepanjang Periode]
    E --> F["Pegawai Isi Log Harian"]
    F --> G["Kaitkan Log ke RHK\n(Many-to-Many, opsional)"]
    G --> H[Akhir Periode]
    H --> I["Atasan Buka Form Evaluasi"]
    I --> J["Nilai Hasil Kerja\nSangat Baik / Baik / Butuh Perbaikan / Kurang"]
    I --> K["Nilai Perilaku BerAKHLAK\n(Kriteria sama)"]
    J --> L[Nilai SKP Final]
    K --> L
    L --> M[Input ke Kalkulator IP-ASN]
```

## 7 · Kalkulator IP-ASN (Advanced)

PermenPANRB 38/2018 · PerBKN 8/2019 — dihitung otomatis dari data yang sudah ada di sistem

```mermaid
%%{init:{"flowchart":{"nodeSpacing":100,"rankSpacing":90,"padding":30},"theme":"dark"}}%%
flowchart LR
    subgraph DIM["4 Dimensi Penilaian"]
        A["Data Pendidikan\neducation_histories\n \nDi bawah SLTA = 1\nSLTA, D1, D2 = 5\nD3 = 10\nS1, D4 = 15\nS2, Profesi = 20\nS3 = 25"]:::d1
        B["Riwayat Pelatihan\ntraining_records\n \nDiklat Struktural / Fungsional = 15\nPelatihan Teknis >= 20 JP = 15\nWorkshop / Seminar = 10"]:::d2
        C["Nilai SKP Terakhir\nperformance_reviews\n \nNilai dari hasil evaluasi\nperiode SKP terakhir"]:::d3
        D["Riwayat Disiplin\ndiscipline_records\n \nTidak ada hukuman = 5\nHukuman Ringan = 3\nHukuman Sedang = 2\nHukuman Berat = 1"]:::d4
    end
    A -->|"x 0.25\n(Kualifikasi)"| E(["IP-ASN\nScore"])
    B -->|"x 0.40\n(Kompetensi)"| E
    C -->|"x 0.30\n(Kinerja)"| E
    D -->|"x 0.05\n(Disiplin)"| E
    E --> F["Simpan ke\nipasn_snapshots"]
    F --> G["Hitung Ulang\nKapan Saja"]
    F --> H["Export PDF\ndan Excel"]
    H --> I(["Laporan ke\nKemendiktisaintek"])
    classDef d1 fill:#1a2d50,stroke:#3b82f6,color:#bfdbfe
    classDef d2 fill:#1a3020,stroke:#10b981,color:#a7f3d0
    classDef d3 fill:#2d1f10,stroke:#f59e0b,color:#fde68a
    classDef d4 fill:#2d1f3a,stroke:#8b5cf6,color:#c4b5fd
```

## 8 · Tracker 20 JP Pengembangan Kompetensi

PP 11/2017 Pasal 203–217 · SE Kepala BKN No. 3/2022 — wajib 20 JP per tahun, bukan hak pilihan

```mermaid
%%{init:{"flowchart":{"nodeSpacing":95,"rankSpacing":85,"padding":30},"theme":"dark"}}%%
flowchart LR
    subgraph JENIS["6 Jenis Pengembangan Kompetensi (Semua Masuk Hitungan 20 JP)"]
        J1["Diklat Struktural\nPKP, PKA, Latsar\nLainnya"]:::j1
        J2["Diklat Fungsional\nDiklat JF Arsiparis\nPranata Komputer, dll"]:::j2
        J3["Pelatihan Teknis\nBimtek, Workshop\nSeminar, Kursus"]:::j3
        J4["Pelatihan SIASN\nModul SIASN BKN"]:::j4
        J5["Coaching dan Mentoring\noleh Atasan Langsung"]:::j5
        J6["Belajar Mandiri\nE-learning, Buku"]:::j6
    end
    J1 --> T(["Total JP\nTahun Berjalan\nper Pegawai"])
    J2 --> T
    J3 --> T
    J4 --> T
    J5 --> T
    J6 --> T
    T --> CH{Cek Otomatis\ndi Bulan Oktober}
    CH -->|"Sudah >= 20 JP"| OK(["Kewajiban\nTerpenuhi"])
    CH -->|"Belum 20 JP"| EWS["EWS: Kirim Peringatan\nke Pegawai dan Admin"]
    EWS --> ACT["Pegawai Ikut\nPengembangan Tambahan"]
    ACT --> T
    classDef j1 fill:#1a2d50,stroke:#3b82f6,color:#bfdbfe
    classDef j2 fill:#1a3020,stroke:#10b981,color:#a7f3d0
    classDef j3 fill:#2d1f10,stroke:#f59e0b,color:#fde68a
    classDef j4 fill:#2d1f3a,stroke:#8b5cf6,color:#c4b5fd
    classDef j5 fill:#301a1a,stroke:#ef4444,color:#fca5a5
    classDef j6 fill:#1a2a2a,stroke:#06b6d4,color:#a5f3fc
```

## 9 · Hierarki Role SIMPEG

5 role — Keycloak SSO, email di-cache lokal dari Keycloak saat login pertama

```mermaid
%%{init:{"flowchart":{"nodeSpacing":60,"rankSpacing":70,"padding":22},"theme":"dark"}}%%
flowchart TD
    SA["Super Admin\n(Semua fitur + konfigurasi sistem)"] --> AK
    AK["Admin Kepegawaian\n(Kelola semua data pegawai\nProses pengajuan dan verifikasi)"] --> PM
    PM["Pimpinan\n(Approve cuti dan klaim\nDashboard semua pegawai)"] --> AL
    AL["Atasan Langsung\n(Approve cuti dan klaim\nbawahan langsung)"] --> PG
    PG["Pegawai\n(Lihat data sendiri\nAjukan cuti dan self-service)"]
```

## 10 · Roadmap Pengembangan SIMPEG

4 fase — Core harus stabil dan dipakai sebelum Advanced dikerjakan

```mermaid
%%{init:{"flowchart":{"nodeSpacing":90,"rankSpacing":80,"padding":25},"theme":"dark"}}%%
flowchart LR
    F1["Fase 1: Core\n \nSetup + Keycloak SSO\nSeed Reference Tables\nRBAC Roles dan Permissions\nData Pegawai Lengkap\nRiwayat Kepangkatan, Jabatan\nKGB, Disiplin, Pengangkatan\nImport Excel/CSV awal\nEWS (KP, KGB, Pensiun, PPPK)\nModul Cuti + Approval\nNotifikasi In-App dan Email\nAudit Log\nDashboard Dasar"]:::f1
    F2["Fase 2: Advanced 1\n \nSelf-Service Pegawai\nPending Changes\nKlaim Kehadiran + Kuota\nSurat Tugas\nKalender Virtual\nLog Harian"]:::f2
    F3["Fase 3: Advanced 2\n \nSKP + RHK\nLog Harian ke RHK\nEvaluasi Kinerja\nRiwayat Pendidikan\nRiwayat Pelatihan\nTracker 20 JP\nArsip Dokumen\nLaporan PDF dan Excel"]:::f3
    F4["Fase 4: Advanced 3\n \nKalkulator IP-ASN\nAsesmen Kompetensi\nEkspor Data SIASN\n(Format CSV atau JSON BKN)\nIntegrasi API SIASN\n(jika akses tersedia)"]:::f4
    F1 -->|"Core stabil\ndan dipakai"| F2
    F2 -->|"Advanced 1\nstabil"| F3
    F3 -->|"Advanced 2\nstabil"| F4
    classDef f1 fill:#1a2d50,stroke:#3b82f6,color:#bfdbfe,font-size:13px
    classDef f2 fill:#14301f,stroke:#10b981,color:#a7f3d0,font-size:13px
    classDef f3 fill:#2d1f10,stroke:#f59e0b,color:#fde68a,font-size:13px
    classDef f4 fill:#2d1f3a,stroke:#8b5cf6,color:#c4b5fd,font-size:13px
```
