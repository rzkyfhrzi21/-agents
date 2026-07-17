🪐 YUBASHOP - Database & API Endpoints Blueprint Spec v1.0

Spesifikasi ini dirancang khusus untuk integrasi Laravel 12 (Backend) + Inertia.js & React (Frontend) dengan standar keamanan industri dan performa basis data yang optimal.

🛠️ Aturan Global & Arsitektur Sistem

1. Sistem Autentikasi (Laravel Session & Web Middleware)

Backend Layer: Menggunakan Laravel Session & Web Middleware. Status login dikelola secara stateful oleh server.

Frontend Layer (Inertia React): Status autentikasi dan data user aktif dibagikan langsung dari backend via Inertia Shared Props (`auth.user`) dan dapat diakses dengan hook `usePage().props`.

Proteksi Route: Halaman dan endpoint admin dilindungi oleh middleware `auth` bawaan Laravel di `routes/web.php`.

2. Standar Pagination Dinamis (Eloquent Model & Base Controller)

Setiap model Eloquent diatur default $perPage = 10. Pada controller, penanganan pagination diimplementasikan menggunakan logika dinamis berikut:

$perPage = $request->query('per_page', $model->getPerPage());
$perPage = min((int)$perPage, 50); // Maksimal limit dibatasi 50


3. Skema Routing & Penamaan Halaman (Inertia Web Routes)

Backend & Routing (Laravel): Menggunakan RESTful standard web routes via Route::resource('nama-modul', Controller::class).

Frontend Pages (Inertia): File UI React diletakkan di bawah `resources/js/Pages/Admin/` dan `resources/js/Pages/Guest/`.

Keterangan Kode CRUDTZB pada Modul:

C (Create): Memerlukan form input (halaman Inertia Create / modal), data dikirim via POST request Inertia ke `/admin/{resource}`.

R (Read): Mengambil list data (halaman Index dengan pagination ter-render), atau filter dinamis menggunakan Inertia query parameters.

U (Update): Memerlukan form edit (halaman Inertia Edit / modal), data dikirim via PUT/PATCH request Inertia ke `/admin/{resource}/{id}`.

D (Delete): Menjalankan DELETE request Inertia ke `/admin/{resource}/{id}`.

Z (Detail): Halaman Inertia Show detail `/admin/{resource}/{id}`.

T (Toggle Status): Request POST/PATCH ke `/admin/{resource}/{id}/toggle-status` untuk toggle boolean (seperti `is_active` atau `status`).

B (Bulk Action): Request POST ke `/admin/{resource}/bulk-action` untuk memproses mass-data (contoh: hapus massal atau perubahan status massal).

📁 Bagian 1: Modul Konfigurasi (Direct Update Form Mode - Tanpa Datatable)

Halaman-halaman berikut tidak menampilkan Datatable, melainkan langsung menyajikan form update (U) ketika dibuka.

1. Modul Konfigurasi Website

Frontend Halaman: https://URL_WEBSITE/admin/config/website

Laravel API Resource: Route::apiResource('config-website', ConfigWebsiteController::class)->only(['index', 'update'])

Tipe Akses Form: GET (untuk memuat data), PUT/PATCH (untuk update data)

Table Name: CONFIG_WEBSITE

id : INT PK Auto-Increment

website_name : STRING (default: 'YUBASHOP')

title : STRING (default: 'WEBSITE PENYEDIA LAYANAN TOPUP GAME XXXX')

icon : STRING (Path icon file)

logo : STRING (Path logo file)

author : STRING (default: 'YUBASHOP')

keywords : TEXT (JSON keywords array)

desc : TEXT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'keywords' => 'array',
    ];
}


2. Modul Konfigurasi Pembelian

Frontend Halaman: https://URL_WEBSITE/admin/config/transaction

Laravel API Resource: Route::apiResource('config-transaction', ConfigTransactionController::class)->only(['index', 'update'])

Table Name: CONFIG_TRANSACTION

id : INT PK Auto-Increment

trx_prefix_id : STRING (4) (default: 'YUBA')

trx_start_id : INT (default: 1, format output string maks 5 karakter seperti '00001' s.d '99999')

trx_expired : INT (Minutes, default: 60)

trx_delay : INT (Seconds, default: 5)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'trx_start_id' => 'integer',
        'trx_expired' => 'integer',
        'trx_delay' => 'integer',
    ];
}


