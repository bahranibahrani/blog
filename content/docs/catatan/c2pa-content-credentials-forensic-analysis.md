---
title: "Forensic Analysis C2PA Content Credentials: Mengapa Watermark Kriptografi AI Hilang Saat Di-Crop, Dibalik, atau Dikirim via WhatsApp?"
date: 2026-09-24T08:18:16+07:00
draft: false
description: "Eksperimen forensik digital menggunakan C2PA Forensic Scanner berbasis Python (c2pa-python, Pydantic, Rich). Membedah struktur JUMBF manifest, integrasi badge OpenAI di LinkedIn & Google Search, serta analisis kegagalan validasi pada cropping dan kompresi WhatsApp."
tags: ["C2PA", "Cyber Security", "Digital Forensics", "Python", "Pydantic", "AI Provenance", "SynthID", "OpenAI", "Google", "Cryptography", "WhatsApp"]
categories: ["Cyber Security", "Digital Forensics & AI"]
showToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---

## 1. Fenomena Badge "Content Credentials" di LinkedIn & Google

Saat mengamati gambar hasil *generate* AI (seperti DALL-E 3 atau ChatGPT) yang diunggah ke LinkedIn, kita sering melihat ikon pin kecil bertuliskan **"Content Credentials"**. Ketika diklik, muncul informasi forensik:

```text
Content Credentials
Source or history information is available for this media. Learn more

App or device used : OpenAI Media Service API
Issued by          : OpenAI OpCo, LLC
Issued on          : September 23, 2026
```
![OpenAI](/img/c2pa/1.png)
### Mengapa LinkedIn dan Google Menampilkan Metadata Ini?

LinkedIn, OpenAI, Google, Microsoft, dan Adobe tergabung dalam *Steering Committee* **C2PA (Coalition for Content Provenance and Authenticity)**. 

