# Bukti QA — Kelola Akses User Super Admin

> Tanggal retest: 23 Juli 2026.
>
> Branch: `fix/user-management-phase-1`.
>
> Status: seluruh skenario Fase 4 yang berada dalam scope Kelola Akses User lulus. Release gate repository belum dapat dinyatakan hijau karena `composer qa` berhenti pada masalah format yang telah ada pada file di luar scope perubahan ini.

## Keputusan yang diuji

- `users.employee_id` adalah target mapping canonical; email tidak menjadi identifier POST.
- `users.keycloak_id` menyimpan identifier Keycloak yang stabil dan wajib terisi. Disconnect belum tersedia, sehingga pengosongan identifier ditolak dan tidak menghasilkan audit mutasi.
- `users` adalah sumber kebenaran mapping dan role.
- audit mapping bersifat strict: kegagalan audit membatalkan transaction.

## Matriks hasil

| ID | Bukti | Hasil |
|---|---|---|
| UA-40 | `UserMappingControllerTest`: guest redirect; empat role non-Super Admin menerima 403 pada GET dan POST; Super Admin mendapat 200. | Lulus. |
| UA-41 | Kontrak HTML memeriksa lima `option value` kode internal dan label Indonesia; role `null` tetap `Belum diberi role`; lima role berhasil disimpan lewat kontrak form. | Lulus. |
| UA-42 | Role label lama, role invalid, UUID malformed/tidak ada, identifier kosong/duplikat, dan email tampered ditolak; modal/error field kembali secara aksesibel. | Lulus. |
| UA-43 | Feature test menolak dua user berbeda untuk satu employee dan mapping identifier yang sudah dipakai. PostgreSQL 17 mengembalikan SQLSTATE `23505` untuk `users_keycloak_id_unique` dan `users_employee_id_unique`; dua koneksi paralel juga menghasilkan `23505` pada koneksi kedua. | Lulus. |
| UA-44 | Audit create/update menyimpan actor, `employee_id`, role, status, IP, User-Agent, dan identifier masked. Tidak ada identifier mentah. Kegagalan audit diuji menyebabkan rollback. Pengosongan identifier ditolak sebelum mutation/audit. | Lulus. |
| UA-45 | Pagination, pencarian, role/status filter, query string, canonical `employee_id`, fallback legacy aman, dan payload PII minimal diuji server-side. | Lulus. |
| UA-46 | Retest browser desktop dan lebar 390 px membuka modal, fokus awal pada identifier, `Tab` memindahkan fokus ke role, dan tidak ada overflow horizontal. Retest ini menemukan atribut Alpine yang rusak akibat quote selector; markup kemudian diperbaiki dan retest lulus. | Lulus. |
| UA-47 | Test terarah, PHPStan penuh, Pint pada file scope, verifikasi route, dan compile Blade dijalankan. `composer qa` penuh dicoba dan tercatat pada bagian release gate. | Lulus untuk scope; release gate repository masih blocked. |

## Hasil command yang dapat direproduksi

| Command | Hasil |
|---|---|
| `php artisan test tests/Feature/UserMappingControllerTest.php --testdox` | 22 test, 161 assertion lulus. |
| `php artisan test tests/Feature/KeycloakCallbackMappingTest.php --testdox` | 19 test, 95 assertion lulus. |
| `php artisan test tests/Feature/CutiAuditCoverageTest.php tests/Feature/AuditPageIntegrationTest.php --testdox` | 3 test, 12 assertion lulus. |
| `php vendor/bin/phpstan analyse --memory-limit=1G --no-progress` | 489 file, tidak ada error. |
| `php vendor/bin/pint --test` pada file PHP yang diubah | Lulus. |
| `php artisan route:list --name=user-management --json` | GET/POST, nama route, controller, dan middleware role Super Admin tetap sesuai kontrak. |
| `php artisan view:cache` lalu `php artisan view:clear` | Blade dapat dikompilasi dan cache dibersihkan. |

## Verifikasi PostgreSQL dan data uji

Verifikasi dilakukan terhadap PostgreSQL 17 pada container development, bukan SQLite saja.

1. Transaction yang di-rollback membuktikan duplicate `keycloak_id` dan duplicate `employee_id` ditolak oleh constraint aktual dengan SQLSTATE `23505`.
2. Uji dua koneksi paralel membuat koneksi pertama commit mapping sementara dan koneksi kedua memakai identifier sama. Koneksi kedua menerima `23505` dari `users_keycloak_id_unique`.
3. Satu row sementara yang sempat di-commit untuk mensimulasikan race diverifikasi unik, kemudian dihapus; verifikasi akhir menunjukkan nol row data uji tersisa. Uji transaction lainnya seluruhnya di-rollback.

## Retest browser

Retest browser memakai ChromeDriver lokal dan container aplikasi pada `http://127.0.0.1:8000`. Agar akun demo lokal dapat menjalankan skenario Super Admin, role demo dinaikkan hanya selama retest lalu dipulihkan kembali ke `pimpinan`. Tidak ada mapping pegawai maupun audit mapping yang diubah oleh retest ini.

Temuan yang ditutup dalam retest adalah quote ganda pada selector `querySelectorAll` di atribut `x-data`. Quote tersebut merusak parsing Alpine dan membuat kode JavaScript tampil sebagai teks. Selector kini memfilter `tabindex="-1"` di JavaScript tanpa quote bersarang di markup HTML.

## Release gate yang masih blocked

`composer qa` belum melewati tahap Pint karena perubahan yang sudah ada di workspace, di luar file Fase 1–4 ini:

- `bootstrap/app.php`: `concat_space`.
- `test_show.php`: `method_argument_space`, `fully_qualified_strict_types`, `concat_space`, `statement_indentation`, `single_line_after_imports`, dan `no_whitespace_in_blank_line`.

Kedua file tidak diubah untuk pekerjaan Kelola Akses User. Setelah pemilik perubahan tersebut memformat atau memisahkannya, `composer qa` harus dijalankan ulang sebelum merge/release. Tidak ada defect P0/P1 Kelola Akses User yang terbuka dari retest ini.
