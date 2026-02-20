---
summary: "Ringkasan penyedia model dengan contoh konfigurasi + alur CLI"
read_when:
  - Anda memerlukan referensi pengaturan model per penyedia
  - Anda menginginkan contoh konfigurasi atau perintah onboarding CLI untuk penyedia model
title: "Penyedia Model"
---

# Penyedia model

Halaman ini membahas tentang **penyedia model/LLM** (bukan saluran obrolan seperti WhatsApp/Telegram). Untuk aturan pemilihan model, lihat [/id-ID/concepts/models](/id-ID/concepts/models).

## Aturan Cepat

- Referensi model menggunakan format `penyedia/model` (contoh: `opencode/claude-opus-4-6`).
- Jika Anda menyetel `agents.defaults.models`, itu akan menjadi daftar izinkan (allowlist).
- Pembantu CLI: `openclaw onboard`, `openclaw models list`, `openclaw models set <penyedia/model>`.

## Rotasi Kunci API

- Mendukung rotasi penyedia generik untuk penyedia yang dipilih.
- Konfigurasikan beberapa kunci melalui:
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (satu-satunya penimpa langsung, prioritas tertinggi).
  - `<PROVIDER>_API_KEYS` (daftar dengan pemisah koma atau titik koma).
  - `<PROVIDER>_API_KEY` (kunci utama).
  - `<PROVIDER>_API_KEY_*` (daftar bernomor, misal: `<PROVIDER>_API_KEY_1`).
- Untuk penyedia Google, `GOOGLE_API_KEY` juga disertakan sebagai cadangan (fallback).
- Urutan pemilihan kunci menjaga prioritas dan menghapus nilai duplikat.
- Permintaan akan dicoba ulang dengan kunci berikutnya hanya pada respons batas tingkat (rate-limit) (misalnya `429`, `rate_limit`, `quota`, `resource exhausted`).
- Kegagalan yang bukan karena batas tingkat akan langsung gagal; tidak ada rotasi kunci yang dicoba.
- Ketika semua kunci kandidat gagal, kesalahan terakhir akan dikembalikan dari upaya terakhir.

## Penyedia Bawaan (Katalog pi-ai)

OpenClaw menyertakan katalog pi-ai. Penyedia ini **tidak** memerlukan konfigurasi `models.providers`; cukup atur autentikasi dan pilih model.

### OpenAI

- Penyedia: `openai`
- Autentikasi: `OPENAI_API_KEY`
- Rotasi opsional: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, ditambah `OPENCLAW_LIVE_OPENAI_KEY` (penimpa satu-satunya).
- Contoh model: `openai/gpt-5.1-codex`
- CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

- Penyedia: `anthropic`
- Autentikasi: `ANTHROPIC_API_KEY` atau `claude setup-token`
- Rotasi opsional: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`, ditambah `OPENCLAW_LIVE_ANTHROPIC_KEY` (penimpa satu-satunya).
- Contoh model: `anthropic/claude-opus-4-6`
- CLI: `openclaw onboard --auth-choice token` (tempel setup-token) atau `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

