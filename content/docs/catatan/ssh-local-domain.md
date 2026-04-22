---
title: "Server Local Bisa Di-SSH Lewat Domain"
date: 2026-04-23T02:06:37+07:00
draft: false
description: "Cara mengakses server lokal dari mana saja lewat internet menggunakan Cloudflare Zero Trust Tunnel. Server di rumah, akses dari mana saja via domain."
tags: ["SSH", "Cloudflare", "Zero Trust", "Tunnel", "Server"]
categories: ["Catatan"]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---
## Pernah Bingung Bagaimana Cara Akses Server Lokal dari Jauh?

Bayangkan kamu punya server lokal di rumah atau di kantor. Server itu jalan baik, tapi masalahnya — **kamu cuma bisa akses saat berada di jaringan yang sama**. Mau akses dari kafe, dari rumah teman, atau dari mana saja lewat internet? **Tidak bisa.**

Secara default, server lokal itu "tersembunyi" di balik jaringan lokal (LAN). Dia tidak punya alamat publik yang bisa dijangkau dari internet. Belum lagi kalau kamu pakai ISP rumahan yang ganti-ganti IP publik (dynamic IP), atau bahkan di-NAT sehingga kamu tidak bisa langsung port-forward.

Nah, bayangkan kalau kamu bisa akses server lokal itu dengan nama domain, dari **mana saja di dunia ini**, selama ada koneksi internet:

```bash
ssh user@serverkamu.com
```

Keren, kan? Di artikel ini, kita akan bahas cara membuatnya terwujud menggunakan **Cloudflare Zero Trust Tunnel**.

![Konsep akses server lokal via internet](/img/ssh/2026-04-23_014505.png)

---

## Konsep Dasar: Cloudflare Zero Trust Tunnel

Sebelum masuk ke praktik, mari kita pahami konsepnya dengan analogi sederhana.

Bayangkan server lokal kamu itu seperti **rumah pribadi yang terletak di gang kecil** — tidak ada alamat resmi yang bisa dicari di Google Maps. Teman kamu yang mau berkunjung pasti bingung nyarinya.

**Cloudflare Tunnel** itu seperti kamu memasang **"portal ajaib"** di dalam rumah kamu. Portal ini terhubung langsung ke **Cloudflare** (sebuah platform besar yang punya alamat jelas di internet). Jadi teman kamu tidak perlu tahu di mana rumah kamu yang sebenarnya — mereka cukup datang ke alamat Cloudflare, dan Cloudflare akan meneruskan mereka melalui portal ajaib itu langsung ke rumah kamu.

### Gimana Cara Kerjanya?

1. **Cloudflare Tunnel** (`cloudflared`) berjalan di server lokal kamu
2. Tunnel ini membuat koneksi **outbound** (keluar) dari server kamu menuju jaringan Cloudflare
3. Cloudflare memberikan kamu sebuah **domain** (misalnya `serverkamu.com`)
4. Ketika seseorang mengakses domain itu, Cloudflare meneruskan traffic-nya melalui tunnel ke server lokal kamu
5. **Tidak perlu buka port**, tidak perlu IP publik static, dan tidak perlu setting port-forwarding di router!


---

## Apa yang Kita Butuhkan?

Sebelum mulai, pastikan kamu sudah punya:

1. **Server lokal** yang sudah berjalan dan terhubung ke internet (tidak perlu IP publik!)
2. **SSH Server** sudah terinstall di server tersebut
3. **Akun Cloudflare** (gratis bisa!) dengan sebuah domain yang sudah ditambahkan
4. **Akses terminal** di server lokal untuk menginstall `cloudflared`

> **Keuntungan besar:** Karena tunnel bekerja secara **outbound**, kamu **tidak perlu** membuka port di router, tidak perlu IP publik static, dan server kamu tetap aman karena tidak terekspos langsung ke internet.


---

## Langkah 1: Setup Cloudflare Zero Trust

Pertama, kita perlu setup Cloudflare Zero Trust. Login ke dashboard Cloudflare, lalu:

1. Buka menu **Zero Trust** di sidebar
2. Buat sebuah **Team Name** untuk organisasi kamu
3. Pilih plan **Free** (sudah cukup untuk kebutuhan ini)


---

## Langkah 2: Buat Tunnel

Setelah Zero Trust aktif, saatnya membuat tunnel:

1. Di dashboard Zero Trust, buka **Networks → Tunnels**
2. Klik **Create a tunnel**
3. Pilih tipe **Cloudflared**
4. Beri nama tunnel kamu, misalnya `server-lokal`
5. Cloudflare akan memberikan sebuah **token** — simpan baik-baik token ini!

---

## Langkah 3: Install Cloudflared di Server Lokal

Sekarang kita install `cloudflared` di server lokal. SSH ke server kamu (masih lewat jaringan lokal) dan jalankan:

### Di Ubuntu/Debian:
```bash
sudo apt update
sudo apt install cloudflared
```

Atau kalau tidak tersedia di repository, bisa install manual:
```bash
# Download dan install
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb -o cloudflared.deb
sudo dpkg -i cloudflared.deb
```

### Di CentOS/RHEL:
```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.rpm -o cloudflared.rpm
sudo rpm -i cloudflared.rpm
```

---

## Langkah 4: Jalankan Tunnel di Server

Setelah `cloudflared` terinstall, saatnya menghubungkan server ke Cloudflare. Gunakan token yang kamu dapat dari Langkah 2:

```bash
sudo cloudflared service install eyJh...token_kamu_disini...
```

Perintah ini akan mendaftarkan `cloudflared` sebagai service yang berjalan otomatis di background.

