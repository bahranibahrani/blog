---
title: "Self-Hosting Made Easy: Konfigurasi Home Server Menggunakan Domain Cloudflare"
date: 2025-09-03T10:46:27+07:00
draft: true
showToc: true
searchHidden: false
UseHugoToc: true
  - Catatan
tags:
  - "Cloudflare"
  - "Tutorial"
#![Image](https://blog.bahrani.my.id/img/1.png)
#![Image](/static/img/1.png)
#![Image](https://www.greenheirloom.in/cdn/shop/products/Bharani2_1024x1024@2x.jpg?v=1608273224)

---


## Pendahuluan

![Image](/static/img/1.png)
![images](/static/img/1.png)
Self-hosting menjadi semakin populer bagi para penggemar teknologi yang ingin memiliki kontrol penuh atas data dan aplikasi mereka. Dengan self-hosting, Anda bisa menjalankan berbagai layanan, mulai dari situs web pribadi, cloud storage, hingga media server, langsung dari perangkat keras di rumah Anda. Namun, salah satu tantangan umum adalah mengekspos server rumah Anda ke internet dengan aman dan tanpa IP statis.

Di sinilah Cloudflare Tunnel datang untuk menyelamatkan. Cloudflare Tunnel (sebelumnya dikenal sebagai Argo Tunnel) memungkinkan Anda menghubungkan server Anda ke jaringan Cloudflare tanpa harus membuka port pada router atau memiliki alamat IP publik yang statis. Ini menciptakan koneksi yang aman dan terenkripsi antara server Anda dan Cloudflare, melindungi Anda dari serangan langsung.

Dalam tutorial ini, kita akan membahas langkah-langkah untuk mengkonfigurasi server rumah (home server) dan menghubungkannya ke domain Anda menggunakan Cloudflare Tunnel.

## Prasyarat

Sebelum memulai, pastikan Anda memiliki:

1.  **Perangkat Server**: Komputer atau perangkat apa pun yang akan Anda jadikan server (misalnya, Raspberry Pi, PC lama, atau VM).
2.  **Domain Terdaftar**: Anda harus memiliki nama domain yang sudah terdaftar (misalnya, `namadomainanda.com`).
3.  **Akun Cloudflare**: Akun gratis di [Cloudflare](https://www.cloudflare.com/).


## Langkah 1: Tambahkan Domain Anda ke Cloudflare

Jika Anda belum melakukannya, langkah pertama adalah menambahkan domain Anda ke akun Cloudflare Anda.

1.  Login ke dashboard Cloudflare.
2.  Klik **"Add a site"** dan masukkan nama domain Anda.
3.  Pilih paket **Free**.
4.  Cloudflare akan memindai catatan DNS Anda yang ada. Setelah selesai, klik **Continue**.
5.  Anda akan diberikan dua nameserver Cloudflare. Perbarui nameserver di registrar domain Anda (tempat Anda membeli domain) dengan yang diberikan oleh Cloudflare.

Perubahan nameserver bisa memakan waktu hingga 24 jam untuk diterapkan sepenuhnya.

## Langkah 2: Instal `cloudflared`

`cloudflared` adalah perangkat lunak command-line yang akan menjalankan Tunnel di server Anda. Instal `cloudflared` pada perangkat yang akan menjadi server Anda.

Anda dapat mengunduhnya dari [halaman unduhan resmi Cloudflare](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/).

-   **Untuk Linux (Debian/Ubuntu)**:

    ```bash
    curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
    sudo dpkg -i cloudflared.deb
    ```

-   **Untuk Windows**:

    Unduh file `.msi` dan jalankan installer.

-   **Untuk macOS**:

    ```bash
    brew install cloudflare/cloudflare/cloudflared
    ```

## Langkah 3: Autentikasi `cloudflared`

Selanjutnya, Anda perlu menghubungkan `cloudflared` dengan akun Cloudflare Anda.

1.  Buka terminal atau command prompt di server Anda.
2.  Jalankan perintah berikut:

    ```bash
    cloudflared tunnel login
    ```

3.  Sebuah jendela browser akan terbuka. Login dengan akun Cloudflare Anda dan pilih domain yang ingin Anda gunakan.
4.  Setelah otorisasi berhasil, sebuah file sertifikat (`cert.pem`) akan diunduh ke direktori default `cloudflared` (misalnya, `~/.cloudflared/` di Linux).

## Langkah 4: Buat dan Konfigurasi Tunnel

Sekarang kita akan membuat Tunnel dan mengkonfigurasinya untuk mengarahkan lalu lintas ke layanan lokal di server Anda.

1.  **Buat Tunnel**:

    Jalankan perintah ini untuk membuat Tunnel. Ganti `my-tunnel` dengan nama yang Anda inginkan.

    ```bash
    cloudflared tunnel create my-tunnel
    ```

    Perintah ini akan menghasilkan file kredensial JSON (misalnya, `<UUID>.json`) di direktori `~/.cloudflared/`. Simpan UUID Tunnel yang ditampilkan, karena Anda akan membutuhkannya.

2.  **Buat File Konfigurasi**:

    Buat file `config.yml` di dalam direktori `~/.cloudflared/`. File ini akan menentukan bagaimana lalu lintas diarahkan.

    ```yaml
    tunnel: <UUID_TUNNEL_ANDA>
    credentials-file: /root/.cloudflared/<UUID_TUNNEL_ANDA>.json

    ingress:
      - hostname: myapp.namadomainanda.com
        service: http://localhost:8080
      # Tambahkan aturan lain di sini
      - service: http_status:404
    ```

    -   Ganti `<UUID_TUNNEL_ANDA>` dengan UUID yang Anda dapatkan sebelumnya.
    -   `hostname`: Subdomain yang akan Anda gunakan.
    -   `service`: Alamat lokal layanan Anda. Contoh di atas mengarahkan lalu lintas ke server web yang berjalan di `localhost` pada port `8080`.
    -   Baris terakhir (`service: http_status:404`) adalah aturan *catch-all* untuk mencegah permintaan ke hostname yang tidak terdefinisi terekspos.

3.  **Hubungkan DNS ke Tunnel**:

    Jalankan perintah berikut untuk membuat catatan DNS CNAME di Cloudflare yang mengarah ke Tunnel Anda.

    ```bash
    cloudflared tunnel route dns my-tunnel myapp.namadomainanda.com
    ```

## Langkah 5: Jalankan Tunnel

Anda dapat menjalankan Tunnel secara manual untuk pengujian:

```bash
cloudflared tunnel run my-tunnel
```

Namun, cara terbaik adalah menjalankannya sebagai layanan (service) agar tetap berjalan bahkan setelah server di-reboot.

1.  **Instal Tunnel sebagai Layanan**:

    ```bash
    sudo cloudflared service install
    ```

2.  **Mulai Layanan**:

    Layanan akan dimulai secara otomatis. Anda bisa memeriksanya dengan:

    ```bash
    sudo systemctl start cloudflared
    sudo systemctl status cloudflared
    ```

Sekarang, jika Anda membuka `myapp.namadomainanda.com` di browser, permintaan akan diarahkan dengan aman melalui Cloudflare ke layanan yang berjalan di `localhost:8080` di server rumah Anda!

## Kesimpulan

Dengan Cloudflare Tunnel, mengkonfigurasi home server menjadi jauh lebih mudah dan aman. Anda tidak perlu lagi khawatir tentang IP dinamis, konfigurasi port forwarding yang rumit, atau mengekspos server Anda secara langsung ke internet. Selamat mencoba dunia self-hosting!