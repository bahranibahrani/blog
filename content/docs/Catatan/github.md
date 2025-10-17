---
title: "GIT CHEAT SHEET"
date: "2025-10-17"
lastmod: "2025-10-17"
author: "Nama Anda"
draft: false
tags: ["Git"]
categories: ["Pribadi"]
description: "Git"
showToc: true
ShowReadingTime: true
ShowWordCount: true
---

### **Git Cheatsheet untuk GitHub & GitLab**

Dokumen ini berisi kumpulan perintah Git yang paling sering digunakan, dari konfigurasi awal hingga sinkronisasi dengan remote repository di GitHub atau GitLab.

---

### **1. Konfigurasi Awal (Hanya perlu dilakukan sekali)**

Perintah ini untuk mengatur identitas Anda pada Git di mesin lokal.

```bash
# Mengatur nama pengguna Anda
git config --global user.name "Nama Anda"

# Mengatur email pengguna Anda (gunakan email yang terdaftar di GitHub/GitLab)
git config --global user.email "email@anda.com"
```

---

### **2. Membuat atau Mengambil Proyek**

```bash
# Inisialisasi repository baru di folder yang sudah ada
git init

# Clone (mengunduh) repository yang sudah ada di GitHub/GitLab
git clone [URL_REPOSITORY]
```
*   **URL_REPOSITORY**: Ganti dengan URL SSH atau HTTPS dari repository Anda.
    *   Contoh GitHub: `https://github.com/username/nama-repo.git`
    *   Contoh GitLab: `https://gitlab.com/username/nama-repo.git`

---

### **3. Sinkronisasi dengan Remote Repository**

Ini adalah bagian terpenting untuk berkolaborasi dan menjaga kode Anda tetap up-to-date.

```bash
# Menambahkan remote repository (biasanya bernama 'origin')
# Ini yang ada di catatan Anda.
git remote add origin [URL_REPOSITORY]

# Melihat daftar remote repository yang sudah terhubung
git remote -v

# Mengambil perubahan terbaru dari remote tanpa menggabungkannya (aman)
git fetch origin

# Menarik (pull) perubahan dari branch 'main' di remote 'origin'
# Opsi 1: Menggunakan merge (default, membuat commit merge)
git pull origin main

# Opsi 2: Menggunakan rebase (seperti di catatan Anda, membuat histori lebih bersih)
# Ini akan menempatkan perubahan lokal Anda di atas perubahan dari remote.
git pull origin main --rebase

# Mendorong (push) perubahan dari branch lokal ke remote
# Perintah ini mendorong branch 'main' dan mengaturnya agar terhubung (-u)
git push -u origin main

# Push biasa setelah koneksi diatur
git push origin main

# Mendorong (push) secara paksa. HATI-HATI!
# Ini akan menimpa histori di remote. Hanya gunakan jika Anda tahu apa yang Anda lakukan.
# '-f' adalah singkatan dari '--force'.
git push -f origin main
```

---

### **4. Bekerja dengan Perubahan (Workflow Harian)**

```bash
# Melihat status perubahan di working directory
git status

# Menambahkan satu file ke staging area (area persiapan sebelum commit)
git add [NAMA_FILE]

# Menambahkan semua perubahan di direktori saat ini ke staging area
git add .

# Menyimpan perubahan yang ada di staging area ke dalam sebuah 'commit'
git commit -m "Pesan commit yang deskriptif"
```

---

### **5. Bekerja dengan Branch**

Branch memungkinkan Anda untuk bekerja pada fitur baru atau perbaikan bug secara terisolasi.

```bash
# Melihat semua branch yang ada (lokal dan remote)
git branch -a

# Membuat branch baru
git branch [NAMA_BRANCH_BARU]

# Pindah ke branch lain
git checkout [NAMA_BRANCH]

# Membuat dan langsung pindah ke branch baru (shortcut)
git checkout -b [NAMA_BRANCH_BARU]

# Mengganti nama branch saat ini (seperti di catatan Anda)
# '-M' akan mengganti nama secara paksa, bagus untuk branch 'master' ke 'main'.
git branch -M main

# Menggabungkan perubahan dari branch lain ke branch saat ini
git merge [NAMA_BRANCH_YANG_AKAN_DIGABUNG]

# Menghapus branch lokal (setelah digabungkan)
git branch -d [NAMA_BRANCH]
```

---

### **6. Melihat Riwayat & Mengurungkan Perubahan**

```bash
# Melihat riwayat commit
git log

# Melihat riwayat commit dalam format yang lebih ringkas
git log --oneline --graph

# Mengurungkan perubahan pada file di working directory (sebelum di-commit)
git checkout -- [NAMA_FILE]

# Membuat commit baru yang merupakan kebalikan dari commit sebelumnya (aman)
git revert HEAD

# Reset ke commit tertentu dan menghapus semua perubahan setelahnya (berbahaya)
# Gunakan dengan hati-hati, terutama pada branch yang sudah di-push.
git reset --hard [HASH_COMMIT]
```