3. Modul Konfigurasi Produk

Frontend Halaman: https://URL_WEBSITE/admin/product/config

Laravel API Resource: Route::apiResource('config-product', ConfigProductController::class)->only(['index', 'update'])

Table Name: CONFIG_PRODUCT

id : INT PK Auto-Increment

product_limit : BOOLEAN (default: true)

start_display_product_desktop : INT (default: 12)

more_display_product_desktop : INT (default: 6)

start_display_product_mobile : INT (default: 6)

more_display_product_mobile : INT (default: 4)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'product_limit' => 'boolean',
        'start_display_product_desktop' => 'integer',
        'more_display_product_desktop' => 'integer',
        'start_display_product_mobile' => 'integer',
        'more_display_product_mobile' => 'integer',
    ];
}


4. Modul Konfigurasi WA

Frontend Halaman: https://URL_WEBSITE/admin/config/whatsapp

Laravel API Resource: Route::apiResource('config-whatsapp', ConfigWhatsappController::class)->only(['index', 'update'])

Table Name: CONFIG_WHATSAPP

id : INT PK Auto-Increment

provider : STRING (default: 'Fonnte')

device_name : STRING (default: 'Admin Yuba 1')

url : STRING

token : STRING

status : ENUM ('aktif', 'tidak') (default: 'tidak')

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts: None (Casts standar string/enum)

5. Modul Konfigurasi SMTP

Frontend Halaman: https://URL_WEBSITE/admin/smtp

Laravel API Resource: Route::apiResource('config-smtp', ConfigSmtpController::class)->only(['index', 'update'])

Table Name: CONFIG_SMTP

id : INT PK Auto-Increment

smtp_name : STRING

smtp_host : STRING

smtp_port : INT

smtp_username : STRING

smtp_password : STRING (Terenkripsi)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'smtp_password' => 'encrypted',
        'smtp_port' => 'integer',
    ];
}


6. Modul Konfigurasi Popup

Frontend Halaman: https://URL_WEBSITE/admin/config/popup

Laravel API Resource: Route::apiResource('config-popup-setting', ConfigPopupSettingController::class)->only(['index', 'update'])

Table Name: CONFIG_POPUP

id : INT PK Auto-Increment

size : ENUM ('Small', 'Medium', 'Large', 'Xtra Large')

btn_is_read : BOOLEAN (default: false)

duration_is_read : INT (Minutes, default: 60)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'btn_is_read' => 'boolean',
        'duration_is_read' => 'integer',
    ];
}


📁 Bagian 2: Modul Utama & Kelola Data (Dengan Datatable & API Lengkap)

7. Modul Login Admin

Frontend Halaman: https://URL_WEBSITE/login

Laravel API: POST /api/v1/login (Auth Session Sanctum), POST /api/v1/logout

Table Name: Menggunakan tabel USERS untuk validasi kredensial.

8. Modul Catatan Aktivitas (R)

Frontend Halaman: https://URL_WEBSITE/admin/logs

Laravel API: Route::get('logs', ActivityLogsController::class)->middleware('auth')

Table Name: ACTIVITY_LOGS

id : INT PK Auto-Increment

user_id : INT FK (Relasi ke USERS.id)

ip_address : STRING (45)

action : STRING (255)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-activity-logs

