---
summary: "Gunakan Anthropic Claude via kunci API atau setup-token di OpenClaw"
read_when:
  - Anda ingin menggunakan model Anthropic di OpenClaw
  - Anda memilih menggunakan setup-token alih-alih kunci API
title: "Anthropic"
---

# Anthropic (Claude)

Anthropic membuat keluarga model **Claude** dan menyediakan akses melalui API. Di OpenClaw, Anda dapat melakukan autentikasi menggunakan kunci API (API Key) atau **setup-token**.

## Opsi A: Kunci API Anthropic

**Sangat Direkomendasikan untuk:** Penggunaan standar berbasis API (Pay-as-you-go). Buat kunci API Anda di Anthropic Console.

### Penyiapan via CLI

```bash
openclaw onboard
# pilih: Anthropic API key
```

### Cuplikan Konfigurasi

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Caching Prompt (Anthropic API)

OpenClaw mendukung fitur caching prompt dari Anthropic untuk menghemat biaya dan mempercepat respon. Fitur ini hanya tersedia melalui **link API**; autentikasi langganan (subscription) tidak menggunakan fitur ini.

### Konfigurasi Cache

Gunakan parameter `cacheRetention` dalam konfigurasi model Anda:

- `none`: Tanpa caching.
- `short`: 5 menit (Default jika menggunakan Kunci API).
- `long`: 1 jam (Memerlukan flag beta).

## Opsi B: Claude setup-token

**Sangat Direkomendasikan untuk:** Menggunakan langganan (subscription) Claude Pro Anda langsung di OpenClaw.

### Cara mendapatkan setup-token

Setup-token dibuat melalui **Claude Code CLI**, bukan Anthropic Console. Jalankan perintah ini di mesin mana pun:

```bash
claude setup-token
```

Salin token tersebut ke OpenClaw (gunakan wizard: **Anthropic token (paste setup-token)**) atau jalankan langsung di host gateway:

```bash
openclaw models auth paste-token --provider anthropic
```

## Pemecahan Masalah

- **401 Error / Token Tidak Valid**: Autentikasi langganan Claude dapat kedaluwarsa. Jalankan ulang `claude setup-token` dan masukkan token baru ke host gateway.
- **Kunci API Tidak Ditemukan**: Autentikasi dilakukan **per agen**. Agen baru tidak secara otomatis mewarisi kunci dari agen utama. Jalankan ulang onboarding untuk agen tersebut.

## Terkait
- [OAuth & Refresh Token](/id-ID/concepts/oauth)
- [Troubleshooting](/id-ID/gateway/troubleshooting)



