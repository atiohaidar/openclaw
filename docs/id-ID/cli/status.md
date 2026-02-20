---
summary: "Referensi CLI untuk `openclaw status` (diagnosa, pemeriksaan, dan ringkasan penggunaan)"
read_when:
  - Anda ingin diagnosa cepat tentang kesehatan saluran + penerima sesi terbaru
  - Anda ingin status lengkap ("all") yang bisa disalin-tempel untuk pemecahan masalah
title: "status"
---

# `openclaw status`

Diagnosa untuk saluran (channels) dan sesi.

```bash
# Menampilkan status ringkas
openclaw status

# Menampilkan semua informasi diagnosa (siap tempel untuk debug)
openclaw status --all

# Melakukan pemeriksaan mendalam (live probes) pada saluran
openclaw status --deep

# Menampilkan penggunaan/kuota dari penyedia model (jika didukung)
openclaw status --usage
```

## Catatan:
- `--deep` melakukan pemeriksaan langsung pada WhatsApp Web, Telegram, Discord, Google Chat, Slack, dan Signal.
- Output mencakup status instalasi/runtime layanan Gateway dan host node jika tersedia.
- Ringkasan juga menampilkan saluran pembaruan dan git SHA (untuk instalasi sumber).
- Jika ada pembaruan tersedia, status akan menampilkan petunjuk untuk menjalankan `openclaw update`.



