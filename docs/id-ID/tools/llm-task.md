---
summary: "Tugas LLM khusus JSON untuk alur kerja (alat plugin opsional)"
read_when:
  - Anda menginginkan langkah LLM khusus JSON di dalam alur kerja
  - Anda membutuhkan output LLM yang divalidasi skema untuk otomatisasi
title: "LLM Task"
---

# LLM Task

`llm-task` adalah **alat plugin opsional** yang menjalankan tugas LLM khusus JSON dan mengembalikan output terstruktur (opsional divalidasi dengan JSON Schema).

Ini sangat ideal untuk mesin alur kerja seperti Lobster: Anda dapat menambahkan satu langkah LLM tanpa menulis kode OpenClaw khusus untuk setiap alur kerja.

## Mengaktifkan Plugin

1. Aktifkan plugin di file konfigurasi Anda:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Masukkan alat ke dalam daftar izinkan (allowlist) agen Anda (karena alat ini terdaftar sebagai `optional: true`):

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## Konfigurasi (Opsional)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.3-codex"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` adalah daftar izinkan untuk string `provider/model`. Jika diatur, permintaan apa pun di luar daftar ini akan ditolak.

## Parameter Alat

- `prompt` (string, wajib)
- `input` (apa saja, opsional)
- `schema` (objek, opsional JSON Schema)
- `provider` (string, opsional)
- `model` (string, opsional)
- `authProfileId` (string, opsional)
- `temperature` (angka, opsional)
- `maxTokens` (angka, opsional)
- `timeoutMs` (angka, opsional)

## Output

Mengembalikan `details.json` yang berisi JSON yang telah diurai (dan divalidasi terhadap `schema` jika disediakan).

## Contoh: Langkah Alur Kerja Lobster

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Berdasarkan input email, kembalikan niat (intent) dan draf.",
  "input": {
    "subject": "Halo",
    "body": "Bisakah Anda membantu saya?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## Catatan Keamanan

- Alat ini **khusus JSON** dan menginstruksikan model untuk mengeluarkan hanya JSON (tanpa blok kode ` ``` `, tanpa komentar).
- Tidak ada alat yang diekspos ke model selama eksekusi ini.
- Perlakukan output sebagai data yang tidak dipercaya kecuali jika Anda memvalidasinya dengan `schema`.
- Tempatkan langkah persetujuan sebelum langkah apa pun yang memiliki efek samping (kirim pesan, posting, eksekusi perintah).


