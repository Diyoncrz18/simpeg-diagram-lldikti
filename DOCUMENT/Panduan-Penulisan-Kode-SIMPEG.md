# Panduan Penulisan Kode SIMPEG

> Dokumen ini menjadi panduan praktis untuk menulis kode SIMPEG agar semua anggota tim memakai pola yang sama.
> Panduan ini dibuat sebagai referensi tim dan berdiri sendiri di dalam folder `DOCUMENT/`.

## Tujuan

Kode SIMPEG harus mudah dibaca, aman, mudah dites, dan mudah dikembangkan oleh anggota tim lain. Setiap fitur baru harus mengikuti pola arsitektur yang sudah dipakai di Laravel app saat ini:

```text
Route -> Middleware/RBAC -> FormRequest -> Controller -> Action -> Service/Model -> Resource/Payload -> Response
```

Prinsip utama:

- controller tipis;
- route hanya mendaftarkan endpoint dan middleware;
- validasi request ada di FormRequest;
- logic use case ada di Action;
- logic reusable ada di Service;
- response shaping tidak dicampur dengan query/business logic;
- audit, authorization, dan masking data sensitif harus dijaga di backend.

## Struktur File Wajib

File baru harus ditempatkan berdasarkan surface dan domain, bukan ditaruh di root folder HTTP secara acak.

```text
app/Http/Controllers/Api/V1/{Domain}Controller.php      JSON API v1
app/Http/Controllers/Admin/{Domain}Controller.php       Blade/admin page
app/Http/Controllers/Auth/{Domain}Controller.php        SSO/OIDC/auth flow
app/Http/Requests/{Domain}/{UseCase}Request.php         FormRequest per domain
app/Actions/{Domain}/{UseCase}Action.php                satu use case
app/Services/{Domain}/{Domain}Service.php               logic reusable/kompleks
resources/views/admin/{domain}/*.blade.php              halaman admin
resources/views/components/{group}/*.blade.php          UI reusable
```

Aturan:

- jangan membuat controller API baru di `app/Http/Controllers/` root;
- jangan membuat FormRequest baru di `app/Http/Requests/` root;
- boleh menyentuh file legacy root hanya saat mempertahankan behavior lama atau memindahkannya secara eksplisit;
- nama folder domain harus konsisten dengan domain fitur, seperti `Employee`, `Cuti`, `Import`, `History`, `HariLibur`, atau `Rbac`.

## Aturan Wajib

- Tidak boleh membuat fat controller.
- Tidak boleh menaruh business logic di route.
- Tidak boleh mengandalkan UI hiding sebagai security.
- Mutation endpoint wajib memakai FormRequest.
- Authorization wajib ditegakkan di backend.
- Route middleware boleh menjadi coarse gate, tetapi data-scope rule harus dicek di policy/service/action sesuai kebutuhan.
- Data sensitif pegawai harus dimasking atau tidak dikembalikan jika tidak dibutuhkan role tersebut.
- Riwayat pegawai dan audit log harus preserve auditability.
- Jangan mengubah response contract lama tanpa regression test dan persetujuan scope.
- Jangan menambah fallback/legacy path spekulatif tanpa kebutuhan nyata.
- Jangan mencampur scope SIMPEG Fase 1 dengan SAKIP atau roadmap future.

## Struktur Route API

Route API v1 memakai hierarchy berikut:

```text
routes/api.php
-> routes/api_v1.php          (aggregator v1)
-> routes/api/v1/*.php        (per domain)
```

### Menambah Domain API Baru

1. Buat file domain baru:

```text
routes/api/v1/{domain}.php
```

2. Tambahkan satu baris require di `routes/api_v1.php`:

```php
require __DIR__.'/api/v1/{domain}.php';
```

3. Pastikan route baru tetap memakai middleware role/permission yang sesuai.

Contoh domain:

```text
routes/api/v1/pegawai.php
routes/api/v1/profil.php
routes/api/v1/hari_libur.php
routes/api/v1/audit_log.php
routes/api/v1/notifikasi.php
```

