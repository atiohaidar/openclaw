---
summary: "Referensi CLI OpenClaw untuk perintah `openclaw`, sub-perintah, dan opsi"
read_when:
  - Menambah atau memodifikasi perintah atau opsi CLI
  - Mendokumentasikan permukaan perintah baru
title: "Referensi CLI"
---

# Referensi CLI (Antarmuka Baris Perintah)

Halaman ini mendeskripsikan perilaku CLI OpenClaw saat ini. Gunakan referensi ini untuk memahami perintah dan parameter yang tersedia.

## Halaman Perintah

Klik tautan di bawah ini untuk melihat detail masing-masing perintah:

- [`setup`](/id-ID/cli/setup): Inisialisasi awal.
- [`onboard`](/id-ID/cli/onboard): Panduan penyiapan interaktif.
- [`configure`](/id-ID/cli/configure): Panduan konfigurasi interaktif.
- [`config`](/id-ID/cli/config): Pengaturan konfigurasi non-interaktif.
- [`doctor`](/id-ID/cli/doctor): Pemeriksaan kesehatan dan perbaikan otomatis.
- [`status`](/id-ID/cli/status): Status kesehatan dan sesi.
- [`gateway`](/id-ID/cli/gateway): Manajemen layanan Gateway.
- [`message`](/id-ID/cli/message): Pengiriman pesan dan aksi saluran.
- [`agent`](/id-ID/cli/agent): Menjalankan putaran agen.
- [`channels`](/id-ID/cli/channels): Manajemen saluran perpesanan.
- [`skills`](/id-ID/cli/skills): Daftar dan informasi skill.
- [`plugins`](/id-ID/cli/plugins): Manajemen plugin.
- [`cron`](/id-ID/cli/cron): Penjadwalan pekerjaan latar belakang.
- [`browser`](/id-ID/cli/browser): Kontrol browser.

## Flag Global

- `--dev`: Mengisolasi status di bawah `~/.openclaw-dev` dan mengubah port default.
- `--profile <nama>`: Mengisolasi status di bawah `~/.openclaw-<nama>`.
- `--no-color`: Menonaktifkan warna ANSI pada terminal.
- `-V`, `--version`, `-v`: Mencetak versi aplikasi dan keluar.

## Palet Warna

OpenClaw menggunakan palet warna khusus untuk output CLI:
- **Aksen** (#FF5A2D): Judul, label, sorotan utama.
- **Success** (#2FBF71): Status sukses.
- **Warn** (#FFB020): Peringatan atau perhatian.
- **Error** (#E23D2D): Kesalahan atau kegagalan.
- **Muted** (#8B7F77): Metadata atau teks yang tidak ditekankan.

## Pohon Perintah (Command Tree)

```bash
openclaw [--dev] [--profile <nama>] <perintah>
  setup
  onboard
  configure
  config
    get | set | unset
  doctor
  status
  gateway
    start | stop | restart | status | health
  channels
    list | add | remove | login | logout
  skills
    list | info | check
  plugins
    list | install | enable | disable
  message
    send | poll | react
  cron
    list | add | edit | rm
  browser
    start | stop | open | screenshot
```

## Keamanan

- `openclaw security audit`: Melakukan audit konfigurasi dan status lokal untuk menemukan potensi celah keamanan.
- `openclaw security audit --fix`: Memperketat pengaturan default yang aman.

## Memori

OpenClaw dapat melakukan pencarian vektor pada file memori:
- `openclaw memory status`: Menampilkan statistik indeks.
- `openclaw memory index`: Membuat ulang indeks file memori.
- `openclaw memory search "<query>"`: Melakukan pencarian semantik pada memori.

## Penyiapan (Setup & Onboarding)

### `setup`
Menginisialisasi konfigurasi dan ruang kerja (workspace).
- `--workspace <dir>`: Jalur folder ruang kerja agen (default: `~/.openclaw/workspace`).
- `--wizard`: Menjalankan panduan onboarding.

### `onboard`
Panduan interaktif untuk menyiapkan gateway, ruang kerja, dan skill.
- `--reset`: Mereset konfigurasi, kredensial, dan sesi sebelum memulai panduan.
- `--mode <local|remote>`: Memilih mode lokal atau gateway jarak jauh.

## Manajemen Saluran (Channels)

### `channels add`
Menyiapkan akun saluran obrolan (WhatsApp, Telegram, Discord, dll).
```bash
openclaw channels add --channel telegram --token $TELEGRAM_TOKEN
```

### `channels login`
Login interaktif (khusus WhatsApp Web).
```bash
openclaw channels login --channel whatsapp
```

## Pesan & Agen

### `message send`
Mengirim pesan keluar ke target tertentu.
```bash
openclaw message send --target +628123456789 --message "Halo!"
```

### `agent`
Menjalankan satu putaran (turn) agen secara langsung.
```bash
openclaw agent --message "Buat skrip python sederhana"
```
Opsi: `--to`, `--model`, `--thinking`, `--verbose`.



