# PANDUAN PENULISAN KODE — Laravel API + Vue.js (SPA / Inertia)

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek berbasis **Laravel** sebagai backend dengan **Vue.js** sebagai frontend (bisa SPA terpisah atau monolith via Inertia).

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Backend | Laravel 11/12 (PHP 8.2+) |
| Frontend | Vue 3 (Composition API) |
| Bridge (Opsional) | Inertia.js (monolith) |
| Styling | Tailwind CSS |
| State | Pinia (SPA) / Inertia Shared Props (monolith) |
| Form | Inertia `useForm` / VeeValidate + Zod |
| HTTP Client | Axios |
| Build Tool | Vite |
| Auth | Laravel Sanctum (SPA) / Session (Inertia) |

---

## 1. STRUKTUR FOLDER (Monolith Inertia)

```
project-root/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   │   └── HandleInertiaRequests.php
│   │   └── Requests/               # Form Request Validation
│   ├── Models/
│   ├── Services/
│   └── Jobs/
│
├── resources/
│   ├── js/
│   │   ├── Pages/                  # Halaman Inertia (PascalCase)
│   │   │   ├── Auth/
│   │   │   ├── Admin/
│   │   │   └── Guest/
│   │   ├── Components/
│   │   │   ├── Admin/
│   │   │   ├── Guest/
│   │   │   └── Shared/
│   │   ├── Composables/            # Custom composables (hooks)
│   │   ├── Layouts/
│   │   ├── Types/                  # TypeScript interfaces
│   │   └── app.ts                  # Bootstrap
│   └── views/
│       └── app.blade.php
│
├── routes/
│   ├── web.php
│   └── api.php
└── package.json
```

---

## 2. VUE 3 COMPOSITION API

### 2.1 Script Setup (Wajib)
```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

interface Props {
  title: string;
  items: Item[];
}

const props = defineProps<Props>();
const emit = defineEmits<{
  (e: 'select', id: number): void;
}>();

const search = ref('');
const filtered = computed(() =>
  props.items.filter(i => i.name.toLowerCase().includes(search.value.toLowerCase()))
);
</script>

<template>
  <div>
    <input v-model="search" placeholder="Cari..." />
    <div v-for="item in filtered" :key="item.id" @click="emit('select', item.id)">
      {{ item.name }}
    </div>
  </div>
</template>
```

### 2.2 Composables (Custom Hooks)
```typescript
// composables/useDebounce.ts
import { ref, watch } from 'vue';

export function useDebounce<T>(value: Ref<T>, delay = 300) {
  const debounced = ref(value.value) as Ref<T>;
  let timeout: ReturnType<typeof setTimeout>;

  watch(value, (newVal) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => { debounced.value = newVal; }, delay);
  });

  return debounced;
}
```

---

## 3. FORM HANDLING

### 3.1 Inertia useForm (Monolith)
```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3';

const form = useForm({
  name: '',
  price: 0,
  is_active: true,
});

function submit() {
  form.post(route('admin.items.store'), {
    onError: () => {
      // Toast error count
    },
  });
}
</script>

<template>
  <form @submit.prevent="submit">
    <input v-model="form.name" />
    <p v-if="form.errors.name" class="text-red-500">{{ form.errors.name }}</p>
    <button :disabled="form.processing">Simpan</button>
  </form>
</template>
```

### 3.2 VeeValidate + Zod (SPA)
```vue
<script setup lang="ts">
import { useForm } from 'vee-validate';
import { toTypedSchema } from '@vee-validate/zod';
import { z } from 'zod';

const schema = toTypedSchema(z.object({
  name: z.string().min(1, 'Nama wajib diisi'),
  price: z.number().positive('Harga harus positif'),
}));

const { handleSubmit, errors, defineField } = useForm({ validationSchema: schema });
const [name, nameAttrs] = defineField('name');
const [price, priceAttrs] = defineField('price');

const onSubmit = handleSubmit(async (values) => {
  await api.post('/items', values);
});
</script>
```

---

## 4. STATE MANAGEMENT (Pinia)

```typescript
// stores/auth.ts
import { defineStore } from 'pinia';

export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null as User | null,
    token: null as string | null,
  }),
  getters: {
    isAuthenticated: (state) => !!state.token,
  },
  actions: {
    async login(credentials: LoginDto) {
      const { data } = await api.post('/auth/login', credentials);
      this.user = data.user;
      this.token = data.token;
    },
    logout() {
      this.user = null;
      this.token = null;
    },
  },
  persist: true, // pinia-plugin-persistedstate
});
```

---

## 5. DATATABLE (AJAX PAGINATION)

```vue
<script setup lang="ts">
import { ref, watch } from 'vue';
import { useDebounce } from '@/composables/useDebounce';

const items = ref([]);
const meta = ref({ total: 0, lastPage: 1 });
const page = ref(1);
const search = ref('');
const loading = ref(false);
const debouncedSearch = useDebounce(search);

async function fetchData() {
  loading.value = true;
  const { data } = await api.get('/items', {
    params: { page: page.value, per_page: 25, search: debouncedSearch.value },
  });
  items.value = data.data;
  meta.value = data.meta;
  loading.value = false;
}

watch([page, debouncedSearch], fetchData, { immediate: true });
</script>

<template>
  <div class="relative">
    <input v-model="search" placeholder="Cari..." />

    <!-- Spinner overlay -->
    <div v-if="loading" class="absolute inset-0 bg-black/50 flex items-center justify-center z-10">
      <div class="w-8 h-8 border-2 border-blue-500 border-t-transparent rounded-full animate-spin" />
    </div>

    <table>
      <tr v-for="item in items" :key="item.id">
        <td>{{ item.name }}</td>
      </tr>
    </table>

    <Pagination :page="page" :last-page="meta.lastPage" @change="page = $event" />
  </div>
</template>
```

---

## 6. LOADING UX

### 6.1 Skeleton Component
```vue
<template>
  <div class="space-y-3">
    <div v-for="i in rows" :key="i" class="h-12 bg-gray-200 rounded animate-pulse" />
  </div>
</template>

<script setup lang="ts">
defineProps<{ rows?: number }>();
</script>
```

### 6.2 Image Skeleton
```vue
<template>
  <div class="relative" :class="className">
    <div v-if="!loaded" class="absolute inset-0 bg-gray-200 animate-pulse rounded" />
    <img :src="src" :alt="alt" @load="loaded = true"
      :class="['transition-opacity duration-300', loaded ? 'opacity-100' : 'opacity-0']" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
defineProps<{ src: string; alt: string; className?: string }>();
const loaded = ref(false);
</script>
```

---

## 7. LARAVEL BACKEND (Ringkasan)

### 7.1 Slim Controller + Service
```php
// Controller hanya panggil service
public function index(): \Illuminate\Http\JsonResponse
{
    $data = $this->itemService->getAll(request());
    return response()->json($data);
}

public function store(ItemRequest $request): \Illuminate\Http\JsonResponse
{
    $item = $this->itemService->create($request->validated());
    return response()->json(['success' => true, 'data' => $item], 201);
}
```

### 7.2 Sanctum Auth (SPA)
```php
// routes/api.php
Route::post('/auth/login', [AuthController::class, 'login']);
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('items', ItemController::class);
});
```

### 7.3 CORS (SPA terpisah)
```php
// config/cors.php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:5173')],
'supports_credentials' => true,
```

---

## 8. ENVIRONMENT

```env
# Laravel .env
SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost
FRONTEND_URL=http://localhost:5173

# Vue .env
VITE_API_URL=http://localhost:8000/api
```