### Versioned Routes

Semua endpoint bisnis SIMPEG masuk ke `routes/api/v1/*.php`.

Jika nanti ada breaking change API, buat versi baru seperti:

```text
routes/api_v2.php
routes/api/v2/*.php
```

Jangan mengubah kontrak v1 secara diam-diam.

### Non-Versioned Routes

Callback eksternal yang URL-nya harus stabil jangka panjang langsung diletakkan di `routes/api.php`, bukan di `routes/api_v1.php`.

Contoh:

- callback Keycloak;
- payment gateway callback;
- webhook provider eksternal yang URL-nya sudah fixed.

Alasan: provider eksternal tidak selalu bisa digiring update URL saat API berubah dari v1 ke v2.

### Route UUID

Route yang memakai UUID model binding wajib membatasi parameter dengan `whereUuid(...)`.

```php
Route::get('/pegawai/{employee}', [EmployeeController::class, 'show'])
    ->whereUuid('employee')
    ->name('show');
```

Tujuannya agar UUID rusak menghasilkan 404 di layer route, bukan error database 500.

### Webhook Baru

- Jika kontrak webhook ikut versi aplikasi, gunakan `routes/api/v1/webhooks.php`.
- Jika callback berasal dari provider eksternal dan URL harus tetap, gunakan `routes/api.php`.

## Controller Pattern

Controller hanya bertugas sebagai adapter HTTP:

1. menerima FormRequest atau Request;
2. menerima model binding jika ada;
3. memanggil satu Action;
4. mengembalikan response.

Controller tidak boleh berisi:

- query panjang;
- transaksi database kompleks;
- kalkulasi business rule;
- SSO/OIDC mapping;
- audit decision;
- notification dispatch;
- parsing file import;
- mapping response besar;
- authorization inline yang tersebar.

Contoh controller yang benar:

```php
public function store(StoreEmployeeFamilyRequest $request, Employee $employee, CreateEmployeeFamilyAction $action): JsonResponse
{
    $family = $action->execute($employee, $request->validated(), $request);

    return response()->json([
        'message' => 'Data keluarga berhasil ditambahkan.',
        'family' => $family,
    ], 201);
}
```

Contoh yang harus dihindari:

```php
public function store(Request $request)
{
    $validated = $request->validate([...]);
    DB::transaction(function () use ($validated) {
        // query, business rule, audit, notification, response mapping bercampur
    });
}
```

## FormRequest Pattern

Pakai FormRequest untuk semua mutation endpoint:

- `POST`
- `PUT`
- `PATCH`
- `DELETE` jika butuh payload atau authorization khusus

FormRequest digunakan untuk:

- validasi field;
- normalisasi input sederhana;
- authorize gate pada boundary request;
- pesan error validasi yang konsisten.

Contoh:

```php
class StoreEmployeeFamilyRequest extends FormRequest
{
    public function authorize(): bool
    {
        return in_array($this->user()?->role, ['super_admin', 'admin_kepegawaian'], true);
    }

    public function rules(): array
    {
        return [
            'nama_anggota' => ['required', 'string', 'max:255'],
            'hubungan' => ['required', Rule::in(['Suami', 'Istri', 'Anak'])],
            'nik' => ['nullable', 'digits:16'],
            'tanggal_lahir' => ['required', 'date', 'before_or_equal:today'],
            'jenis_kelamin' => ['required', Rule::in(['L', 'P'])],
            'status_tunjangan' => ['required', 'boolean'],
            'pekerjaan' => ['nullable', 'string', 'max:100'],
        ];
    }
}
```

Jangan validasi manual di controller jika FormRequest bisa dipakai.

Gunakan `authorize()` untuk gate mutation yang melekat pada request. Jika authorization butuh ownership/data-scope yang panjang, panggil Policy atau scoped service dari `authorize()` atau Action, jangan tulis rule panjang langsung di controller.

## Action Pattern

Action mewakili satu use case aplikasi.

Nama Action harus jelas:

- `CreateEmployeeAction`
- `UpdateEmployeeAction`
- `DeactivateEmployeeAction`
- `RestoreEmployeeAction`
- `CreateEmployeeFamilyAction`
- `GenerateEwsEventsAction`

Action boleh melakukan:

- orchestration use case;
- transaksi database;
- pemanggilan Service;
- audit logging;
- notification dispatch;
- data-scope checks;
- payload helper calls.

Action tidak boleh menjadi tempat semua logic domain jika logic tersebut dipakai ulang oleh banyak use case. Jika logic mulai berulang, pindahkan ke Service.

Action juga tidak boleh dipakai untuk mengubah response contract secara diam-diam. Jika response akan dimigrasikan ke API Resource atau envelope baru, kunci kontrak lama dengan test terlebih dahulu.

Contoh Action:

```php
class DeactivateEmployeeAction
{
    public function execute(Employee $employee, Request $request): void
    {
        DB::transaction(function () use ($employee, $request): void {
            $oldValues = $employee->only(['id', 'nama', 'nip', 'email']);

            $employee->delete();

            AuditService::log('SOFT_DELETE', 'Employee', $employee->id, $oldValues, null, $request);
        });
    }
}
```

## Service Pattern

Service dipakai untuk behavior yang reusable atau kompleks.

Gunakan Service ketika:

- logic dipakai oleh beberapa Action;
- ada kalkulasi domain yang panjang;
- ada integrasi eksternal;
- ada parsing file atau transformasi data yang perlu dites terpisah.

Contoh kandidat Service:

- `EmployeeImportService`
- `EmployeeHistoryService`
- `CutiApprovalService`
- `EwsEligibilityService`
- `NotificationDispatcher`

Jangan membuat Service untuk semua hal kecil. Jika logic hanya satu use case dan masih mudah dibaca, cukup Action.

## Authorization dan RBAC

Authorization harus berlapis:

1. route middleware untuk authentication/role/permission gate;
2. FormRequest `authorize()` untuk mutation boundary;
3. Policy atau scoped service untuk data ownership;
4. Action untuk orchestration dan pemanggilan policy/service.

Contoh route gate:

```php
Route::middleware(['web', 'keycloak.auth', 'role:super_admin,admin_kepegawaian'])
    ->prefix('pegawai')
    ->name('pegawai.')
    ->group(function (): void {
        Route::post('/', [EmployeeController::class, 'store'])
            ->middleware('permission:employees.create')
            ->name('store');
    });
```

Jangan hanya menyembunyikan tombol di Blade. Jika user tidak boleh melakukan aksi, backend harus menolak request.

## Blade Component Pattern

Blade component dipakai untuk bagian UI yang berulang, punya variasi state yang jelas, atau menjadi elemen dasar tampilan SIMPEG. Tujuannya adalah menjaga tampilan konsisten, mengurangi copy-paste HTML, dan membuat perubahan desain lebih mudah dilakukan dari satu tempat.

### Kapan Membuat Blade Component

Buat component jika UI:

- dipakai minimal di dua halaman atau berpotensi dipakai ulang;
- memiliki style yang harus konsisten, seperti button, badge, alert, card, table toolbar, empty state, modal, pagination, atau form field;
- memiliki beberapa variasi yang masih satu keluarga, seperti `primary`, `secondary`, `danger`, `success`, `warning`;
- punya state yang berulang, seperti loading, disabled, error, empty, active, selected;
- membutuhkan slot agar isi bisa fleksibel tetapi wrapper/style tetap sama.

Jangan membuat component jika:

- hanya dipakai sekali dan tidak ada pola reuse yang jelas;
- component hanya membungkus satu tag tanpa nilai konsistensi;
- props terlalu banyak sampai component sulit dipahami;
- component mencampur query database, authorization business rule, atau logic controller.

Prioritas component SIMPEG:

1. primitive UI yang sering muncul: button, badge, alert, card, table, modal;
2. form control: input, select, textarea, checkbox, date input, file input;
3. page helper: page header, filter bar, action group, empty state, pagination summary;
4. domain display yang benar-benar berulang: status cuti, status pegawai, ringkasan notifikasi.

