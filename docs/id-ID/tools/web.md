---
summary: "Alat pencarian web + pengambilan konten (Brave Search API, Perplexity langsung/OpenRouter)"
read_when:
  - Anda ingin mengaktifkan web_search atau web_fetch
  - Anda perlu menyiapkan kunci API Brave Search
  - Anda ingin menggunakan Perplexity Sonar untuk pencarian web
title: "Alat Web (Web Tools)"
---

# Alat Web (Web Tools)

OpenClaw menyediakan dua alat web ringan:

- `web_search` — Cari di web melalui Brave Search API (default) atau Perplexity Sonar (langsung atau melalui OpenRouter).
- `web_fetch` — Ambil konten via HTTP + ekstraksi teks yang dapat dibaca (HTML → markdown/teks).

Alat-alat ini **bukan** otomasi browser. Untuk situs yang berat JavaScript atau memerlukan login, gunakan [Alat Browser](/id-ID/tools/browser).

## Cara Kerja

- `web_search` memanggil penyedia yang dikonfigurasi dan mengembalikan hasil.
  - **Brave** (default): mengembalikan hasil terstruktur (judul, URL, cuplikan).
  - **Perplexity**: mengembalikan jawaban sintetis AI dengan sitasi dari pencarian web real-time.
- Hasil pencarian disimpan dalam cache selama 15 menit.
- `web_fetch` melakukan HTTP GET biasa dan mengekstrak konten yang dapat dibaca. Alat ini **tidak** mengeksekusi JavaScript.
- `web_fetch` diaktifkan secara default.

## Memilih Penyedia Pencarian

| Penyedia            | Kelebihan                                  | Kekurangan                                  | Kunci API                                      |
| ------------------- | ------------------------------------------ | ------------------------------------------- | ---------------------------------------------- |
| **Brave** (default) | Cepat, hasil terstruktur, ada paket gratis | Hasil pencarian tradisional                 | `BRAVE_API_KEY`                                |
| **Perplexity**      | Jawaban sintetis AI, sitasi, real-time     | Memerlukan akses Perplexity atau OpenRouter | `OPENROUTER_API_KEY` atau `PERPLEXITY_API_KEY` |

Atur penyedia di konfigurasi:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave", // atau "perplexity"
      },
    },
  },
}
```

## Mendapatkan Kunci API Brave

1. Buat akun Brave Search API di [https://brave.com/search/api/](https://brave.com/search/api/).
2. Di dashboard, pilih paket **Data for Search** (bukan “Data for AI”).
3. Jalankan `openclaw configure --section web` untuk menyimpan kunci dalam konfigurasi, atau set `BRAVE_API_KEY` di lingkungan Anda.

## Menggunakan Perplexity (Langsung atau via OpenRouter)

Model Perplexity Sonar memiliki kemampuan pencarian web bawaan dan mengembalikan jawaban beserta sitasi. Anda dapat menggunakannya melalui OpenRouter (mendukung pembayaran kripto/prepaid).

### Menyiapkan Pencarian Perplexity

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "sk-or-v1-...",
          baseUrl: "https://openrouter.ai/api/v1",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## web_search

Cari di web menggunakan penyedia yang telah dikonfigurasi.

### Parameter Alat

- `query` (wajib)
- `count` (1–10; default dari konfigurasi)
- `country` (opsional): kode negara 2 huruf (misal: "ID", "US").
- `search_lang` (opsional): kode bahasa ISO untuk hasil pencarian.
- `freshness` (opsional): filter berdasarkan waktu penemuan (misal: `pd` untuk past day, `pw` untuk past week).

## web_fetch

Ambil URL dan ekstrak konten yang dapat dibaca.

### Konfigurasi web_fetch

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE"
        },
      },
    },
  },
}
```

### Parameter alat web_fetch

- `url` (wajib, hanya http/https)
- `extractMode` (`markdown` | `text`)
- `maxChars` (batasi panjang teks yang diambil)

Catatan:

- `web_fetch` menggunakan Readability terlebih dahulu, kemudian Firecrawl (jika dikonfigurasi) jika yang pertama gagal.
- Firecrawl berguna untuk melewati perlindungan anti-bot pada beberapa situs.
- `web_fetch` memblokir hostname pribadi/internal untuk keamanan.
- Hasil pengambilan disimpan dalam cache selama 15 menit.