Berdasarkan [publikasi resmi Google](https://blog.google/innovation-and-ai/products/google-gen-ai-content-transparency-c2pa/), industri teknologi kini beralih ke standar **C2PA versi 2.1** untuk menciptakan transparansi digital di berbagai platform:
* **LinkedIn:** Memvalidasi sertifikat digital penanda tangan dari penyedia AI (`OpenAI OpCo, LLC`).
* **Google Search ("About this image"):** Menampilkan riwayat apakah gambar dibuat atau diedit dengan AI pada Google Images, Google Lens, dan *Circle to Search*.
* **YouTube & Google Ads:** Menggunakan sinyal C2PA untuk membedakan rekaman kamera fisik asli vs manipulasi AI melalui **C2PA Trust List**.

---

## 2. Membangun Tool Forensik: "C2PA Content Credentials Forensic Scanner"

Untuk menguji keaslian dan batas ketahanan C2PA secara empiris, saya mengembangkan sebuah aplikasi forensik berbasis Python: **C2PA Content Credentials Forensic Scanner**.

Aplikasi ini menggunakan pustaka resmi `c2pa-python` untuk memecahkan kode *JUMBF payload* (bukan sekadar pencocokan *regex/string* biasa), dibungkus dengan data model `Pydantic` yang *type-safe*, serta antarmuka `Rich Terminal UI` dan export JSON.

### Arsitektur Alur Logika Scanner (Logical Flow)

```text
 ┌──────────────────────┐
 │   Input Media File   │ (User mengirimkan gambar via CLI)
 └──────────┬───────────┘
            ▼
 ┌──────────────────────┐
 │  Integrity Checker   │ (Menghitung Hash SHA-256 asli dari byte file)
 └──────────┬───────────┘
            ▼
 ┌──────────────────────┐ (c2pa-python Engine)
 │    JUMBF & C2PA      │ ──► Jika Tidak Ditemukan ──► Exit(1) [C2PA Absent]
 │  Manifest Extractor  │
 └──────────┬───────────┘
            ▼
 ┌──────────────────────┐
 │ Analysis & Filtering │ (Menyaring Validation Status dan Assertions)
 └──┬────────────────┬──┘
    │                │
    ▼                ▼
[ Actions ]    [ Cryptographic Signature Check ]
 (created,      (Data Hash match? Certificate valid?)
  edited)            │
                     ▼
 ┌──────────────────────────────────────┐
 │   Mapping to Pydantic Data Models    │ (Membungkus ke objek ScanResult)
 └───────────────────┬──────────────────┘
                     ▼
 ┌──────────────────────────────────────┐
 │   Rich CLI / JSON Output Formatter   │ (Mencetak UI Terminal / File JSON)
 └──────────────────────────────────────┘
```
![Google](/img/c2pa/2.png) 

### Standar Spesifikasi Exit Codes (Scripting & Automation):
* `0` : **OK** — Validasi sukses & integritas piksel terverifikasi (Data Hash Match).
* `1` : **C2PA Absent** — Blok C2PA tidak ditemukan (misal: gambar non-AI atau metadata terhapus).
* `2` : **Cryptographic Failure** — Blok C2PA ada, namun gagal verifikasi kriptografik (*Hash Mismatch / Tampered*).
* `3` : **File Not Found** — Direktori media tidak ditemukan.
* `4` : **Unsupported Format** — Format file tidak didukung.
* `5` : **Internal Error** — Kesalahan runtime pustaka/engine.
![OpenAI](/img/c2pa/3.png)
---

## 3. Hasil Temuan Eksperimen Forensik Nyata

Dengan menjalankan `c2pa-scanner` terhadap puluhan sampel gambar dari berbagai skenario transmisi dan manipulasi, didapatkan hasil pengujian berikut:

| Skenario Pengujian | Exit Code Scanner | Status Manifest C2PA | Keterbacaan di Verifier |
|---|:---:|---|---|
| **File Asli (Direct Download API)** | `0` | ✅ Valid | **Terbaca Sempurna** (*Issued by OpenAI OpCo, LLC*) |
| **Kirim via WhatsApp (Mode Dokumen)** | `0` | ✅ Valid & Utuh | **Terbaca Sempurna** (*Raw Binary Preserved*) |
| **Kirim via WhatsApp (Kirim Gambar Galeri)** | `1` | ❌ Hilang Total | **C2PA Absent** (*Metadata Di-strip WhatsApp*) |
| **Screenshot Layar (Screen Capture)** | `1` | ❌ Hilang Total | **C2PA Absent** (*Metadata Tidak Ikut Ter-capture*) |
| **Gambar Di-Crop (Potong Sebagian)** | `2` | ❌ Rusak / Tampered | **Cryptographic Failure** (*Piksel Hash Berubah*) |
| **Gambar Dibalik (Flip Horizontal/Vertical)** | `2` | ❌ Rusak / Tampered | **Cryptographic Failure** (*Hash Mismatch*) |
| **Re-save via Image Editor (Non-C2PA App)** | `1` / `2` | ❌ Rusak / Hilang | **Gagal Validasi** |

---

### Analisis Akar Masalah (Root Cause Analysis):

### A. Mengapa Kirim Gambar Biasa di WhatsApp Mengubah Status Jadi Exit Code 1, tetapi Mode Dokumen Exit Code 0?
* **Pengiriman Gambar Normal:** WhatsApp menerapkan kompresi otomatis untuk efisiensi *bandwidth*. Dalam proses ini, seluruh metadata non-visual (EXIF, XMP, dan **JUMBF box C2PA**) di-*strip* (dibuang total). Akibatnya, file yang diterima bersih dari blok C2PA (`Exit Code 1: C2PA Absent`).
* **Pengiriman sebagai Dokumen:** WhatsApp memperlakukan file sebagai *raw byte stream* bit-by-bit. Struktur container JUMBF dan tanda tangan PKI tetap utuh tanpa perubahan, sehingga scanner menghasilkan `Exit Code 0: OK`.

### B. Mengapa Crop dan Flip Menghasilkan Exit Code 2 (Cryptographic Failure)?
C2PA bekerja dengan prinsip integritas matematis yang kaku (**Asset Binding**). 
* OpenAI menghitung nilai hash SHA-256 dari susunan matriks piksel asli dan menandatanganinya dengan sertifikat X.509 resmi.
* Ketika gambar di-*crop* atau di-*flip*, susunan matriks piksel berubah drastis.
* Saat scanner menghitung ulang hash SHA-256 dari piksel baru dan membandingkannya dengan *Signed Hash* di dalam manifest, hasilnya tidak cocok (*Hash Mismatch*). Scanner mendeteksi adanya manipulasi dan mengembalikan `Exit Code 2`.

---

## 4. Analisis Komparasi: C2PA vs SynthID (Google DeepMind)

Sebagaimana ditekankan dalam riset Google, transparansi AI memerlukan pendekatan **Dual-Layer** karena tidak ada satu teknologi yang menjadi solusi tunggal (*no silver bullet*):

```text
┌───────────────────────────┬─────────────────────────────────┬─────────────────────────────────┐
│ Fitur / Parameter         │ C2PA (Content Credentials)      │ Google SynthID (DeepMind)       │
├───────────────────────────┼─────────────────────────────────┼─────────────────────────────────┤
│ **Metode Penyematan**     │ Container Metadata File (JUMBF) │ Domain Frekuensi Piksel Gambar  │
│ **Kirim WhatsApp Biasa**  │ ❌ Hilang Total (Exit Code 1)   │ ✅ Tetap Bertahan (Tahan Kompresi│
│ **Kirim WhatsApp Dokumen**│ ✅ Utuh 100% (Exit Code 0)      │ ✅ Tetap Bertahan               │
│ **Ketahanan Crop & Flip** │ ❌ Gagal (Exit Code 2)          │ ✅ Bertahan (Robust Imperceptible│
│ **Screenshot Layar**      │ ❌ Hilang (Exit Code 1)         │ ✅ Tetap Terdeteksi             │
│ **Transparansi Standar**  │ ✅ Terbuka untuk Publik (Open)  │ ⚠️ Membutuhkan Model AI Detector│
│ **Fungsi Utama**          │ Rantai Jejak Hukum (Provenance) │ Ketahanan Anti-Bypass Forensik  │
└───────────────────────────┴─────────────────────────────────┴─────────────────────────────────┘
```

* **C2PA** ibarat **"Sertifikat Berstempel Kemenkumham"** yang dilampirkan pada berkas: sertifikat ini sah secara hukum, tetapi jika berkas difotokopi ulang atau amplopnya dibuang (seperti kirim via WhatsApp biasa), stempelnya ikut hilang.
* **SynthID** ibarat **"Serat Pengaman Hologram pada Uang Kertas"**: tanda air menyatu ke dalam struktur fisik piksel, sehingga tetap terdeteksi oleh scanner khusus meskipun uang tersebut dipotong atau dilipat.

---

## 5. Kesimpulan & Rekomendasi Forensik

1. **C2PA Efektif untuk Provenance Distribusi Asli:** Standar C2PA v2.1 sangat ideal untuk platform enterprise (LinkedIn, Google Search, News Media) dalam melacak asal-usul kamera fisik (Sony/Leica) dan software AI (OpenAI/Google Imagen).
2. **Keterbatasan di Jalur Chatting:** Masyarakat tidak bisa mengandalkan C2PA jika gambar sudah beredar lewat kompresi media sosial atau grup chat WhatsApp biasa (kecuali dikirim sebagai dokumen).
3. **Standar Masa Depan:** Autentikasi media digital modern wajib menggabungkan **C2PA (untuk legal audit trail terbuka)** bersama **SynthID / Latent Watermarking (untuk ketahanan terhadap manipulasi piksel, crop, dan re-encoding)**.

