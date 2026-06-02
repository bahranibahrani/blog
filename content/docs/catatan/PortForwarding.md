---
title: "Port Forwarding"
date: 2026-06-02T19:13:53+07:00
draft: false
description: "Panduan lengkap tentang Port Forwarding, NAT, CGNAT, dan Port Mapping pada router"
tags: ["networking", "port-forwarding", "NAT", "router"]
categories: ["catatan"]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---

## Apa Itu Port Forwarding?

**Port Forwarding** adalah teknik jaringan yang memungkinkan perangkat dari luar jaringan lokal (internet) untuk mengakses layanan/server yang berjalan di dalam jaringan lokal kamu. Secara sederhana, port forwarding adalah cara **"membuka pintu"** tertentu di router agar traffic dari luar bisa diteruskan ke perangkat spesifik di jaringan internal.

---

## Kenapa Perlu Port Forwarding?

Karena router kamu menggunakan **NAT (Network Address Translation)**. 

### Bagaimana NAT Bekerja?

- **IP Publik** → dimiliki oleh router (diberikan oleh ISP), ini adalah alamat yang terlihat dari internet.
- **IP Private** → digunakan oleh perangkat di dalam jaringan lokal kamu (contoh: `192.168.x.x`, `10.x.x.x`).

NAT bertugas menerjemahkan IP Private ke IP Publik saat perangkat lokal mengakses internet. Namun, **secara default traffic dari luar tidak bisa masuk** ke perangkat lokal karena router tidak tahu harus meneruskan ke perangkat mana.

**Port Forwarding** menyelesaikan masalah ini dengan memberi tahu router:
> *"Jika ada traffic masuk di port X, teruskan ke perangkat Y di jaringan lokal pada port Z."*

### Contoh Penggunaan:
- Mengakses **web server** lokal dari internet
- Menjalankan **game server** yang bisa diakses teman
- Remote access ke **CCTV/IP Camera** dari luar rumah
- Mengakses **NAS (Network Attached Storage)** dari mana saja
- Menjalankan **SSH server** agar bisa remote ke komputer rumah

---

## Masalah CGNAT (Carrier Grade NAT)

### Apa Itu CGNAT?

Banyak ISP di Indonesia seperti **Telkom IndiHome, Biznet, MyRepublic, XL, Smartfren**, dan lainnya menggunakan **CGNAT (Carrier Grade NAT)**.

CGNAT adalah lapisan NAT tambahan yang dilakukan di sisi ISP. Artinya, sebelum traffic sampai ke router kamu, sudah ada proses NAT di level ISP.

### Dampak CGNAT:

- ❌ Kamu **tidak bisa melakukan Port Forwarding secara normal**
- ❌ Bahkan kalau sudah di-forward di router kamu, **orang luar tetap tidak bisa mengakses**
- ❌ **IP Publik** yang kamu dapat **dipakai bersama** dengan pelanggan lain (shared IP)

### Cara Mengecek Apakah ISP Kamu Pakai CGNAT

**Langkah 1:** Cek IP Publik kamu dari internet

