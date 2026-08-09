# Tracking Role — SIMPEG Fase 1

Folder ini menampung tracking kesesuaian implementasi untuk **kelima role** SIMPEG terhadap PRD
v1.4, User Stories, dan Panduan Kode. Satu file per role, masing-masing berisi: apa yang **sudah
sesuai** (✅) dan apa yang **belum sesuai** (⚠️/❌) beserta bukti `file:line` dan rujukan task
tracker.

Folder ini menggantikan lima dokumen analisis role terpisah (audit 21–23 Juli 2026). Seluruh
temuan lama direkonsiliasi terakhir terhadap branch `development` @ `1fd99cb` pada 10 Agustus
2026. File role masih mempertahankan tabel audit Juli sebagai kronologi; bagian “Rekonsiliasi
Temuan” pada tiap file menjadi status aktif bila ada pernyataan lama yang bertentangan.

## Ringkasan Lintas Role

| Role | File | Status | Sorotan |
|---|---|:---:|---|
| Super Admin | [Role-Super-Admin.md](Role-Super-Admin.md) | ⚠️ | Data Master, Hari Libur, dashboard, laporan, masking, immutability, dan audit server-side sudah ditutup. Sisa utama: Pengaturan Sistem, beberapa gap RBAC/integrasi, dan hardening fail-closed menyeluruh |
| Admin Kepegawaian | [Role-Admin-Kepegawaian.md](Role-Admin-Kepegawaian.md) | ⚠️ | Dashboard/laporan/audit utama sudah ditutup; sisa: navigasi lifecycle pegawai, konsistensi tombol vs permission, batas kelola dokumen, dan hardening fail-closed |
| Pimpinan | [Role-Pimpinan.md](Role-Pimpinan.md) | ⚠️→✅ | Read-only UI, laporan, dashboard, dan tren W7 sudah tersedia; sisa: keputusan produk link audit W6 dan verifikasi akhir lintas-role |
| Kepala Bagian | [Role-Kepala-Bagian.md](Role-Kepala-Bagian.md) | ✅ | Paling sesuai; sisa: bersih-bersih kode Dinas Luar (keputusan K-2) dan paginasi EWS |
| Pegawai | [Role-Pegawai.md](Role-Pegawai.md) | ⚠️ | Widget notifikasi dan validasi lintas tahun sudah ditutup; sisa utama dua halaman fungsional yang belum memiliki tautan navigasi |

## Temuan Lintas Role (berlaku untuk lebih dari satu role)

| Temuan | Role terdampak | Rujukan |
|---|---|---|
| Hardening audit fail-closed belum menyeluruh pada CRUD pegawai/import; masking, immutability, dan audit server-side sudah ditutup PR #174 | Super Admin, Admin | Sprint 7 (7.2-9) |
| L3 belum tersedia untuk Admin dan beberapa gap lifecycle/navigasi masih terbuka; export custom/PDF utama sudah ditutup PR #154/#162/#167 | Super Admin, Admin | Backlog role Admin |
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
