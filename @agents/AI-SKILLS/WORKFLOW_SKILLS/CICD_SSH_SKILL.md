# CI/CD SSH DEPLOYMENT SKILL

Skill ini aktif ketika pengguna meminta untuk menyiapkan deployment otomatis, CI/CD, atau integrasi Github Actions untuk melakukan push ke server (cPanel/VPS) menggunakan SSH/Rsync.

Jika dipicu, AI **wajib** menjalankan langkah-langkah berikut secara berurutan:

## Langkah 1: Buat Kunci SSH (Otomatis)
AI harus menjalankan perintah di terminal CMD pengguna untuk menghasilkan pasangan kunci SSH baru (Private & Public) yang akan diletakkan di dalam folder `@agents/SSH/`. Pastikan untuk mengganti `[NAMA_PROJECT]` dengan nama proyek yang sebenarnya.
Gunakan *passphrase* yang kuat atau kosongkan jika dirasa aman (sesuaikan dengan konteks/keinginan pengguna).

```cmd
mkdir "@agents\SSH"
ssh-keygen -t rsa -b 4096 -f "@agents\SSH\deploy_key_[NAMA_PROJECT]" -N "" -C "github-actions-deploy"
```
*Catatan: Eksekusi kode di atas secara nyata di terminal.*

## Langkah 2: Buat File Workflow Github Actions (Otomatis)
AI harus membuat struktur direktori `.github/workflows/` (jika belum ada) dan menghasilkan file `.github/workflows/deploy.yml` menggunakan *template* di bawah ini. AI **wajib** menanyakan atau menyesuaikan parameter `TARGET`, `REMOTE_USER`, dan nama branch (`master`/`main`) sesuai detail proyek.

```yaml
name: Deploy to Production (SSH Rsync)

# Memicu alur kerja hanya saat push ke branch utama
on:
  push:
    branches:
      - master # Sesuaikan menjadi main jika perlu

jobs:
  production-deploy:
    name: 🚀 Deploy to Server
    runs-on: ubuntu-latest

    steps:
      # Langkah 1: Mengambil kode proyek
      - name: 🚚 Get latest code
        uses: actions/checkout@v3

      # Langkah 2: Deploy menggunakan Rsync over SSH
      - name: 🚀 Deploy via Rsync
        uses: easingthemes/ssh-deploy@main
        with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          PASSPHRASE: ${{ secrets.SSH_PASSPHRASE }}
          PORT: ${{ secrets.SSH_PORT }}
          ARGS: "-rlgoDzvO"
          SOURCE: "./"
          REMOTE_HOST: ${{ secrets.SSH_HOST }}
          REMOTE_USER: ${{ secrets.SSH_USERNAME }}
          # Sesuaikan TARGET dan REMOTE_USER
          TARGET: "/home/USERNAME_CPANEL/DOMAIN_ANDA"
          # Mengecualikan folder sensitif dan konfigurasi agen
          EXCLUDE: "/.git/, /.github/, /db/, /referensi/, *.sql, /Zzz/, README.md, /laporan, /@agents/"
```

## Langkah 3: Berikan Instruksi Lanjutan ke Pengguna
Setelah langkah 1 dan 2 selesai, AI **tidak dapat** memasukkan *secrets* ke GitHub atau server secara langsung. Oleh karena itu, AI **wajib mencetak panduan berikut** (atau ringkasannya) ke pengguna agar mereka menyelesaikannya secara manual:

1. **Pasang Public Key di Server (cPanel/VPS):**
   Minta pengguna membuka file `@agents/SSH/deploy_key_[NAMA_PROJECT].pub` dan memasukkan isinya ke server. Jika cPanel, gunakan menu **SSH Access -> Manage SSH Keys -> Import Public Key** (lalu *Authorize*). Jika VPS, letakkan di `~/.ssh/authorized_keys`.

2. **Pasang GitHub Secrets:**
   Minta pengguna membuka Repositori GitHub -> **Settings -> Secrets and variables -> Actions -> New repository secret** dan memasukkan data berikut:
   - `SSH_PRIVATE_KEY`: Isi dengan seluruh teks dari file `@agents/SSH/deploy_key_[NAMA_PROJECT]` (termasuk `--BEGIN--` dan `--END--`).
   - `SSH_PASSPHRASE`: Kata sandi SSH (jika dikosongkan pada langkah 1, *secret* ini bisa dilewati).
   - `SSH_PORT`: Port SSH Server (biasanya 22 atau port modifikasi cPanel).
   - `SSH_HOST`: IP Address atau Domain Server.
   - `SSH_USERNAME`: Username login server (contoh: `uucdjd7c`).

AI harus menunggu pengguna mengonfirmasi bahwa kedua langkah manual ini selesai sebelum melanjutkan tugas lain.
