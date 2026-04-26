---
title: "Penerapan CI/CD pada Server Lokal"
date: 2026-04-26T23:22:21+07:00
draft: false
description: "Cara menerapkan CI/CD di server lokal menggunakan GitLab Runner agar setiap push ke Git otomatis di-deploy. Tidak perlu lagi clone dan deploy manual!"
tags: ["CI/CD", "GitLab Runner", "Self-Hosted Runner", "DevOps"]
categories: ["Catatan"]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---
## Dari SSH ke Server, Sekarang Saatnya Otomasi!

Di artikel sebelumnya, kita sudah berhasil **SSH ke server lokal dari mana saja lewat internet** menggunakan Cloudflare Tunnel. Sekarang server lokal kita bisa dijangkau dari mana saja. Tapi ada satu masalah lagi…

Bayangkan kamu punya project — bisa jadi aplikasi web, API, atau script kecil — yang kode-nya kamu taruh di GitLab. Setiap kali kamu update kode, kamu harus:

1. SSH ke server
2. `git pull` untuk ambil kode terbaru
3. Build aplikasi (kalau perlu)
4. Restart service

Ulangi itu **setiap kali** ada perubahan. Capek, kan? Apalagi kalau kamu deploy **beberapa kali sehari**.

Nah, di sinilah **CI/CD** masuk!

---

## Apa Itu CI/CD?

Sebelum praktik, pahami dulu konsepnya dengan analogi sederhana.

Bayangkan kamu punya **restoran**:

- **CI (Continuous Integration)** itu seperti **kitchen quality check** — setiap kali chef baru masak, makanannya otomatis dicek rasanya, kebersihannya, dan presentasinya. Kalau ada yang salah, langsung ditolak sebelum sampai ke pelanggan.
- **CD (Continuous Deployment/Delivery)** itu seperti **waiter otomatis** — begitu makanan lolos quality check, waiter langsung mengantarkannya ke meja pelanggan **tanpa perlu disuruh**.

Dalam dunia software:

- **CI** = Setiap kali kamu **push kode ke Git**, sistem otomatis menjalankan **test dan build** untuk memastikan kode tidak error
- **CD** = Kalau test dan build berhasil, sistem otomatis **deploy ke server** tanpa campur tangan kamu

Jadi alurnya jadi gini:
```
Kamu push kode → GitLab terima → Otomatis test & build → Otomatis deploy ke server → Selesai!
```

---

### Kenapa Pakai CI/CD di Server Lokal?

Beberapa orang mungkin berpikir, *"CI/CD itu kan buat perusahaan besar, buat apa di server lokal?"*

Eits, jangan salah! CI/CD itu **sangat berguna** bahkan untuk project kecil:

- 🚀 **Hemat waktu** — tidak perlu manual deploy setiap kali update
- 🔒 **Konsisten** — proses deploy selalu sama, tidak ada langkah yang terlewat
- 🐛 **Deteksi error lebih cepat** — kalau kode bermasalah, langsung ketahuan saat build
- 📋 **Riwayat deploy jelas** — tahu kapan dan apa yang di-deploy
- 😌 **Tenang** — push kode, duduk manis, server yang kerja

---

### Apa yang Kita Butuhkan?

Sebelum mulai, pastikan kamu sudah punya:

1. **Server lokal** yang sudah bisa di-SSH (lihat artikel sebelumnya tentang Cloudflare Tunnel)
2. **Project** yang sudah di-push ke GitLab
3. **Akses Maintainer/Owner** ke repository GitLab
4. **Terminal** di server lokal

---

### Langkah 1: Pilih Project yang Mau Diterapkan CI/CD

Pertama, buka GitLab dan pilih project yang mau kamu terapkan CI/CD. Bisa project baru atau project yang sudah ada.

![Pilih project yang mau diterapkan CI/CD](/img/cicd/2026-04-23_195114.png)

---

### Langkah 2: Buka Settings untuk Aktifkan CI/CD

Setelah masuk ke project, pergi ke bagian **Settings** lalu cari menu **CI/CD**. Di sinilah kita akan mengatur runner untuk project ini.

![Ke bagian Settings untuk aktifkan CI/CD](/img/cicd/2026-04-23_195327.png)

---

### Langkah 3: Buat Runner Baru

