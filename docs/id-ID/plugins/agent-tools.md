---
summary: "Menulis alat agen di dalam plugin (skema, alat opsional, daftar izinkan)"
read_when:
  - Anda ingin menambahkan alat agen baru di dalam sebuah plugin
  - Anda perlu membuat alat menjadi opsional melalui daftar izinkan
title: "Alat Agen Plugin"
---

# Alat Agen Plugin (Plugin Agent Tools)

Plugin OpenClaw dapat mendaftarkan **alat agen** (fungsi JSON-schema) yang diekspos ke LLM saat agen berjalan. Alat dapat bersifat **wajib** (selalu tersedia) atau **opsional** (harus dipilih secara eksplisit).

Alat agen dikonfigurasi di bawah `tools` pada konfigurasi utama, atau per agen di bawah `agents.list[].tools`. Kebijakan daftar izinkan (allowlist) dan daftar cekal (denylist) mengontrol alat mana yang dapat dipanggil oleh agen.

## Alat Dasar

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "alat_saya",
    description: "Melakukan sesuatu",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## Alat Opsional (Opt-in)

Alat opsional **tidak pernah** diaktifkan secara otomatis. Pengguna harus menambahkannya ke daftar izinkan agen secara manual.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "alat_alur_kerja",
      description: "Menjalankan alur kerja lokal",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Aktifkan alat opsional di `agents.list[].tools.allow` (atau `tools.allow` global):

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "alat_alur_kerja", // nama alat spesifik
            "nama_plugin", // ID plugin (mengaktifkan semua alat dari plugin tersebut)
            "group:plugins", // semua alat plugin
          ],
        },
      },
    ],
  },
}
```

## Aturan + Tips

- Nama alat **tidak boleh** bentrok dengan nama alat inti (core); alat yang bentrok akan dilewati.
- Utamakan penggunaan `optional: true` untuk alat yang memicu efek samping (side effects) atau memerlukan biner/kredensial tambahan.



