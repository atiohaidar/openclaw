---
summary: "Panduan ClawHub: registri skill publik + alur kerja CLI"
read_when:
  - Memperkenalkan ClawHub kepada pengguna baru
  - Menginstal, mencari, atau memublikasikan skill
  - Menjelaskan flag CLI ClawHub dan perilaku sinkronisasi
title: "ClawHub"
---

# ClawHub

ClawHub adalah **registri skill publik untuk OpenClaw**. Ini adalah layanan gratis: semua skill bersifat publik, terbuka, dan dapat dilihat oleh siapa saja untuk dibagikan dan digunakan kembali. Sebuah skill hanyalah folder yang berisi file `SKILL.md` (ditambah file pendukung lainnya). Anda dapat menjelajahi skill di web atau menggunakan CLI untuk mencari, menginstal, memperbarui, dan memublikasikan skill.

Situs: [clawhub.ai](https://clawhub.ai)

## Mengenal ClawHub

- Registri publik untuk skill OpenClaw.
- Penyimpanan metadata dan bundel skill yang memiliki versi.
- Permukaan penemuan melalui fitur pencarian, tag, dan sinyal penggunaan.

## Cara Kerjanya

1. Pengguna memublikasikan bundel skill (file + metadata).
2. ClawHub menyimpan bundel tersebut, mengurai metadata, dan memberikan versi.
3. Registri mengindeks skill tersebut untuk memudahkan pencarian.
4. Pengguna dapat mencari, mengunduh, dan menginstal skill di OpenClaw.

## Untuk Siapa Ini Dibuat? (Ramah Pemula)

Jika Anda ingin menambah kemampuan baru pada agen OpenClaw Anda, ClawHub adalah cara termudah untuk menemukan dan menginstal skill. Anda tidak perlu tahu cara kerja teknis di belakangnya. Anda bisa:

- Mencari skill dengan bahasa sehari-hari.
- Menginstal skill ke dalam workspace Anda.
- Memperbarui skill nantinya dengan satu perintah.
- Mencadangkan skill buatan Anda sendiri dengan memublikasikannya.

## Memulai dengan Cepat

1. Instal CLI ClawHub.
2. Cari sesuatu yang Anda butuhkan: `clawhub search "calendar"`
3. Instal sebuah skill: `clawhub install <slug-skill>`
4. Mulai sesi OpenClaw baru agar skill baru tersebut terdeteksi.

## Instalasi CLI

Gunakan salah satu:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## Hubungan dengan OpenClaw

Secara default, CLI menginstal skill ke dalam folder `./skills`. OpenClaw memuat skill workspace dari `<workspace>/skills` dan akan mendeteksinya pada sesi **berikutnya**.

Untuk detail lebih lanjut tentang cara pemuatan dan penyaringan skill, lihat [Skill](/id-ID/tools/skills).

## Fitur Layanan

- **Penjelajahan Publik**: Melihat skill dan konten `SKILL.md`-nya.
- **Pencarian**: Didukung oleh pencarian vektor (embedding), bukan hanya kata kunci.
- **Versi**: Menggunakan semver, changelog, dan tag (termasuk `latest`).
- **Unduhan**: File zip per versi.
- **Sinyal Komunitas**: Bintang dan komentar untuk umpan balik.

## Keamanan dan Moderasi

ClawHub bersifat terbuka secara default. Siapa pun dapat mengunggah skill, tetapi akun GitHub harus berusia minimal satu minggu untuk memublikasikannya demi mencegah penyalahgunaan.

- Pengguna dapat melaporkan skill yang mencurigakan atau berbahaya.
- Skill dengan laporan lebih dari 3 akan disembunyikan secara otomatis.
- Moderator dapat meninjau, menghapus, atau memblokir pengguna yang melanggar aturan.

## Perintah CLI

Opsi Global:
- `--workdir <dir>`: Direktori kerja.
- `--dir <dir>`: Direktori skill (default: `skills`).

Autentikasi:
- `clawhub login`: Login melalui browser.
- `clawhub logout`: Keluar.
- `clawhub whoami`: Lihat identitas Anda.

Perintah Utama:
- `clawhub search "kueri"`: Cari skill.
- `clawhub install <slug>`: Instal skill.
- `clawhub update --all`: Perbarui semua skill yang terinstal.
- `clawhub publish <path>`: Publikasikan skill Anda.
- `clawhub sync`: Sinkronkan skill lokal dan publikasikan pembaruan secara massal.

## Alur Kerja Umum

### Mencari Skill
```bash
clawhub search "backup database"
```

### Mengunduh Skill Baru
```bash
clawhub install paket-skill-saya
```

### Memperbarui Skill
```bash
clawhub update --all
```

### Mencadangkan Skill (Publish atau Sync)
Untuk satu folder skill:
```bash
clawhub publish ./nama-skill --slug nama-skill --name "Nama Skill" --version 1.0.0
```

Untuk memindai dan mencadangkan banyak skill sekaligus:
```bash
clawhub sync --all
```