- Penyedia: `openai-codex`
- Autentikasi: OAuth (ChatGPT)
- Contoh model: `openai-codex/gpt-5.3-codex`
- CLI: `openclaw onboard --auth-choice openai-codex` atau `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

- Penyedia: `opencode`
- Autentikasi: `OPENCODE_API_KEY` (atau `OPENCODE_ZEN_API_KEY`)
- Contoh model: `opencode/claude-opus-4-6`
- CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (Kunci API)

- Penyedia: `google`
- Autentikasi: `GEMINI_API_KEY`
- Rotasi opsional: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, fallback `GOOGLE_API_KEY`, dan `OPENCLAW_LIVE_GEMINI_KEY` (penimpa satu-satunya).
- Contoh model: `google/gemini-3-pro-preview`
- CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex, Antigravity, dan Gemini CLI

- Penyedia: `google-vertex`, `google-antigravity`, `google-gemini-cli`
- Autentikasi: Vertex menggunakan gcloud ADC; Antigravity/Gemini CLI menggunakan alur autentikasi masing-masing.
- OAuth Antigravity disertakan sebagai plugin bawaan (`google-antigravity-auth`, dinonaktifkan secara default).
  - Aktifkan: `openclaw plugins enable google-antigravity-auth`
  - Login: `openclaw models auth login --provider google-antigravity --set-default`
- OAuth Gemini CLI disertakan sebagai plugin bawaan (`google-gemini-cli-auth`, dinonaktifkan secara default).
  - Aktifkan: `openclaw plugins enable google-gemini-cli-auth`
  - Login: `openclaw models auth login --provider google-gemini-cli --set-default`
  - Catatan: Anda **tidak** menempelkan ID klien atau rahasia ke `openclaw.json`. Alur login CLI menyimpan token dalam profil autentikasi di host gateway.

### Z.AI (GLM)

- Penyedia: `zai`
- Autentikasi: `ZAI_API_KEY`
- Contoh model: `zai/glm-4.7`
- CLI: `openclaw onboard --auth-choice zai-api-key`
  - Alias: `z.ai/*` dan `z-ai/*` dinormalisasi menjadi `zai/*`.

### Vercel AI Gateway

- Penyedia: `vercel-ai-gateway`
- Autentikasi: `AI_GATEWAY_API_KEY`
- Contoh model: `vercel-ai-gateway/anthropic/claude-opus-4.6`
- CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Penyedia bawaan lainnya

- OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
- Contoh model: `openrouter/anthropic/claude-sonnet-4-5`
- xAI: `xai` (`XAI_API_KEY`)
- Groq: `groq` (`GROQ_API_KEY`)
- Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  - Model GLM di Cerebras menggunakan ID `zai-glm-4.7` dan `zai-glm-4.6`.
  - Base URL yang kompatibel dengan OpenAI: `https://api.cerebras.ai/v1`.
- Mistral: `mistral` (`MISTRAL_API_KEY`)
- GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
- Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` atau `HF_TOKEN`) — router yang kompatibel dengan OpenAI; contoh model: `huggingface/deepseek-ai/DeepSeek-R1`; CLI: `openclaw onboard --auth-choice huggingface-api-key`. Lihat [Hugging Face (Inference)](/id-ID/providers/huggingface).

## Penyedia melalui `models.providers` (Kustom/Base URL)

Gunakan `models.providers` (atau `models.json`) untuk menambahkan penyedia **kustom** atau proxy yang kompatibel dengan OpenAI/Anthropic.

### Moonshot AI (Kimi)

Moonshot menggunakan endpoint yang kompatibel dengan OpenAI, jadi konfigurasikan sebagai penyedia kustom:

- Penyedia: `moonshot`
- Autentikasi: `MOONSHOT_API_KEY`
- Contoh model: `moonshot/kimi-k2.5`

ID model Kimi K2:

{/_moonshot-kimi-k2-model-refs:start_/ && null}

- `moonshot/kimi-k2.5`
- `moonshot/kimi-k2-0905-preview`
- `moonshot/kimi-k2-turbo-preview`
- `moonshot/kimi-k2-thinking`
- `moonshot/kimi-k2-thinking-turbo`
  {/_moonshot-kimi-k2-model-refs:end_/ && null}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      },
    },
  },
}
```

### Kimi Coding

Kimi Coding menggunakan endpoint yang kompatibel dengan Anthropic dari Moonshot AI:

- Penyedia: `kimi-coding`
- Autentikasi: `KIMI_API_KEY`
- Contoh model: `kimi-coding/k2p5`

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (Tingkat Gratis)

Qwen menyediakan akses OAuth ke Qwen Coder + Vision melalui alur kode perangkat. Aktifkan plugin bawaan, lalu login:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Referensi model:

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

Lihat [/id-ID/providers/qwen](/id-ID/providers/qwen) untuk detail penyiapan dan catatan.

### Synthetic

Synthetic menyediakan model yang kompatibel dengan Anthropic di balik penyedia `synthetic`:

- Penyedia: `synthetic`
- Autentikasi: `SYNTHETIC_API_KEY`
- Contoh model: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
- CLI: `openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }],
      },
    },
  },
}
```

### MiniMax

MiniMax dikonfigurasi melalui `models.providers` karena menggunakan endpoint kustom:

- MiniMax (kompatibel dengan Anthropic): `--auth-choice minimax-api`
- Autentikasi: `MINIMAX_API_KEY`

Lihat [/id-ID/providers/minimax](/id-ID/providers/minimax) untuk detail penyiapan, opsi model, dan potongan kode konfigurasi.

### Ollama

Ollama adalah runtime LLM lokal yang menyediakan API yang kompatibel dengan OpenAI:

- Penyedia: `ollama`
- Autentikasi: Tidak diperlukan (server lokal)
- Contoh model: `ollama/llama3.3`
- Instalasi: [https://ollama.ai](https://ollama.ai)

```bash
# Instal Ollama, lalu ambil model:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollama terdeteksi secara otomatis saat berjalan secara lokal di `http://127.0.0.1:11434/v1`. Lihat [/id-ID/providers/ollama](/id-ID/providers/ollama) untuk rekomendasi model dan konfigurasi kustom.

### vLLM

vLLM adalah server yang kompatibel dengan OpenAI yang berjalan lokal (atau di-hos sendiri):

- Penyedia: `vllm`
- Autentikasi: Opsional (tergantung server Anda)
- Base URL default: `http://127.0.0.1:8000/v1`

Untuk mengaktifkan penemuan otomatis secara lokal (nilai apa pun berfungsi jika server Anda tidak mewajibkan autentikasi):

```bash
export VLLM_API_KEY="vllm-local"
```

Kemudian setel model (ganti dengan salah satu ID yang dikembalikan oleh `/v1/models`):

```json5
{
  agents: {
    defaults: { model: { primary: "vllm/id-model-anda" } },
  },
}
```

Lihat [/id-ID/providers/vllm](/id-ID/providers/vllm) untuk detail lebih lanjut.

### Proxy Lokal (LM Studio, vLLM, LiteLLM, dll.)

Contoh (kompatibel dengan OpenAI):

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Catatan:

- Untuk penyedia kustom, `reasoning`, `input`, `cost`, `contextWindow`, dan `maxTokens` bersifat opsional. Jika tidak diisi, OpenClaw akan menggunakan nilai default:
  - `reasoning: false`
  - `input: ["text"]`
  - `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  - `contextWindow: 200000`
  - `maxTokens: 8192`
- Direkomendasikan: setel nilai eksplisit yang sesuai dengan batas proxy/model Anda.

## Contoh CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

Lihat juga: [/id-ID/gateway/configuration](/id-ID/gateway/configuration) untuk contoh konfigurasi lengkap.
