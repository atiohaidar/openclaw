---
summary: "Platform perpesanan yang dapat dihubungkan ke OpenClaw"
read_when:
  - Anda ingin memilih saluran obrolan (chat) untuk OpenClaw
  - Anda butuh ringkasan cepat platform perpesanan yang didukung
title: "Saluran Chat"
---

# Saluran Chat

OpenClaw dapat berkomunikasi dengan Anda melalui aplikasi chat apa pun yang sudah Anda gunakan. Setiap saluran terhubung melalui Gateway. Teks didukung di semua saluran, sementara dukungan media dan reaksi bervariasi.

## Saluran yang didukung

- [WhatsApp](/id-ID/channels/whatsapp) — Paling populer; menggunakan Baileys dan memerlukan pairing QR.
- [Telegram](/id-ID/channels/telegram) — Menggunakan Bot API via grammY; mendukung grup.
- [Discord](/id-ID/channels/discord) — Discord Bot API + Gateway; mendukung server, saluran, dan DM.
- [Slack](/id-ID/channels/slack) — Bolt SDK; aplikasi workspace.
- [Google Chat](/id-ID/channels/googlechat) — Aplikasi Google Chat API melalui HTTP webhook.
- [Signal](/id-ID/channels/signal) — signal-cli; fokus pada privasi.
- [BlueBubbles](/id-ID/channels/bluebubbles) — **Direkomendasikan untuk iMessage**; menggunakan REST API server BlueBubbles macOS dengan dukungan fitur lengkap (edit, tarik pesan, efek, reaksi, manajemen grup).
- [Microsoft Teams](/id-ID/channels/msteams) — Bot Framework; dukungan enterprise (plugin, diinstal terpisah).
- [Zalo Personal](/id-ID/channels/zalouser) — Akun Zalo pribadi melalui login QR (plugin, diinstal terpisah).

## Catatan Penting

- Beberapa saluran dapat berjalan secara bersamaan; OpenClaw akan merutekan pesan sesuai dengan tempat obrolan berlangsung.
- Penyiapan tercepat biasanya adalah **Telegram** (hanya butuh token bot sederhana). WhatsApp memerlukan pairing QR dan menyimpan lebih banyak data status di disk.
- Perilaku grup bervariasi di setiap saluran; lihat [Grup](/id-ID/channels/groups).
- Pairing DM dan daftar izinkan (allowlist) diberlakukan demi keamanan; lihat [Keamanan](/id-ID/gateway/security).
- Pemecahan Masalah: [Troubleshooting Saluran](/id-ID/channels/troubleshooting).


