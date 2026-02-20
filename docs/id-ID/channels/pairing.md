---
summary: "Ringkasan Pairing: setujui siapa yang dapat mengirim DM + node mana yang dapat bergabung"
read_when:
  - Mengatur kontrol akses DM
  - Memasangkan node iOS/Android baru
title: "Pairing"
---

# Pairing

“Pairing” (Pemasangan) adalah langkah **persetujuan pemilik** (owner approval) yang eksplisit di OpenClaw. Langkah ini digunakan di dua tempat:

1. **Pairing DM**: Siapa yang diizinkan untuk berbicara dengan bot.
2. **Pairing Node**: Perangkat/node mana yang diizinkan untuk bergabung ke jaringan gateway.

## 1) Pairing DM (Akses Chat Masuk)

Saat sebuah saluran dikonfigurasi dengan kebijakan DM `pairing`, pengirim yang tidak dikenal akan menerima kode singkat dan pesan mereka **tidak akan diproses** sampai Anda menyetujuinya.

### Menyetujui Pengirim

```bash
# Menampilkan daftar permintaan pairing yang masuk
openclaw pairing list telegram

# Menyetujui pengirim berdasarkan kode yang diberikan
openclaw pairing approve telegram <KODE>
```

Saluran yang didukung: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

**Penting:** Kode pairing berlaku selama **1 jam**. Jika tidak disetujui dalam waktu tersebut, pengirim harus memicu permintaan baru.

## 2) Pairing Perangkat Node (iOS/Android/macOS/Headless)

Node terhubung ke Gateway sebagai perangkat. Gateway akan membuat permintaan pairing perangkat yang harus Anda setujui agar node dapat beroperasi.

### Pairing via Telegram (Direkomendasikan untuk iOS)

Jika Anda menggunakan plugin `device-pair`, Anda dapat melakukan pairing perangkat sepenuhnya dari Telegram:

1. Di Telegram, chat bot Anda: `/pair`
2. Bot akan memberikan petunjuk dan **kode penyiapan (setup code)**.
3. Di iPhone Anda, buka aplikasi OpenClaw → Settings → Gateway.
4. Tempel kode penyiapan dan hubungkan.
5. Kembali ke Telegram: `/pair approve`

### Menyetujui Perangkat Node via CLI

```bash
# Lihat daftar perangkat yang menunggu persetujuan
openclaw devices list

# Setujui perangkat
openclaw devices approve <requestId>
```

## Terkait
- [Model Keamanan](/id-ID/gateway/security)
- [Saluran Telegram](/id-ID/channels/telegram)
- [Saluran WhatsApp](/id-ID/channels/whatsapp)


