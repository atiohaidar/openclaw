---
summary: "Perilaku streaming + pemecahan pesan (balasan blok, streaming pratinjau Telegram, batasan)"
read_when:
  - Menjelaskan cara kerja streaming atau pemecahan pesan (chunking) pada saluran
  - Mengubah perilaku streaming blok atau pemecahan saluran
  - Men-debug balasan blok duplikat/awal atau streaming pratinjau Telegram
title: "Streaming dan Pemecahan Pesan"
---

# Streaming + Pemecahan Pesan (Chunking)

OpenClaw memiliki dua lapisan "streaming" yang terpisah:

-   **Streaming Blok (Saluran)**: mengirimkan **blok** yang sudah selesai saat asisten sedang menulis. Ini adalah pesan saluran normal (bukan delta token).
-   **Streaming ala Token (Hanya Telegram)**: memperbarui **pesan pratinjau** sementara dengan teks parsial saat sedang dibuat.

Saat ini **tidak ada streaming delta token yang sebenarnya** ke pesan saluran. Streaming pratinjau Telegram adalah satu-satunya permukaan streaming parsial.

## Streaming Blok (Pesan Saluran)

Streaming blok mengirimkan output asisten dalam potongan kasar saat sudah tersedia.

```
Output model
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker mengeluarkan blok saat buffer membesar
       └─ (blockStreamingBreak=message_end)
            └─ chunker menguras (flush) pada message_end
                   └─ kirim ke saluran (balasan blok)
```

Legenda:

-   `text_delta/events`: event stream model (mungkin jarang untuk model non-streaming).
-   `chunker`: `EmbeddedBlockChunker` yang menerapkan batas min/max + preferensi pemisah.
-   `kirim ke saluran`: pesan keluar yang sebenarnya (balasan blok).

**Kontrol:**

-   `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (defaultnya off).
-   Penimpa Saluran: `*.blockStreaming` (dan varian per akun) untuk memaksa `"on"`/`"off"` per saluran.
-   `agents.defaults.blockStreamingBreak`: `"text_end"` atau `"message_end"`.
-   `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
-   `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (menggabungkan blok yang distreaming sebelum dikirim).
-   Batas keras saluran: `*.textChunkLimit` (misalnya, `channels.whatsapp.textChunkLimit`).
-   Mode potongan saluran: `*.chunkMode` (default `length`, `newline` memecah pada garis kosong/batas paragraf sebelum pemecahan berdasarkan panjang).
-   Batas lunak Discord: `channels.discord.maxLinesPerMessage` (default 17) memecah balasan yang terlalu panjang untuk menghindari pemotongan UI.

**Semantik Batas:**

-   `text_end`: streaming blok segera setelah chunker mengeluarkannya; kuras pada setiap `text_end`.
-   `message_end`: tunggu sampai pesan asisten selesai, lalu kuras output yang ada di buffer.

`message_end` tetap menggunakan chunker jika teks di buffer melebihi `maxChars`, sehingga bisa mengeluarkan beberapa potongan di akhir.

## Algoritme Pemecahan (Batas Bawah/Atas)

Pemecahan blok diimplementasikan oleh `EmbeddedBlockChunker`:

-   **Batas Bawah**: jangan keluarkan sampai buffer >= `minChars` (kecuali diperintahkan).
-   **Batas Atas**: lebih disukai diputus sebelum `maxChars`; jika terpaksa, putus di `maxChars`.
-   **Preferensi Pemisah**: `paragraph` → `newline` → `sentence` → `whitespace` → pemutusan keras.
-   **Blok Kode (Code Fences)**: jangan pernah memutus di dalam blok kode; jika terpaksa di `maxChars`, tutup dan buka kembali blok kode agar Markdown tetap valid.

`maxChars` dibatasi oleh `textChunkLimit` saluran, sehingga Anda tidak akan melebihi kapasitas per saluran.

## Koalesensi (Penggabungan Blok yang Distreaming)

Saat streaming blok diaktifkan, OpenClaw dapat **menggabungkan potongan blok yang berurutan** sebelum mengirimkannya. Ini mengurangi "spam satu baris" sambil tetap memberikan output progresif.

-   Koalesensi menunggu **celah waktu diam** (`idleMs`) sebelum menguras.
-   Buffer dibatasi oleh `maxChars` dan akan dikuras jika melebihi batas tersebut.
-   `minChars` mencegah pengiriman fragmen kecil sampai teks terkumpul cukup banyak (pengurasan akhir akan selalu mengirimkan teks yang tersisa).
-   Penyambung diambil dari `blockStreamingChunk.breakPreference` (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → spasi).
-   Penimpa saluran tersedia via `*.blockStreamingCoalesce` (termasuk konfigurasi per akun).
-   Default `minChars` untuk koalesensi dinaikkan ke 1500 untuk Signal/Slack/Discord kecuali jika diatur lain.

## Kecepatan Berbalas ala Manusia Antar Blok

Saat streaming blok diaktifkan, Anda dapat menambahkan **jeda acak** di antara balasan blok (setelah blok pertama). Ini membuat respons yang terdiri dari banyak balon pesan terasa lebih alami.

-   Konfigurasi: `agents.defaults.humanDelay` (timpa per agen via `agents.list[].humanDelay`).
-   Mode: `off` (default), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
-   Hanya berlaku untuk **balasan blok**, bukan balasan akhir atau ringkasan alat.

## "Streaming Potongan atau Semuanya"

Hal ini dipetakan ke:

-   **Streaming Potongan**: `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (mengeluarkan sambil jalan). Saluran non-Telegram juga memerlukan `*.blockStreaming: true`.
-   **Streaming Semuanya di Akhir**: `blockStreamingBreak: "message_end"` (kuras sekali, kemungkinan menjadi beberapa potongan jika sangat panjang).
-   **Tanpa Streaming Blok**: `blockStreamingDefault: "off"` (hanya balasan akhir).

