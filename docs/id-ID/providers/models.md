---
summary: "Penyedia model (LLM) yang didukung oleh OpenClaw"
read_when:
  - Anda ingin memilih penyedia model
  - Anda ingin melihat contoh penyiapan cepat untuk autentikasi LLM + pemilihan model
title: "Panduan Cepat Penyedia Model"
---

# Penyedia Model

OpenClaw dapat menggunakan banyak penyedia model LLM. Pilih satu, lakukan autentikasi, lalu atur model default Anda dalam format `penyedia/model`.

## Sorotan: Venice (Venice AI)

Venice adalah rekomendasi kami untuk penyiapan Venice AI yang mengutamakan privasi, dengan opsi menggunakan Claude Opus untuk tugas-tugas tersulit.

- Standar: `venice/llama-3.3-70b`
- Terbaik: `venice/claude-opus-45` (Opus tetap yang terkuat)

Lihat [Venice AI](/id-ID/providers/venice).

## Penyiapan Cepat (Dua Langkah)

1. Lakukan autentikasi dengan penyedia (biasanya melalui `openclaw onboard`).
2. Atur model default di konfigurasi:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Penyedia yang Didukung (Set Awal)

- [OpenAI](/id-ID/providers/openai)
- [Anthropic](/id-ID/providers/anthropic)
- [OpenRouter](/id-ID/providers/openrouter)
- [Venice AI](/id-ID/providers/venice)
- [Moonshot AI (Kimi)](/id-ID/providers/moonshot)
- [Amazon Bedrock](/id-ID/providers/bedrock)

Untuk katalog lengkap (xAI, Groq, Mistral, dll) dan konfigurasi tingkat lanjut, lihat [Penyedia Model](/id-ID/concepts/model-providers).