Buka [whatismyipaddress.com](https://whatismyipaddress.com) atau jalankan perintah:

```bash
curl ifconfig.me
```

![Check IP Publik](/img/port/2026-06-02_182517.png)

**Langkah 2:** Cek IP yang tertera di modem/router kamu

Login ke halaman admin router (biasanya `192.168.1.1` atau `192.168.0.1`), lalu lihat IP WAN.

![Check IP Publik di Router](/img/port/2026-06-02_182553.png)

**Kesimpulan:**
- Jika **IP Publik dari internet = IP WAN di router** → ✅ Kamu **tidak kena CGNAT**, port forwarding bisa dilakukan
- Jika **IP Publik dari internet ≠ IP WAN di router** → ❌ Kamu **kena CGNAT**, perlu solusi alternatif

> **Solusi jika kena CGNAT:**
> - Hubungi ISP untuk minta **IP Publik statis/dinamis** (beberapa ISP menyediakan dengan biaya tambahan)
> - Gunakan layanan **tunneling** seperti Cloudflare Tunnel, Ngrok, atau Tailscale
> - Gunakan **VPN** dengan port forwarding (contoh: WireGuard pada VPS)

---

## Tutorial Port Forwarding di Router

Jika kamu sudah memastikan tidak kena CGNAT, berikut langkah-langkah melakukan port forwarding:

### Langkah 1: Masuk ke Menu Forward Rules

Login ke halaman admin router, lalu navigasi ke menu **Forward Rules** atau **Port Forwarding** (nama menu bisa berbeda tergantung merek router).

![Menu Forward Rules](/img/port/2026-06-02_182652.png)

### Langkah 2: Buat Rule Forward

Buat rule baru untuk menentukan:

![Buat Rule Forward](/img/port/2026-06-02_182705.png)

### Langkah 3: Verifikasi Sebelum Port Forward

Sebelum port forwarding aktif, lakukan scan dengan **nmap** ke IP Publik untuk melihat kondisi awal:

```bash
nmap <IP_PUBLIK_KAMU>
```

![Nmap Sebelum Port Forward](/img/port/2026-06-02_182736.png)

Terlihat port yang dituju masih **tertutup/filtered** karena belum ada rule forwarding.

### Langkah 4: Verifikasi Setelah Port Forward

Setelah rule forwarding diaktifkan, scan ulang dengan nmap:

```bash
nmap <IP_PUBLIK_KAMU>
```

![Nmap Setelah Port Forward](/img/port/2026-06-02_183021.png)

Sekarang port yang di-forward sudah terlihat **open** ✅

### Langkah 5: Akses dari Luar

Coba akses layanan melalui IP Publik dari jaringan luar (bisa pakai data seluler atau minta teman mengecek):

![Akses IP Publik Berhasil](/img/port/2026-06-02_183138.png)

🎉 **Berhasil!** Layanan di jaringan lokal sudah bisa diakses dari internet.

---

## Port Mapping (Advance)

**Port Mapping** adalah fitur lanjutan yang memungkinkan kamu memetakan port eksternal ke port internal yang **berbeda**. Ini berguna ketika:

- Kamu ingin menyembunyikan port asli layanan (keamanan)
- Menjalankan beberapa layanan serupa di port yang berbeda
- Port standar sudah digunakan oleh layanan lain

### Langkah 1: Masuk ke Menu Port Mapping

Navigasi ke menu **Port Mapping** pada router (biasanya ada di bagian Advanced/NAT settings).

![Menu Port Mapping](/img/port/advance/2026-06-02_190531.png)

### Langkah 2: Deploy Port Mapping

Konfigurasikan port mapping sesuai kebutuhan, lalu simpan dan terapkan konfigurasi.
- **Port eksternal** → port yang dibuka dari sisi internet
- **IP tujuan** → IP perangkat lokal yang ingin diakses
- **Port internal** → port layanan di perangkat lokal
- **Protokol** → TCP, UDP, atau keduanya

![Deploy Port Mapping](/img/port/advance/2026-06-02_190644.png)

### Langkah 3: Verifikasi Port Mapping

Cek apakah port mapping sudah terbaca dengan benar menggunakan **nmap**:

```bash
nmap <IP_PUBLIK_KAMU> -p <PORT_YANG_DIMAPPING>
```

![Verifikasi Port Mapping dengan Nmap](/img/port/advance/2026-06-02_190721.png)

Port mapping berhasil diterapkan dan terdeteksi oleh nmap ✅

---

## Ringkasan

| Aspek | Keterangan |
|-------|-----------|
| **Port Forwarding** | Membuka port di router agar traffic luar bisa masuk ke perangkat lokal |
| **NAT** | Menerjemahkan IP Private ↔ IP Publik |
| **CGNAT** | NAT tambahan di sisi ISP, menghalangi port forwarding normal |
| **Cek CGNAT** | Bandingkan IP di `whatismyipaddress.com` dengan IP WAN di router |
| **Port Mapping** | Memetakan port eksternal ke port internal yang berbeda |
| **Tool Verifikasi** | Gunakan `nmap` untuk memastikan port sudah terbuka |

> ⚠️ **Tips Keamanan:**
> - Hanya forward port yang benar-benar diperlukan
> - Gunakan port non-standar jika memungkinkan
> - Pastikan layanan yang di-forward sudah diamankan (password, firewall, SSL/TLS)
> - Monitor log akses secara berkala