Di halaman CI/CD Settings, cari bagian **Runners** lalu klik untuk membuat runner baru. Runner ini yang nanti akan menjalankan pipeline CI/CD di server lokal kamu.

![Buat runner baru](/img/cicd/2026-04-23_195429.png)

---

### Langkah 4: Beri Nama Runner

Kasih nama untuk runner kamu. Misalnya `server-lokal` atau nama lain yang mudah dikenali. Nama ini akan muncul di GitLab supaya kamu tahu runner mana yang sedang bekerja.

![Kasih nama runner](/img/cicd/2026-04-23_195621.png)

---

### Langkah 5: Pilih Sistem Operasi Runner

Pilih sistem operasi di mana runner akan dijalankan. Sesuaikan dengan OS server lokal kamu — misalnya Linux, macOS, atau Windows.

![Pilih sistem operasi di mana menjalankan runner](/img/cicd/2026-04-23_195706.png)

---

### Langkah 6: Copy Perintah Instalasi Runner

GitLab akan menampilkan perintah untuk mendownload dan menginstall runner. **Copy perintah-perintah tersebut** — kita akan menjalankannya di server lokal.

![Copy perintah instalasi runner](/img/cicd/2026-04-23_195834.png)

---

#### Langkah 7: Jalankan Perintah di Server — Cek Runner Sudah Terinstall

SSH ke server lokal kamu, lalu jalankan perintah-perintah yang sudah di-copy tadi satu per satu. Setelah selesai, pastikan runner sudah berhasil terinstall:

```bash
gitlab-runner --version
```

![Check apa runner sudah terinstall](/img/cicd/2026-04-23_200026.png)

---

### Langkah 8: Paste Token untuk Registrasi Runner

Saat proses registrasi, kamu akan diminta memasukkan **token** yang kamu dapat dari GitLab. Token ini menghubungkan runner di server kamu dengan project di GitLab.

```bash
sudo gitlab-runner register
```

Masukkan token dan informasi yang diminta:
- **GitLab URL:** biasanya `https://gitlab.com`
- **Token:** paste token dari GitLab
- **Executor:** pilih `shell` (supaya bisa langsung menjalankan command di server)

![Paste token yang didapat di runner](/img/cicd/2026-04-23_200148.png)

---

### Langkah 9: Runner Berjalan!

Setelah registrasi berhasil, jalankan runner:

```bash
sudo gitlab-runner start
```

Kalau tidak ada error, runner sudah berjalan di server lokal kamu!

![Runner berjalan](/img/cicd/2026-04-23_200526.png)

---

### Langkah 10: Cek di GitLab — Pasti Ada New Runner!

Buka kembali halaman **Settings → CI/CD → Runners** di GitLab. Runner kamu seharusnya sudah muncul di sana dengan status aktif. Kalau muncul, berarti server lokal kamu sudah terhubung dengan GitLab!

![Check GitLab runner, pasti ada new runner!](/img/cicd/2026-04-23_200730.png)

---

### Langkah 11: Pastikan Runner Online dan Terdaftar

Sebelum lanjut, pastikan runner menunjukkan status **online** (ikon hijau). Kalau offline, coba restart runner di server:

```bash
sudo gitlab-runner restart
```

![Pastikan runner online dan sudah terdaftar](/img/cicd/2026-04-23_200926.png)

---

### Langkah 12: Buat Workflow File

Sekarang runner sudah siap. Saatnya membuat **workflow file** — file yang mendefinisikan apa yang harus dilakukan CI/CD setiap kali ada push.

Di root project kamu, buat file:

```
.gitlab-ci.yml
```

Contoh isi file:

```yaml
stages:
  - deploy

deploy:
  stage: deploy
  only:
    - main
  script:
    - cd /path/to/project
    - git pull origin main
    - # Tambahkan perintah build di sini, misalnya:
    - # npm install && npm run build
    - # atau: docker compose up -d --build
    - sudo systemctl restart aplikasi
  tags:
    - server-lokal
```

> **Penjelasan:**
> - `only: main` → Pipeline hanya berjalan saat ada push ke branch `main`
> - `script` → Command yang dijalankan di server lokal
> - `tags: server-lokal` → Pastikan pipeline dijalankan oleh runner yang sudah kita buat

![Buat workflow file .gitlab-ci.yml](/img/cicd/2026-04-23_201134.png)

---

### Langkah 13: Push ke GitLab

