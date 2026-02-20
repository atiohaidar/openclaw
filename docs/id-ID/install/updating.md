---
summary: "Memperbarui OpenClaw dengan aman (instalasi global atau sumber), plus strategi rollback"
read_when:
  - Memperbarui OpenClaw
  - Terjadi masalah setelah pembaruan
title: "Memperbarui"
---

# Memperbarui

OpenClaw berkembang sangat cepat (masih dalam tahap pra “1.0”). Perlakukan pembaruan seperti infrastruktur pengiriman: perbarui → jalankan pemeriksaan → restart (atau gunakan `openclaw update`, yang otomatis merestart) → verifikasi.

## Direkomendasikan: jalankan ulang skrip instalasi (upgrade di tempat)

Jalur pembaruan yang **disukai** adalah dengan menjalankan ulang skrip instalasi dari situs web. Skrip ini mendeteksi instalasi yang sudah ada, melakukan upgrade di tempat (upgrade in place), dan menjalankan `openclaw doctor` saat dibutuhkan.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Catatan:

- Tambahkan `--no-onboard` jika Anda tidak ingin wizard onboarding dijalankan lagi.
- Untuk **instalasi dari sumber (source)**, gunakan:

  ```bash
  curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
  ```

  Installer akan menjalankan `git pull --rebase` **hanya** jika repositori dalam keadaan bersih (clean).

- Untuk **instalasi global**, skrip secara internal menggunakan `npm install -g openclaw@latest`.
- Catatan lama: `clawdbot` tetap tersedia sebagai shim kompatibilitas.

## Sebelum Anda memperbarui

- Ketahui metode instalasi Anda: **global** (npm/pnpm) vs **dari sumber** (git clone).
- Ketahui cara Gateway Anda berjalan: **terminal latar depan (foreground)** vs **layanan terawasi** (launchd/systemd).
- Ambil snapshot konfigurasi Anda:
  - Konfigurasi: `~/.openclaw/openclaw.json`
  - Kredensial: `~/.openclaw/credentials/`
  - Workspace: `~/.openclaw/workspace`

## Perbarui (instalasi global)

Instalasi global (pilih salah satu):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Kami **tidak** merekomendasikan Bun untuk runtime Gateway (ada bug pada WhatsApp/Telegram).

Untuk berpindah saluran pembaruan (instalasi git + npm):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Gunakan `--tag <tag-distribusi|versi>` untuk instalasi tag/versi tertentu secara sekali jalan.

Lihat [Saluran pengembangan](/id-ID/install/development-channels) untuk detail saluran dan catatan rilis (release notes).

Catatan: pada instalasi npm, gateway akan mencatat petunjuk pembaruan saat startup (memeriksa tag saluran saat ini). Nonaktifkan via `update.checkOnStart: false`.

Setelah itu:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Catatan:

- Jika Gateway Anda berjalan sebagai layanan, perintah `openclaw gateway restart` lebih disukai daripada menghentikan (kill) PID secara manual.
- Jika Anda mengunci versi tertentu, lihat “Rollback / penguncian versi” di bawah.

## Perbarui (`openclaw update`)

Untuk **instalasi dari sumber** (git checkout), lebih disarankan menggunakan:

```bash
openclaw update
```

Perintah ini menjalankan alur pembaruan yang cukup aman:

- Memerlukan worktree yang bersih (clean).
- Berpindah ke saluran yang dipilih (tag atau branch).
- Mengambil (fetch) + rebase terhadap upstream yang dikonfigurasi (saluran dev).
- Menginstal dependensi, membangun biner (build), membangun Control UI, dan menjalankan `openclaw doctor`.
- Merestart gateway secara default (gunakan `--no-restart` untuk melewatkan).

Jika Anda menginstal via **npm/pnpm** (tidak ada metadata git), `openclaw update` akan mencoba memperbarui melalui pengelola paket Anda. Jika tidak dapat mendeteksi metode instalasi, gunakan cara “Perbarui (instalasi global)”.

## Perbarui (Control UI / RPC)

Control UI memiliki opsi **Update & Restart** (RPC: `update.run`). Opsi ini:

1. Menjalankan alur pembaruan sumber yang sama dengan `openclaw update` (hanya untuk git checkout).
2. Menulis sentinel restart dengan laporan terstruktur (hasil akhir stdout/stderr).
3. Merestart gateway dan mengirimkan ping ke sesi aktif terakhir dengan laporan tersebut.

Jika rebase gagal, gateway akan membatalkan proses dan merestart tanpa menerapkan pembaruan.

## Perbarui (dari sumber)

Dari folder repositori (checkout):

Disukai:

```bash
openclaw update
```

Manual (kurang lebih setara):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # otomatis menginstal dependensi UI pada jalankan pertama
openclaw doctor
openclaw health
```

Catatan:

- `pnpm build` penting saat Anda menjalankan biner `openclaw` yang sudah dikemas ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) atau menggunakan Node untuk menjalankan `dist/`.
- Jika Anda menjalankan dari folder repositori tanpa instalasi global, gunakan `pnpm openclaw ...` untuk perintah CLI.
- Jika Anda menjalankan langsung dari TypeScript (`pnpm openclaw ...`), pembangunan ulang (rebuild) biasanya tidak diperlukan, tetapi **migrasi konfigurasi tetap berlaku** → jalankan doctor.
- Beralih antara instalasi global dan git sangat mudah: instal rasa lainnya, lalu jalankan `openclaw doctor` agar titik masuk (entrypoint) layanan gateway ditulis ulang ke instalasi saat ini.

## Selalu Jalankan: `openclaw doctor`

Doctor adalah perintah “pembaruan aman”. Sengaja dibuat membosankan: memperbaiki + migrasi + memperingatkan.

Catatan: jika Anda menggunakan **instalasi dari sumber** (git checkout), `openclaw doctor` akan menawarkan untuk menjalankan `openclaw update` terlebih dahulu.

Hal-hal umum yang dilakukan:

- Migrasi kunci konfigurasi yang sudah usang / lokasi file konfigurasi lama.
- Audit kebijakan DM dan memperingatkan pengaturan “terbuka” yang berisiko.
- Memeriksa kesehatan Gateway dan dapat menawarkan untuk merestart.
- Mendeteksi dan migrasi layanan gateway lama (launchd/systemd; schtasks lama) ke layanan OpenClaw saat ini.
- Di Linux, memastikan user lingering systemd aktif (agar Gateway tetap berjalan setelah logout).

Detail: [Doctor](/id-ID/gateway/doctor)

## Mulai / stop / restart Gateway

CLI (berfungsi di OS apa pun):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Jika Anda menggunakan layanan terawasi:

- macOS launchd (LaunchAgent bawaan aplikasi): `launchctl kickstart -k gui/$UID/bot.molt.gateway` (gunakan `bot.molt.<profil>`; `com.openclaw.*` lama masih berfungsi)
- Layanan pengguna systemd di Linux: `systemctl --user restart openclaw-gateway[-<profil>].service`
- Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profil>].service`
  - `launchctl`/`systemctl` hanya berfungsi jika layanan sudah terinstal; jika belum, jalankan `openclaw gateway install`.

Panduan prosedur (runbook) + label layanan yang tepat: [Runbook Gateway](/id-ID/gateway)

## Rollback / penguncian versi (saat terjadi masalah)

### Rollback (instalasi global)

Instal versi yang diketahui berfungsi baik (ganti `<versi>` dengan versi terakhir yang berfungsi):

```bash
npm i -g openclaw@<versi>
```

```bash
pnpm add -g openclaw@<versi>
```

Tips: untuk melihat versi yang dipublikasikan saat ini, jalankan `npm view openclaw version`.

Kemudian restart + jalankan ulang doctor:

```bash
openclaw doctor
openclaw gateway restart
```

### Rollback (sumber) berdasarkan tanggal

Pilih commit berdasarkan tanggal (contoh: “kondisi main pada 2026-01-01”):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before="2026-01-01" origin/main)"
```

Kemudian instal ulang dependensi + restart:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Jika Anda ingin kembali ke versi terbaru nanti:

```bash
git checkout main
git pull
```

## Jika Anda buntu

- Jalankan `openclaw doctor` lagi dan baca outputnya dengan teliti (seringkali memberikan solusi).
- Periksa: [Pemecahan Masalah](/id-ID/gateway/troubleshooting)
- Bertanya di Discord: [https://discord.gg/clawd](https://discord.gg/clawd)



