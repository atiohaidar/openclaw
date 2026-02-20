---
summary: "Manifest plugin + persyaratan skema JSON (validasi konfigurasi yang ketat)"
read_when:
  - Anda sedang membangun plugin OpenClaw
  - Anda perlu menyertakan skema konfigurasi plugin atau menelusuri kesalahan validasi plugin
title: "Manifest Plugin"
---

# Manifest Plugin (openclaw.plugin.json)

Setiap plugin **wajib** menyertakan file `openclaw.plugin.json` di dalam **root plugin**. OpenClaw menggunakan manifest ini untuk memvalidasi konfigurasi **tanpa menjalankan kode plugin**. Manifest yang hilang atau tidak valid akan dianggap sebagai kesalahan plugin dan memblokir validasi konfigurasi.

Lihat panduan sistem plugin lengkap: [Plugin](/id-ID/tools/plugin).

## Bidang Wajib

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Kunci wajib:
- `id` (string): ID kanonik plugin.
- `configSchema` (object): Skema JSON untuk konfigurasi plugin (inline).

## Bidang Opsional

- `kind` (string): Jenis plugin (contoh: `"memory"`).
- `channels` (array): ID saluran yang didaftarkan oleh plugin ini.
- `providers` (array): ID penyedia (provider) yang didaftarkan oleh plugin ini.
- `skills` (array): Direktori skill yang akan dimuat (relatif terhadap root plugin).
- `name` (string): Nama tampilan untuk plugin.
- `description` (string): Ringkasan singkat plugin.
- `uiHints` (object): Label/placeholder bidang konfigurasi untuk perenderan UI.

## Persyaratan Skema JSON

- **Setiap plugin harus menyertakan Skema JSON**, meskipun tidak menerima konfigurasi apa pun.
- Skema kosong diperbolehkan (contoh: `{ "type": "object", "additionalProperties": false }`).
- Skema divalidasi saat konfigurasi dibaca atau ditulis, bukan saat runtime.

## Perilaku Validasi

- Kunci `channels.*` yang tidak dikenal akan dianggap sebagai **kesalahan**, kecuali ID saluran tersebut dideklarasikan oleh manifest plugin.
- `plugins.entries.<id>`, `plugins.allow`, `plugins.deny`, dan `plugins.slots.*` harus merujuk pada ID plugin yang **dapat ditemukan (discoverable)**. ID yang tidak dikenal adalah **kesalahan**.
- Jika plugin terinstal tetapi manifest atau skemanya rusak atau hilang, validasi akan gagal dan Doctor akan melaporkan kesalahan plugin tersebut.
- Jika konfigurasi plugin ada tetapi plugin tersebut **dinonaktifkan**, konfigurasi akan tetap disimpan dan **peringatan** akan muncul di Doctor + log.