### Lokasi dan Penamaan

Simpan Blade component di:

```text
resources/views/components/
```

Gunakan struktur folder berdasarkan jenis UI:

```text
resources/views/components/ui/button.blade.php
resources/views/components/ui/badge.blade.php
resources/views/components/ui/alert.blade.php
resources/views/components/form/input.blade.php
resources/views/components/form/select.blade.php
resources/views/components/layouts/app.blade.php
resources/views/components/admin/page-header.blade.php
```

Gunakan nama kebab-case dan panggil dengan prefix yang sesuai:

```blade
<x-ui.button variant="primary" type="submit">
    Simpan
</x-ui.button>

<x-form.input
    name="nip"
    label="NIP"
    :value="old('nip', $pegawai->nip ?? '')"
    required
/>
```

### Props dan Slot

Props harus sederhana dan eksplisit. Gunakan props untuk konfigurasi kecil, bukan untuk membawa data domain besar.

Contoh props yang boleh:

- `variant`
- `size`
- `type`
- `label`
- `name`
- `value`
- `required`
- `disabled`
- `href`
- `icon`

Contoh props yang harus dihindari:

- seluruh model Eloquent jika hanya butuh satu atau dua field;
- collection besar;
- flag business rule yang seharusnya dihitung di controller/action;
- array konfigurasi panjang yang membuat component sulit dibaca.

Gunakan slot untuk konten utama:

```blade
<x-ui.alert variant="warning">
    Data pegawai belum lengkap. Lengkapi data SK dan riwayat kepangkatan.
</x-ui.alert>
```

Gunakan escaped output `{{ }}` untuk data user atau data database. Jangan memakai `{!! !!}` kecuali HTML sudah disanitasi dan alasan keamanannya jelas di kode.

Gunakan named slot hanya jika benar-benar perlu area khusus:

```blade
<x-admin.page-header title="Data Pegawai">
    <x-slot:actions>
        <x-ui.button href="{{ route('pegawai.create') }}" variant="primary">
            Tambah Pegawai
        </x-ui.button>
    </x-slot:actions>
</x-admin.page-header>
```

### Styling Component

Semua class visual utama harus berada di dalam component, bukan disalin berulang di halaman.

Aturan styling:

- gunakan token/class design system yang sudah dipakai project;
- variasi style dikontrol lewat props seperti `variant` dan `size`;
- gabungkan class default dengan `$attributes->class([...])` atau `$attributes->merge([...])` agar caller masih bisa menambah atribut HTML standar;
- jangan membuat style inline kecuali untuk nilai dinamis yang tidak bisa diwakili class;
- jangan menyisipkan CSS panjang di file Blade halaman;
- pastikan state `hover`, `focus`, `disabled`, dan `loading` konsisten;
- component form wajib menampilkan error validation dengan pola yang sama.

Contoh pola variant:

```blade
@props([
    'variant' => 'primary',
    'type' => 'button',
    'disabled' => false,
])

@php
    $variants = [
        'primary' => 'bg-primary text-white hover:bg-primary/90',
        'secondary' => 'bg-surface text-ink border border-border hover:bg-soft',
        'danger' => 'bg-danger text-white hover:bg-danger/90',
    ];
@endphp

<button
    type="{{ $type }}"
    @disabled($disabled)
    {{ $attributes->class([
        'inline-flex items-center justify-center rounded-md px-3 py-2 text-sm font-semibold transition',
        $variants[$variant] ?? $variants['primary'],
    ]) }}
>
    {{ $slot }}
</button>
```

Jika component bisa render `<a>` dan `<button>`, bedakan behavior secara eksplisit. Link untuk navigasi, button untuk submit/action. Jangan membuat satu component yang menebak behavior dari terlalu banyak prop.

### Component Form

Form component harus mengikuti pola validasi Laravel.

Untuk input, minimal dukung:

