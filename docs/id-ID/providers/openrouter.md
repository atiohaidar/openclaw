---
summary: "Gunakan API tunggal OpenRouter untuk mengakses banyak model di OpenClaw"
read_when:
  - Anda ingin menggunakan satu kunci API untuk banyak LLM
  - Anda ingin menjalankan model via OpenRouter di OpenClaw
title: "OpenRouter"
---

# OpenRouter

OpenRouter menyediakan **API tunggal** yang merutekan permintaan ke berbagai model di balik satu titik akhir (endpoint) dan satu kunci API. Layanan ini kompatibel dengan OpenAI, sehingga sebagian besar SDK OpenAI dapat bekerja hanya dengan mengganti alamat URL dasarnya.

## Penyiapan via CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "SK_OR_API_KEY_ANDA"
```

## Cuplikan Konfigurasi

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Catatan
- Referensi model menggunakan format `openrouter/penyedia/model`.
- OpenRouter menggunakan Bearer token dengan kunci API Anda di balik layar.
- Untuk opsi model dan penyedia lainnya, lihat [Penyedia Model](/id-ID/concepts/model-providers).



