# PANDUAN PENULISAN KODE — UNIVERSAL

Dokumen ini berisi standar dan konvensi penulisan kode (coding style) yang wajib dipatuhi oleh AI, **berlaku untuk semua tech stack** (PHP, Node.js, Python, Go, dll). Prinsip-prinsip di bawah bersifat fundamental dan tidak terikat pada framework tertentu.

---

## 1. ARSITEKTUR & POLA KODE

### 1.1 Slim Controller / Handler
- Controller/handler hanya bertugas: menerima request, memvalidasi input, memanggil service/use-case, dan mengembalikan response.
- **Jangan** tulis logika bisnis, query database kompleks, atau integrasi API langsung di dalam controller/handler.
- Delegasikan ke layer Service / Use Case / Repository.

### 1.2 Service Layer (Business Logic)
- Seluruh logika bisnis kompleks, integrasi API eksternal, dan query database multi-tabel harus ditulis di layer terpisah (Service, Use Case, atau Repository).
- Gunakan *Dependency Injection* untuk memanggil service lain, bukan instansiasi langsung (`new ServiceX()`).

### 1.3 Transaksi Database
- Untuk mutasi data yang melibatkan lebih dari satu tabel, **wajib** membungkus proses tersebut dalam transaksi database demi menjaga integritas data.
- Gunakan mekanisme transaksi bawaan framework/ORM (contoh: `DB::transaction`, `session.begin()`, `with conn.begin()`, dsb).
- Selalu sediakan mekanisme rollback jika terjadi error di tengah proses.

### 1.4 Caching
- Cache data statis atau konfigurasi yang sering diakses untuk menghemat query database (contoh: daftar kategori, pengaturan website, menu navigasi).
- Tentukan TTL (Time-To-Live) yang wajar dan pastikan cache di-flush saat data sumbernya berubah (cache invalidation).

### 1.5 Background Jobs / Queue
- Proses berat seperti pengiriman email, notifikasi, generate laporan, dan integrasi API eksternal **wajib** diproses secara asinkron melalui mekanisme queue/job agar response time tetap cepat.
- Jangan blokir request lifecycle pengguna dengan proses yang bisa di-*defer*.

---

## 2. PENULISAN KODE & KONVENSI

### 2.1 Penamaan (Naming Convention)
- **Kolom database / variabel backend:** `snake_case` (contoh: `trx_number`, `is_active`, `created_at`).
- **Fungsi/method:** `camelCase` atau `snake_case` sesuai konvensi bahasa (PHP/JS: `camelCase`, Python: `snake_case`).
- **Komponen UI / Class:** `PascalCase` (contoh: `DataTable.tsx`, `OrderService.php`).
- **Konstanta:** `UPPER_SNAKE_CASE` (contoh: `MAX_UPLOAD_SIZE`, `API_BASE_URL`).

### 2.2 Dokumentasi Function (Docblock Wajib)
- Setiap function/method baru **wajib** diberi dokumentasi (docblock/docstring) yang menjelaskan:
  - Tujuan function (1 kalimat ringkas).
  - Parameter input (tipe dan keterangan singkat).
  - Return value (tipe dan keterangan singkat).
- Untuk komponen frontend, tambahkan komentar singkat `//` di atas function non-trivial.

### 2.3 Tipe Data Eksplisit
- Hindari penggunaan tipe dinamis/implisit (`any`, `mixed`, `var`) sebisa mungkin.
- Definisikan interface/type/model secara eksplisit untuk setiap struktur data yang digunakan.
- Di ORM/Model, definisikan casting/serialisasi tipe data secara eksplisit (contoh: `boolean`, `array`, `integer`, `datetime`).

### 2.4 Jangan Hardcode
- Nilai yang bisa berubah (URL API, batas ukuran file, kredensial, konfigurasi) **wajib** disimpan di environment variable (`.env`) atau file konfigurasi, bukan ditulis langsung di dalam kode logika.

---

## 3. FRONTEND & UX

### 3.1 Skeleton / Shimmer Loading
- Gunakan **Skeleton / Shimmer UI** sebagai placeholder visual saat data sedang dimuat secara asinkron, untuk menghindari *layout shifting* dan menjaga *perceived performance*.
- Desain skeleton harus mendekati bentuk visual asli data (tabel, card, statistik).
- **Hindari** spinner melingkar penuh sebagai satu-satunya loading indicator pada konten utama halaman.

### 3.2 Lazy Loading
- Komponen/halaman yang berat atau jarang diakses harus di-lazy load (contoh: `React.lazy()`, dynamic `import()`, `defineAsyncComponent`).
- Gambar/media yang berada di bawah viewport harus menggunakan `loading="lazy"` atau *Intersection Observer*.

### 3.3 Pagination (AJAX/Async)
- Halaman dengan data list/tabel berjumlah besar **wajib** menggunakan pagination berbasis AJAX/fetch, bukan full-page reload.
- Format response pagination standar:
  ```json
  {
    "data": [...],
    "meta": {
      "total": 150,
      "per_page": 25,
      "current_page": 2,
      "last_page": 6
    }
  }
  ```
