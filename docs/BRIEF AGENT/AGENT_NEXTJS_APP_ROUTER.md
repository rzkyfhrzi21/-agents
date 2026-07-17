# PANDUAN PENULISAN KODE — Next.js (App Router) + TypeScript

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek berbasis **Next.js** (App Router) dengan **TypeScript**.

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Framework | Next.js 14/15 (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS / CSS Modules |
| UI Library | shadcn/ui (opsional) |
| State Management | React Server Components + Client Hooks |
| ORM | Prisma / Drizzle |
| Auth | NextAuth.js / Clerk / Lucia |
| Deployment | Vercel / VPS / Docker |

---

## 1. STRUKTUR FOLDER PROJECT

```
project-root/
├── src/
│   ├── app/                        # App Router
│   │   ├── layout.tsx              # Root layout
│   │   ├── page.tsx                # Homepage (/)
│   │   ├── loading.tsx             # Loading UI (Suspense fallback)
│   │   ├── error.tsx               # Error boundary
│   │   ├── not-found.tsx           # 404 page
│   │   ├── (auth)/                 # Route group: auth
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── dashboard/              # Protected routes
│   │   │   ├── layout.tsx          # Dashboard layout (sidebar)
│   │   │   ├── page.tsx            # /dashboard
│   │   │   └── [slug]/page.tsx     # Dynamic route
│   │   └── api/                    # API Routes (Route Handlers)
│   │       └── webhook/route.ts
│   │
│   ├── components/
│   │   ├── ui/                     # Atomic/shared components
│   │   ├── forms/                  # Form components
│   │   └── layouts/                # Layout wrappers
│   │
│   ├── lib/
│   │   ├── db.ts                   # Database client (Prisma)
│   │   ├── auth.ts                 # Auth config
│   │   ├── utils.ts                # Helper functions
│   │   └── validations.ts          # Zod schemas
│   │
│   ├── hooks/                      # Custom React hooks
│   ├── types/                      # TypeScript types/interfaces
│   ├── actions/                    # Server Actions
│   └── services/                   # Business logic layer
│
├── prisma/
│   └── schema.prisma               # Database schema
├── public/                          # Static assets
├── .env.local                       # Environment variables
└── next.config.ts
```

---

## 2. SERVER COMPONENTS vs CLIENT COMPONENTS

### 2.1 Aturan Default
- **Semua komponen adalah Server Component** secara default.
- Tambahkan `'use client'` **hanya** jika komponen membutuhkan: `useState`, `useEffect`, event handlers (`onClick`, `onChange`), atau browser API.

### 2.2 Pola Komposisi
```tsx
// Server Component (default) - fetch data di sini
export default async function DashboardPage() {
  const data = await getItems();  // langsung fetch di server
  return <ItemList items={data} />;
}

// Client Component - interaktivitas di sini
'use client'
export function ItemList({ items }: { items: Item[] }) {
  const [search, setSearch] = useState('');
  // ... interactive logic
}
```

### 2.3 Jangan Fetch di Client Jika Bisa di Server
```tsx
// ✅ BENAR: Fetch di Server Component
export default async function Page() {
  const items = await db.item.findMany();
  return <ItemTable items={items} />;
}

// ❌ SALAH: Fetch di Client Component tanpa alasan
'use client'
export default function Page() {
  const [items, setItems] = useState([]);
  useEffect(() => { fetch('/api/items')... }, []);
}
```

---

## 3. SERVER ACTIONS

### 3.1 Definisi Server Action
```tsx
// src/actions/item.ts
'use server'

import { revalidatePath } from 'next/cache';
import { db } from '@/lib/db';
import { itemSchema } from '@/lib/validations';

export async function createItem(formData: FormData) {
  const parsed = itemSchema.safeParse({
    name: formData.get('name'),
    price: Number(formData.get('price')),
  });

  if (!parsed.success) {
    return { error: parsed.error.flatten().fieldErrors };
  }

  await db.item.create({ data: parsed.data });
  revalidatePath('/dashboard/items');
  return { success: true };
}
```

### 3.2 Penggunaan di Form
```tsx
'use client'
import { createItem } from '@/actions/item';
import { useFormState, useFormStatus } from 'react-dom';

export function CreateItemForm() {
  const [state, formAction] = useFormState(createItem, null);

  return (
    <form action={formAction}>
      <input name="name" required />
      {state?.error?.name && <p className="text-red-500">{state.error.name}</p>}
      <SubmitButton />
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Menyimpan...' : 'Simpan'}</button>;
}
```

---

## 4. LOADING & ERROR UI

### 4.1 Loading UI (Automatic Suspense)
```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      <div className="h-8 bg-gray-200 rounded animate-pulse w-1/3" />
      <div className="h-64 bg-gray-200 rounded animate-pulse" />
    </div>
  );
}
```

### 4.2 Streaming dengan Suspense
```tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<TableSkeleton />}>
        <ItemTable />
      </Suspense>
      <Suspense fallback={<StatsSkeleton />}>
        <StatsCards />
      </Suspense>
    </div>
  );
}
```

### 4.3 Error Boundary
```tsx
// app/dashboard/error.tsx
'use client'
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="text-center py-10">
      <h2>Terjadi kesalahan</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Coba lagi</button>
    </div>
  );
}
```

---

## 5. DATA FETCHING & CACHING

### 5.1 Fetch dengan Revalidasi
```tsx
// Revalidate setiap 60 detik
const data = await fetch('https://api.example.com/items', {
  next: { revalidate: 60 },
});

// Atau pakai tag untuk invalidasi manual
const data = await fetch('...', { next: { tags: ['items'] } });

// Invalidasi via Server Action
import { revalidateTag } from 'next/cache';
revalidateTag('items');
```

### 5.2 Database Query (Prisma)
```tsx
// Langsung di Server Component
const items = await db.item.findMany({
  where: { isActive: true },
  orderBy: { createdAt: 'desc' },
  take: 20,
});
```

---

## 6. API ROUTES (Route Handlers)

```tsx
// app/api/webhook/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    // Proses webhook
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: 'Internal error' },
      { status: 500 }
    );
  }
}
```

---

## 7. VALIDASI (Zod)

```tsx
// src/lib/validations.ts
import { z } from 'zod';

export const itemSchema = z.object({
  name: z.string().min(1, 'Nama wajib diisi').max(100),
  price: z.number().int().positive('Harga harus positif'),
  isActive: z.boolean().default(true),
  tags: z.array(z.string()).optional(),
});

export type ItemInput = z.infer<typeof itemSchema>;
```

---

## 8. MIDDLEWARE (Auth Protection)

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('session');
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*'],
};
```

---

## 9. PAGINATION (Client-Side Fetch)

```tsx
'use client'
import { useState, useEffect } from 'react';

export function ItemTable() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [meta, setMeta] = useState({ total: 0, lastPage: 1 });
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/items?page=${page}&per_page=25`)
      .then(res => res.json())
      .then(data => {
        setItems(data.data);
        setMeta(data.meta);
      })
      .finally(() => setLoading(false));
  }, [page]);

  return (
    <div className="relative">
      {loading && <SpinnerOverlay />}
      <table>...</table>
      <Pagination page={page} lastPage={meta.lastPage} onChange={setPage} />
    </div>
  );
}
```

---

## 10. ENVIRONMENT VARIABLES

```env
# .env.local
DATABASE_URL="postgresql://..."
NEXTAUTH_SECRET="random-secret"
NEXTAUTH_URL="http://localhost:3000"
API_KEY="your-api-key"
```

- Prefix `NEXT_PUBLIC_` untuk variabel yang diakses di client.
- Variabel tanpa prefix hanya tersedia di server.
- **Jangan** commit `.env.local` ke repository.
