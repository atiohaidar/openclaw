---
summary: "Kapabilitas OpenClaw di seluruh saluran, perutean, media, dan UX."
read_when:
  - Anda menginginkan daftar lengkap dukungan OpenClaw
title: "Fitur"
---

## Sorotan

<Columns>
  <Card title="Saluran (Channels)" icon="message-square">
    WhatsApp, Telegram, Discord, dan iMessage dengan satu Gateway.
  </Card>
  <Card title="Plugin" icon="plug">
    Tambahkan Mattermost dan lainnya dengan ekstensi.
  </Card>
  <Card title="Perutean (Routing)" icon="route">
    Perutean multi-agen dengan sesi terisolasi.
  </Card>
  <Card title="Media" icon="image">
    Gambar, audio, dan dokumen masuk dan keluar.
  </Card>
  <Card title="Aplikasi dan UI" icon="monitor">
    Web Control UI dan aplikasi pendamping macOS.
  </Card>
  <Card title="Node seluler" icon="smartphone">
    Node iOS dan Android dengan dukungan Canvas.
  </Card>
</Columns>

## Daftar lengkap

- Integrasi WhatsApp melalui WhatsApp Web (Baileys)
- Dukungan bot Telegram (grammY)
- Dukungan bot Discord (channels.discord.js)
- Dukungan bot Mattermost (plugin)
- Integrasi iMessage melalui CLI imsg lokal (macOS)
- Agen bridge untuk Pi dalam mode RPC dengan streaming tool
- Streaming dan chunking untuk respons panjang
- Perutean multi-agen untuk sesi terisolasi per workspace atau pengirim
- Autentikasi berlangganan untuk Anthropic dan OpenAI melalui OAuth
- Sesi: chat langsung (direct chat) digabung ke dalam `main` bersama; grup terisolasi
- Dukungan chat grup dengan aktivasi berbasis mention
- Dukungan media untuk gambar, audio, dan dokumen
- Hook transkripsi voice note opsional
- WebChat dan aplikasi menu bar macOS
- Node iOS dengan pairing dan Canvas surface
- Node Android dengan pairing, Canvas surface, chat, dan kamera

<Note>
Jalur Claude, Codex, Gemini, dan Opencode lama (legacy) telah dihapus. Pi adalah satu-satunya jalur agen coding.
</Note>


