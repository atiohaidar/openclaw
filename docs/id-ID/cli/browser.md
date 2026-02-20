---
summary: "Referensi CLI untuk `openclaw browser` (profil, tab, aksi, relay ekstensi)"
read_when:
  - Anda menggunakan `openclaw browser` dan menginginkan contoh tugas umum
  - Anda ingin mengontrol browser di mesin lain melalui host node
  - Anda ingin menggunakan relay ekstensi Chrome
title: "browser"
---

# `openclaw browser`

Mengelola server kontrol browser OpenClaw dan menjalankan berbagai aksi browser (tab, cuplikan/snapshots, tangkapan layar, navigasi, klik, pengetikan).

## Terkait:
- Alat Browser & API: [Browser tool](/id-ID/tools/browser)
- Relay Ekstensi Chrome: [Chrome extension](/id-ID/tools/chrome-extension)

## Contoh Penggunaan (Lokal)

```bash
# Menampilkan tab yang aktif di profil chrome
openclaw browser --browser-profile chrome tabs

# Memulai instansi browser OpenClaw yang terdedikasi
openclaw browser --browser-profile openclaw start

# Membuka URL tertentu
openclaw browser --browser-profile openclaw open https://example.com

# Mengambil cuplikan (snapshot) halaman saat ini
openclaw browser --browser-profile openclaw snapshot
```

## Profil Browser
Profil adalah konfigurasi perutean browser yang diberi nama:
- `openclaw`: Menjalankan/menghubungkan ke instansi Chrome khusus yang dikelola OpenClaw.
- `chrome`: Mengontrol tab Chrome yang sudah ada melalui relay ekstensi Chrome.

## Aksi Browser (Navigasi / Klik / Ketik)

```bash
# Navigasi ke alamat baru
openclaw browser navigate https://example.com

# Mengklik elemen berdasarkan referensi (ref)
openclaw browser click <ref>

# Mengetik teks ke elemen tertentu
openclaw browser type <ref> "halo"
```

## Kontrol Browser Jarak Jauh (Remote)
Jika Gateway berjalan di mesin yang berbeda dengan browser, jalankan **host node** di mesin yang memiliki Chrome/Brave/Edge/Chromium. Gateway akan meneruskan perintah browser ke node tersebut.



