---
summary: "Akses jarak jauh menggunakan terowongan (tunnel) SSH dan tailnet"
read_when:
  - Menjalankan atau mendebug penyiapan gateway jarak jauh
title: "Akses Jarak Jauh"
---

# Akses Jarak Jauh (SSH & VPN)

OpenClaw mendukung penggunaan "pos pemeriksaan jarak jauh" dengan membiarkan satu Gateway utama (master) berjalan di host khusus (komputer rumah atau server) dan menghubungkan berbagai klien ke sana.

- **Untuk Operator (Anda/App Mac)**: Tunneling SSH adalah solusi cadangan universal.
- **Untuk Node (iOS/Android)**: Menghubungkan ke Gateway melalui WebSocket (baik di LAN atau jaringan Tailscale).

## Konsep Utama

- Gateway WebSocket secara default berjalan secara lokal (`loopback`) pada port 18789.
- Untuk penggunaan jarak jauh, Anda bisa meneruskan port tersebut melalui SSH atau menggunakan VPN seperti Tailscale.

## Skenario Penggunaan Umum

### 1) Gateway Selalu Aktif (Server/VPS)
Jalankan Gateway di server yang selalu menyala (misal: di [Hetzner](/id-ID/install/hetzner) atau server rumah).
- **Terbaik**: Biarkan gateway di mode `loopback` dan gunakan **Tailscale Serve** untuk mengakses UI Kontrol secara aman.
- **Cadangan**: Gunakan terowongan SSH dari perangkat apa pun yang butuh akses.

### 2) Komputer Rumah (Host) + Laptop (Remote)
Laptop Anda tidak menjalankan agen, melainkan mengontrol komputer rumah Anda.
- Gunakan mode **Remote over SSH** di aplikasi macOS untuk kemudahan penyiapan otomatis.

## Terowongan (Tunnel) SSH via CLI

Gunakan perintah ini untuk membuat terowongan dari komputer lokal ke gateway jarak jauh:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host-anda
```

Setelah terowongan aktif, perintah CLI seperti `openclaw health` akan menjangkau gateway jarak jauh seolah-olah berjalan secara lokal.

## Keamanan

- **Gunakan Loopback**: Sangat disarankan untuk membiarkan gateway hanya bisa diakses secara lokal kecuali jika Anda benar-benar butuh membukanya ke jaringan.
- **Token/Password**: Jika Anda membuka akses di luar loopback, pastikan menggunakan token autentikasi.
- **Tailscale**: Gunakan fitur `gateway.auth.allowTailscale: true` untuk mempermudah login otomatis berbasis identitas Tailscale.

## Terkait
- [Keamanan](/id-ID/gateway/security)
- [Penemuan Jaringan](/id-ID/gateway/discovery)
- [Akses Remote macOS](/id-ID/platforms/mac/remote)



