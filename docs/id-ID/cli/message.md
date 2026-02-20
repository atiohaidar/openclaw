---
summary: "Referensi CLI untuk `openclaw message` (pengiriman pesan + aksi saluran)"
read_when:
  - Menambah atau memodifikasi aksi CLI pesan
  - Mengubah perilaku saluran keluar (outbound channel)
title: "message"
---

# `openclaw message`

Perintah tunggal untuk mengirim pesan dan melakukan aksi saluran (Discord, Telegram, WhatsApp, Slack, dll).

## Penggunaan

```bash
openclaw message <sub-perintah> [flags]
```

### Pemilihan Saluran
- `--channel` wajib diisi jika lebih dari satu saluran dikonfigurasi.
- Nilai: `whatsapp`, `telegram`, `discord`, `googlechat`, `slack`, `signal`, `imessage`, `msteams`.

### Format Target (`--target`)
- **WhatsApp**: nomor E.164 atau JID grup.
- **Telegram**: ID obrolan atau `@username`.
- **Discord**: `channel:<id>` atau `user:<id>`.
- **Signal**: `+E.164`, `group:<id>`, atau `username:<nama>`.
- **MS Teams**: ID percakapan atau `user:<id>`.

## Aksi Utama

### `send` (Kirim)
Mengirim pesan teks atau media.
- Wajib: `--target`, dan salah satu dari `--message` atau `--media`.
- Opsional: `--reply-to`, `--thread-id`.

### `poll` (Jajak Pendapat)
Membuat polling di saluran yang mendukung.
- Wajib: `--target`, `--poll-question`, `--poll-option` (bisa diulang).
- Opsional: `--poll-multi` (izinkan pilih banyak).

### `react` (Reaksi)
Memberikan reaksi emoji pada pesan.
- Wajib: `--message-id`, `--target`, `--emoji`.

## Contoh

### Mengirim pesan balasan di Discord
```bash
openclaw message send --channel discord \
  --target channel:123 --message "halo" --reply-to 456
```

### Membuat polling di Telegram
```bash
openclaw message poll --channel telegram \
  --target @grupku \
  --poll-question "Makan siang?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120
```

### Memberikan reaksi di Slack
```bash
openclaw message react --channel slack \
  --target C123 --message-id 1625... --emoji "✅"
```

### Mengirim tombol inline di Telegram
```bash
openclaw message send --channel telegram --target @grupku --message "Pilih:" \
  --buttons '[ [{"text":"Ya","callback_data":"ya"}], [{"text":"Tidak","callback_data":"tidak"}] ]'
```



