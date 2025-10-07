---
title: "Proyek Visualisasi Jaringan CDP dengan Streamlit"
date: 2025-10-07T12:00:00+07:00
tags: ["python", "network", "streamlit"]
categories: ["Catatan"]
draft: false
---

Aplikasi web sederhana untuk memvisualisasikan topologi jaringan berdasarkan data dari Cisco Discovery Protocol (CDP) yang diekspor ke dalam format file Excel.

## Deskripsi

Aplikasi ini menggunakan library **Streamlit** di Python untuk membuat aplikasi web interaktif. Ia akan membaca file `.xlsx` yang berisi data CDP (seperti nama perangkat, alamat IP, port yang terhubung, dll.), kemudian menggunakan library `networkx` dan `matplotlib` untuk membuat dan menampilkan gambar topologi jaringan.

## Fitur

- Membaca data perangkat jaringan dari file Excel.
- Membangun model graf dari topologi jaringan.
- Menampilkan visualisasi topologi jaringan melalui antarmuka web interaktif.
- Menggunakan ikon SVG untuk merepresentasikan tipe perangkat yang berbeda (router, switch, dll).

## Struktur Proyek

```
.
├── app.py                # Logika utama aplikasi Streamlit
├── data/
│   └── cdp_neighbors.xlsx  # Contoh file data CDP
└── img/                  # Ikon untuk perangkat jaringan
```

## Instalasi & Penggunaan

1.  **Instalasi library yang dibutuhkan:**
    ```bash
    pip install streamlit pandas networkx matplotlib openpyxl
    ```

2.  **Jalankan aplikasi:**
    ```bash
    streamlit run app.py
    ```

3.  Buka browser web Anda dan kunjungi alamat yang muncul di terminal (biasanya `http://localhost:8501`).

Anda akan melihat visualisasi topologi jaringan yang dihasilkan dari data Anda.

## Contoh Hasil

Berikut adalah contoh visualisasi yang dihasilkan oleh aplikasi:

![Contoh Topologi Jaringan](/img/1.png "Visualisasi Topologi Jaringan")

## Kontribusi

Kontribusi dalam bentuk *pull request* sangat diterima. Untuk perubahan besar, silakan buka *issue* terlebih dahulu untuk mendiskusikan apa yang ingin Anda ubah.
