---
summary: "Hapus instalasi OpenClaw sepenuhnya (CLI, layanan, state, workspace)"
read_when:
  - Anda ingin menghapus OpenClaw dari mesin
  - Layanan gateway masih berjalan setelah penghapusan instalasi
title: "Hapus Instalasi"
---

# Hapus Instalasi

Ada dua jalur:

- **Jalur mudah** jika `openclaw` masih terinstal.
- **Penghapusan layanan manual** jika CLI sudah tidak ada tetapi layanan gateway masih berjalan.

## Jalur mudah (CLI masih terinstal)

Direkomendasikan: gunakan fitur uninstaller bawaan:

```bash
openclaw uninstall
```

Mode non-interaktif (untuk otomatisasi / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Langkah-langkah manual (hasilnya sama):

1. Hentikan layanan gateway:

```bash
openclaw gateway stop
```

2. Hapus instalasi layanan gateway (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. Hapus state + konfigurasi:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Jika Anda mengatur `OPENCLAW_CONFIG_PATH` ke lokasi kustom di luar direktori state, hapus juga file tersebut.

4. Hapus workspace Anda (opsional, akan menghapus file agen):

```bash
rm -rf ~/.openclaw/workspace
```

5. Hapus instalasi CLI (pilih sesuai metode yang Anda gunakan):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Jika Anda menginstal aplikasi macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

Catatan:

- Jika Anda menggunakan profil (`--profile` / `OPENCLAW_PROFILE`), ulangi langkah 3 untuk setiap direktori state (defaultnya adalah `~/.openclaw-<profil>`).
- Dalam mode remote, direktori state berada di **host gateway**, jadi jalankan langkah 1-4 di sana juga.

## Penghapusan layanan manual (CLI tidak terinstal)

Gunakan cara ini jika layanan gateway terus berjalan tetapi perintah `openclaw` sudah tidak ada.

### macOS (launchd)

Label defaultnya adalah `bot.molt.gateway` (atau `bot.molt.<profil>`; label lama `com.openclaw.*` mungkin masih ada):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

Jika Anda menggunakan profil, ganti label dan nama plist dengan `bot.molt.<profil>`. Hapus juga plist `com.openclaw.*` lama jika ada.

### Linux (unit pengguna systemd)

Nama unit defaultnya adalah `openclaw-gateway.service` (atau `openclaw-gateway-<profil>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Tugas Terjadwal / Scheduled Task)

Nama tugas defaultnya adalah `OpenClaw Gateway` (atau `OpenClaw Gateway (<profil>)`).
Skrip tugas berada di bawah direktori state Anda.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Jika Anda menggunakan profil, hapus nama tugas yang sesuai dan `~\.openclaw-<profil>\gateway.cmd`.

## Instalasi Normal vs Checkout Sumber

### Instalasi Normal (install.sh / npm / pnpm / bun)

Jika Anda menggunakan `https://openclaw.ai/install.sh` atau `install.ps1`, CLI diinstal dengan `npm install -g openclaw@latest`.
Hapus dengan `npm rm -g openclaw` (atau `pnpm remove -g` / `bun remove -g` jika Anda menginstalnya dengan cara tersebut).

### Checkout Sumber / Source Checkout (git clone)

Jika Anda menjalankan dari folder repositori (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1. Hapus instalasi layanan gateway **sebelum** menghapus repositori (gunakan jalur mudah di atas atau penghapusan layanan manual).
2. Hapus direktori repositori.
3. Hapus state + workspace seperti yang ditunjukkan di atas.