- Sediakan opsi jumlah per halaman: `10`, `25`, `50`, `100`.
- Sertakan fitur pencarian/filter yang tidak memicu reload halaman penuh.

### 3.4 Spinner Overlay (Tabel / Partial Loading)
- Saat tabel/data sedang di-fetch ulang (pagination, filter, search), tampilkan spinner overlay **hanya di area tabel/konten tersebut**, bukan memblokir seluruh halaman.

### 3.5 Responsive Design
- Semua halaman harus responsif (mobile-first atau adaptive).
- Tabel: gunakan horizontal scroll (`overflow-x-auto`) di layar kecil.
- Form: format kolom tunggal di mobile (<768px), multi-kolom di desktop.
- Sidebar/navigasi: berubah menjadi drawer/hamburger di mobile.

### 3.6 Image Skeleton
- Untuk elemen gambar/video, tampilkan placeholder animasi (pulse/shimmer) hingga media selesai dimuat (`onLoad`), lalu tampilkan media asli dengan *fade-in transition*.

---

## 4. VALIDASI & PENANGANAN ERROR

### 4.1 Validasi Input
- Validasi data input mutasi **wajib** dilakukan di sisi server (Form Request, Schema Validation, Middleware, dsb).
- Tampilkan pesan error validasi secara **inline** di bawah setiap field yang bermasalah, bukan dalam modal/popup terpisah.
- Tambahkan notifikasi toast ringkas sebagai *summary* jumlah error (opsional).

### 4.2 Standard JSON Response (API)
- Untuk semua API endpoint (REST/webhook), gunakan format JSON standar yang konsisten:
  ```json
  {
    "success": true,
    "message": "Pesan informatif.",
    "data": { ... }
  }
  ```
  Untuk error:
  ```json
  {
    "success": false,
    "message": "Pesan error yang informatif.",
    "error": "Detail kesalahan (opsional)"
  }
  ```

### 4.3 Try-Catch pada Integrasi Eksternal
- Seluruh pemanggilan API eksternal (payment gateway, email service, storage, dll) **wajib** dibungkus dalam blok try-catch.
- Catat kegagalan ke log sistem dengan detail yang cukup untuk debugging.
- Jangan pernah menampilkan stack trace atau detail internal error ke pengguna akhir.

### 4.4 PRG Pattern (Post-Redirect-Get)
- Untuk form submission berbasis server-rendered (bukan SPA), selalu gunakan pola PRG: setelah proses POST berhasil, redirect ke halaman tujuan (GET) agar form tidak ter-submit ulang saat user refresh.

---

## 5. DATABASE & MODEL

### 5.1 Prepared Statements / Parameterized Queries
- Semua query yang menerima input dari pengguna **wajib** menggunakan prepared statement / parameterized query untuk mencegah SQL Injection.
- **Jangan** pernah melakukan string concatenation langsung ke dalam query SQL.

### 5.2 Soft Delete
- Untuk data penting (transaksi, user, produk), pertimbangkan penggunaan *soft delete* (`deleted_at`) alih-alih penghapusan permanen.
- Sediakan halaman/fitur *Recycle Bin* terpusat untuk melihat dan memulihkan data yang dihapus.

### 5.3 Migration & Seeder
- Semua perubahan skema database harus ditulis dalam file *migration*, bukan dijalankan langsung via SQL manual.
- Sediakan *seeder* untuk data awal/default yang dibutuhkan aplikasi saat pertama kali dijalankan.

---

## 6. KEAMANAN

### 6.1 Autentikasi & Session
- Cek status login di setiap halaman/endpoint yang membutuhkan proteksi.
- Jangan simpan data sensitif (password, token) di local storage atau cookie tanpa enkripsi.
- Gunakan mekanisme CSRF protection bawaan framework untuk setiap form/request mutasi.

### 6.2 Upload File
- Batasi ekstensi file yang diizinkan (whitelist, bukan blacklist).
- Batasi ukuran file maksimal.
- Berikan nama file unik (timestamp/UUID) untuk menghindari konflik dan directory traversal.
- Validasi MIME type di sisi server, jangan hanya mengandalkan ekstensi file.

### 6.3 Environment & Secrets
- File `.env` **tidak boleh** di-commit ke repository (pastikan tercantum di `.gitignore`).
- Dokumentasikan semua env key yang dibutuhkan di file `.env.example`.

---

## 7. KOMENTAR & BAHASA

- Gunakan **Bahasa Indonesia** untuk komentar logika bisnis dan penamaan variabel domain-specific.
- Gunakan **Bahasa Inggris** untuk komentar teknis, nama function/method, dan komentar di library/utility.
- Jangan tinggalkan komentar `TODO` atau `FIXME` tanpa keterangan yang jelas.
