# PANDUAN PENULISAN KODE — PHP Native + MySQL + Bootstrap

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek berbasis **PHP Native** (tanpa framework) dengan **MySQL/MariaDB** dan **Bootstrap** sebagai UI framework.

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Backend | PHP Native (8.x+) |
| Database | MySQL / MariaDB (MySQLi Extension) |
| Frontend | HTML + CSS + JavaScript |
| UI Framework | Bootstrap 5 |
| Hosting Target | cPanel Shared Hosting |
| Auth | Session-based (`$_SESSION`) |

---

## 1. STRUKTUR FOLDER PROJECT

```
project-root/
├── dashboard/
│   ├── admin.php               # Router utama + layout sidebar
│   └── pages/                  # Partial halaman (di-include oleh admin.php)
│       ├── dashboard.php
│       ├── data_master.php
│       ├── transaksi.php
│       ├── profile.php
│       ├── css.php             # Semua <link> CSS dikumpulkan
│       └── js.php              # Semua <script> JS dikumpulkan
│
├── functions/
│   ├── koneksi.php             # Koneksi MySQLi
│   ├── function_auth.php       # Login, logout, register
│   ├── function_master.php     # CRUD data master
│   ├── function_transaksi.php  # Logika transaksi
│   ├── log_akses.php           # Pencatatan log akses
│   └── data.php                # Data statis / lookup array
│
├── auth/                       # Halaman login, logout, register
├── db/                         # File SQL schema / migration manual
├── uploads/                    # Folder upload file (dibuat otomatis)
├── assets/
│   ├── css/
│   ├── js/
│   └── img/
└── laporan/                    # Generate laporan (opsional)
```

---

## 2. POLA ROUTER (`admin.php`)

`admin.php` adalah single-file router yang meng-include halaman via parameter `$page`:

```php
<?php
session_start();
require_once '../functions/koneksi.php';

// Cek login
if (empty($_SESSION['sesi_id'])) {
    header('location: ../auth/logout.php');
    exit;
}

$page = $_GET['page'] ?? 'dashboard';

// Layout header + sidebar
include 'pages/css.php';
?>
<!-- Sidebar & Navbar HTML di sini -->
<?php
switch ($page) {
    case 'data master':    include 'pages/data_master.php'; break;
    case 'transaksi':      include 'pages/transaksi.php'; break;
    case 'profile':        include 'pages/profile.php'; break;
    default:               include 'pages/dashboard.php';
}
include 'pages/js.php';
?>
```

**Aturan:**
- Tambah halaman baru = tambah `case` + buat file di `pages/`.
- Jangan buat file PHP terpisah di luar pola ini untuk halaman admin.
- Link navigasi sidebar: `?page=nama halaman`.

---

## 3. POLA FUNCTIONS

### 3.1 Koneksi Database
```php
<?php
$koneksi = new mysqli('localhost', 'root', '', 'nama_db');
if ($koneksi->connect_error) {
    die('Koneksi gagal: ' . $koneksi->connect_error);
}
$koneksi->set_charset('utf8mb4');
```

### 3.2 Prepared Statement (WAJIB)
```php
$stmt = $koneksi->prepare("SELECT * FROM users WHERE id_user = ?");
$stmt->bind_param('s', $id_user);
$stmt->execute();
$result = $stmt->get_result();
$data = $result->fetch_assoc();
$stmt->close();
```
- Tipe binding: `'i'` = integer, `'s'` = string, `'d'` = double.
- **JANGAN** pernah concatenate input user langsung ke query SQL.

### 3.3 File Function
- Setiap function file di-`require_once` dari halaman/controller yang butuh.
- Gunakan `session_start()` di paling atas jika butuh `$_SESSION`.
- **Jangan** tulis query langsung di halaman `pages/*.php` — selalu taruh di `functions/`.

---

## 4. SESSION & AUTENTIKASI

### 4.1 Session Keys Standar
```php
$_SESSION['sesi_id']    // ID user (VARCHAR, contoh: "USER001")
$_SESSION['sesi_nama']  // Nama user
$_SESSION['sesi_role']  // Role user (admin, operator, dll)
```

