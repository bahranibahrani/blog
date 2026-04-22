---
title: "Promo Domain Berkahost"
date: "2026-02-03"
draft: false
tags: ["Domain"]
categories: ["Pribadi"]
showToc: true
hidemeta: false
comments: false
description: "Kapan Lagi Promo domain my.id web.id dan biz.id hanya 3000 Pertahun"
disableHLJS: true # to disable highlightjs
disableShare: false
#disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
## Apa itu Domain?
Pernahkah kamu ingin mengunjungi sebuah website dan mengetikkan alamat seperti google.com atau tokopedia.com di browser? Nah, alamat inilah yang disebut domain.

Secara sederhana, domain adalah alamat unik di internet yang digunakan untuk mempermudah kita mengakses sebuah situs web. Tanpa domain, kita harus mengetik IP address sekumpulan angka seperti 192.168.1.1 agar bisa membuka suatu website. Bayangkan kalau setiap kali mau buka YouTube kamu harus mengingat angka panjang seperti itu. Pasti repot, kan?

Di sinilah domain berperan, karena sistem yang disebut DNS (Domain Name System) akan menerjemahkan nama domain menjadi alamat IP. Prinsip kerjanya mirip seperti mencari alamat rumah: kalau kamu ingin mengirim paket ke teman, kamu cukup tahu alamatnya tidak perlu tahu koordinat GPS-nya.

Jadi, bisa dibilang domain adalah “alamat rumah” bagi website di dunia digital. Ia ada agar kita, manusia, bisa mengingat dan menemukan situs web dengan mudah, tanpa perlu menghafal deretan angka yang membingungkan.


## Berkahost

