---
summary: "Plugin komunitas: standar kualitas, persyaratan hosting, dan jalur pengajuan PR"
read_when:
  - Anda ingin mempublikasikan plugin OpenClaw pihak ketiga
  - Anda ingin mengajukan plugin untuk dimasukkan ke dalam daftar dokumentasi
title: "Plugin Komunitas"
---

# Plugin Komunitas

Halaman ini mencantumkan **plugin yang dikelola komunitas** dengan kualitas tinggi untuk OpenClaw.

Kami menerima Pull Request (PR) untuk menambahkan plugin komunitas di sini jika memenuhi standar kualitas kami.

## Persyaratan untuk Masuk Daftar

- Paket plugin sudah dipublikasikan di npmjs (dapat diinstal melalui `openclaw plugins install <npm-spec>`).
- Kode sumber di-host di GitHub (repositori publik).
- Repositori menyertakan dokumentasi penyiapan/penggunaan dan pelacak masalah (issue tracker).
- Plugin memiliki sinyal pemeliharaan yang jelas (pemelihara aktif, pembaruan terkini, atau penanganan masalah yang responsif).

## Cara Mengajukan

Buka PR untuk menambahkan plugin Anda ke halaman ini dengan menyertakan:
- Nama plugin
- Nama paket npm
- URL repositori GitHub
- Deskripsi singkat satu baris
- Perintah instalasi

## Standar Peninjauan (Review)

Kami mengutamakan plugin yang berguna, terdokumentasi dengan baik, dan aman untuk dijalankan. Paket yang dibuat seadanya (low-effort wrappers), tidak memiliki kepemilikan yang jelas, atau tidak dipelihara mungkin akan ditolak.

## Format Kandidat

Gunakan format ini saat menambahkan entri:
- **Nama Plugin** — deskripsi singkat
  npm: `@scope/package`
  repo: `https://github.com/org/repo`
  install: `openclaw plugins install @scope/package`