### 4.2 Cek Login di Setiap Halaman
```php
if (empty($_SESSION['sesi_id'])) {
    header('location: ../auth/logout.php');
    exit;
}
```

### 4.3 PRG Pattern (Post-Redirect-Get)
```php
// Setelah proses POST berhasil, SELALU redirect:
header('Location: ../dashboard/admin.php?page=data master');
exit;
```
**Jangan** tampilkan response langsung setelah POST — selalu redirect.

---

## 5. UPLOAD FILE

- Folder tujuan: `uploads/{kategori}/` (relatif dari root project).
- Ekstensi diizinkan: whitelist (`['jpg', 'jpeg', 'png', 'pdf']`).
- Ukuran maksimal: definisikan di konstanta (contoh: `2 * 1024 * 1024` = 2MB).
- Nama file: prefix + `time()` + `.ext` (unik, tidak bentrok).
- Buat folder jika belum ada: `mkdir($dir, 0777, true)`.

```php
$ext = strtolower(pathinfo($_FILES['foto']['name'], PATHINFO_EXTENSION));
$allowed = ['jpg', 'jpeg', 'png'];
if (!in_array($ext, $allowed)) {
    $_SESSION['error'] = 'Format file tidak didukung.';
    header('Location: ...');
    exit;
}
$filename = 'item_' . time() . '.' . $ext;
move_uploaded_file($_FILES['foto']['tmp_name'], $uploadDir . $filename);
```

---

## 6. PENANGANAN ERROR

- Error dikemas ke `$_SESSION` lalu redirect, **bukan** `die()` atau `exit` tanpa redirect.
```php
$_SESSION['error'] = 'Pesan error yang informatif.';
header('Location: ../dashboard/admin.php?page=data master');
exit;
```
- Sukses juga via session:
```php
$_SESSION['success'] = 'Data berhasil disimpan.';
header('Location: ...');
exit;
```
- Tampilkan notifikasi di halaman tujuan:
```php
<?php if (!empty($_SESSION['success'])): ?>
  <div class="alert alert-success"><?= $_SESSION['success'] ?></div>
  <?php unset($_SESSION['success']); ?>
<?php endif; ?>
```

---

## 7. JAVASCRIPT (Vanilla / jQuery)

### 7.1 AJAX Pagination
```javascript
function loadData(page = 1) {
  const search = document.getElementById('search').value;
  fetch(`../functions/function_master.php?action=get_data&page=${page}&search=${search}`)
    .then(res => res.json())
    .then(data => {
      renderTable(data.rows);
      renderPagination(data.total_pages, page);
    });
}
```

### 7.2 Konfirmasi Delete
```javascript
function hapusData(id) {
  if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
    window.location.href = `../functions/function_master.php?action=delete&id=${id}`;
  }
}
```

### 7.3 DataTables (Opsional)
```html
<script src="https://cdn.datatables.net/1.x/js/jquery.dataTables.min.js"></script>
<script>
  $(document).ready(function() {
    $('#tabelData').DataTable({
      processing: true,
      serverSide: true,
      ajax: '../functions/function_master.php?action=datatables',
    });
  });
</script>
```

---

## 8. KONVENSI DATABASE

- Nama tabel: `snake_case` (contoh: `data_master`, `log_transaksi`).
- Nama kolom: `snake_case`.
- Primary key: `id` atau `id_{nama_tabel}` (contoh: `id_user`).
- Timestamp: `created_at`, `updated_at` (format `DATETIME`).
- Soft delete: kolom `deleted_at` (nullable `DATETIME`).
- Engine: `InnoDB` (untuk foreign key support).
- Charset: `utf8mb4`.

---

## 9. HOSTING (cPanel)

- Upload via FTP / File Manager / Git Pull.
- Pastikan `functions/koneksi.php` menggunakan kredensial produksi dari cPanel MySQL.
- Folder `uploads/` harus writable (chmod 755 atau 777).
- File `.htaccess` untuk proteksi folder sensitif:
```apache
# Di dalam folder functions/
<Files "*.php">
    Order Deny,Allow
    Deny from all
</Files>
```
