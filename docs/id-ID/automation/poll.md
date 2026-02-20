---
summary: "Pengiriman jajak pendapat (poll) melalui gateway + CLI"
read_when:
  - Menambah atau memodifikasi dukungan jajak pendapat
  - Mendebug pengiriman jajak pendapat dari CLI atau gateway
title: "Jajak Pendapat (Polls)"
---

# Jajak Pendapat (Polls)

## Saluran yang Didukung

- WhatsApp (saluran web)
- Discord
- MS Teams (Kartu Adaptif / Adaptive Cards)

## CLI

```bash
# WhatsApp
openclaw message poll --target +628123456789 \
  --poll-question "Makan siang hari ini?" --poll-option "Ya" --poll-option "Tidak" --poll-option "Mungkin"

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Camilan?" --poll-option "Pizza" --poll-option "Sushi"
```

Opsi:
- `--channel`: `whatsapp` (default), `discord`, atau `msteams`.
- `--poll-multi`: izinkan memilih lebih dari satu opsi.
- `--poll-duration-hours`: Khusus Discord (default 24 jam).

## Perbedaan Saluran

- **WhatsApp**: 2-12 opsi, mengabaikan `durationHours`.
- **Discord**: 2-10 opsi, `durationHours` dibatasi antara 1-768 jam.
- **MS Teams**: Polling dikelola oleh OpenClaw menggunakan Kartu Adaptif. Tidak ada API jajak pendapat asli; `durationHours` diabaikan.

## Alat Agen (Message Tool)

Gunakan alat `message` dengan tindakan `poll` (`to`, `pollQuestion`, `pollOption`, opsional `pollMulti`, `pollDurationHours`, `channel`).

Catatan: Jajak pendapat Teams dirender sebagai Kartu Adaptif dan mengharuskan gateway tetap online untuk mencatat suara di `~/.openclaw/msteams-polls.json`.

