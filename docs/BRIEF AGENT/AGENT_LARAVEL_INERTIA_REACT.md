# PANDUAN PENULISAN KODE — Laravel 12 + Inertia.js + React

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek berbasis **Laravel 12** (backend) dengan **Inertia.js** sebagai bridge dan **React** (frontend).

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Backend Engine | Laravel 12 (PHP 8.2+) — Eloquent ORM |
| Frontend Framework | React (Vite integration inside Laravel) |
| Monolith Bridge | Inertia.js (v2.x) |
| Styling & UI | Tailwind CSS + shadcn/ui |
| State Management | Inertia Shared Props (via `usePage().props`) |
| Tabel Data | @tanstack/react-table + Pagination |
| Icons & Toast | lucide-react + sonner |
| HTTP Client | Axios (configured globally with CSRF token) |
| Auth System | Laravel Session & Web Middleware (Stateful) |
| Routing Helper | Ziggy (route() di frontend) |

---

## 1. STRUKTUR FOLDER PROJECT (Monolith)

```
project-root/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Admin/              # Admin Area Controllers
│   │   │   └── Guest/              # Public Controllers
│   │   ├── Middleware/
│   │   │   └── HandleInertiaRequests.php  # Shared Props
│   │   ├── Requests/               # Form Request Validation
│   │   └── Resources/              # API Resource Transformers
│   ├── Models/                     # Eloquent Models
│   ├── Services/                   # Business Logic Services
│   └── Jobs/                       # Queue Jobs (async tasks)
│
├── config/
│   └── filesystems.php             # Storage disk configuration
│
├── database/
│   ├── migrations/                 # Schema migrations
│   └── seeders/                    # Data seeders
│
├── resources/
│   ├── js/
│   │   ├── Pages/                  # Halaman Inertia (CamelCase)
│   │   │   ├── Auth/
│   │   │   ├── Admin/
│   │   │   └── Guest/
│   │   ├── Components/             # React Components
│   │   │   ├── Admin/
│   │   │   ├── Guest/
│   │   │   └── Shared/             # 100% reusable
│   │   ├── Hooks/                  # Custom React Hooks
│   │   ├── Layouts/                # Layout Wrappers
│   │   ├── Lib/                    # Utils, Axios config
│   │   ├── Types/                  # TypeScript Interfaces
│   │   ├── app.tsx                 # Bootstrap Inertia
│   │   └── ssr.tsx                 # SSR setup (opsional)
│   └── views/
│       └── app.blade.php           # Root Blade (@inertia)
│
├── routes/
│   ├── web.php                     # Inertia routes (session auth)
│   └── api.php                     # Stateless API / Webhook
└── package.json
```

---

## 2. PENULISAN KODE BACKEND (Laravel)

### 2.1 Slim Controller
Controller hanya bertugas: menerima request → validasi (FormRequest) → panggil Service → return response.

**Render Inertia:**
```php
public function index(): \Inertia\Response
{
    $items = $this->itemService->getAll();
    return Inertia::render('Admin/Items/Index', [
        'items' => ItemResource::collection($items)
    ]);
}
```

**Mutasi + Redirect:**
```php
public function store(ItemRequest $request): \Illuminate\Http\RedirectResponse
{
    $this->itemService->create($request->validated());
    return redirect()->route('admin.items.index')
        ->with('success', 'Data berhasil ditambahkan.');
}
```

### 2.2 Service Layer
- Logika bisnis, query kompleks, dan integrasi API eksternal ditulis di `app/Services/`.
- Gunakan Dependency Injection di constructor.

### 2.3 Transaksi Database
```php
use Illuminate\Support\Facades\DB;

public function createOrder(array $data)
{
    return DB::transaction(function () use ($data) {
        $order = Order::create($data);
        $order->items()->createMany($data['items']);
        return $order;
    });
}
```

### 2.4 Caching & Queue
```php
// Caching data statis
$categories = Cache::remember('categories:all', now()->addHours(6), fn() =>
    Category::all()
);

// Background job untuk proses berat
SendNotificationJob::dispatch($order);
```

### 2.5 Model & Database
- Kolom database: `snake_case`.
- Definisikan casts secara eksplisit:
```php
protected function casts(): array
{
    return [
        'tags'       => 'array',
        'is_active'  => 'boolean',
        'price'      => 'integer',
        'secret_data'=> 'encrypted:array',
    ];
}
```
- Gunakan `BIGINT` untuk kolom harga Rupiah (tanpa desimal).

### 2.6 Docblock Wajib
```php
/**
 * Mengambil semua item aktif berdasarkan kategori.
 *
 * @param  int  $categoryId
 * @return \Illuminate\Database\Eloquent\Collection
 */
public function getByCategory(int $categoryId): Collection
{ ... }
```

---

## 3. PENULISAN KODE FRONTEND (React + Inertia)

