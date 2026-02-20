---
summary: "Perbaiki masalah startup CDP Chrome/Brave/Edge/Chromium untuk kontrol browser OpenClaw di Linux"
read_when: "Kontrol browser gagal di Linux, terutama dengan snap Chromium"
title: "Pemecahan Masalah Browser (Linux)"
---

# Pemecahan Masalah Browser (Linux)

## Masalah: "Failed to start Chrome CDP on port 18800"

Server kontrol browser OpenClaw gagal meluncurkan Chrome/Brave/Edge/Chromium dengan pesan kesalahan:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile "openclaw"."}
```

### Penyebab Utama

Pada Ubuntu (dan banyak distro Linux lainnya), instalasi Chromium default adalah sebuah **paket snap**. Pembatasan (confinement) AppArmor dari Snap mengganggu cara OpenClaw memulai dan memantau proses browser.

Perintah `apt install chromium` sebenarnya menginstal paket perantara yang mengarahkan ke snap. Ini bukanlah browser asli, melainkan hanya sebuah pembungkus (wrapper).

### Solusi 1: Instal Google Chrome (Direkomendasikan)

Instal paket resmi Google Chrome `.deb`, yang tidak dibatasi oleh snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # jika ada kesalahan dependensi
```

Kemudian perbarui konfigurasi OpenClaw Anda (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Solusi 2: Gunakan Snap Chromium dengan Mode Attach-Only

Jika Anda harus menggunakan snap Chromium, konfigurasi OpenClaw untuk menempel (attach) pada browser yang dijalankan secara manual:

1. Perbarui konfigurasi:
```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "noSandbox": true
  }
}
```

2. Jalankan Chromium secara manual:
```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. (Opsional) Buat layanan pengguna systemd untuk menjalankan Chrome secara otomatis.

### Verifikasi Kerja Browser

Periksa status:
```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Uji penjelajahan:
```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Referensi Konfigurasi

| Opsi                     | Deskripsi                                                | Default          |
| ------------------------ | -------------------------------------------------------- | ---------------- |
| `browser.enabled`        | Mengaktifkan kontrol browser                             | `true`           |
| `browser.executablePath` | Jalur ke biner browser berbasis Chromium                 | deteksi otomatis |
| `browser.headless`       | Jalankan tanpa antarmuka GUI                             | `false`          |
| `browser.noSandbox`      | Tambahkan flag `--no-sandbox`                            | `false`          |
| `browser.attachOnly`     | Jangan luncurkan browser, hanya tempel ke yang sudah ada | `false`          |

### Masalah: "Chrome extension relay is running, but no tab is connected"

Anda menggunakan profil `chrome` (relay ekstensi). Ini mengharuskan ekstensi browser OpenClaw menempel pada tab yang aktif.

Solusi:
1. **Gunakan browser terkelola**: `openclaw browser start --browser-profile openclaw`.
2. **Gunakan relay ekstensi**: instal ekstensi, buka tab, dan klik ikon ekstensi OpenClaw untuk menghubungkannya.