- `name`;
- `label`;
- `value`;
- `required`;
- `disabled`;
- `placeholder`;
- error dari `$errors`;
- `old()` dari halaman pemanggil atau value yang dikirim sebagai props.
- atribut aksesibilitas seperti `id`, `for`, `aria-invalid`, dan `aria-describedby` saat ada error/help text.

Contoh pemakaian:

```blade
<x-form.input
    name="tanggal_sk"
    label="Tanggal SK"
    type="date"
    :value="old('tanggal_sk', $rankHistory->tanggal_sk ?? '')"
    required
/>
```

Component form tidak boleh melakukan validasi sendiri. Validasi tetap ada di FormRequest. Component hanya menampilkan state error dari backend.

Form Blade wajib memakai fitur Laravel untuk request state:

- `@csrf` untuk semua form mutation;
- `@method('PUT')`, `@method('PATCH')`, atau `@method('DELETE')` saat method HTML perlu spoofing;
- `old()` untuk mempertahankan input setelah validasi gagal;
- `@error('field')` atau error bag yang sesuai untuk pesan validasi;
- `route(...)` untuk URL internal, bukan hardcoded path.

Contoh form singkat:

```blade
<form method="POST" action="{{ route('pegawai.update', $pegawai) }}">
    @csrf
    @method('PATCH')

    <x-form.input
        name="email"
        label="Email"
        type="email"
        :value="old('email', $pegawai->email)"
        required
    />

    <x-ui.button type="submit" variant="primary">
        Simpan
    </x-ui.button>
</form>
```

### Authorization di Blade

Blade component boleh menerima prop seperti `can` atau memakai `@can` untuk mengatur tampilan tombol/aksi, tetapi itu hanya untuk UX.

Contoh:

```blade
@can('create', App\Models\Employee::class)
    <x-ui.button href="{{ route('pegawai.create') }}" variant="primary">
        Tambah Pegawai
    </x-ui.button>
@endcan
```

Aturan penting:

- component boleh menyembunyikan tombol berdasarkan permission;
- backend tetap wajib menolak request tanpa izin;
- jangan menaruh decision business rule kompleks di Blade component;
- jika logic permission mulai panjang, pindahkan ke Policy, middleware, Action, atau helper yang jelas.

### Data dan Query

Blade component tidak boleh menjalankan query database langsung.

Hindari:

```blade
@php
    $totalPegawai = \App\Models\Employee::count();
@endphp
```

Data harus disiapkan oleh controller/action/service lalu dikirim ke view:

```php
return view('admin.pegawai.index', [
    'totalPegawai' => $summary->totalPegawai,
]);
```

Component hanya menerima data siap tampil.

Blade halaman juga tidak boleh melakukan kalkulasi domain panjang. Format tampilan ringan boleh, seperti `->format('d/m/Y')`, tetapi status, eligibility, saldo cuti, dan rule akses harus disiapkan sebelum view.

Untuk list besar, controller/action harus mengirim paginator atau collection yang sudah dibatasi. Jangan melakukan filtering collection besar di Blade.

### Alpine/JavaScript di Component

Jika component membutuhkan Alpine.js:

- scope `x-data` harus kecil dan jelas;
- jangan menyimpan business rule backend di JavaScript;
- gunakan event name yang deskriptif;
- state UI seperti open/close, selected tab, loading indicator boleh berada di component;
- jika logic JavaScript panjang, pindahkan ke file JS terpisah.

Gunakan progressive enhancement: fitur utama tetap harus aman lewat backend request. Alpine hanya boleh memperbaiki pengalaman UI, bukan menggantikan validasi, authorization, audit, atau perubahan data.

### Blade Layout dan Partial

- Layout halaman utama simpan di `resources/views/layouts/` atau component layout yang sudah dipakai project.
- Partial biasa boleh dipakai untuk potongan satu halaman yang belum layak jadi component reusable.
- Jika partial mulai dipakai lintas halaman atau butuh variasi state, naikkan menjadi Blade component.
- Hindari markup tabel/form/action button copy-paste di banyak halaman; buat component atau partial kecil.
- Gunakan `@once`, `@push`, dan `@stack` untuk script/style per halaman agar asset tidak terduplikasi.

