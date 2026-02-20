---
summary: "Cadangan Firecrawl untuk web_fetch (anti-bot + ekstraksi cache)"
read_when:
  - Anda ingin ekstraksi web yang didukung Firecrawl
  - Anda memerlukan kunci API Firecrawl
  - Anda ingin ekstraksi anti-bot untuk web_fetch
title: "Firecrawl"
---

# Firecrawl

OpenClaw dapat menggunakan **Firecrawl** sebagai ekstraktor cadangan untuk `web_fetch`. Firecrawl adalah layanan ekstraksi konten ter-host yang mendukung penghindaran bot (bot circumvention) dan caching, yang membantu saat mengakses situs berat JavaScript atau halaman yang memblokir pengambilan HTTP biasa.

## Mendapatkan Kunci API

1. Buat akun Firecrawl dan hasilkan kunci API.
2. Simpan di konfigurasi atau atur `FIRECRAWL_API_KEY` di lingkungan gateway.

## Konfigurasi Firecrawl

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "KUNCI_API_FIRECRAWL_DI_SINI",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000, // 2 hari
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Catatan:
- `firecrawl.enabled` secara default bernilai true jika kunci API ditemukan.
- `maxAgeMs` mengontrol seberapa lama hasil cache dianggap valid (dalam ms). Defaultnya adalah 2 hari.

## Mode Stealth / Penghindaran Bot

Firecrawl menyediakan parameter **proxy mode** untuk penghindaran bot (`basic`, `stealth`, atau `auto`). OpenClaw selalu menggunakan `proxy: "auto"` ditambah `storeInCache: true` untuk permintaan Firecrawl.

Mode `auto` akan mencoba kembali dengan proxy stealth jika percobaan dasar gagal, yang mungkin menggunakan lebih banyak pulsa daripada scraping dasar saja.

## Cara `web_fetch` Menggunakan Firecrawl

Urutan ekstraksi `web_fetch`:
1. Readability (lokal)
2. Firecrawl (jika dikonfigurasi)
3. Pembersihan HTML dasar (cadangan terakhir)

Lihat [Alat Web](/id-ID/tools/web) untuk pengaturan alat web lengkap.


