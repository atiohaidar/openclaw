---
title: "Pemangkasan Sesi (Session Pruning)"
summary: "Pemangkasan sesi: pemotongan hasil alat untuk mengurangi pembengkakan konteks"
read_when:
  - Anda ingin mengurangi pertumbuhan konteks LLM dari output alat
  - Anda sedang mengatur agents.defaults.contextPruning
---

# Pemangkasan Sesi (Session Pruning)

Pemangkasan sesi memotong **hasil alat yang lama** dari konteks dalam memori tepat sebelum setiap panggilan LLM. Ini **tidak** menulis ulang riwayat sesi yang ada di disk (`*.jsonl`).

## Kapan Ini Berjalan

- Ketika `mode: "cache-ttl"` diaktifkan dan panggilan Anthropic terakhir untuk sesi tersebut lebih lama dari `ttl`.
- Hanya memengaruhi pesan yang dikirim ke model untuk permintaan tersebut.
- Hanya aktif untuk panggilan API Anthropic (dan model Anthropic di OpenRouter).
- Untuk hasil terbaik, sesuaikan `ttl` dengan `cacheControlTtl` model Anda.
- Setelah pemangkasan, jendela TTL direset sehingga permintaan berikutnya tetap menggunakan cache sampai `ttl` kedaluwarsa kembali.

## Default Cerdas (Anthropic)

- Profil **OAuth atau setup-token**: mengaktifkan pemangkasan `cache-ttl` dan menyetel heartbeat ke `1h`.
- Profil **Kunci API**: mengaktifkan pemangkasan `cache-ttl`, menyetel heartbeat ke `30m`, dan default `cacheControlTtl` ke `1h` pada model Anthropic.
- Jika Anda menyetel salah satu nilai ini secara eksplisit, OpenClaw **tidak** akan menimpanya.

## Apa yang Diperbaiki (Biaya + Perilaku Cache)

- **Mengapa dipangkas**: Prompt caching Anthropic hanya berlaku di dalam jendela TTL. Jika sesi tidak aktif melampaui TTL, permintaan berikutnya akan menyimpan cache ulang seluruh prompt kecuali Anda memotongnya terlebih dahulu.
- **Apa yang menjadi lebih murah**: pemangkasan mengurangi ukuran **cacheWrite** untuk permintaan pertama setelah TTL kedaluwarsa.
- **Mengapa reset TTL penting**: setelah pemangkasan berjalan, jendela cache direset, sehingga permintaan tindak lanjut dapat menggunakan kembali prompt yang baru saja disimpan di cache daripada menyimpan ulang seluruh riwayat lagi.
- **Apa yang tidak dilakukan**: pemangkasan tidak menambahkan token atau biaya "ganda"; ini hanya mengubah apa yang disimpan di cache pada permintaan pertama setelah TTL terlampaui.

## Apa yang Bisa Dipangkas

- Hanya pesan `toolResult` (hasil alat).
- Pesan pengguna (user) + asisten (assistant) **tidak pernah** dimodifikasi.
- Sejumlah `keepLastAssistants` pesan asisten terakhir dilindungi; hasil alat setelah batas tersebut tidak akan dipangkas.
- Jika tidak ada cukup pesan asisten untuk menetapkan batas, pemangkasan akan dilewati.
- Hasil alat yang berisi **blok gambar** akan dilewati (tidak pernah dipotong/dikosongkan).

## Estimasi Jendela Konteks (Context Window)

Pemangkasan menggunakan estimasi jendela konteks (karakter ≈ token × 4). Jendela dasar diselesaikan dalam urutan berikut:

1. Penimpaan `models.providers.*.models[].contextWindow`.
2. Definisi model `contextWindow` (dari registri model).
3. Default `200000` token.

Jika `agents.defaults.contextTokens` disetel, nilai tersebut diperlakukan sebagai batas (minimum) pada jendela yang diselesaikan.

## Mode

### cache-ttl

- Pemangkasan hanya berjalan jika panggilan Anthropic terakhir lebih lama dari `ttl` (default `5m`).
- Saat berjalan: perilaku pemotongan lunak (soft-trim) + pengosongan keras (hard-clear) yang sama seperti sebelumnya.

## Pemangkasan Lunak vs Keras

- **Pemotongan Lunak (Soft-trim)**: hanya untuk hasil alat yang berukuran sangat besar.
    - Menyimpan bagian awal (head) + akhir (tail), menyisipkan `...`, dan menambahkan catatan dengan ukuran aslinya.
    - Melewati hasil yang memiliki blok gambar.
- **Pengosongan Keras (Hard-clear)**: mengganti seluruh hasil alat dengan `hardClear.placeholder`.

## Pemilihan Alat

- `tools.allow` / `tools.deny` mendukung wildcard `*`.
- `deny` (tolak) menang atas `allow` (izinkan).
- Pencocokan tidak peka huruf besar-kecil (case-insensitive).
- Daftar izinkan kosong berarti semua alat diizinkan.

## Interaksi dengan Batas Lainnya

- Alat bawaan sudah memotong output mereka sendiri; pemangkasan sesi adalah lapisan tambahan yang mencegah obrolan yang berjalan lama menumpuk terlalu banyak output alat dalam konteks model.
- Pemadatan (Compaction) berbeda: pemadatan meringkas dan menetap (persist), pemangkasan bersifat sementara per permintaan. Lihat [/id-ID/concepts/compaction](/id-ID/concepts/compaction).

## Default (Saat Diaktifkan)

- `ttl`: `"5m"`
- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3`
- `hardClearRatio`: `0.5`
- `minPrunableToolChars`: `50000`
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
- `hardClear`: `{ enabled: true, placeholder: "[Konten hasil alat lama telah dibersihkan]" }`

## Contoh

Default (mati):

```json5
{
  agent: {
    contextPruning: { mode: "off" },
  },
}
```

Aktifkan pemangkasan berbasis TTL:

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" },
  },
}
```

Batasi pemangkasan pada alat tertentu saja:

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] },
    },
  },
}
```

Lihat referensi konfigurasi: [Konfigurasi Gateway](/id-ID/gateway/configuration)