### Aksesibilitas Blade

- Label form harus terhubung ke input lewat `for` dan `id`.
- Tombol icon-only wajib punya teks screen-reader atau `aria-label`.
- Error validasi harus bisa dibaca pembaca layar lewat `aria-describedby`.
- Modal/dropdown harus punya state focus dan keyboard yang aman jika dipakai untuk aksi penting.
- Warna status tidak boleh menjadi satu-satunya pembeda; tambahkan teks atau icon yang jelas.

### Dokumentasi Mini

Untuk component yang dipakai luas, tambahkan komentar singkat di awal file jika props-nya tidak langsung jelas.

Contoh:

```blade
{{-- Props: variant=primary|secondary|danger, size=sm|md, href optional untuk render link --}}
```

Jangan memberi komentar yang hanya mengulang nama variable.

## Audit Log Pattern

Setiap mutation penting harus audit-aware:

- create;
- update;
- soft delete;
- restore;
- import;
- approval/rejection;
- config changes.

Gunakan event yang eksplisit:

```text
CREATE
UPDATE
SOFT_DELETE
RESTORE
IMPORT
APPROVE
REJECT
```

Audit log harus menyimpan old/new values jika relevan, tetapi jangan simpan data sensitif yang tidak perlu.

Contoh data sensitif:

- NIK;
- No KK;
- token;
- secret;
- credential;
- raw Keycloak payload.

Jika perlu audit payload, buat helper payload khusus agar masking konsisten.

## Response dan Payload Pattern

Untuk response sederhana, controller boleh membentuk JSON final.

Untuk response besar atau berulang, gunakan:

- API Resource;
- payload helper;
- mapper class.

Contoh payload helper:

```php
class EmployeeFamilyPayload
{
    public function response(EmployeeFamily $family): array
    {
        return [
            'id' => $family->id,
            'nama_anggota' => $family->nama_anggota,
            'hubungan' => $family->hubungan,
            'tanggal_lahir' => $family->tanggal_lahir?->toDateString(),
        ];
    }

    public function audit(EmployeeFamily $family): array
    {
        return Arr::except($this->response($family), ['nik']);
    }
}
```

Jangan mengubah bentuk response endpoint lama tanpa test yang mengunci kontrak lama.

Sebelum migrasi response ke `JsonResource` atau envelope baru, inventarisasi shape endpoint lama: key utama, `meta`, pagination, message, status code, dan masking field sensitif. Migrasi hanya boleh dilakukan saat kontrak baru disetujui atau test membuktikan shape lama tetap sama.

## Query dan Filter Pattern

List endpoint yang punya banyak filter tidak boleh membengkakkan controller.

Jika filter mulai banyak, pindahkan ke query/filter object:

- `EmployeeQuery`
- `AuditLogQuery`
- `NotificationQuery`

Aturan query/filter:

- default sorting eksplisit;
- pagination eksplisit;
- filter invalid divalidasi di FormRequest;
- response shape tidak berubah tanpa scope khusus.

## File Upload dan Import Pattern

Untuk import atau upload file:

- validasi file di FormRequest;
- parsing file di Action/Service/support class, bukan controller;
- jangan percaya nama file dari user;
- normalisasi header/input sebelum insert;
- gunakan transaksi database untuk all-or-nothing jika requirement begitu;
- audit hasil import;
- untuk file besar, gunakan queue job jika proses bisa lama.

Flow import yang ideal:

```text
upload -> preview -> validate -> execute -> result/report
```

Jika flow dibuat bertahap, setiap step harus punya controller method, Action/Service, response contract, dan test.

## Blade dan Frontend Integration

Blade boleh menangani tampilan, state UI ringan, dan interaksi pengguna.

Blade tidak boleh menjadi sumber security rule.

Jika Blade memanggil API:

- endpoint harus benar-benar ada;
- middleware backend harus enforce permission;
- error response harus ditangani;
- jangan biarkan tombol hanya pura-pura berhasil di JavaScript tanpa request backend.

