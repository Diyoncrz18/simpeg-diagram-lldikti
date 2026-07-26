# Tracking Role — SIMPEG Fase 1

Folder ini menampung tracking kesesuaian implementasi untuk **kelima role** SIMPEG terhadap PRD
v1.3, User Stories, dan Panduan Kode. Satu file per role, masing-masing berisi: apa yang **sudah
sesuai** (✅) dan apa yang **belum sesuai** (⚠️/❌) beserta bukti `file:line` dan rujukan task
tracker.

Folder ini menggantikan lima dokumen analisis role terpisah (audit 21–23 Juli 2026). Seluruh
temuan lama telah **diverifikasi ulang terhadap kode aktual** branch `development` @ `e82b527`
pada 27 Juli 2026 — sesudah PR #121–#131 — sehingga status di sini mencerminkan kondisi sekarang,
bukan kondisi saat audit awal.

## Ringkasan Lintas Role

| Role | File | Status | Sorotan |
|---|---|:---:|---|
| Super Admin | [Role-Super-Admin.md](Role-Super-Admin.md) | ⚠️ | Operasional kuat; Kelola Akses User tuntas (PR #126). Sisa terbesar: Data Master, Hari Libur web, Pengaturan Sistem, RBAC, dashboard, pengerasan audit |
| Admin Kepegawaian | [Role-Admin-Kepegawaian.md](Role-Admin-Kepegawaian.md) | ⚠️ | Penetapan atasan tuntas; sisa: navigasi pegawai nonaktif, konsistensi tombol vs permission, laporan L1/L3, audit NIK |
| Pimpinan | [Role-Pimpinan.md](Role-Pimpinan.md) | ⚠️→✅ | Hampir tuntas (read-only UI, laporan, dashboard via PR #125); sisa: tren W7, label `Perubahan`, keputusan produk link audit W6 |
| Kepala Bagian | [Role-Kepala-Bagian.md](Role-Kepala-Bagian.md) | ✅ | Paling sesuai; sisa: bersih-bersih kode Dinas Luar (keputusan K-2) dan paginasi EWS |
| Pegawai | [Role-Pegawai.md](Role-Pegawai.md) | ⚠️ | Semua halaman target ada dan self-scoped; sisa: bug widget notifikasi, 2 halaman tanpa tautan navigasi, validasi lintas tahun |

## Temuan Lintas Role (berlaku untuk lebih dari satu role)

| Temuan | Role terdampak | Rujukan |
|---|---|---|
| Audit update pegawai bocor NIK/No. KK plaintext; halaman Audit Log tanpa masking + tanpa paginasi server-side; model `AuditLog` tanpa guard immutable | Super Admin, Admin | Task #2, #3; Sprint 7 (7.1-1, 7.2-1, 7.2-2) |
| Preview laporan pegawai (closure lama, PII ke browser), modal custom mati, PDF L1 belum ada, L3 belum tersedia untuk Admin | Super Admin, Admin | Task #18, #19, #20 |
| Dashboard admin masih dummy kecuali panel EWS | Super Admin, Admin | Task #16 (kontrak K-3) |
| Label `Perlu Perubahan` (istilah resmi: `Perubahan`) di view cuti bersama | Pimpinan, Admin, Pegawai | Task #25 |
| Form ubah password lokal tidak sinkron dengan SSO Keycloak | Semua role | Keputusan produk |

## Cara Memakai

- Status diperbarui setiap kali PR yang menyentuh role terkait di-merge ke `development` — catat
  nomor PR pada baris temuan yang tertutup, pindahkan barisnya ke tabel ✅.
- Temuan baru dari QA/review dimasukkan ke tabel "Belum Sesuai" file role terkait, dengan bukti
  `file:line` dan prioritas.
- Rujukan silang: tracking pekerjaan per sprint ada di folder `../Tracking-Sprint-1-6/`
  (Sprint 1–7); daftar issue resmi di `../Issues-SIMPEG-Fase1.md`; keputusan produk terbaru di
  `../Kickoff-Sprint-6-Kontrak-dan-Keputusan.md`.

## Dokumen Asal yang Dikonsolidasi

| Dokumen lama (dihapus dari folder DOCUMENT) | Menjadi |
|---|---|
| `Analisis-Frontend-Backend-Role-Super-Admin.md` (21 Juli) | Role-Super-Admin.md |
| `Analisis-Kesesuaian-Administrasi-Sistem-Super-Admin.md` (23 Juli) | Role-Super-Admin.md |
| `Analisis-Frontend-Backend-Role-Admin-Kepegawaian.md` (21 Juli) | Role-Admin-Kepegawaian.md |
| `Analisis-Frontend-Role-Pimpinan.md` (21 Juli) | Role-Pimpinan.md |
| `Analisis-Frontend-Role-Kepala-Bagian.md` (21 Juli) | Role-Kepala-Bagian.md |
| `Halaman-dan-Hak-Akses-Role-Pegawai.md` (22 Juli, dokumen target produk) | Role-Pegawai.md |

`Bukti-QA-Kelola-Akses-User-Super-Admin.md` dan `Rencana-Eksekusi-Kelola-Akses-User-Super-Admin.md`
tetap di folder `DOCUMENT` sebagai arsip bukti QA dan rencana eksekusi (bukan dokumen analisis role).
