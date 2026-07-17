# AI Workspace Configuration

Repositori ini berisi standar konfigurasi, *skills*, aturan (rules), dan integrasi alat (*tools*) AI yang dapat digunakan untuk berbagai asisten AI (seperti Claude, Copilot, Codex, Antigravity, dan lainnya). Konfigurasi ini dirancang agar dapat disalin (*copy-paste*) ke setiap proyek baru untuk langsung memberikan konteks, aturan kerja, dan kemampuan khusus kepada AI.

## Struktur Direktori & File

Berikut adalah penjelasan masing-masing komponen yang ada di dalam konfigurasi ini:

### 1. `AI_RULES.md`
Ini adalah **file induk aturan AI**. File ini mendikte bagaimana AI harus berperilaku saat mengerjakan proyek, file apa saja yang boleh dan tidak boleh dibaca, serta konvensi umum proyek (misalnya, kewajiban menggunakan terminal CMD Mode Administrator). 
- **Penggunaan:** Biarkan file ini di *root* direktori proyek. AI akan membaca file ini terlebih dahulu untuk memahami batasan dan prioritas kerjanya.

### 2. `@agents/AI-SKILLS/`
Folder ini berisi kumpulan *file* yang mendefinisikan *skill* (kemampuan khusus), persona, integrasi MCP, dan pedoman perilaku AI secara modular. AI diwajibkan untuk membaca seluruh isi folder ini di awal pengerjaan, kecuali untuk beberapa *file* khusus (seperti `KOMENTAR_ORANG_TUA.md`) yang hanya boleh dibaca/diterapkan jika pengguna memintanya secara eksplisit.

### 3. `@agents/MCP/`
Folder ini diperuntukkan bagi konfigurasi **Model Context Protocol (MCP)**. Ini memastikan AI memiliki panduan atau konfigurasi yang diperlukan untuk berinteraksi dengan berbagai macam CLI, *text editor*, basis data, dan *tools* sistem lainnya.

### 4. `@agents/RTK/`
Berisi utilitas bawaan (seperti `rtk.exe`) dan perlengkapan *toolkit* lainnya yang dapat dieksekusi atau dimanfaatkan oleh AI saat membantu proses *development* dalam proyek.

### 5. Folder Lainnya yang Terkait Aturan
- **`Zzz/` (Dibuat otomatis oleh AI):** Berdasarkan instruksi di `AI_RULES.md`, saat bekerja, AI diinstruksikan untuk menyimpan semua *script* sementara (*temporary scripts*), file eksekusi sementara, atau file uji coba (*scratch*) di dalam folder `Zzz/` agar tidak mengotori *root* direktori proyek Anda.

## Cara Menggunakan pada Proyek Baru

1. **Salin Konfigurasi Utama:** *Copy* file `AI_RULES.md` beserta folder `@agents/AI-SKILLS/`, `@agents/MCP/`, dan `@agents/RTK/` ke dalam *root* direktori proyek baru Anda.
2. **Siapkan Folder Panduan Proyek (`docs/`):** 
   - Folder `docs/` yang ada di konfigurasi ini **hanya sebagai contoh/struktur dasar**. 
   - Pada proyek riil Anda, buat folder `docs/` dan isi dengan dokumen spesifik yang merujuk pada `AI_RULES.md`, seperti `BRIEF.txt`, `CANVAS.md`, `ENDPOINT.txt`, serta subfolder `BRIEF KOMPONENT/` dan `BRIEF MODUL/`.
3. **Mulai Sesi AI:** Saat Anda membuka *workspace* atau memulai obrolan dengan AI di proyek yang baru, sistem AI (yang mendukung konteks/aturan proyek) akan secara otomatis membaca aturan dari `AI_RULES.md` dan *skills* di folder `@agents/AI-SKILLS/`.

## Penambahan *Skill* Baru

Jika Anda ingin meminta AI untuk menginstal *skill* baru dari sebuah repositori (contoh: dari GitHub), Anda cukup memberikan tautan repositori tersebut kepada AI. AI akan secara otomatis melakukan langkah berikut:
1. Mengunduh (kloning) repositori asli tersebut ke dalam folder `@agents/SKILLS/` sebagai arsip referensi sumber.
2. Mengekstrak dan menggabungkan seluruh instruksi dari file *skill* (misalnya `SKILL.md`) yang ada di dalam repositori tersebut.
3. Memasukkannya menjadi **satu file Markdown (.md) khusus (1 file per repositori)** di dalam folder `@agents/AI-SKILLS/` agar dapat dibaca dan digunakan berulang kali oleh AI. File hasil gabungan ini akan tetap dipisah untuk setiap repositori yang Anda instal (tidak digabungkan menjadi satu file raksasa), sehingga lebih rapi dan modular.

*Contoh: Menginstal repositori `obra/superpowers` akan meletakkan repositori aslinya di folder `@agents/SKILLS/superpowers`, dan kemudian menghasilkan file gabungan `SUPERPOWERS_SKILLS.md` di dalam folder `@agents/AI-SKILLS/`.*

---
*Dengan menyalin konfigurasi ini ke setiap proyek, Anda memastikan bahwa semua AI *Assistant* yang Anda gunakan selalu mengikuti standar operasional yang sama persis (penggunaan CMD, pengelolaan file sementara, dan aturan pembacaan dokumentasi) tanpa perlu Anda latih berulang kali setiap memulai proyek baru.*

