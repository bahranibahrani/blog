---
title: "Cisco Software-Defined Networking"
date: 2026-04-22T14:08:33+07:00
draft: true
description: ""
tags: ["SDN","Cisco"]
categories: [catatan]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---
## Software-Defined Networking (SDN)
Pernahkah kalian merasa pusing mengelola ratusan bahkan ribuan perangkat jaringan yang harus di-setting satu per satu? Nah, kalau kalian masih memakai cara lama ini, saatnya kalian kenalan dengan SDN (Software-Defined Networking)
### Apa sih SDN itu?
Bayangkan sebuah kota besar di mana setiap lampu merah bekerja secara individual hanya mengandalkan timer statis. Inilah gambaran **jaringan tradisional**. Terkadang jalanan sudah sepi, namun lampu tetap merah dalam waktu lama, sehingga menciptakan kemacetan yang tidak perlu karena setiap lampu tidak tahu kondisi jalan di blok sebelah.

**Software-Defined Networking (SDN)** hadir sebagai **"Pusat Komando Cerdas"**. Alih-alih membiarkan setiap lampu merah menebak-nebak, kita memiliki satu pengawas pusat yang bisa melihat seluruh kondisi kota secara *real-time*. Jika satu jalur padat, pusat komando akan langsung mengubah lampu menjadi hijau secara otomatis.

### Perbedaan Struktur: Tradisional vs SDN

Dalam dunia jaringan, terdapat dua elemen kunci yang menentukan cara kerja sebuah perangkat:
1.  **Control Plane**: Si "Otak" atau pengambil keputusan (menentukan jalur mana yang harus dilewati data).
2.  **Data Plane**: Si "Otot" atau penerus paket data (tugasnya hanya mengirimkan data sesuai perintah).

#### 1. Jaringan Tradisional
Pada jaringan konvensional, **Control Plane** dan **Data Plane** digabung di dalam satu perangkat fisik (seperti router atau switch). 
* **Masalahnya:** Setiap perangkat harus berpikir sendiri-sendiri. Jika ada perubahan kebijakan, administrator harus mengonfigurasi perangkat satu per satu.

#### 2. Jaringan SDN (Software-Defined Networking)
SDN melakukan revolusi dengan **memisahkan** kedua fungsi tersebut:
* **Control Plane (Terpusat):** "Otak" ditarik keluar dari perangkat fisik dan dipindahkan ke sebuah software terpusat yang disebut **Controller**.
* **Data Plane (Terdistribusi):** Perangkat fisik (switch/router) kini hanya fokus menjadi "Otot" yang bertugas meneruskan data sesuai instruksi dari Controller.