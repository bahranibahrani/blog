# 🌐 Bahrani Blog

Blog pribadi tentang **Networking**, **Network Automation**, dan **Teknologi**.

🌐 **Live**: [blog.bahrani.my.id](https://blog.bahrani.my.id)

---

## 🛠️ Tech Stack

| Komponen | Teknologi |
|----------|-----------|
| Static Site Generator | [Hugo](https://gohugo.io/) |
| Tema | [PaperMod](https://github.com/adityatelange/hugo-PaperMod) |
| DNS & CDN | Cloudflare |
| Version Control | Git + GitHub |

---

## 📁 Struktur Proyek

```
blog/
├── archetypes/          # Template untuk post baru
│   ├── default.md       # Template default
│   ├── catatan.md       # Template untuk kategori Catatan
│   └── pribadi.md       # Template untuk kategori Pribadi
├── content/
│   └── docs/            # Semua post blog
│       ├── catatan/     # Catatan teknis
│       └── pribadi/     # Post personal
├── layouts/
│   └── partials/        # Custom partials (override tema)
│       ├── extend_head.html
│       ├── extend_footer.html
│       └── index_profile.html
├── static/
│   └── img/             # Gambar dan aset statis
├── themes/
│   └── PaperMod/        # Tema Hugo PaperMod (submodule)
├── config.yml           # Konfigurasi utama Hugo
└── config.development.yml # Override untuk development lokal
```

---

## 🚀 Quick Start

### Prasyarat

- [Hugo Extended](https://gohugo.io/installation/) v0.157+
- [Git](https://git-scm.com/)

### Clone & Jalankan

```bash
# Clone repository
git clone https://github.com/bahranibahrani/blog.git
cd blog

# Jalankan development server (termasuk draft)
hugo server -D --config config.yml,config.development.yml --disableFastRender
```

Buka [http://localhost:1313](http://localhost:1313)

### Build Production

```bash
# Build tanpa draft
hugo --config config.yml

# Output ada di folder public/
```

---

## 📝 Membuat Post Baru

```bash
# Post di catatan (otomatis pakai template catatan)
hugo new content/docs/catatan/judul-post.md

# Post di pribadi (otomatis pakai template pribadi)
hugo new content/docs/pribadi/judul-post.md

# Post di folder lain (pakai template default)
hugo new content/docs/judul-post.md
```

Setelah itu edit file yang sudah dibuat, lalu ganti `draft: true` → `draft: false` untuk publish.

---

## ✨ Fitur

- 🏠 **Profile Mode** — Halaman depan dengan foto profil dan bio
- 🔍 **Search** — Pencarian full-text menggunakan Fuse.js
- 🌙 **Dark/Light Mode** — Toggle tema otomatis atau manual
- 📱 **Responsive** — Tampilan optimal di desktop dan mobile
- 🖼️ **Lightbox** — Klik gambar untuk zoom
- 🔗 **Social Links** — Instagram, GitHub, LinkedIn
- 📊 **Reading Stats** — Waktu baca dan jumlah kata
- 🗂️ **Categories & Tags** — Organisasi konten
- 📰 **RSS Feed** — Subscribe via RSS
- 🧭 **Breadcrumbs** — Navigasi yang mudah

---

## ⚙️ Konfigurasi

File utama: `config.yml`

| Setting | Nilai |
|---------|-------|
| `baseURL` | `https://blog.bahrani.my.id/` |
| `languageCode` | `id-id` |
| `theme` | PaperMod |
| `defaultTheme` | auto |
| `buildDrafts` | false |

---

## 📦 Deployment

Blog ini di-deploy menggunakan **Cloudflare Pages**:

1. Push ke branch `main`
2. Cloudflare Pages auto-build dengan command `hugo`
3. Output di folder `public/` di-serve ke CDN global

> **Catatan**: Folder `public/` ada di `.gitignore` dan tidak di-commit ke repository.

---

## 📄 Lisensi

Konten blog © Bahrani. Tema PaperMod menggunakan lisensi masing-masing.