Awalnya saya juga tidak tahu ada penyedia layanan bernama **Berkahost**. Selama ini yang sering saya dengar justru nama-nama besar seperti DomaiNesia, Rumahweb, Niagahoster, IDwebhost, JagoanHosting, Dewaweb, atau IDCloudHost yang sudah lebih dulu populer di Indonesia. Setelah saya telusuri, ternyata Berkahost adalah layanan web hosting dan domain asal Serang, Banten, yang fokus menawarkan paket murah untuk pemula, termasuk domain dan hosting dengan harga terjangkau. [berkahost](https://www.berkahost.com)
<div style="display: flex; justify-content: space-between; align-items: center;">
  <img src="/img/domain/berkah.jpeg" alt="Promo Berkahost" title="Promo" style="width: 45%; height: auto;">
  <img src="/img/domain/pembelian.jpeg" alt="Promo Berkahost" title="Promo" style="width: 45%; height: auto;">
</div>

Yang membuat saya tertarik adalah adanya promo domain dengan harga yang sangat rendah, sehingga dengan modal beberapa ribu rupiah saja kamu sudah bisa punya alamat domain sendiri. Promo seperti ini biasanya dibatasi untuk ekstensi tertentu atau periode tertentu, tapi sudah lebih dari cukup kalau kamu hanya ingin:

- Iseng bikin blog pribadi.
- Punya domain untuk main-main dengan homeserver di rumah.
- Menghubungkan domain ke GitHub Pages atau layanan static site lain.
- Mencoba-coba proyek kecil tanpa keluar biaya besar.

Dengan memiliki domain sendiri, kamu bisa melakukan **customisasi alamat** sesuai keinginan, misalnya memakai nama brand pribadi, nama proyek, atau sekadar username favoritmu. Rasanya mirip seperti punya “nama jalan” sendiri di internet: lebih rapi, lebih mudah diingat, dan terlihat lebih serius, meskipun proyeknya masih sebatas eksperimen.

## Setelah Pembelian
Setelah domainnya berhasil saya beli, pengelolaan DNS yang sebelumnya berada di Berkahost langsung saya pindahkan ke Cloudflare. Langkah ini saya ambil karena paket yang saya beli hanya berisi domain saja tanpa hosting dan tanpa sertifikat SSL bawaan, sehingga saya membutuhkan tempat pengelolaan DNS yang lebih fleksibel.

Saya memilih Cloudflare karena sebagian besar domain saya juga sudah dikelola di sana, dan dashboard-nya menurut saya sangat nyaman untuk kustomisasi record DNS, pointing ke server lokal, hingga integrasi dengan berbagai layanan lain. Cloudflare sendiri menyediakan DNS yang cepat, mudah diatur, dan dapat digunakan meskipun hosting berada di tempat lain. Selain itu, pada plan gratis saja Cloudflare sudah memberikan fitur penting seperti SSL/TLS gratis, perlindungan dasar terhadap DDoS, serta CDN untuk mempercepat akses konten.

Bagi saya, Cloudflare menjadi semacam “pusat kontrol” untuk beberapa proyek pribadi: mulai dari blog sederhana, website statis, sampai server lokal yang saya expose dengan berbagai trik networking. Tingkat keamanannya sudah cukup nyaman untuk kebutuhan personal, apalagi dengan adanya fitur seperti DNSSEC, proteksi serangan berbasis DNS, dan pengelolaan aturan keamanan dasar di level jaringan. Untuk blog atau website pribadi, kombinasi domain murah dari Berkahost dan DNS plus keamanan gratis dari Cloudflare sudah lebih dari cukup untuk mulai membangun “rumah” kecil di internet tanpa keluar biaya besar

## Setelah Beli Domain: Lanjut Ngapain?
Kadang sebagai pemula kita suka bingung: “Sudah beli domain, terus mau diapain?” Domain akhirnya cuma jadi pajangan, dibeli lalu dibiarkan selesai begitu saja. Padahal, dengan domain saja (tanpa hosting berbayar), sebenarnya sudah banyak hal yang bisa kita lakukan.

Salah satu yang paling menyenangkan adalah deploy website statis: blog sederhana, cepat, murah, dan mudah dirawat. Website statis ini biasanya dibangun dari file markdown, lalu di-generate menjadi HTML oleh static site generator seperti Hugo atau Jekyll.

Main Web Statis dengan Markdown
Sebelumnya, blog saya ini saya hosting di GitHub, karena GitHub Pages memang sangat cocok untuk website statis: gratis, stabil, dan cukup untuk blog pribadi atau dokumentasi. Setelah saya baca-baca, ternyata ada banyak sekali tema kece untuk website statis yang bisa kita kostumisasi isinya hanya dengan file markdown, persis seperti yang sedang kamu baca sekarang.
​

Di blog ini saya menggunakan Hugo dengan tema bernama PaperMod, sebuah tema Hugo yang cepat, bersih, dan sangat populer di komunitas Hugo. Struktur kontennya cukup tulis markdown di folder content, lalu Hugo akan mengubahnya menjadi file HTML siap deploy. Rasanya mirip menulis catatan di Notion, tapi outputnya jadi website beneran.

Untuk percobaan lain pada domain yang baru saya beli, saya juga mencoba Jekyll dan menggunakan tema Minimal Mistakes, salah satu tema Jekyll yang sangat fleksibel untuk blog dan personal site. Kelebihannya, Jekyll terintegrasi dengan GitHub Pages sehingga kamu bisa langsung hosting dari repo GitHub tanpa perlu server sendiri.

Belajar dan Eksperimen Tema
Kabar baiknya, hampir semua tema populer punya dokumentasi dan tutorial lengkap di internet maupun YouTube: cara install, cara ganti warna, cara tambah menu, sampai optimasi SEO dasar. Tinggal pilih:

- Mau pakai Hugo + PaperMod.
- Mau pakai Jekyll + Minimal Mistakes.
- Atau generator/tema lain yang sesuai selera.

Setelah itu, alurnya kira-kira seperti ini:
- Pilih generator (Hugo/Jekyll)
- Pilih tema yang kamu suka.
- Tulis konten dalam bentuk file markdown.
- Generate jadi website statis.
- Deploy ke GitHub Pages atau layanan serupa.
- Point domain yang sudah kamu beli ke hosting statis tersebut (misalnya lewat Cloudflare atau DNS provider lain).

Pelan-pelan, domain yang tadinya “nganggur” bisa berubah jadi blog pribadi, dokumentasi, catatan teknis, atau apa pun yang kamu suka. Contohnya bisa kamu lihat di blog saya: [TipsIT](https://tipsit.web.id)

![TipsIT](/img/domain/tips.png "TipsIT")

## Summary
Tantangan terbesar setelah punya blog atau domain sebenarnya bukan teknis, tapi bagaimana tetap konsisten menulis dan kembali pada tujuan awal: blog ini untuk apa. Banyak blogger menyarankan untuk selalu mengingat alasan pribadi saat mulai menulis, misalnya sebagai ruang ekspresi diri, dokumentasi perjalanan, atau sekadar catatan yang bisa diakses kapan saja selama ada internet.

Kalau tujuanmu memang untuk konsumsi pribadi, sebagai arsip catatan teknis atau jurnal kecil yang bisa kamu buka dari mana saja, itu sudah alasan yang sangat kuat untuk terus menulis. Banyak penulis menyarankan mulai dari langkah kecil: menulis sedikit demi sedikit, tidak perlu panjang atau sempurna, yang penting rutin dan terus berjalan. Tanpa memulai dan membiasakan diri, kebiasaan itu tidak akan pernah terbentuk, jadi satu paragraf pun sudah sebuah kemajuan.