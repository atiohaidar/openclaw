---
summary: "CLI Gateway OpenClaw (`openclaw gateway`) — menjalankan, query, dan menemukan gateway"
read_when:
  - Menjalankan Gateway dari CLI (dev atau server)
  - Mendebug autentikasi, mode bind, dan konektivitas Gateway
  - Menemukan gateway melalui Bonjour (LAN + tailnet)
title: "gateway"
---

# Gateway CLI

Gateway adalah server WebSocket OpenClaw yang menangani saluran (channels), node, sesi, dan hook.

## Menjalankan Gateway

Jalankan proses Gateway lokal:
```bash
openclaw gateway
```

Atau gunakan alias:
```bash
openclaw gateway run
```

### Opsi Utama
- `--port <port>`: Port WebSocket (default: `18789`).
- `--bind <loopback|lan|tailnet|auto|custom>`: Mode pengikatan (bind) pendengar.
- `--token <token>`: Memberikan token autentikasi.
- `--tailscale <off|serve|funnel>`: Mengekspos Gateway melalui Tailscale.
- `--dev`: Menggunakan folder konfigurasi dan ruang kerja pengembangan.

## Memeriksa Gateway yang Berjalan

Semua perintah pemeriksaan menggunakan WebSocket RPC.

### `gateway health`
Memeriksa kesehatan dasar Gateway.
```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### `gateway status`
Menampilkan status layanan Gateway (systemd/launchd) dan pemeriksaan RPC opsional.
```bash
openclaw gateway status
```

### `gateway probe`
Perintah debug lengkap untuk memeriksa konektivitas ke gateway lokal maupun jarak jauh.
```bash
openclaw gateway probe
```

## Mengelola Layanan Gateway

Layanan gateway dapat dikelola menggunakan sub-perintah berikut:
```bash
openclaw gateway install    # Menginstal sebagai layanan latar belakang
openclaw gateway start      # Memulai layanan
openclaw gateway stop       # Menghentikan layanan
openclaw gateway restart    # Memulai ulang layanan
openclaw gateway uninstall  # Menghapus layanan
```

## Penemuan Gateway (Bonjour)

`gateway discover` memindai beacon Gateway (`_openclaw-gw._tcp`) di jaringan lokal (LAN) atau Tailscale.
```bash
openclaw gateway discover --timeout 4000
```
Opsi `--json` akan menampilkan output dalam format JSON untuk kebutuhan integrasi skrip.


