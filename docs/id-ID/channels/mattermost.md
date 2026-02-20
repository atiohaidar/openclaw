---
summary: "Penyiapan bot Mattermost dan konfigurasi OpenClaw"
read_when:
  - Menyiapkan Mattermost
  - Mendebug perutean Mattermost
title: "Mattermost"
---

# Mattermost (Plugin)

Status: Didukung melalui plugin (token bot + event WebSocket). Mendukung saluran (channels), grup, dan pesan langsung (DM). Mattermost adalah platform perpesanan tim yang dapat di-host sendiri.

## Instalasi Plugin

Saluran Mattermost tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/mattermost
```

## Penyiapan Cepat (Quick Setup)

1. **Instal Plugin** Mattermost.
2. **Buat Akun Bot**: Dapatkan **bot token** dari pengaturan integrasi di server Mattermost Anda.
3. **Dapatkan URL Dasar**: Gunakan URL server Mattermost Anda (misal: `https://chat.contoh.com`).
4. **Konfigurasi OpenClaw**: Tambahkan kredensial ke file `openclaw.json`.
5. **Mulai Gateway**: Bot akan mulai mendengarkan event dari server Mattermost.

Contoh konfigurasi minimal:
```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.contoh.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Mode Chat

Mattermost menanggapi DM secara otomatis. Perilaku di dalam saluran chat dikontrol oleh parameter `chatmode`:
- `oncall` (default): Hanya merespons jika di-@mention di saluran.
- `onmessage`: Merespons setiap pesan di saluran.
- `onchar`: Merespons jika pesan diawali dengan prefiks pemicu (seperti `>` atau `!`).

## Kontrol Akses

- **Pesan Langsung (DM)**: Secara default menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui `openclaw pairing approve mattermost <KODE>`.
- **Saluran (Grup)**: Secara default menggunakan mode **allowlist**. Anda dapat membatasi siapa saja yang bisa memicu bot menggunakan `@username` atau ID pengguna di `groupAllowFrom`.

## Aksi Reaksi

Agen dapat memberikan atau menghapus reaksi emoji pada postingan di Mattermost. Anda dapat menggunakan alat perpesanan dengan menyertakan `postId` dan nama emoji (misal: `thumbsup`).

## Pemecahan Masalah

- **Bot tidak membalas di saluran**: Pastikan bot sudah diundang ke saluran tersebut dan periksa apakah Anda perlu melakukan @mention atau menggunakan prefiks pemicu.
- **Kesalahan Autentikasi**: Periksa kembali token bot dan URL dasar server Anda.
- **Variabel Lingkungan**: Ingat bahwa variabel lingkungan (`MATTERMOST_BOT_TOKEN`) hanya berlaku untuk akun default.

Untuk rujukan lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



