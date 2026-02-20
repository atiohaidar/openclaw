---
summary: "Referensi CLI untuk `openclaw tui` (UI terminal yang terhubung ke Gateway)"
read_when:
  - Anda menginginkan UI terminal untuk Gateway (ramah untuk akses jarak jauh)
  - Anda ingin memberikan url/token/sesi melalui skrip
title: "tui"
---

# `openclaw tui`

Membuka antarmuka pengguna terminal (Terminal UI / TUI) yang terhubung ke Gateway.

## Terkait:
- Panduan TUI: [TUI](/id-ID/web/tui)

## Contoh

```bash
# Menjalankan TUI standar
openclaw tui

# Menghubungkan ke Gateway jarak jauh (remote)
openclaw tui --url ws://127.0.0.1:18789 --token <token>

# Menjalankan sesi tertentu dan mengaktifkan pengiriman otomatis (deliver)
openclaw tui --session main --deliver
```
