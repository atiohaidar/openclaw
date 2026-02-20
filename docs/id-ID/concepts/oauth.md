---
summary: "OAuth di OpenClaw: pertukaran token, penyimpanan, dan pola multi-akun"
read_when:
  - Anda ingin memahami OAuth OpenClaw dari ujung ke ujung
  - Anda mengalami masalah invalidasi token / logout
  - Anda menginginkan alur autentikasi setup-token atau OAuth
  - Anda menginginkan banyak akun atau perutean profil
title: "OAuth"
---

# OAuth

OpenClaw mendukung "autentikasi berlangganan" melalui OAuth untuk penyedia yang menawarkannya (terutama **OpenAI Codex (ChatGPT OAuth)**). Untuk langganan Anthropic, gunakan alur **setup-token**. Halaman ini menjelaskan:

- bagaimana **token exchange** OAuth bekerja (PKCE)
- di mana token **disimpan** (dan mengapa)
- cara menangani **banyak akun** (profil + override per-sesi)

OpenClaw juga mendukung **plugin penyedia** yang mengirimkan alur OAuth atau kunci API mereka sendiri. Jalankan mereka melalui:

```bash
openclaw models auth login --provider <id>
```

## Token sink (mengapa ini ada)

Penyedia OAuth biasanya **menerbitkan refresh token baru** selama alur login/refresh. Beberapa penyedia (atau klien OAuth) dapat membatalkan refresh token lama ketika yang baru diterbitkan untuk pengguna/aplikasi yang sama.

Masalah yang sering muncul:

- Anda login melalui OpenClaw _dan_ melalui Claude Code / Codex CLI → salah satunya secara acak menjadi "logged out" nanti

Untuk menguranginya, OpenClaw memperlakukan `auth-profiles.json` sebagai **token sink**:

- runtime membaca kredensial dari **satu tempat**
- kita dapat menyimpan beberapa profil dan merutekannya secara deterministik

## Penyimpanan (di mana token berada)

Rahasia disimpan **per-agen**:

- Profil Auth (OAuth + Kunci API): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- Cache Runtime (dikelola otomatis; jangan diedit): `~/.openclaw/agents/<agentId>/agent/auth.json`

File impor-saja lama (legacy) (masih didukung, tetapi bukan penyimpanan utama):

- `~/.openclaw/credentials/oauth.json` (diimpor ke `auth-profiles.json` pada penggunaan pertama)

Semua hal di atas juga menghormati `$OPENCLAW_STATE_DIR` (override direktori state). Referensi lengkap: [/id-ID/gateway/configuration](/id-ID/gateway/configuration#auth-storage-oauth--api-keys)

## Setup-token Anthropic (autentikasi berlangganan)

Jalankan `claude setup-token` di mesin mana pun, lalu tempelkan ke OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Jika Anda membuat token di tempat lain, tempelkan secara manual:

```bash
openclaw models auth paste-token --provider anthropic
```

Verifikasi:

```bash
openclaw models status
```

## Pertukaran OAuth (bagaimana login bekerja)

Alur login interaktif OpenClaw diimplementasikan di `@mariozechner/pi-ai` dan dihubungkan ke wizard/perintah.

### Setup-token Anthropic (Claude Pro/Max)

Bentuk alur:

1. jalankan `claude setup-token`
2. tempelkan token ke OpenClaw
3. simpan sebagai profil autentikasi token (tanpa refresh)

Jalur wizard adalah `openclaw onboard` → pilihan auth `setup-token` (Anthropic).

### OpenAI Codex (ChatGPT OAuth)

Bentuk alur (PKCE):

1. buat verifier/challenge PKCE + `state` acak
2. buka `https://auth.openai.com/oauth/authorize?...`
3. coba tangkap callback di `http://127.0.0.1:1455/auth/callback`
4. jika callback tidak dapat mengikat (atau Anda berada di remote/headless), tempelkan URL/kode redirect
5. tukar di `https://auth.openai.com/oauth/token`
6. ekstrak `accountId` dari token akses dan simpan `{ access, refresh, expires, accountId }`

Jalur wizard adalah `openclaw onboard` → pilihan auth `openai-codex`.

## Refresh + kedaluwarsa

Profil menyimpan stempel waktu `expires`.

Saat runtime:

- jika `expires` ada di masa depan → gunakan token akses yang tersimpan
- jika kedaluwarsa → refresh (di bawah kunci file) dan timpa kredensial yang tersimpan

Alur refresh otomatis; Anda umumnya tidak perlu mengelola token secara manual.

## Banyak akun (profil) + perutean

Dua pola:

### 1) Disukai: agen terpisah

Jika Anda ingin "pribadi" dan "kerja" tidak pernah berinteraksi, gunakan agen yang terisolasi (sesi terpisah + kredensial + workspace):

```bash
openclaw agents add work
openclaw agents add personal
```

Kemudian konfigurasikan auth per-agen (wizard) dan rutekan chat ke agen yang tepat.

### 2) Tingkat lanjut: banyak profil dalam satu agen

`auth-profiles.json` mendukung beberapa ID profil untuk penyedia yang sama.

Pilih profil mana yang digunakan:

- secara global melalui urutan konfigurasi (`auth.order`)
- per-sesi melalui `/model ...@<profileId>`

Contoh (override sesi):

- `/model Opus@anthropic:work`

Cara melihat ID profil apa yang ada:

- `openclaw channels list --json` (menampilkan `auth[]`)

Dokumen terkait:

- [/concepts/model-failover](/concepts/model-failover) (aturan rotasi + cooldown)
- [/tools/slash-commands](/tools/slash-commands) (command surface)


