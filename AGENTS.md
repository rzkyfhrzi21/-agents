---
name: ai-master-rules
description: >
  File induk aturan AI untuk proyek.
  Seluruh skill telah dipecah ke file masing-masing di folder @agents/AI-SKILLS/
  agar lebih modular dan mudah di-maintain.
---

# DAFTAR ATURAN & SKILL BACA AI

Berikut adalah aturan prioritas pembacaan file yang wajib dipatuhi oleh AI saat memulai sesi atau melakukan pengerjaan.

## 1. Aturan Pembacaan Folder Skill (`@agents/AI-SKILLS/`)
AI **wajib** membaca semua file skill di folder `@agents/AI-SKILLS/` dan subfoldernya. Berikut adalah fungsi garis besar dari tiap folder *skill* yang tersedia:

- **📁 PERSONA_SKILLS**: Kumpulan persona dan gaya komunikasi AI.
  - `CAVEMAN_SKILL.md`: Mode komunikasi sangat singkat dan padat (hemat token).
  - `PONYTAIL_SKILL.md`: Mode "Senior Dev Malas", memprioritaskan solusi paling simpel, bersih, dan menolak *over-engineering* (YAGNI).
  - `KOMENTAR_ORANG_TUA.md`: (Pengecualian) Jangan dibaca/diterapkan kecuali diminta secara eksplisit oleh pengguna.

- **📁 TOOL_SKILLS**: Panduan integrasi dan penggunaan alat (tools/MCP) pendukung.
  - `CODEBASE_MEMORY.md`: Wajib digunakan di awal untuk memahami struktur *repository* secara menyeluruh.
  - `CONTEXT7.md`: Digunakan untuk mencari dokumentasi resmi teknologi utama proyek.
  - `FIRECRAWL.md`: Digunakan untuk *web scraping* atau mencari dokumentasi dari sumber luar.
  - `RTK_SKILL.md`: Panduan inisialisasi Repository Toolkit (RTK) demi optimasi token terminal.

- **📁 WORKFLOW_SKILLS**: Aturan standar operasional, alur kerja, dan lingkungan kerja AI.
  - `AGENTS-GENERAL.md` / `WORKFLOW_MODE.md` / `AI_SETUP_MACHINE.md`: Aturan terkait alur kerja AI, penyiapan lingkungan (*environment*), dan persona.
  - `CICD_SSH_SKILL.md`: Prosedur wajib untuk setup otomatis CI/CD menggunakan SSH Rsync dan Github Actions.

- **📁 SUPERPOWERS**: 
  - `SUPERPOWERS_SKILLS.md`: Kumpulan *skill* modular untuk alur kerja seperti *brainstorming*, penulisan *plan*, dan *debugging*.

- **📁 AGENT_SKILLS**: (Dari repo `agent-skills`)
  - Kumpulan *skill* engineering (TDD, CI/CD, Code Review, SDD, Debugging, Refactoring, dsb). Diambil dari koleksi Addy Osmani untuk menstandarkan siklus hidup rekayasa perangkat lunak (*software engineering lifecycle*) AI.

- **📁 TASTE_SKILLS**: (Dari repo `taste-skill`)
  - Kumpulan *skill* (seperti `brandkit`, `minimalist-skill`, dll) untuk merancang antarmuka (UI/UX) yang premium, dinamis, dan terhindar dari gaya bawaan yang kaku (*anti-slop*).

- **📁 ECC_SKILLS**: (Dari repo `ecc`)
  - Kumpulan masif ratusan *skill* modular (mulai dari *frontend*, *backend*, *security*, hingga DevOps) yang bertindak sebagai *Agent Harness Operating System*. File dipisah per-topik agar AI dapat membaca yang paling relevan dengan tumpukan teknologi proyek yang sedang dikerjakan.

- **📁 IMPECCABLE_SKILLS**: (Dari repo `impeccable`)
  - *Skill* sistem desain (*design tokens*, tipografi, spasi, dsb) untuk meningkatkan kualitas dan konsistensi antarmuka (UI) dari agen AI serta mencegah keluaran desain *slop*.