9. Modul Kelola Akun (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/users

Laravel API Resource: Route::apiResource('users', UserController::class)->middleware('auth')

Table Name: USERS

id : INT PK Auto-Increment

name : STRING (50)

email : STRING (50) -> UNIQUE

no_whatsapp : STRING (20)

password : STRING (255)

address : TEXT

role_id : INT FK (Relasi ke ROLES.id)

status : ENUM ('aktif', 'tidak') (default: 'aktif')

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'password' => 'hashed',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-users

C (Create): Page /admin/users/add -> API POST /api/v1/users

U (Update): Page /admin/users/edit/{id} -> API PUT /api/v1/users/{id}

D (Delete): API DELETE /api/v1/users/{id}

T (Toggle): POST /api/v1/users/toggle-status

10. Modul Kelola Profile (U)

Frontend Halaman: https://URL_WEBSITE/admin/profile

Laravel API: GET /api/v1/profile (Fetch data user aktif), PUT /api/v1/profile (Update data diri)

Table Name: Menggunakan relasi tabel USERS untuk data update profile. No Datatable.

11. Modul Kelola Roles (CRUDZ)

Frontend Halaman: https://URL_WEBSITE/admin/level

Laravel API Resource: Route::apiResource('roles', RoleController::class)->middleware('auth')

Table Name: ROLES

id : INT PK Auto-Increment

name : STRING (25)

img : STRING (255) (Logo/Avatar role)

point : INT (1: superadmin, 2: admin)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-roles

C (Create): Page /admin/level/add -> API POST /api/v1/roles

U (Update): Page /admin/level/edit/{id} -> API PUT /api/v1/roles/{id}

D (Delete): API DELETE /api/v1/roles/{id}

Z (Detail): Page /admin/level/detail/{id} -> API GET /api/v1/roles/{id}

12. Modul Riwayat Saldo (R)

Frontend Halaman: https://URL_WEBSITE/admin/balance

Laravel API: Route::get('balances', BalanceController::class)->middleware('auth')

Table Name: BALANCES

id : INT PK Auto-Increment

user_id : INT FK (Relasi ke USERS.id)

keterangan : STRING (255)

beginning_balance : DECIMAL (12, 2)

amount : DECIMAL (12, 2)

ending_balance : DECIMAL (12, 2)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-balance

13. Modul Data Transaksi (CRUDZ)

Frontend Halaman: https://URL_WEBSITE/admin/transaction

Laravel API Resource: Route::apiResource('transactions', TransactionController::class) (Guest checkout tidak menggunakan middleware auth saat POST, tetapi admin operations require auth)

Table Name: TRANSACTIONS

id : INT PK Auto-Increment

trx_number : STRING (11) -> UNIQUE (Skema: YUBA + 00001 = YUBA00001)

product_id : INT FK (Relasi ke PRODUCTS.id)

total_price : DECIMAL (12, 2)

purchase_status : ENUM ('Pending', 'Canceled', 'Processing', 'Success') (default: 'Pending')

payment_status : ENUM ('Paid', 'Unpaid') (default: 'Unpaid')

payment_method_id : INT FK (Relasi ke PAYMENT_METHODS.id)

voucher_id : INT FK Nullable (Relasi ke vouchers.id)

notes : STRING (255) Nullable

tax : DECIMAL (12, 2) (default: 0)

snap_token : STRING (255) Nullable

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'total_price' => 'decimal:2',
        'tax' => 'decimal:2',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-transaction

C (Create): Endpoint checkout publik -> POST /api/v1/transactions

U (Update): Page /admin/transaction/edit/{id} -> API PUT /api/v1/transactions/{id}

D (Delete): API DELETE /api/v1/transactions/{id}

Z (Detail): Page /admin/transaction/detail/{id} -> API GET /api/v1/transactions/{id}

14. Modul Review Transaksi (CRUDZ)

Frontend Halaman: https://URL_WEBSITE/admin/transaction/review

Laravel API Resource: Route::apiResource('transaction-reviews', TransactionReviewController::class)

Table Name: TRANSACTION_REVIEWS

id : INT PK Auto-Increment

trx_number : STRING (11) FK (Relasi ke TRANSACTIONS.trx_number)

rating : ENUM ('1 star', '2 star', '3 star', '4 star', '5 star')

message : TEXT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-transaction-review

C (Create): POST /api/v1/transaction-reviews

U (Update): Page /admin/transaction/review/edit/{id} -> API PUT /api/v1/transaction-reviews/{id}

D (Delete): API DELETE /api/v1/transaction-reviews/{id}

Z (Detail): Page /admin/transaction/review/detail/{id} -> API GET /api/v1/transaction-reviews/{id}

15. Modul Laporan Pembelian (R)

Frontend Halaman: https://URL_WEBSITE/admin/transaction/report

Laravel API: GET /api/v1/transaction/report (Ekspor/ambil semua data transaksi terfilter)

Table Name: Membaca data agregat dari tabel TRANSACTIONS

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-transaction-report

16. Modul Konfigurasi Transaction Contact (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/config/transaction-contact

Laravel API Resource: Route::apiResource('config-transaction-contact', ConfigTransactionContactController::class)->middleware('auth')

Table Name: CONFIG_TRANSACTION_CONTACT

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

title : STRING (100) (Contoh: "Form Validasi Data Mobile Legends")

content : TEXT

img : STRING Nullable

separator : STRING (default: '-')

value : TEXT (JSON input mapping fields)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'value' => 'array',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-config-transaction-contact

C (Create): Page /admin/config/transaction-contact/add -> API POST /api/v1/config-transaction-contact

U (Update): Page /admin/config/transaction-contact/edit/{id} -> API PUT /api/v1/config-transaction-contact/{id}

D (Delete): API DELETE /api/v1/config-transaction-contact/{id}

17. Modul Kontak Buyer Transaksi (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/config/transaction-contact (Detail Tab)

Laravel API Resource: Route::apiResource('transaction-contacts', TransactionContactController::class)

Table Name: TRANSACTION_CONTACTS

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

trx_number : STRING (11) FK (Relasi ke TRANSACTIONS.trx_number)

value : TEXT (JSON array values user input form contact)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'value' => 'array',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-transaction-contacts

C (Create): POST /api/v1/transaction-contacts

U (Update): API PUT /api/v1/transaction-contacts/{id}

D (Delete): API DELETE /api/v1/transaction-contacts/{id}

18. Modul Kelola Games (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/games

Laravel API Resource: Route::apiResource('games', GameController::class)

Table Name: GAMES

id : INT PK Auto-Increment

name : STRING (50)

sub_name : STRING (50)

img : STRING (Ratio 1:1)

banner : STRING (Ratio 3:1)

slug : STRING (55) -> UNIQUE

id_config_transaction_contact : INT FK Nullable (Relasi ke CONFIG_TRANSACTION_CONTACT.id)

tag_games : TEXT (JSON tags, contoh: ["Layanan Resmi", "Proses Instan"])

desc : TEXT

is_popular : BOOLEAN (default: false)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'tag_games' => 'array',
        'is_popular' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-games

C (Create): Page /admin/games/add -> API POST /api/v1/games

U (Update): Page /admin/games/edit/{id} -> API PUT /api/v1/games/{id}

D (Delete): API DELETE /api/v1/games/{id}

T (Toggle): POST /api/v1/games/toggle-status

19. Modul Kelola Kategori (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/games/category

Laravel API Resource: Route::apiResource('game-categories', GameCategoryController::class)

Table Name: GAME_CATEGORIES

id : INT PK Auto-Increment

name : ENUM ('Topup', 'Jual Akun', 'Joki Akun')

queue_number : INT (default: 1)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-game-categories

C (Create): Page /admin/games/category/add -> API POST /api/v1/game-categories

U (Update): Page /admin/games/category/edit/{id} -> API PUT /api/v1/game-categories/{id}

D (Delete): API DELETE /api/v1/game-categories/{id}

20. Modul Kelola Tag (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/games/tag

Laravel API Resource: Route::apiResource('game-tags', GameTagController::class)

Table Name: GAME_TAGS

id : INT PK Auto-Increment

name : STRING

img : STRING

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-game-tags

C (Create): Page /admin/games/tag/add -> API POST /api/v1/game-tags

U (Update): Page /admin/games/tag/edit/{id} -> API PUT /api/v1/game-tags/{id}

D (Delete): API DELETE /api/v1/game-tags/{id}

21. Modul Kelola Produk (CRUDB)

Frontend Halaman: https://URL_WEBSITE/admin/product

Laravel API Resource: Route::apiResource('products', ProductController::class)

Table Name: PRODUCTS

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

name : STRING (100) (Contoh: "1450 Diamond")

capital_price : DECIMAL (12, 2)

guest_price : DECIMAL (12, 2)

digital_product_provider_id : INT FK Nullable (Relasi ke digital_product_providers.id)

product_code : STRING (UNIQUE SKU)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'capital_price' => 'decimal:2',
        'guest_price' => 'decimal:2',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-products

C (Create): Page /admin/product/add -> API POST /api/v1/products

U (Update): Page /admin/product/edit/{id} -> API PUT /api/v1/products/{id}

D (Delete): API DELETE /api/v1/products/{id}

B (Bulk Action): POST /api/v1/products/bulk-action

22. Modul Konfigurasi Flash Sale (UB)

Diintegrasikan sebagai modul setelan di atas halaman utama daftar flash sale

Frontend Halaman: https://URL_WEBSITE/admin/product/flash-sale (Form Atas)

Laravel API Resource: Route::apiResource('flash-sale-configs', FlashSaleConfigController::class)->only(['index', 'update'])

Table Name: FLASH_SALE_CONFIGS

id : INT PK Auto-Increment

status : BOOLEAN (default: false)

end_date : DATE

end_time : TIME

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'status' => 'boolean',
        'end_date' => 'date',
    ];
}


Endpoint & Page Mapping:

U (Update): PUT /api/v1/flash-sale-configs/{id}

B (Bulk Action): POST /api/v1/flash-sale-configs/bulk-action

23. Modul Produk Flash Sale (CUT)

Diintegrasikan di bawah form konfigurasi flash sale

Frontend Halaman: https://URL_WEBSITE/admin/product/flash-sale (Tabel Bawah)

Laravel API Resource: Route::apiResource('product-flash-sales', ProductFlashSaleController::class)

Table Name: PRODUCT_FLASH_SALES

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

product_id : INT FK (Relasi ke PRODUCTS.id - Varian dinamis terpilih berdasarkan game_id)

harga_coret : DECIMAL (12, 2)

label : STRING

is_flash_sale : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'harga_coret' => 'decimal:2',
        'is_flash_sale' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-product-flash-sales

C (Create): Page /admin/product/flash-sale/add -> API POST /api/v1/product-flash-sales

U (Update): Page /admin/product/flash-sale/edit/{id} -> API PUT /api/v1/product-flash-sales/{id}

D (Delete): API DELETE /api/v1/product-flash-sales/{id}

T (Toggle): POST /api/v1/product-flash-sales/toggle-status

24. Modul Kelola Joki Akun (CRUDZ)

Frontend Halaman: https://URL_WEBSITE/admin/joki

Laravel API Resource: Route::apiResource('account-joki', AccountJokiController::class)->middleware('auth')

Table Name: ACCOUNT_JOKI

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

product_id : INT FK (Relasi ke PRODUCTS.id - Menentukan varian harga paket joki)

trx_number : STRING (11) FK (Relasi ke TRANSACTIONS.trx_number)

joki_account : TEXT (JSON terenkripsi data kredensial akun pembeli)

joki_detail : TEXT (JSON data detail target push joki)

status : ENUM ('Pending', 'Processing', 'Success', 'Failed') (default: 'Pending')

buyer_note : TEXT Nullable

admin_note : TEXT Nullable

start_at : TIMESTAMP Nullable

end_at : TIMESTAMP Nullable

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'joki_account' => 'encrypted:array',
        'joki_detail' => 'array',
        'start_at' => 'datetime',
        'end_at' => 'datetime',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-account-joki

C (Create): Page /admin/joki/add -> API POST /api/v1/account-joki

U (Update): Page /admin/joki/edit/{id} -> API PUT /api/v1/account-joki/{id}

D (Delete): API DELETE /api/v1/account-joki/{id}

Z (Detail): Page /admin/joki/detail/{id} -> API GET /api/v1/account-joki/{id}

25. Modul Kelola Stok Akun (CRUDZ)

Frontend Halaman: https://URL_WEBSITE/admin/stocks

Laravel API Resource: Route::apiResource('account-stocks', AccountStockController::class)->middleware('auth')

Table Name: ACCOUNT_STOCKS

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

config_transaction_contact_id : INT FK Nullable (Relasi template kontak)

product_id : INT FK (Relasi ke PRODUCTS.id - Menentukan harga stok akun tersebut)

trx_number : STRING (11) FK Nullable (Terekam otomatis ke TRANSACTIONS.trx_number saat terjual)

title : STRING (100) (Contoh: "Akun MLBB 40+ Skin Epic")

account_data : TEXT (JSON terenkripsi data kredensial login akun)

status : ENUM ('available', 'sold') (default: 'available')

sold_at : TIMESTAMP Nullable

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'account_data' => 'encrypted:array',
        'sold_at' => 'datetime',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-account-stocks

C (Create): Page /admin/stocks/add -> API POST /api/v1/account-stocks

U (Update): Page /admin/stocks/edit/{id} -> API PUT /api/v1/account-stocks/{id}

D (Delete): API DELETE /api/v1/account-stocks/{id}

Z (Detail): Page /admin/stocks/detail/{id} -> API GET /api/v1/account-stocks/{id}

26. Modul Kelola Kategori Produk (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/product/category

Laravel API Resource: Route::apiResource('product-categories', ProductCategoryController::class)

Table Name: PRODUCT_CATEGORIES

id : INT PK Auto-Increment

name : STRING

queue_number : INT (default: 1)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-product-categories

C (Create): Page /admin/product/category/add -> API POST /api/v1/product-categories

U (Update): Page /admin/product/category/edit/{id} -> API PUT /api/v1/product-categories/{id}

D (Delete): API DELETE /api/v1/product-categories/{id}

27. Modul Kelola Icon Produk (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/product/icon

Laravel API Resource: Route::apiResource('product-icons', ProductIconController::class)

Table Name: PRODUCT_ICONS

id : INT PK Auto-Increment

name : STRING

img : STRING

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-product-icons

C (Create): Page /admin/product/icon/add -> API POST /api/v1/product-icons

U (Update): Page /admin/product/icon/edit/{id} -> API PUT /api/v1/product-icons/{id}

D (Delete): API DELETE /api/v1/product-icons/{id}

28. Modul Kelola Provider (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/provider

Laravel API Resource: Route::apiResource('digital-product-providers', DigitalProductProviderController::class)->middleware('auth')

Table Name: digital_product_providers

id : INT PK Auto-Increment

name : STRING

username : STRING Nullable

production_key : STRING Nullable

url_callback : STRING Nullable

webhook_id : STRING Nullable

secret : STRING Nullable (Terenkripsi)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'secret' => 'encrypted',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-digital-product-providers

C (Create): Page /admin/provider/add -> API POST /api/v1/digital-product-providers

U (Update): Page /admin/provider/edit/{id} -> API PUT /api/v1/digital-product-providers/{id}

D (Delete): API DELETE /api/v1/digital-product-providers/{id}

29. Modul Markup Harga Produk (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/product/markup

Laravel API Resource: Route::apiResource('product-markups', ProductMarkupController::class)->middleware('auth')

Table Name: PRODUCT_MARKUP

id : INT PK Auto-Increment

game_id : INT FK (Relasi ke GAMES.id)

digital_product_provider_id : INT FK Nullable (Relasi ke digital_product_providers.id)

product_id : INT FK (Relasi ke PRODUCTS.id)

markup_type : ENUM ('rupiah', 'percent')

markup_value : INT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'markup_value' => 'integer',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-product-markups

C (Create): Page /admin/product/markup/add -> API POST /api/v1/product-markups

U (Update): Page /admin/product/markup/edit/{id} -> API PUT /api/v1/product-markups/{id}

D (Delete): API DELETE /api/v1/product-markups/{id}

30. Modul Kelola Metode Pembayaran (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/payment-method

Laravel API Resource: Route::apiResource('payment-methods', PaymentMethodController::class)

Table Name: PAYMENT_METHODS

id : INT PK Auto-Increment

name : STRING

payment_categories_id : INT FK (Relasi ke PAYMENT_METHOD_CATEGORIES.id)

img : STRING

admin_fee_rupiah : DECIMAL (12, 2) (default: 0)

admin_fee_percent : DECIMAL (5, 2) (default: 0)

uniq_code : INT (default: 0)

digital_product_provider_id : INT FK Nullable

value : STRING Nullable (No Rekening manual / Kode gateway)

is_display_footer : BOOLEAN (default: true)

is_active : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'admin_fee_rupiah' => 'decimal:2',
        'admin_fee_percent' => 'decimal:2',
        'uniq_code' => 'integer',
        'is_display_footer' => 'boolean',
        'is_active' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-payment-methods

C (Create): Page /admin/payment-method/add -> API POST /api/v1/payment-methods

U (Update): Page /admin/payment-method/edit/{id} -> API PUT /api/v1/payment-methods/{id}

D (Delete): API DELETE /api/v1/payment-methods/{id}

T (Toggle): POST /api/v1/payment-methods/toggle-status

31. Modul Kelola Kategori Metode Pembayaran (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/payment-method/category

Laravel API Resource: Route::apiResource('payment-method-categories', PaymentMethodCategoryController::class)

Table Name: PAYMENT_METHOD_CATEGORIES

id : INT PK Auto-Increment

name : STRING

queue_number : INT (default: 1)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-payment-method-categories

C (Create): Page /admin/payment-method/category/add -> API POST /api/v1/payment-method-categories

U (Update): Page /admin/payment-method/category/edit/{id} -> API PUT /api/v1/payment-method-categories/{id}

D (Delete): API DELETE /api/v1/payment-method-categories/{id}

32. Modul Kelola Voucher (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/voucher

Laravel API Resource: Route::apiResource('vouchers', VoucherController::class)

Table Name: vouchers

id : INT PK Auto-Increment

voucher_code : STRING -> UNIQUE

discount : INT (Percentage value, 1-100)

min_transaction : DECIMAL (12, 2)

max_discount : DECIMAL (12, 2)

is_active : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'discount' => 'integer',
        'min_transaction' => 'decimal:2',
        'max_discount' => 'decimal:2',
        'is_active' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-vouchers

C (Create): Page /admin/voucher/add -> API POST /api/v1/vouchers

U (Update): Page /admin/voucher/edit/{id} -> API PUT /api/v1/vouchers/{id}

D (Delete): API DELETE /api/v1/vouchers/{id}

T (Toggle): POST /api/v1/vouchers/toggle-status

33. Modul Kelola Banner (CRDT)

Frontend Halaman: https://URL_WEBSITE/admin/banner

Laravel API Resource: Route::apiResource('banners', BannerController::class)

Table Name: banners

id : INT PK Auto-Increment

img : STRING (Spesifikasi: 1472 x 832 px)

is_active : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'is_active' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-banners

C (Create): Page /admin/banner/add -> API POST /api/v1/banners

D (Delete): API DELETE /api/v1/banners/{id}

T (Toggle): POST /api/v1/banners/toggle-status

34. Modul Notifikasi Whatsapp (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/notif/whatsapp

Laravel API Resource: Route::apiResource('notif-whatsapps', NotifWhatsappController::class)->middleware('auth')

Table Name: notif_whatsapps

id : INT PK Auto-Increment

title : STRING (100)

subject : STRING (150)

guide : TEXT

content : TEXT

is_active : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'is_active' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-notif-whatsapps

C (Create): Page /admin/notif/whatsapp/add -> API POST /api/v1/notif-whatsapps

U (Update): Page /admin/notif/whatsapp/edit/{id} -> API PUT /api/v1/notif-whatsapps/{id}

D (Delete): API DELETE /api/v1/notif-whatsapps/{id}

T (Toggle): POST /api/v1/notif-whatsapps/toggle-status

35. Modul Response Whatsapp (RD)

Frontend Halaman: https://URL_WEBSITE/admin/notif/whatsapp/response

Laravel API Resource: Route::apiResource('response-whatsapps', ResponseWhatsappController::class)->only(['index', 'destroy'])->middleware('auth')

Table Name: response_whatsapps

id : INT PK Auto-Increment

config_whatsapp_id : INT FK (Relasi ke CONFIG_WHATSAPP.id)

status : ENUM ('success', 'failed')

message : STRING (255)

response : TEXT (JSON response payload)

datetime : TIMESTAMP

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'response' => 'array',
        'datetime' => 'datetime',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-response-whatsapps

D (Delete): API DELETE /api/v1/response-whatsapps/{id}

36. Modul Notifikasi Email (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/notif/email

Laravel API Resource: Route::apiResource('notif-emails', NotifEmailController::class)->middleware('auth')

Table Name: notif_emails

id : INT PK Auto-Increment

transaction_contact_id : INT FK Nullable

subject : STRING (150)

content : TEXT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-notif-emails

C (Create): Page /admin/notif/email/add -> API POST /api/v1/notif-emails

U (Update): Page /admin/notif/email/edit/{id} -> API PUT /api/v1/notif-emails/{id}

D (Delete): API DELETE /api/v1/notif-emails/{id}

37. Modul Response Email (RD)

Frontend Halaman: https://URL_WEBSITE/admin/notif/email/response

Laravel API Resource: Route::apiResource('response-emails', ResponseEmailController::class)->only(['index', 'destroy'])->middleware('auth')

Table Name: response_emails

id : INT PK Auto-Increment

config_smtp_id : INT FK (Relasi ke CONFIG_SMTP.id)

status : ENUM ('success', 'failed')

response : TEXT (JSON response payload)

datetime : TIMESTAMP

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'response' => 'array',
        'datetime' => 'datetime',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-response-emails

D (Delete): API DELETE /api/v1/response-emails/{id}

38. Modul Kelola Popup (CRUDT)

Frontend Halaman: https://URL_WEBSITE/admin/popup

Laravel API Resource: Route::apiResource('popup', PopupController::class)

Table Name: popup

id : INT PK Auto-Increment

title : STRING (100)

content : TEXT

img : STRING

display_locate : ENUM ('beranda', 'halaman games')

game_id : INT FK Nullable (Relasi ke GAMES.id)

is_active : BOOLEAN (default: true)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'is_active' => 'boolean',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-popup

C (Create): Page /admin/popup/add -> API POST /api/v1/popup

U (Update): Page /admin/popup/edit/{id} -> API PUT /api/v1/popup/{id}

D (Delete): API DELETE /api/v1/popup/{id}

T (Toggle): POST /api/v1/popup/toggle-status

39. Modul Kelola Sosmed (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/sosmed

Laravel API Resource: Route::apiResource('sosmed', SosmedController::class)

Table Name: sosmed

id : INT PK Auto-Increment

name : STRING (50)

img : STRING

link : STRING (255)

locate : ENUM ('navbar', 'contact', 'footer')

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-sosmed

C (Create): Page /admin/sosmed/add -> API POST /api/v1/sosmed

U (Update): Page /admin/sosmed/edit/{id} -> API PUT /api/v1/sosmed/{id}

D (Delete): API DELETE /api/v1/sosmed/{id}

40. Modul Kelola Tampilan (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/preference

Laravel API Resource: Route::apiResource('preference', PreferenceController::class)

Table Name: preference

id : INT PK Auto-Increment

profile_pic : STRING Nullable

teks_navbar : STRING Nullable

hero_img : TEXT (JSON Array slide banner login)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'hero_img' => 'array',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-preference

C (Create): Page /admin/preference/add -> API POST /api/v1/preference

U (Update): Page /admin/preference/edit/{id} -> API PUT /api/v1/preference/{id}

D (Delete): API DELETE /api/v1/preference/{id}

41. Modul Kelola Pertanyaan Umum (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/faq

Laravel API Resource: Route::apiResource('faq', FaqController::class)

Table Name: faq

id : INT PK Auto-Increment

faq_categories_id : INT FK (Relasi ke FAQ_CATEGORIES.id)

title : STRING (255)

content : TEXT

queue_number : INT (default: 1)

datetime : TIMESTAMP

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts:

protected function casts(): array {
    return [
        'datetime' => 'datetime',
        'queue_number' => 'integer',
    ];
}


Endpoint & Page Mapping:

R (Read): POST /api/v1/data-faq

C (Create): Page /admin/faq/add -> API POST /api/v1/faq

U (Update): Page /admin/faq/edit/{id} -> API PUT /api/v1/faq/{id}

D (Delete): API DELETE /api/v1/faq/{id}

42. Modul Kelola Kategori Pertanyaan Umum (CRUD)

Frontend Halaman: https://URL_WEBSITE/admin/faq/category

Laravel API Resource: Route::apiResource('faq-categories', FaqCategoryController::class)

Table Name: FAQ_CATEGORIES

id : INT PK Auto-Increment

name : STRING

img : STRING

queue_number : INT (default: 1)

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Endpoint & Page Mapping:

R (Read): POST /api/v1/data-faq-categories

C (Create): Page /admin/faq/category/add -> API POST /api/v1/faq-categories

U (Update): Page /admin/faq/category/edit/{id} -> API PUT /api/v1/faq-categories/{id}

D (Delete): API DELETE /api/v1/faq-categories/{id}

43. Modul Syarat dan Ketentuan (U)

Frontend Halaman: https://URL_WEBSITE/admin/pages/terms

Laravel API Resource: Route::apiResource('pages-terms', PagesTermsController::class)->only(['index', 'update'])

Table Name: PAGES_TERMS

id : INT PK Auto-Increment

content : TEXT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts: None

44. Modul Kebijakan Privasi (U)

Frontend Halaman: https://URL_WEBSITE/admin/pages/privacy

Laravel API Resource: Route::apiResource('pages-privacy', PagesPrivacyController::class)->only(['index', 'update'])

Table Name: PAGES_PRIVACY

id : INT PK Auto-Increment

content : TEXT

created_at : TIMESTAMP

updated_at : TIMESTAMP

deleted_at : TIMESTAMP (Soft Delete)

Model Casts: None