**Catatan Saluran**: Untuk saluran non-Telegram, streaming blok dalam keadaan **mati kecuali** `*.blockStreaming` secara eksplisit disetel ke `true`. Telegram dapat melakukan streaming pratinjau langsung (`channels.telegram.streamMode`) tanpa balasan blok.

Pengingat lokasi konfigurasi: default `blockStreaming*` berada di bawah `agents.defaults`, bukan di akar konfigurasi.

## Streaming Pratinjau Telegram (Ala Token)

Telegram adalah satu-satunya saluran dengan streaming pratinjau langsung:

-   Menggunakan Bot API `sendMessage` (pembaruan pertama) + `editMessageText` (pembaruan berikutnya).
-   `channels.telegram.streamMode: "partial" | "block" | "off"`.
    -   `partial`: pratinjau diperbarui dengan teks stream terbaru.
    -   `block`: pratinjau diperbarui dalam blok potongan (aturan chunker yang sama).
    -   `off`: tanpa streaming pratinjau.
-   Konfigurasi potongan pratinjau (hanya untuk `streamMode: "block"`): `channels.telegram.draftChunk` (defaultnya: `minChars: 200`, `maxChars: 800`).
-   Streaming pratinjau terpisah dari streaming blok.
-   Saat streaming blok Telegram diaktifkan secara eksplisit, streaming pratinjau akan dilewati untuk menghindari double-streaming.
-   Balasan akhir berupa teks saja diterapkan dengan mengedit pesan pratinjau di tempatnya.
-   Balasan akhir non-teks/kompleks akan kembali ke pengiriman pesan akhir normal.
-   `/reasoning stream` menulis penalaran (thinking) ke dalam pratinjau langsung (hanya Telegram).

```
Telegram
  └─ sendMessage (pesan pratinjau sementara)
       ├─ streamMode=partial → edit teks terbaru
       └─ streamMode=block   → pemecahan blok + edit pembaruan
  └─ balasan akhir teks saja → edit akhir di pesan yang sama
  └─ fallback: hapus pratinjau + pengiriman akhir normal (media/kompleks)
```

Legenda:

-   `pesan pratinjau`: pesan sementara Telegram yang diperbarui selama pembuatan respons.
-   `edit akhir`: pengeditan langsung pada pesan pratinjau yang sama (hanya teks).