- **📁 UIUX_SKILLS**: (Dari repo `ui-ux-pro-max-skill`)
  - Koleksi skill UI/UX lengkap dengan database 84 gaya visual, 192 palet warna, 74 font pairing, 98 panduan UX, dan 16 GSAP motion preset. Mencakup: `ui-ux-pro-max` (skill utama), `design` (identitas brand, logo, banner, ikon, social media), `design-system` (token arsitektur tiga-layer), `ui-styling` (shadcn/ui + Tailwind), `brand` (brand voice & visual identity), `banner-design` (22 gaya banner multi-platform), `slides` (presentasi HTML + Chart.js).



## 2. Aturan Pembacaan Dokumen Panduan (`docs/`)
AI **wajib** membaca panduan dari file dan subfolder di dalam `docs/` berikut **(bila ada filenya)**:
- `docs/BRIEF.txt`
- `docs/CANVAS.md`
- `docs/PRD.md`
- Folder `docs/BRIEF KOMPONENT/` (Semua file di dalamnya, bila ada)
- Folder `docs/BRIEF MODUL/` (Semua file di dalamnya, bila ada)

**DILARANG KERAS** membaca subfolder berikut di dalam `docs/`:
1. `docs/BRIEF SAAT DEPLOY/` (Folder brief saat deploy - JANGAN DIBACA)
2. `docs/PROJECT LAIN (CONTOH)/` (Folder project lain - JANGAN DIBACA)
3. `docs/DATA KLIEN/` (Folder data klien - JANGAN DIBACA)

## 3. Aturan Pembacaan README Utama
- **DILARANG KERAS** membaca file `README.md` yang berada di root project karena itu hanya panduan untuk instalasi & deployment saja (JANGAN DIBACA). Gunakan dokumen panduan `docs/` yang diperbolehkan di atas untuk referensi teknis.

---

# INISIALISASI PROYEK BARU (AUTO-SETUP)

Jika AI mendeteksi bahwa ini adalah proyek atau *workspace* baru, AI **wajib** secara proaktif menjalankan (atau mengingatkan pengguna untuk menjalankan) langkah-langkah inisialisasi *tools* berikut:
1. **RTK (Repository Toolkit):** Jalankan perintah `./@agents/RTK/rtk.exe init` (atau sesuaikan path rtk) di terminal untuk menghasilkan folder `.rtk` (berisi konfigurasi `filters.toml`) dan folder `rules` (berisi aturan `antigravity-rtk-rules.md`).
2. **Context7:** Pastikan integrasi sudah terpasang dengan menjalankan perintah `npx ctx7 setup` untuk autentikasi dan pembuatan *rules* otomatis di dalam agen.
3. **Codebase Memory MCP:** Lakukan pemetaan struktur proyek secara otomatis. Jika server MCP sudah tersambung, AI harus segera memindai (melakukan aksi *"Index this project"*) agar grafik pengetahuan kode terbentuk di memori.
4. **Firecrawl MCP:** Ingatkan pengguna untuk memastikan variabel lingkungan `FIRECRAWL_API_KEY` sudah terpasang jika proyek membutuhkan fitur pencarian web/*scraping* lanjutan.

---

# MANAJEMEN FILE/SCRIPT SEMENTARA
- Simpan semua file eksekusi, script sementara (temporary scripts), atau file uji coba/scratch di dalam folder `Zzz/` yang ada di root proyek.
- DILARANG mengotori root directory atau folder lain dengan script sekali pakai.

---

# ENVIRONMENT TERMINAL USER

- User menggunakan **CMD (Command Prompt) mode Administrator**, BUKAN PowerShell.
- AI **wajib** memberikan perintah dalam sintaks CMD, bukan PowerShell.
- Contoh perbedaan:
  - PowerShell: `New-Item`, `Remove-Item`, `Move-Item`
  - CMD: `mkdir`, `del`, `move`, `rmdir /s /q`

---

# ATURAN REFACTORING & MODIFIKASI KODE LAMA

- **DILARANG KERAS** mengubah, merombak (refactor), atau menulis ulang seluruh kode pada file yang sudah ada secara tiba-tiba hanya karena membaca aturan di `AGENTS.md` atau file skill lainnya.
- Aturan penulisan kode **hanya berlaku** untuk kode baru yang sedang ditulis atau fitur baru yang sedang ditambahkan.
- Pengecualian: AI hanya diizinkan melakukan refactor pada kode lama **JIKA DAN HANYA JIKA** pengguna secara eksplisit menginstruksikan atau meminta refactor tersebut.

---