Setelah file `.gitlab-ci.yml` selesai, commit dan push ke GitLab:

```bash
git add .gitlab-ci.yml
git commit -m "Tambah CI/CD pipeline"
git push origin main
```

![Push ke GitLab](/img/cicd/2026-04-23_201655.png)

---

### Langkah 14: Pipeline Berjalan!

Buka project di GitLab dan pergi ke **CI/CD → Pipelines**. Kamu akan melihat pipeline sedang berjalan! GitLab otomatis mendeteksi file `.gitlab-ci.yml` dan mulai mengeksekusi perintah-perintah yang kita tulis.

![Pipeline berjalan](/img/cicd/2026-04-23_204150.png)

---

### Langkah 15: Lihat Detail Pipeline

Klik pipeline yang sedang berjalan untuk melihat detail. Kamu bisa lihat stage apa saja yang dijalankan dan status masing-masing — apakah sedang running, berhasil, atau gagal.

![Detail pipeline apa saja](/img/cicd/2026-04-23_204235.png)

---

### Langkah 16: Log di Server — Berhasil Dieksekusi

Klik job di pipeline untuk melihat **log output** secara real-time. Di sini kamu bisa lihat command apa saja yang dijalankan oleh runner di server lokal, dan apakah semuanya berhasil tanpa error.

![Log di server — runner berhasil eksekusi](/img/cicd/2026-04-23_204544.png)

---

#### Langkah 17: Pastikan Runner Bisa Menjalankan Command di Workflow

Penting untuk memastikan bahwa runner bisa menjalankan semua command yang ada di workflow. Kalau ada command yang gagal (misalnya permission denied), cek user yang menjalankan `gitlab-runner` dan pastikan punya akses yang cukup:

```bash
sudo usermod -aG docker gitlab-runner  # kalau pakai docker
sudo chown -R gitlab-runner:gitlab-runner /path/to/project
```

![Pastikan GitLab runner bisa menjalankan command di workflow](/img/cicd/2026-04-23_203044.png)

---

### Langkah 18: Lihat Status di Pipeline — Deploy

Kembali ke halaman pipeline, status deploy akan berubah menjadi **running** (icon biru berputar) saat sedang berjalan.

![Status di pipeline deploy](/img/cicd/2026-04-23_221412.png)

---

### Langkah 19: Deploy Selesai dan Berhasil! 🎉

Kalau semua langkah berhasil, status pipeline akan berubah menjadi **passed** (ikon hijau ✅). Artinya kode sudah berhasil di-deploy ke server lokal kamu secara otomatis!

![Deploy selesai dan berhasil!](/img/cicd/2026-04-23_222442.png)

---

### Langkah 20: Verifikasi Deploy di Server

Terakhir, verifikasi di server lokal bahwa kode terbaru sudah benar-benar ter-deploy. Cek file, versi aplikasi, atau langsung test aplikasinya.

![Verifikasi deploy berhasil pada server](/img/cicd/2026-04-23_203750.png)

---

## Kesimpulan

Dengan menerapkan CI/CD di server lokal menggunakan **GitLab Runner**, proses deploy yang tadinya manual dan membosankan sekarang jadi **sepenuhnya otomatis**.

Ringkasan langkah-langkahnya:
1. **Pilih project** di GitLab dan buka **Settings → CI/CD**
2. **Buat runner baru** dan kasih nama
3. **Copy perintah instalasi**, jalankan di server lokal
4. **Registrasi runner** dengan token dari GitLab
5. **Buat workflow** file `.gitlab-ci.yml`
6. **Push kode** dan biarkan pipeline yang bekerja!

Keuntungannya:

- ⏱️ **Hemat waktu** — push kode, sisanya otomatis
- 🔧 **Konsisten** — setiap deploy mengikuti proses yang sama
- 🐛 **Deteksi error cepat** — gagal build = gagal deploy, langsung ketahuan
- 📊 **Riwayat jelas** — semua deployment terekam di GitLab Pipelines
- 💰 **Gratis** — self-hosted runner tidak ada biaya tambahan

Dari artikel sebelumnya kita bisa **akses server lokal dari mana saja**, sekarang kita bisa **deploy dari mana saja** juga. Cukup push kode dari laptop, HP, atau komputer mana pun — server lokal kamu akan otomatis update sendiri. Praktis banget! 🚀