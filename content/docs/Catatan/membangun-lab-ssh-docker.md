---
title: "Membangun Lab SSH Pribadi dengan Docker: Bahrani Labs"
date: 2025-10-05T00:00:00Z
tags: ["docker", "ssh", "devops", "tutorial", "container"]
categories: ["Catatan"]
draft: false
---

Sering kali kita membutuhkan sebuah lingkungan _sandbox_ atau lab virtual yang ringan untuk eksperimen, development, atau sekadar mencoba perintah-perintah sistem. Menggunakan Virtual Machine (VM) penuh terkadang terasa berlebihan. Artikel ini akan memandu Anda membangun "Bahrani Labs": sebuah lab berbasis kontainer Docker yang bisa diakses via SSH, lengkap dengan persistensi data dan konfigurasi siap pakai.

Lab ini hadir dengan fitur esensial seperti **akses SSH**, user `cisco` dengan hak **sudo**, **volume persisten** untuk data Anda, **healthcheck** untuk memonitor layanan, kebijakan **restart otomatis**, dan zona waktu yang sudah disesuaikan untuk Indonesia (`Asia/Jakarta`).

## Prasyarat

Sebelum memulai, pastikan sistem Anda telah memenuhi syarat berikut:
- **Docker & Docker Compose**: Sudah terpasang dan berjalan.
- **Port Host**: Ada satu port yang tidak terpakai di mesin Anda (dalam contoh ini kita gunakan `9022`).
- **Sistem Operasi**: Tutorial ini diuji pada Linux, MacOS, atau Windows dengan WSL 2.

## Arsitektur Sederhana

Desainnya sangat sederhana. Kita akan menjalankan satu kontainer dari _image_ kustom yang kita bangun. Port `22` (SSH) di dalam kontainer akan dipetakan ke port `9022` di komputer Anda, dan semua data di dalam direktori `/home/cisco` akan disimpan secara persisten.

Secara singkat, alur koneksinya adalah: `User -> SSH Client -> Host:9022 -> Container:22`.

## File Konfigurasi: Dockerfile & Compose

Kita hanya butuh dua file untuk menjalankan lab ini.

### Dockerfile

File ini berisi instruksi untuk membangun _image_ kontainer kita, mulai dari instalasi paket hingga pembuatan user.

```Dockerfile
# Gunakan base image Ubuntu versi stabil
FROM ubuntu:24.04

# Install semua paket yang dibutuhkan dalam satu layer
RUN apt-get update && apt-get install -y \
    sudo \
    openssh-server \
    python3 \
    python3-pip \
    git \
    nano \
    net-tools \
    iputils-ping \
    netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

# Buat user 'cisco' dan direktori home-nya
RUN useradd -rm -d /home/cisco -s /bin/bash -g root -G sudo -u 1000 cisco

# Set password default untuk user 'cisco' (HARAP SEGERA GANTI)
RUN echo 'cisco:cisco' | chpasswd

# Berikan hak sudo tanpa password untuk kemudahan di lingkungan lab
RUN echo 'cisco ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Buat direktori untuk SSH daemon
RUN mkdir /var/run/sshd

# Expose port SSH
EXPOSE 22

# Perintah default untuk menjalankan SSH daemon di foreground
CMD ["/usr/sbin/sshd", "-D"]
```

Penjelasan untuk setiap barisnya cukup sederhana: kita mulai dari `ubuntu:24.04`, menginstal semua _tools_, membuat user `cisco` dengan password `cisco`, memberikannya hak `sudo`, dan terakhir menjalankan server SSH saat kontainer dimulai.

### docker-compose.yml

File ini kita gunakan untuk mendefinisikan dan menjalankan kontainer `bahrani-labs` dengan semua konfigurasinya.

```yaml
version: "3.9"
services:
  bahrani-labs:
    container_name: bahrani-labs
    build:
      context: .
      dockerfile: Dockerfile
    image: bahrani-labs:latest
    restart: unless-stopped
    ports:
      - "9022:22"
    volumes:
      - bahrani-home:/home/cisco
    environment:
      TZ: "Asia/Jakarta"
    healthcheck:
      test: ["CMD", "sh", "-c", "nc -z 127.0.0.1 22 || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 20s
volumes:
  bahrani-home:
```

Di sini kita mendefinisikan semua properti layanan: `build` untuk membangun dari Dockerfile, `restart: unless-stopped` untuk menyala otomatis, `ports` untuk mapping port, `volumes` untuk persistensi data, dan `healthcheck` untuk memastikan SSH selalu berjalan.

## Instalasi dan Penggunaan Pertama

1.  Buat sebuah direktori baru di komputer Anda (misal, `bahrani-labs`).
2.  Di dalam direktori tersebut, buat file `Dockerfile` dan `docker-compose.yml` dengan isi seperti di atas.
3.  Buka terminal di direktori itu dan jalankan:
    ```bash
    docker compose up -d --build
    ```
4.  Setelah proses selesai, Anda bisa langsung terhubung via SSH:
    ```bash
    ssh cisco@localhost -p 9022
    ```
    Masukkan password default: `cisco`.

## Perintah Sehari-hari

Berikut adalah beberapa perintah dasar untuk mengelola lab Anda:

-   **Menghentikan Lab**: `docker compose stop`
-   **Memulai Lab**: `docker compose start`
-   **Melihat Logs**: `docker compose logs -f`
-   **Masuk ke Kontainer (tanpa SSH)**: `docker exec -it bahrani-labs bash`

## Catatan Keamanan (Best Practices)

Meskipun ini untuk lab, praktik keamanan dasar tetap penting:

-   **Ganti Password**: Hal **pertama** yang harus dilakukan setelah login adalah mengganti password default dengan perintah `passwd`.
-   **Gunakan SSH Key**: Untuk keamanan lebih, nonaktifkan login dengan password dan gunakan SSH key.
-   **Batasi Akses Port**: Di lingkungan produksi, batasi akses ke port `9022` hanya dari IP tepercaya menggunakan firewall.

## Kustomisasi dan Pengembangan Lanjutan

-   **Menambah Port Aplikasi**: Ingin menjalankan server web? Tambahkan mapping port di `docker-compose.yml`, misalnya `ports: ["9022:22", "8080:80"]`.
-   **Menambah Tools**: Edit `Dockerfile` dan tambahkan paket lain yang Anda butuhkan di bagian `apt-get install`.

## Troubleshooting dan FAQ

-   **Tidak bisa SSH?** Cek log (`docker compose logs`), pastikan tidak ada konflik port di `9022`, dan periksa status healthcheck (`docker inspect ...`).
-   **Bagaimana cara update paket?** Cara terbaik adalah dengan membangun ulang image: `docker compose build --no-cache`.
-   **Apakah bisa multi-user?** Tentu. Anda bisa memodifikasi `Dockerfile` untuk membuat user tambahan.

## Kesimpulan

"Bahrani Labs" adalah solusi ideal untuk siapa saja yang membutuhkan lingkungan _sandbox_ Linux yang ringan, cepat, dan persisten. Ini sangat cocok untuk skenario eksperimen, belajar, atau bahkan sebagai target untuk otomasi _build/test_ ringan.

## Call-to-Action

-   Coba setup lab ini di mesin Anda, lalu kembangkan dengan menambahkan aplikasi web sederhana yang berjalan di port 8080!
-   Ingin versi yang lebih aman untuk produksi? Nantikan artikel selanjutnya tentang _hardening_ SSH di dalam kontainer Docker.
