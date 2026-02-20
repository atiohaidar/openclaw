---
summary: "Status dukungan aplikasi Google Chat, kemampuan, dan konfigurasi"
read_when:
  - Mengatur atau menggunakan saluran Google Chat
title: "Google Chat"
---

# Google Chat (Chat API)

Status: Siap digunakan untuk Pesan Langsung (DM) dan Ruang (Spaces) melalui webhook HTTP Google Chat API.

## Penyiapan Cepat (Pemula)

1. **Buat Proyek Google Cloud** dan aktifkan **Google Chat API** di [Konsol Google Cloud](https://console.cloud.google.com/apis/api/chat.googleapis.com).
2. **Buat Akun Layanan (Service Account)**:
   - Pilih **Create Credentials** > **Service Account**.
   - Beri nama (misal: `openclaw-chat`) dan lanjutkan sampai selesai tanpa perlu mengatur peran (role) khusus.
3. **Unduh Kunci JSON**:
   - Klik pada Akun Layanan yang baru dibuat.
   - Buka tab **Keys**, pilih **Add Key** > **Create new key**.
   - Pilih format **JSON** dan simpan file tersebut di host gateway Anda (misal: `~/.openclaw/googlechat-key.json`).
4. **Konfigurasi Aplikasi Google Chat**:
   - Di Konsol Google Cloud Chat API, isi info aplikasi.
   - Aktifkan **Interactive features**.
   - Centang **Join spaces and group conversations**.
   - Pilih **HTTP endpoint URL** sebagai metode koneksi.
   - Setel URL endpoint ke URL publik gateway Anda diikuti dengan `/googlechat` (misal: `https://domain.anda/googlechat`).
5. **Ubah Status Aplikasi**:
   - Setel status aplikasi ke **Live - available to users** agar dapat digunakan.
6. **Konfigurasi di OpenClaw**:
   - Tambahkan path kunci JSON ke konfigurasi `channels.googlechat.serviceAccountFile`.

## Cara Menambahkan Bot ke Google Chat

1. Buka [Google Chat](https://chat.google.com/).
2. Klik ikon **+** di sebelah **Direct Messages**.
3. Cari nama aplikasi yang Anda buat tadi. Bot ini bersifat privat, jadi Anda harus mencari namanya secara lengkap.
4. Klik **Add** untuk memulai percakapan.

## URL Publik (Hanya Webhook)

Google Chat membutuhkan endpoint HTTPS publik. Untuk keamanan, **hanya ekspos jalur `/googlechat`** ke internet.

**Rekomendasi: Tailscale Funnel**
Gunakan Tailscale Funnel untuk mengekspos jalur `/googlechat` saja secara publik, sementara dashboard utama tetap privat di jaringan tailnet Anda.

```bash
tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
```

## Cara Kerja

- **Perutean**: Pesan dirutekan berdasarkan jenis ruang (DM atau Space).
- **Pairing**: Secara default, OpenClaw akan mengirimkan kode pairing kepada pengguna baru. Setujui melalui `openclaw pairing approve googlechat <KODE>`.
- **At-Mention**: Di dalam Ruang (Spaces), bot membutuhkan @mention agar merespons secara otomatis (default).

## Contoh Konfigurasi

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "~/.openclaw/googlechat-key.json",
      audienceType: "app-url",
      audience: "https://gateway.anda/googlechat",
      dm: { policy: "pairing" },
    },
  },
}
```

## Pemecahan Masalah

- **405 Method Not Allowed**: Ini berarti handler webhook belum terdaftar. Pastikan Anda sudah menambahkan konfigurasi `channels.googlechat` dan sudah merestart gateway.
- **Bot tidak membalas**: Pastikan URL webhook di konsol Google Cloud sudah benar dan status aplikasi sudah **Live**.
- **Log detail**: Jalankan `openclaw logs --follow` saat mengirim pesan tes untuk melihat apakah permintaan mencapai gateway.



