---
summary: "Pantau kedaluwarsa OAuth untuk penyedia model"
read_when:
  - Menyiapkan pemantauan atau peringatan kedaluwarsa autentikasi
  - Mengotomatiskan pemeriksaan pembaruan OAuth Claude Code / Codex
title: "Pemantauan Autentikasi (Auth Monitoring)"
---

# Pemantauan Autentikasi

OpenClaw mengekspos status kedaluwarsa OAuth melalui `openclaw models status`. Gunakan fitur ini untuk otomatisasi dan peringatan; skrip hanyalah tambahan opsional untuk alur kerja ponsel.

## Pilihan Utama: Pemeriksaan CLI (Portabel)

```bash
openclaw models status --check
```

Kode Keluar (Exit codes):
- `0`: OK
- `1`: kredensial kedaluwarsa atau hilang
- `2`: segera kedaluwarsa (dalam 24 jam)

Ini berfungsi di cron/systemd dan tidak memerlukan skrip tambahan.

## Skrip Opsional (Ops / Alur Kerja Ponsel)

Skrip-skrip ini berada di bawah folder `scripts/` dan bersifat **opsional**. Skrip ini mengasumsikan akses SSH ke host gateway dan disesuaikan untuk systemd + Termux.

- `scripts/claude-auth-status.sh` kini menggunakan `openclaw models status --json` sebagai sumber kebenaran (sumber data utama). Pastikan `openclaw` ada dalam `PATH` untuk timer.
- `scripts/auth-monitor.sh`: target timer cron/systemd; mengirimkan peringatan (via ntfy atau ponsel).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: timer pengguna systemd.
- `scripts/claude-auth-status.sh`: pemeriksa autentikasi Claude Code + OpenClaw (format full/json/simple).
- `scripts/mobile-reauth.sh`: alur re-autentikasi terpandu melalui SSH.
- `scripts/termux-quick-auth.sh`: status widget satu ketukan + buka URL autentikasi.
- `scripts/termux-auth-widget.sh`: alur kerja widget terpandu lengkap.
- `scripts/termux-sync-widget.sh`: sinkronisasi kredensial Claude Code → OpenClaw.

Jika Anda tidak membutuhkan otomatisasi ponsel atau timer systemd, Anda dapat melewati skrip-skrip ini.
