---
summary: "Penyedia model (LLM) yang didukung oleh OpenClaw"
read_when:
  - Anda ingin memilih penyedia model
  - Anda butuh ringkasan cepat backend LLM yang didukung
title: "Penyedia Model"
---

# Penyedia Model

OpenClaw dapat menggunakan banyak penyedia model LLM. Pilih satu penyedia, lakukan autentikasi, lalu atur model default Anda dalam format `penyedia/model`.

Mencari dokumentasi saluran chat (WhatsApp/Telegram/Discord/dll)? Lihat [Saluran](/id-ID/channels/index).

## Sorotan: Venice (Venice AI)

Venice adalah rekomendasi kami untuk penyiapan Venice AI yang mengutamakan privasi, dengan opsi menggunakan Claude Opus untuk tugas-tugas sulit.

- Standar: `venice/llama-3.3-70b`
- Terbaik: `venice/claude-opus-45` (Opus tetap yang terkuat)

Lihat [Venice AI](/id-ID/providers/venice).

## Penyiapan Cepat

1. Lakukan autentikasi dengan penyedia (biasanya melalui `openclaw onboard`).
2. Atur model default di konfigurasi:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Daftar Penyedia

- [OpenAI](/id-ID/providers/openai)
- [Anthropic](/id-ID/providers/anthropic)
- [OpenRouter](/id-ID/providers/openrouter)
- [LiteLLM](/id-ID/providers/litellm)
- [Together AI](/id-ID/providers/together)
- [Moonshot AI (Kimi)](/id-ID/providers/moonshot)
- [Venice AI](/id-ID/providers/venice)
- [Hugging Face](/id-ID/providers/huggingface)
- [Ollama (Model Lokal)](/id-ID/providers/ollama)
- [vLLM (Model Lokal)](/id-ID/providers/vllm)

## Penyedia Transkripsi

- [Deepgram (Transkripsi Audio)](/id-ID/providers/deepgram)

Untuk katalog lengkap (xAI, Groq, Mistral, dll) dan konfigurasi tingkat lanjut, lihat [Penyedia Model](/id-ID/concepts/model-providers).



