---
summary: "Penanganan zona waktu untuk agen, amplop pesan, dan prompt"
read_when:
  - Anda perlu memahami bagaimana stempel waktu dinormalisasi untuk model
  - Mengonfigurasi zona waktu pengguna untuk prompt sistem
title: "Zona Waktu"
---

# Zona Waktu

OpenClaw menstandarisasi stempel waktu (timestamps) sehingga model melihat **satu waktu referensi tunggal**.

## Amplop Pesan (Local secara default)

Pesan masuk dibungkus dalam amplop seperti berikut:

```
[Provider ... 2026-01-05 16:26 PST] isi pesan
```

Stempel waktu dalam amplop secara default menggunakan **waktu lokal host**, dengan presisi menit.

Anda dapat menimpanya dengan:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | Zona Waktu IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

-   `envelopeTimezone: "utc"` menggunakan UTC.
-   `envelopeTimezone: "user"` menggunakan `agents.defaults.userTimezone` (fallback ke zona waktu host).
-   Gunakan zona waktu IANA yang eksplisit (misalnya, `"Asia/Jakarta"`) untuk offset tetap.
-   `envelopeTimestamp: "off"` menghapus stempel waktu absolut dari header amplop.
-   `envelopeElapsed: "off"` menghapus akhiran waktu yang telah berlalu (gaya `+2m`).

### Contoh

**Lokal (default):**

```
[Signal Andi +62812 2026-01-18 00:19 WIB] halo
```

**Zona waktu tetap:**

```
[Signal Andi +62812 2026-01-18 06:19 GMT+7] halo
```

**Waktu berlalu:**

```
[Signal Andi +62812 +2m 2026-01-18T05:19Z] tindak lanjut
```

## Payload Alat (Data mentah penyedia + bidang yang dinormalisasi)

Pemanggilan alat (`channels.discord.readMessages`, `channels.slack.readMessages`, dll.) mengembalikan **stempel waktu mentah dari penyedia**. Kami juga melampirkan bidang yang dinormalisasi untuk konsistensi:

-   `timestampMs` (mili detik epoch UTC).
-   `timestampUtc` (string ISO 8601 UTC).

Bidang mentah dari penyedia tetap dipertahankan.

## Zona waktu pengguna untuk prompt sistem

Setel `agents.defaults.userTimezone` untuk memberitahu model tentang zona waktu lokal pengguna. Jika tidak disetel, OpenClaw akan menentukan **zona waktu host saat dijalankan** (tanpa menulis ke konfigurasi).

```json5
{
  agents: { defaults: { userTimezone: "Asia/Jakarta" } },
}
```

Prompt sistem akan menyertakan:

-   Bagian `Current Date & Time` (Tanggal & Waktu Saat Ini) dengan waktu lokal dan zona waktu.
-   `Time format`: format 12 jam atau 24 jam.

Anda dapat mengontrol format waktu dengan `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Lihat [Tanggal & Waktu](/id-ID/date-time) (bahasa Inggris) untuk perilaku lengkap dan contoh lainnya.
