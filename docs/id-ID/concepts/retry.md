---
summary: "Kebijakan percobaan ulang (retry) untuk panggilan penyedia keluar"
read_when:
  - Memperbarui perilaku atau default retry penyedia
  - Mendebug kesalahan pengiriman penyedia atau batas tingkat (rate limits)
title: "Kebijakan Retry"
---

# Kebijakan Retry

## Tujuan

- Mencoba ulang per permintaan HTTP, bukan per alur multi-langkah.
- Menjaga urutan dengan hanya mencoba ulang langkah saat ini.
- Menghindari duplikasi operasi yang tidak idempoten.

## Default

- Upaya (Attempts): 3
- Batas jeda maks: 30000 ms
- Jitter: 0.1 (10 persen)
- Default penyedia:
  - Jeda min Telegram: 400 ms
  - Jeda min Discord: 500 ms

## Perilaku

### Discord

- Hanya mencoba ulang pada kesalahan batas tingkat (HTTP 429).
- Menggunakan `retry_after` dari Discord jika tersedia, jika tidak menggunakan backoff eksponensial.

### Telegram

- Mencoba ulang pada kesalahan transien (429, timeout, connect/reset/closed, temporarily unavailable).
- Menggunakan `retry_after` jika tersedia, jika tidak menggunakan backoff eksponensial.
- Kesalahan parsing Markdown tidak dicoba ulang; akan dialihkan ke teks biasa.

## Konfigurasi

Atur kebijakan retry per penyedia di `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## Catatan

- Retry berlaku per permintaan (pengiriman pesan, unggahan media, reaksi, jajak pendapat, stiker).
- Alur komposit tidak mencoba ulang langkah yang sudah selesai.
