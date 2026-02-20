---
summary: "Plugin/ekstensi OpenClaw: penemuan, konfigurasi, dan keamanan"
read_when:
  - Menambah atau memodifikasi plugin/ekstensi
  - Mendokumentasikan aturan instalasi atau pemuatan plugin
title: "Plugin"
---

# Plugin (Ekstensi)

## Memulai dengan Cepat

Plugin adalah **modul kode kecil** yang memperluas kemampuan OpenClaw dengan fitur tambahan (perintah, alat, dan RPC Gateway).

Langkah cepat:

1. Lihat plugin yang sudah terload:
   ```bash
   openclaw plugins list
   ```

2. Instal plugin resmi (contoh: Voice Call):
   ```bash
   openclaw plugins install @openclaw/voice-call
   ```

3. Mulai ulang Gateway, lalu konfigurasi di bawah `plugins.entries.<id>.config`.

Lihat [Voice Call](/id-ID/plugins/voice-call) untuk contoh plugin konkret.
Mencari daftar pihak ketiga? Lihat [Plugin komunitas](/id-ID/plugins/community).

## Plugin Resmi yang Tersedia

- **Microsoft Teams**: Instal `@openclaw/msteams` jika Anda memerlukan integrasi Teams.
- **Memory (Core)**: Plugin pencarian memori bawaan (aktif secara default).
- **[Voice Call](/id-ID/plugins/voice-call)**: `@openclaw/voice-call`.
- **[Zalo Personal](/id-ID/plugins/zalouser)**: `@openclaw/zalouser`.

Plugin OpenClaw adalah **modul TypeScript** yang dimuat saat runtime. Validasi konfigurasi tidak menjalankan kode plugin; ia menggunakan manifes plugin dan JSON Schema.

Plugin dapat mendaftarkan:
- Metode RPC Gateway
- Handler HTTP Gateway
- Alat Agen
- Perintah CLI
- Layanan latar belakang
- Skill (dengan mencantumkan direktori `skills` di manifes plugin)
- **Perintah balas otomatis** (berjalan tanpa memanggil agen AI)

## Penemuan & Prioritas

OpenClaw memindai lokasi berikut sesuai urutan:
1. Jalur konfigurasi (`plugins.load.paths`)
2. Ekstensi workspace (`.openclaw/extensions/`)
3. Ekstensi global (`~/.openclaw/extensions/`)
4. Ekstensi bawaan (`<openclaw>/extensions/`, **nonaktif secara default**)

Plugin bawaan harus diaktifkan secara eksplisit melalui `plugins.entries.<id>.enabled` atau perintah `openclaw plugins enable <id>`.

## ID Plugin

ID default ditentukan dari:
- Paket: Nama di `package.json`.
- File mandiri: Nama dasar file (`voice-call.ts` → `voice-call`).

## Konfigurasi

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Bidang Utama:
- `enabled`: sakelar utama (default: true).
- `allow`: daftar izinkan (opsional).
- `load.paths`: jalur tambahan untuk file/direktori plugin.

Perubahan konfigurasi **memerlukan restart gateway**.

## Slot Plugin (Kategori Eksklusif)

Beberapa kategori plugin bersifat **eksklusif** (hanya satu yang boleh aktif). Gunakan `plugins.slots` untuk memilih:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core",
    },
  },
}
```

## CLI

- `openclaw plugins list`: daftar plugin.
- `openclaw plugins install <path/package>`: instal plugin.
- `openclaw plugins update <id>`: perbarui plugin.
- `openclaw plugins enable <id>`: aktifkan plugin.
- `openclaw plugins doctor`: periksa kesehatan plugin.

## Perintah Balas Otomatis (Auto-reply)

Plugin dapat mendaftarkan perintah slash kustom yang dijalankan **tanpa memanggil agen AI**. Berguna untuk perintah sakelar, pemeriksaan status, atau tindakan cepat.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Tampilkan status plugin",
    handler: (ctx) => ({
      text: `Plugin berjalan! Saluran: ${ctx.channel}`,
    }),
  });
}
```

## Menulis Saluran Pesan Baru

Gunakan ini jika Anda ingin membuat antarmuka chat baru, bukan penyedia model.

Langkah-langkah:
1. Tentukan ID dan bentuk konfigurasi.
2. Definisikan metadata saluran (label, blurb, dll).
3. Implementasikan adaptor yang diperlukan (outbound, capabilities, dll).
4. Daftarkan saluran di plugin Anda menggunakan `api.registerChannel({ plugin })`.

## Catatan Keamanan

Plugin berjalan di dalam proses yang sama dengan Gateway. Perlakukan mereka sebagai kode tepercaya:
- Hanya instal plugin dari sumber yang Anda percayai.
- Gunakan daftar izinkan (`plugins.allow`) untuk membatasi pemuatan.
- Mulai ulang Gateway setelah melakukan perubahan.