Contoh anti-pattern:

```text
Klik restore -> JavaScript hanya hide row -> tidak ada backend call.
```

Contoh yang benar:

```text
Klik restore -> POST/PATCH ke backend -> backend restore -> audit RESTORE -> UI refresh/list update.
```

## Testing Rules

Setiap fitur backend non-trivial harus punya test.

Minimal test untuk CRUD/mutation:

- guest ditolak;
- role tidak berwenang ditolak;
- permission dicabut ditolak;
- valid request berhasil;
- invalid request gagal validasi;
- audit log tertulis jika mutation penting;
- data ownership dicek jika ada nested resource;
- soft delete/restore benar-benar mengubah data.

Untuk refactor route/API:

- snapshot route before/after jika refactor route;
- pastikan URI, method, route name, middleware, controller action tidak berubah;
- pastikan response shape, authorization behavior, dan user-visible behavior tetap sama;
- jalankan focused tests dan `composer qa`.

Backend change yang sensitif database harus diuji di PostgreSQL, bukan hanya SQLite. Contoh sensitif: UUID, JSON column, date casting, FK/index, transaction, pagination query, dan route model binding.

Quality gate lokal default:

```powershell
composer qa
```

Quality gate PR/CI minimal:

- asset build;
- Pint full-project;
- PHPStan minimal level 3;
- test suite Laravel.

Command yang umum dipakai:

```powershell
php artisan test tests/Feature/NamaTest.php
composer qa
php artisan route:list --path=api/v1 --json
```

Catatan: runtime Laravel project ini tidak mendukung `route:list --columns=...`. Gunakan `--json` untuk snapshot route.

## Code Comments dan Docblocks

Komentar kode harus membantu pembaca memahami alasan dan konteks domain.

Tulis komentar dalam Bahasa Indonesia untuk:

- business rule penting;
- keputusan non-obvious;
- public method di Action/Service/Job jika maksudnya tidak langsung jelas;
- request validation yang melindungi rule domain;
- authorization check yang berkaitan dengan role/data scope;
- edge case yang penting.

Jangan menulis komentar untuk:

- syntax obvious;
- getter/setter trivial;
- CRUD yang sudah jelas dari nama method;
- mengulang isi kode;
- menyebut sprint/phase/issue/PRD section di source code.

Komentar harus plain text. Jangan gunakan emoji di source code comment atau docblock.

Contoh komentar yang baik:

```php
// Restore hanya mengaktifkan kembali pegawai; relasi riwayat tetap dipertahankan untuk menjaga auditability.
$employee->restore();
```

Contoh komentar yang buruk:

```php
// Delete employee
$employee->delete();
```

## Checklist Sebelum PR

Sebelum membuat PR, pastikan:

- controller tetap tipis;
- route tidak berisi business logic;
- mutation memakai FormRequest;
- Action dibuat per use case;
- audit log ada untuk mutation penting;
- data sensitif tidak bocor;
- permission/backend authorization diuji;
- response contract tidak berubah diam-diam;
- route UUID binding memakai `whereUuid(...)` jika perlu;
- refactor membuktikan behavior lama tetap sama;
- tests relevan pass;
- `composer qa` pass;
- CI wajib build, Pint, PHPStan, dan test suite pass sebelum siap merge.

## Contoh Struktur Fitur Baru

Contoh fitur: restore pegawai.

```text
routes/web.php atau routes/api/v1/pegawai.php
-> PegawaiController@restore
-> RestoreEmployeeRequest jika perlu payload/authorize khusus
-> RestoreEmployeeAction
-> AuditService::log('RESTORE', ...)
-> response redirect/json
-> Feature test restore + RBAC + audit
```

File yang mungkin dibuat:

```text
app/Actions/Employees/RestoreEmployeeAction.php
app/Http/Requests/Employee/RestoreEmployeeRequest.php
tests/Feature/EmployeeRestoreTest.php
```

Controller tetap hanya memanggil Action.