Cek status tunnel:
```bash
sudo systemctl status cloudflared
```
Pastikan statusnya **active (running)**. Kalau iya, berarti server lokal kamu sudah terhubung ke Cloudflare! 🎉

---

## Langkah 5: Konfigurasi Public Hostname (Route SSH)

Sekarang kita perlu memberi tahu Cloudflare kemana harus meneruskan traffic SSH. Kembali ke dashboard Cloudflare Zero Trust:

1. Di halaman tunnel yang sudah dibuat, klik **Configure**
![Klik Configure pada tunnel](/img/ssh/2026-04-23_014643.png)

2. Buka tab **Public Hostname**
![Tab Public Hostname](/img/ssh/2026-04-23_015007.png)

3. Klik **Add a public hostname**

4. Isi konfigurasi:
   - **Subdomain:** `ssh` (atau sesuai keinginan kamu)
   - **Domain:** pilih domain kamu yang sudah terdaftar di Cloudflare
   - **Service Type:** pilih `SSH`
   - **URL:** `localhost:22` (atau IP lokal server dan port SSH kamu)
![Isi konfigurasi hostname](/img/ssh/2026-04-23_015131.png)

5. Klik **Save**

Sekarang domain `ssh.domainkamu.com` sudah terhubung ke server SSH lokal kamu melalui tunnel!
![Hostname berhasil disimpan](/img/ssh/2026-04-23_015338.png)

---

## Langkah 6: Test SSH Pertama — Masih Gagal! 😅

Dari komputer client, coba akses server menggunakan domain:

```bash
ssh user@ssh.domainkamu.com
```

Tapi... hasilnya malah **gagal**. Kenapa?

![SSH pertama gagal](/img/ssh/2026-04-23_015627.png)

Tenang, ini **normal**! Cloudflare Tunnel untuk SSH tidak bisa langsung dipakai seperti SSH biasa. Kita perlu mengatur **ProxyCommand** di komputer client supaya SSH tahu harus melewati Cloudflare Tunnel terlebih dahulu. Tanpa ProxyCommand, SSH client tidak mengerti cara merutekan koneksi melalui Cloudflare.

---

## Langkah 7: Konfigurasi ProxyCommand di SSH Client

Supaya SSH bisa lewat Cloudflare Tunnel, kita perlu edit file **SSH Config** di komputer client (bukan di server):

```bash
nano ~/.ssh/config
```

Tambahkan konfigurasi berikut:

```
Host ssh.domainkamu.com
    ProxyCommand cloudflared access ssh --hostname %h
```

Apa itu **ProxyCommand**? Singkatnya, ini memberi tahu SSH: *"Sebelum koneksi ke server, lewati dulu `cloudflared` biar bisa tembus ke server lokal."*

Jadi setiap kali kamu SSH ke `ssh.domainkamu.com`, komputer akan otomatis menjalankan `cloudflared access ssh` sebagai perantara.

> **Catatan:** Pastikan `cloudflared` juga sudah terinstall di komputer **client** yang kamu gunakan untuk SSH. Cara install-nya sama seperti di Langkah 3.

![Konfigurasi ProxyCommand](/img/ssh/2026-04-23_015902.png)

---

## Langkah 8: SSH Ulang — Sekarang Berhasil! 🎉

Sekarang coba SSH lagi dari komputer client:

```bash
ssh user@ssh.domainkamu.com
```

Kali ini SSH berjalan melalui Cloudflare Tunnel, dan koneksi berhasil masuk!

![SSH berhasil melalui tunnel](/img/ssh/2026-04-23_020028.png)

Kamu akan diminta memasukkan password server lokal seperti biasa:

![Masukkan password](/img/ssh/2026-04-23_020117.png)

Dan... **berhasil masuk!** 🎉

![Berhasil login ke server](/img/ssh/2026-04-23_020236.png)

Server lokal di rumah kamu sekarang bisa diakses dari **mana saja di dunia** melalui internet, menggunakan nama domain. Tidak perlu buka port, tidak perlu IP publik static, dan koneksi terenkripsi end-to-end oleh Cloudflare.

![Berhasil SSH ke server lokal via domain](/img/ssh/2026-04-23_020320.png)

---

## Kesimpulan

Dengan **Cloudflare Zero Trust Tunnel**, server lokal kamu yang tadinya hanya bisa diakses dari jaringan lokal, sekarang bisa diakses dari **mana saja lewat internet** menggunakan domain.

Langkah-langkahnya ringkas:
1. Setup **Cloudflare Zero Trust** dan buat **Tunnel**
2. Install **cloudflared** di server lokal dan jalankan tunnel
3. Konfigurasi **Public Hostname** dengan tipe SSH
4. Di komputer client, tambahkan **ProxyCommand** di SSH Config
5. SSH via domain — **berhasil!**

Keuntunganya banyak banget:

- 🌐 **Akses dari mana saja** — tidak perlu lagi berada di jaringan yang sama
- 🔒 **Aman** — tidak perlu buka port di router, server tidak terekspos langsung ke internet
- 💰 **Gratis** — Cloudflare Zero Trust plan free sudah mencukupi
- 🏠 **IP dynamic tidak masalah** — tunnel bekerja secara outbound, jadi ganti-ganti IP pun tetap jalan
- ⚡ **Mudah diingat** — pakai domain, bukan IP address

Konsepnya simpel: **pasang Cloudflare Tunnel di server lokal, arahkan domain ke SSH, tambah ProxyCommand di client, dan selesai!** Dampaknya besar banget buat produktivitas, terutama kalau kamu sering bekerja remote dan butuh akses ke server di rumah atau kantor.
