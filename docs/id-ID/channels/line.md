---
summary: "Penyiapan, konfigurasi, dan penggunaan plugin LINE Messaging API"
read_when:
  - Anda ingin menghubungkan OpenClaw ke LINE
title: LINE
---

# LINE (Plugin)

LINE terhubung ke OpenClaw melalui LINE Messaging API. Plugin ini berfungsi sebagai penerima webhook pada gateway dan menggunakan *channel access token* serta *channel secret* untuk autentikasi.

## Instalasi Plugin

Instal plugin LINE terlebih dahulu:

```bash
openclaw plugins install @openclaw/line
```

## Penyiapan (Setup)

1. **LINE Developers Console**: Masuk ke [developers.line.biz/console/](https://developers.line.biz/console/).
2. **Tambah Channel**: Buat atau pilih Provider, lalu tambahkan saluran **Messaging API**.
3. **Kredensial**: Salin **Channel access token** dan **Channel secret** dari pengaturan saluran tersebut.
4. **Aktifkan Webhook**: Di menu Messaging API, pilih **Use webhook**.
5. **Setel URL Webhook**: Gunakan URL HTTPS gateway Anda (misal: `https://domain.anda/line/webhook`).

## Konfigurasi

Contoh konfigurasi minimal di `openclaw.json`:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "AKSES_TOKEN_LINE_ANDA",
      channelSecret: "SECRET_LINE_ANDA",
      dmPolicy: "pairing",
    },
  },
}
```

## Kontrol Akses

- **Pesan Langsung (DM)**: Defaultnya menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui melalui perintah `openclaw pairing approve line <KODE>`.
- **Daftar Izinkan (Allowlist)**: ID pengguna LINE bersifat case-sensitive dan biasanya dimulai dengan huruf `U` diikuti 32 karakter hexadecimal.

## Perilaku Pesan

- **Batas Karakter**: Teks akan dipotong setiap 5000 karakter.
- **Markdown**: Pemformatan Markdown akan dihapus; blok kode dan tabel akan dikonversi menjadi kartu Flex jika memungkinkan.
- **Media**: Batas default unduhan media adalah 10 MB.

## Data Saluran (Rich Messages)

Gunakan `channelData.line` untuk mengirim Balasan Cepat (Quick Replies), lokasi, kartu Flex, atau pesan template. LINE juga menyediakan perintah `/card` untuk preset pesan Flex yang mudah digunakan.

## Pemecahan Masalah

- **Verifikasi Webhook Gagal**: Pastikan URL webhook menggunakan HTTPS dan *Channel Secret* sesuai dengan yang ada di konsol LINE.
- **Event Tidak Masuk**: Pastikan jalur (*path*) webhook sesuai dan gateway dapat diakses dari server LINE.
- **Gagal Mengunduh Media**: Tingkatkan nilai `channels.line.mediaMaxMb` jika file media melebihi batas default.



