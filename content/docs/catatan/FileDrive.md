---
title: "📦 FileDrive Portal: Cara Praktis Bagikan File Besar Tanpa Ribet"
date: 2026-05-15T17:20:44+07:00
draft: false
description: "FileDrive Portal hadir untuk memudahkan saya berbagi file besar dengan cepat, aman, dan gratis, tanpa perlu server atau hosting berbayar."
tags: ["FileDrive", "Google Drive", "File Sharing", "Web App", "Google Apps Script", "GitHub Pages", "Tutorial"]
categories: ["Showcase"]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---
Halo teman-teman! 👋  
Pernah nggak sih, kamu mau share file besar, tapi VPS mahal, storage terbatas, atau takut link Google Drive susah dicari? Saya juga pernah. 😅  

Nah, dari pengalaman itu, lahirlah **FileDrive Portal** — sebuah landing page modern bergaya HFS (*HTTP File Server*) yang memungkinkan kamu **share file besar dengan cepat, aman, dan GRATIS**, tanpa perlu server atau hosting berbayar.  

![Tampilan UI sudah jadi](/img/drive/2026-05-15_170658.png)
*Tampilan UI sudah jadi*

> **🚀 Live Demo:** Kamu bisa melihat langsung hasil jadinya dan mencoba mengakses portal ini melalui tautan berikut:  
> **[https://file.bahrani.my.id/](https://file.bahrani.my.id/)**

Selain itu, kami menyediakan **folder khusus di Google Drive** untuk dibagikan. Jadi, kalau kamu ingin share file, tinggal **pindahkan (move) file ke folder ini**, dan file otomatis muncul di website untuk diakses orang lain. Setelah selesai, tinggal **kembalikan file ke lokasi asal**, sehingga file tetap tersimpan rapi di Drive kamu tanpa hilang atau tercampur. 🎉

![Tampilan google drive](/img/drive/2026-05-15_171143.png)
*Tampilan google drive*

---

## 🔍 Masalah yang Sering Dihadapi

Biasanya kalau mau share file besar, kita menghadapi beberapa kendala:

- Google Drive kadang link-nya panjang dan susah dicari  
- Banyak layanan gratis membatasi ukuran upload  
- User experience kurang nyaman: nggak ada live search, sorting, atau tombol copy link  

FileDrive Portal hadir untuk **menghilangkan semua kerepotan itu**, sekaligus memberi **UX premium** untuk orang yang ingin akses file.

---

## ✨ Solusi dari FileDrive Portal

Dengan FileDrive Portal, kamu bisa:

- Upload file ke Google Drive → otomatis muncul di website  
- Cari file dengan **live search** dan **sort smartly**  
- Copy link dengan satu klik, lengkap dengan toast notification   
- Folder khusus untuk share: **move → tampil online → kembalikan ke lokasi asal**, sehingga file tersimpan rapi  
- Semua orang bisa mengakses folder share secara langsung  
- Google Apps Script menyediakan **endpoint JSON** untuk membaca isi folder dan ditampilkan di frontend  

> Dengan cara ini, membagikan file besar jadi **mudah, aman, dan cepat**. 😎

## 🛠️ Teknologi di Baliknya

Sebelum masuk ke langkah instalasi, kenalan dulu sama “mesin” FileDrive Portal:

- **Frontend:** HTML5 + CSS3 + Vanilla JS (ES6+)  
- **Backend:** Google Drive API via Google Apps Script (JSON endpoint)  
- **Hosting:** GitHub Pages  

![uji coba Script google Cript](/img/drive/2026-05-15_170800.png)
*uji coba Script google Cript*

