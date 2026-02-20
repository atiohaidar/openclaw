---
summary: "Autentikasi model: OAuth, kunci API, dan setup-token"
read_when:
  - Mendebug autentikasi model atau kedaluwarsa OAuth
  - Mendokumentasikan autentikasi atau penyimpanan kredensial
title: "Autentikasi"
---

# Autentikasi

OpenClaw mendukung OAuth dan kunci API untuk penyedia model. Untuk akun Anthropic, kami merekomendasikan penggunaan **kunci API (API key)**. Untuk akses langganan Claude, gunakan token berumur panjang yang dibuat oleh `claude setup-token`.

Lihat [/id-ID/concepts/oauth](/id-ID/concepts/oauth) untuk alur OAuth lengkap dan tata letak penyimpanan.

## Pengaturan Anthropic yang Direkomendasikan (Kunci API)

Jika Anda menggunakan Anthropic secara langsung, gunakan kunci API.

1. Buat kunci API di Konsol Anthropic.
2. Letakkan kunci tersebut di **host gateway** (mesin yang menjalankan `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Jika Gateway berjalan di bawah systemd/launchd, lebih disarankan meletakkan kunci tersebut di `~/.openclaw/.env` agar daemon dapat membacanya:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Kemudian restart daemon (atau restart proses Gateway Anda) dan periksa kembali:

```bash
openclaw models status
openclaw doctor
```

Jika Anda tidak ingin mengelola variabel lingkungan (env var) sendiri, wizard onboarding dapat menyimpan kunci API untuk penggunaan daemon: `openclaw onboard`.

Lihat [Bantuan](/help) untuk detail tentang pewarisan lingkungan (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic: setup-token (autentikasi berlangganan)

Untuk Anthropic, jalur yang direkomendasikan adalah **kunci API**. Jika Anda menggunakan langganan Claude, alur setup-token juga didukung. Jalankan perintah ini di **host gateway**:

```bash
claude setup-token
```

Kemudian tempelkan ke OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Jika token dibuat di mesin lain, tempelkan secara manual:

```bash
openclaw models auth paste-token --provider anthropic
```

Jika Anda melihat kesalahan Anthropic seperti:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…gunakan kunci API Anthropic sebagai gantinya.

Pemasukan token manual (untuk penyedia mana pun; menulis ke `auth-profiles.json` + memperbarui konfigurasi):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Pemeriksaan yang ramah otomatisasi (keluar dengan kode `1` jika kedaluwarsa/hilang, `2` saat hampir kedaluwarsa):

```bash
openclaw models status --check
```

Skrip operasi opsional (systemd/Termux) didokumentasikan di sini:
[/automation/auth-monitoring](/automation/auth-monitoring)

> `claude setup-token` memerlukan TTY interaktif.

## Memeriksa status autentikasi model

```bash
openclaw models status
openclaw doctor
```

## Perilaku rotasi kunci API (gateway)

Beberapa penyedia mendukung upaya ulang (retry) permintaan dengan kunci alternatif saat panggilan API mencapai batas kecepatan (rate limit) penyedia.

- Urutan prioritas:
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (override tunggal)
  - `<PROVIDER>_API_KEYS`
  - `<PROVIDER>_API_KEY`
  - `<PROVIDER>_API_KEY_*`
- Penyedia Google juga menyertakan `GOOGLE_API_KEY` sebagai cadangan tambahan.
- Daftar kunci yang sama akan di-deduplikasi sebelum digunakan.
- OpenClaw mencoba ulang dengan kunci berikutnya hanya untuk kesalahan batas kecepatan (misalnya `429`, `rate_limit`, `quota`, `resource exhausted`).
- Kesalahan selain batas kecepatan tidak akan dicoba ulang dengan kunci alternatif.
- Jika semua kunci gagal, kesalahan terakhir dari upaya terakhir akan dikembalikan.

## Mengontrol kredensial mana yang digunakan

### Per-sesi (perintah chat)

Gunakan `/model <alias-atau-id>@<profileId>` untuk menetapkan kredensial penyedia tertentu untuk sesi saat ini (contoh ID profil: `anthropic:default`, `anthropic:work`).

Gunakan `/model` (atau `/model list`) untuk pemilih ringkas; gunakan `/model status` untuk tampilan penuh (kandidat + profil autentikasi berikutnya, ditambah detail titik akhir penyedia jika dikonfigurasi).

### Per-agen (override CLI)

Atur override urutan profil autentikasi eksplisit untuk agen (disimpan dalam `auth-profiles.json` agen tersebut):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Gunakan `--agent <id>` untuk menargetkan agen tertentu; abaikan untuk menggunakan agen default yang dikonfigurasi.

## Pemecahan Masalah

### “No credentials found” (Kredensial tidak ditemukan)

Jika profil token Anthropic hilang, jalankan `claude setup-token` di **host gateway**, lalu periksa kembali:

```bash
openclaw models status
```

### Token hampir kedaluwarsa / sudah kedaluwarsa

Jalankan `openclaw models status` untuk mengonfirmasi profil mana yang akan kedaluwarsa. Jika profil hilang, jalankan kembali `claude setup-token` dan tempelkan token tersebut lagi.

## Persyaratan

- Langganan Claude Max atau Pro (untuk `claude setup-token`)
- CLI Claude Code telah terinstal (perintah `claude` tersedia)


