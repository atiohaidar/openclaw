---
summary: "Menjalankan jembatan ACP untuk integrasi IDE"
read_when:
  - Menyiapkan integrasi IDE berbasis ACP
  - Mendebug perutean sesi ACP ke Gateway
title: "acp"
---

# `acp`

Menjalankan jembatan **ACP (Agent Client Protocol)** yang terhubung ke Gateway OpenClaw.

Perintah ini menggunakan protokol ACP melalui stdio untuk IDE dan meneruskan prompt ke Gateway melalui WebSocket.

## Contoh Penggunaan

```bash
# Menjalankan jembatan ACP dasar
openclaw acp

# Menghubungkan ke Gateway jarak jauh (remote)
openclaw acp --url wss://gateway-host:18789 --token <token>

# Menggunakan kunci sesi tertentu
openclaw acp --session agent:main:main

# Mereset sesi sebelum memulai prompt pertama
openclaw acp --session agent:main:main --reset-session
```

## Cara Penggunaan
Gunakan ACP saat IDE (seperti Zed) atau klien lain menggunakan protokol ACP dan Anda ingin menghubungkannya ke sesi Gateway OpenClaw.

1. Pastikan Gateway sudah berjalan.
2. Konfigurasi target Gateway (bisa melalui file config atau flag).
3. Atur IDE Anda untuk menjalankan perintah `openclaw acp`.

## Penyiapan di Editor Zed
Tambahkan agen ACP kustom di `~/.config/zed/settings.json`:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

## Opsi
- `--url <url>`: URL WebSocket Gateway.
- `--token <token>`: Token autentikasi Gateway.
- `--token-file <path>`: Membaca token dari file (lebih aman daripada flag `--token`).
- `--session <key>`: Kunci sesi default (misal: `agent:main:main`).
- `--reset-session`: Memulai transkrip baru untuk kunci sesi yang sama.

**Catatan Keamanan:** Gunakan `--token-file` atau variabel lingkungan untuk menghindari token terlihat di daftar proses sistem.



