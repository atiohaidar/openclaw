---
summary: "Gunakan OpenAI via kunci API atau langganan Codex di OpenClaw"
read_when:
  - Anda ingin menggunakan model OpenAI di OpenClaw
  - Anda ingin menggunakan autentikasi langganan Codex alih-alih kunci API
title: "OpenAI"
---

# OpenAI

OpenAI menyediakan API pengembang untuk keluarga model GPT. OpenClaw mendukung akses melalui **Kunci API** (OpenAI Platform) atau **Langganan Codex** (ChatGPT sign-in).

## Opsi A: Kunci API OpenAI

**Sangat Direkomendasikan untuk:** Akses API langsung dengan penagihan berdasarkan penggunaan (Pay-as-you-go). Ambil kunci API Anda dari dashboard OpenAI.

### Penyiapan via CLI

```bash
openclaw onboard --auth-choice openai-api-key
```

### Cuplikan Konfigurasi

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-4o" } } },
}
```

## Opsi B: Langganan OpenAI Code (Codex)

**Sangat Direkomendasikan untuk:** Menggunakan akses langganan ChatGPT/Codex Pro Anda alih-alih membayar per penggunaan API.

### Penyiapan via CLI (Codex OAuth)

```bash
# Jalankan login OAuth via wizard
openclaw onboard --auth-choice openai-codex

# Atau jalankan login langsung
openclaw models auth login --provider openai-codex
```

### Cuplikan Konfigurasi (Langganan Codex)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-4o" } } },
}
```

## Catatan
- Referensi model selalu menggunakan format `penyedia/model`.
- Detail autentikasi dan aturan penggunaan kembali dapat dilihat di [OAuth](/id-ID/concepts/oauth).



