---
summary: "Jalankan OpenClaw dengan Ollama (runtime LLM lokal)"
read_when:
  - Anda ingin menjalankan OpenClaw dengan model lokal via Ollama
  - Anda butuh panduan penyiapan dan konfigurasi Ollama
title: "Ollama"
---

# Ollama

Ollama adalah runtime LLM lokal yang memudahkan Anda menjalankan model sumber terbuka (open-source) di mesin sendiri. OpenClaw terintegrasi dengan API asli Ollama, mendukung streaming, pemanggilan alat (tool calling), dan dapat **mendeteksi model yang mendukung alat secara otomatis**.

## Penyiapan Cepat

1. Instal Ollama: [https://ollama.com](https://ollama.com)

2. Unduh model (pull):
```bash
ollama pull llama3.3
# atau
ollama pull qwen2.5-coder:32b
```

3. Aktifkan Ollama untuk OpenClaw:
```bash
# Atur variabel lingkungan (environment variable)
export OLLAMA_API_KEY="ollama-lokal"

# Atau via konfigurasi CLI
openclaw config set models.providers.ollama.apiKey "ollama-lokal"
```

4. Gunakan model Ollama di konfigurasi:
```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" },
    },
  },
}
```

## Deteksi Model Otomatis

Jika Anda mengatur `OLLAMA_API_KEY` dan **tidak** mendefinisikan daftar model secara manual, OpenClaw akan mendeteksi model dari Ollama lokal Anda (biasanya di `http://127.0.0.1:11434`):
- Hanya model yang melaporkan kemampuan `tools` yang akan ditampilkan.
- Semua biaya (input/output) otomatis diatur ke `0`.
- Jendela konteks (context window) dideteksi secara otomatis dari informasi model.

## Keunggulan Ollama di OpenClaw

- **Privasi Penuh**: Data Anda tidak pernah meninggalkan mesin lokal jika menggunakan model Ollama.
- **Tanpa Biaya**: Tidak perlu membayar per token karena model berjalan di perangkat keras Anda sendiri.
- **Streaming Cepat**: Mendukung pengiriman teks secara bertahap (streaming) saat respon sedang dibuat.

## Pemecahan Masalah

- **Ollama Tidak Terdeteksi**: Pastikan layanan Ollama sedang berjalan (`ollama serve`) dan port `11434` dapat diakses.
- **Model Tidak Muncul di OpenClaw**: OpenClaw hanya mendeteksi model yang mendukung pemanggilan alat (tool calling). Jika model Anda tidak muncul, pastikan Anda menggunakan versi model terbaru yang mendukung fitur tersebut.

## Terkait
- [Penyedia Model](/id-ID/concepts/model-providers)
- [Konfigurasi Gateway](/id-ID/gateway/configuration)



