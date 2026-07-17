# PANDUAN PENULISAN KODE — React + Vite (SPA)

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek **Single Page Application (SPA)** berbasis **React** dengan build tool **Vite**.

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Framework | React 18/19 |
| Build Tool | Vite |
| Language | TypeScript (strict) |
| Routing | React Router v6/v7 |
| State Management | Zustand / TanStack Query |
| Styling | Tailwind CSS |
| UI Library | shadcn/ui / Radix UI |
| HTTP Client | Axios / Fetch API |
| Form | React Hook Form + Zod |
| Icons & Toast | lucide-react + sonner |

---

## 1. STRUKTUR FOLDER PROJECT

```
project-root/
├── src/
│   ├── main.tsx                    # Entry point
│   ├── App.tsx                     # Root component + Router
│   │
│   ├── pages/                      # Route pages
│   │   ├── Home.tsx
│   │   ├── Login.tsx
│   │   ├── Dashboard.tsx
│   │   └── NotFound.tsx
│   │
│   ├── components/
│   │   ├── ui/                     # Atomic components (Button, Input, Modal)
│   │   ├── forms/                  # Form-specific components
│   │   ├── layouts/                # Layout wrappers
│   │   │   ├── MainLayout.tsx
│   │   │   └── DashboardLayout.tsx
│   │   └── shared/                 # Reusable business components
│   │       ├── DataTable.tsx
│   │       ├── Pagination.tsx
│   │       ├── ImageSkeleton.tsx
│   │       └── ConfirmModal.tsx
│   │
│   ├── hooks/                      # Custom hooks
│   │   ├── useAuth.ts
│   │   └── useDebounce.ts
│   │
│   ├── services/                   # API call layer
│   │   ├── api.ts                  # Axios instance
│   │   ├── auth.service.ts
│   │   └── item.service.ts
│   │
│   ├── stores/                     # State management (Zustand)
│   │   ├── auth.store.ts
│   │   └── ui.store.ts
│   │
│   ├── types/                      # TypeScript interfaces
│   │   └── index.ts
│   │
│   ├── lib/
│   │   ├── utils.ts                # Helper functions (cn(), formatCurrency)
│   │   └── validations.ts          # Zod schemas
│   │
│   └── assets/                     # Static assets (images, fonts)
│
├── public/
├── .env
├── index.html
├── vite.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

---

## 2. ROUTING (React Router)

```tsx
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { MainLayout } from '@/components/layouts/MainLayout';
import { DashboardLayout } from '@/components/layouts/DashboardLayout';
import { ProtectedRoute } from '@/components/shared/ProtectedRoute';

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public Routes */}
        <Route element={<MainLayout />}>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
        </Route>

        {/* Protected Routes */}
        <Route element={<ProtectedRoute />}>
          <Route element={<DashboardLayout />}>
            <Route path="/dashboard" element={<Dashboard />} />
            <Route path="/dashboard/items" element={<Items />} />
            <Route path="/dashboard/items/:id" element={<ItemDetail />} />
          </Route>
        </Route>

        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 2.1 Protected Route
```tsx
import { Navigate, Outlet } from 'react-router-dom';
import { useAuthStore } from '@/stores/auth.store';

export function ProtectedRoute() {
  const { isAuthenticated } = useAuthStore();
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
}
```

---

## 3. API SERVICE LAYER

### 3.1 Axios Instance
```tsx
// services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: { 'Content-Type': 'application/json' },
});

// Interceptor: inject token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Interceptor: handle 401
api.interceptors.response.use(
  (res) => res,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### 3.2 Service File
```tsx
// services/item.service.ts
import api from './api';

export const itemService = {
  getAll: (params: { page: number; perPage: number; search?: string }) =>
    api.get('/items', { params }),

  getById: (id: string) => api.get(`/items/${id}`),

  create: (data: CreateItemDto) => api.post('/items', data),

  update: (id: string, data: UpdateItemDto) => api.put(`/items/${id}`, data),

  delete: (id: string) => api.delete(`/items/${id}`),
};
```

---

## 4. STATE MANAGEMENT (Zustand)

```tsx
// stores/auth.store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      login: (user, token) => set({ user, token, isAuthenticated: true }),
      logout: () => set({ user: null, token: null, isAuthenticated: false }),
    }),
    { name: 'auth-storage' }
  )
);
```

---

## 5. FORM (React Hook Form + Zod)

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { toast } from 'sonner';

const schema = z.object({
  name: z.string().min(1, 'Nama wajib diisi'),
  price: z.number().int().positive('Harga harus positif'),
});

type FormData = z.infer<typeof schema>;

export function CreateItemForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    try {
      await itemService.create(data);
      toast.success('Data berhasil ditambahkan');
    } catch (err) {
      toast.error('Gagal menyimpan data');
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <p className="text-red-500 text-sm">{errors.name.message}</p>}

      <input type="number" {...register('price', { valueAsNumber: true })} />
      {errors.price && <p className="text-red-500 text-sm">{errors.price.message}</p>}

      <button disabled={isSubmitting}>
        {isSubmitting ? 'Menyimpan...' : 'Simpan'}
      </button>
    </form>
  );
}
```

---

## 6. DATA FETCHING (TanStack Query)

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// GET (with auto-refetch, caching, loading state)
export function useItems(page: number, search: string) {
  return useQuery({
    queryKey: ['items', page, search],
    queryFn: () => itemService.getAll({ page, perPage: 25, search }),
    select: (res) => res.data,
  });
}

// POST / PUT / DELETE (mutation)
export function useCreateItem() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: itemService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['items'] });
      toast.success('Data berhasil ditambahkan');
    },
    onError: () => toast.error('Gagal menyimpan'),
  });
}
```

---

## 7. LOADING UX

### 7.1 Skeleton Loading
```tsx
export function TableSkeleton({ rows = 5 }: { rows?: number }) {
  return (
    <div className="space-y-3">
      {Array.from({ length: rows }).map((_, i) => (
        <div key={i} className="h-12 bg-gray-200 rounded animate-pulse" />
      ))}
    </div>
  );
}
```

### 7.2 Image Skeleton
```tsx
export function ImageSkeleton({ src, alt, className }: ImageProps) {
  const [loaded, setLoaded] = useState(false);
  return (
    <div className={cn('relative', className)}>
      {!loaded && <div className="absolute inset-0 bg-gray-200 animate-pulse rounded" />}
      <img
        src={src}
        alt={alt}
        onLoad={() => setLoaded(true)}
        className={cn('transition-opacity duration-300', loaded ? 'opacity-100' : 'opacity-0')}
      />
    </div>
  );
}
```

### 7.3 Spinner Overlay (Tabel)
```tsx
{isFetching && (
  <div className="absolute inset-0 bg-black/50 flex items-center justify-center z-10 rounded-lg">
    <div className="w-8 h-8 border-2 border-blue-500 border-t-transparent rounded-full animate-spin" />
  </div>
)}
```

---

## 8. LAZY LOADING

```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('@/pages/Dashboard'));
const Items = lazy(() => import('@/pages/Items'));

// Di router:
<Suspense fallback={<PageSkeleton />}>
  <Route path="/dashboard" element={<Dashboard />} />
</Suspense>
```

---

## 9. ENVIRONMENT VARIABLES

```env
# .env
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=MyApp
```

- Prefix `VITE_` wajib agar bisa diakses di client.
- Akses: `import.meta.env.VITE_API_URL`.
- **Jangan** simpan secret/API key di sini (client-side = public).
