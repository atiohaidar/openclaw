---
summary: "Referensi CLI untuk `openclaw channels` (akun, status, login/logout, log)"
read_when:
  - Anda ingin menambah/menghapus akun saluran (WhatsApp/Telegram/Discord/dll)
  - Anda ingin memeriksa status saluran atau memantau log saluran
title: "channels"
---

# `openclaw channels`

Mengelola akun saluran obrolan dan status runtime-nya di Gateway.

## Terkait:
- Panduan Saluran: [Channels](/id-ID/channels/index)
- Konfigurasi Gateway: [Configuration](/id-ID/gateway/configuration)

## Perintah Umum

```bash
# Menampilkan daftar saluran yang dikonfigurasi
openclaw channels list

# Menampilkan status kesehatan saluran
openclaw channels status

# Melihat kemampuan dasar saluran
openclaw channels capabilities

# Mencari ID saluran/pengguna berdasarkan nama
openclaw channels resolve --channel slack "#general" "@jane"

# Memantau log khusus saluran
openclaw channels logs --channel all
```

## Menambah / Menghapus Akun
```bash
# Menambah Telegram
openclaw channels add --channel telegram --token <bot-token>

# Menghapus Telegram beserta konfigurasinya
openclaw channels remove --channel telegram --delete
```

## Login / Logout (Interaktif)
Berlaku untuk saluran yang memerlukan autentikasi browser atau manual.
```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## Pemecahan Masalah
- Jalankan `openclaw status --deep` untuk pemeriksaan menyeluruh.
- Gunakan `openclaw doctor` untuk perbaikan terpandu.
- Jika `channels list` menunjukkan error 403, pastikan cakupan (scope) token Anda sudah benar.