### 3.1 Komponen & Halaman
- Halaman: `resources/js/Pages/` (CamelCase, contoh: `Items/Index.tsx`).
- Komponen: `resources/js/Components/` (Admin, Guest, Shared).
- Wajib TypeScript. Hindari tipe `any`.

### 3.2 Form (Inertia `useForm`)
```tsx
import { useForm } from '@inertiajs/react'

const { data, setData, post, processing, errors } = useForm({
  name: '',
  is_active: false,
})

const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault()
  post(route('admin.items.store'), {
    onError: (errs) => {
      toast.error(`${Object.keys(errs).length} validasi gagal.`)
    },
  })
}
```

### 3.3 Shared Props (HandleInertiaRequests.php)
```php
public function share(Request $request): array
{
    return [
        ...parent::share($request),
        'auth' => ['user' => $request->user()],
        'flash' => [
            'success' => fn() => $request->session()->get('success'),
            'error'   => fn() => $request->session()->get('error'),
        ],
    ];
}
```

Akses di frontend:
```tsx
const { auth, flash } = usePage().props
```

### 3.4 Deteksi Route & Menu Aktif
```tsx
import { usePage } from '@inertiajs/react'
const { url } = usePage()
const isActive = url.startsWith('/admin/items')
```

### 3.5 Routing (Ziggy)
```tsx
// Gunakan helper route() dari Ziggy
post(route('admin.items.store'))
router.visit(route('admin.items.edit', { item: id }))
```

---

## 4. DATATABLE (AJAX PAGINATION)

### 4.1 Endpoint Dataset
- Method: `GET`
- URL Pattern: `/admin/data-{module}?per_page={n}&page={n}&search={q}`
- Per page: `10`, `25`, `50`, `100`

### 4.2 Controller Dataset
```php
/**
 * Endpoint dataset untuk DataTable (AJAX GET).
 */
public function dataset(Request $request): \Illuminate\Http\JsonResponse
{
    $perPage = min((int) $request->get('per_page', 10), 100);
    $query = Item::query()
        ->when($request->search, fn($q, $s) => $q->where('name', 'like', "%$s%"));
    return response()->json($query->paginate($perPage));
}
```

### 4.3 Komponen DataTable
```tsx
<DataTable
  endpoint="/admin/data-items"
  columns={[
    { key: 'name', label: 'Nama' },
    { key: 'is_active', label: 'Status', render: (v) => v ? 'Aktif' : 'Nonaktif' },
  ]}
  actions={(row) => (
    <>
      <Button onClick={() => router.visit(`/admin/items/${row.id}/edit`)}>Edit</Button>
      <Button variant="danger" onClick={() => handleDelete(row.id)}>Hapus</Button>
    </>
  )}
/>
```

---

## 5. LOADING UX

### 5.1 Spinner Overlay Tabel
```tsx
{loading && (
  <div className="absolute inset-0 bg-black/60 flex items-center justify-center rounded-xl z-10">
    <div className="w-8 h-8 border-2 border-blue-500 border-t-transparent rounded-full animate-spin" />
  </div>
)}
```

### 5.2 Skeleton Loading
```tsx
<ImageSkeleton src={imageUrl} alt="Item" className="w-8 h-8 rounded" />
```
- Sebelum load: `animate-pulse bg-gray-700`
- Setelah `onLoad`: fade-in transition

---

## 6. PROTEKSI ROUTE

```php
// Guest (publik)
Route::get('/', [HomeController::class, 'index'])->name('home');

// Auth
Route::get('/login', [AuthController::class, 'showLogin'])->middleware('guest');
Route::post('/login', [AuthController::class, 'login'])->middleware('guest');
Route::post('/logout', [AuthController::class, 'logout']);

// Protected Admin
Route::middleware(['auth'])->prefix('admin')->name('admin.')->group(function () {
    Route::get('/', [DashboardController::class, 'index'])->name('dashboard');
    Route::resource('items', ItemController::class);
});
```

---

## 7. VALIDASI & ERROR

### 7.1 Form Request
- Validasi mutasi wajib menggunakan Form Request (`app/Http/Requests/`).
- Error 422 otomatis dipetakan ke `errors` di `useForm`.

### 7.2 Error Inline + Toast
- Error validasi ditampilkan inline di bawah field via `errors.fieldName`.
- Toast summary: `toast.error('N validasi gagal.')` di callback `onError`.

### 7.3 API/Webhook Response
```json
{
  "success": false,
  "message": "Pesan error informatif.",
  "error": "Detail kesalahan"
}
```

---

## 8. RESPONSIVE & MOBILE

- Sidebar → drawer overlay di mobile.
- Tabel → `overflow-x-auto` horizontal scroll.
- Form → kolom tunggal di mobile (<768px), multi-kolom di desktop.
