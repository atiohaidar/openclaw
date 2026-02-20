---
summary: "Pipa pemformatan Markdown untuk saluran keluar"
read_when:
  - Anda mengubah pemformatan markdown atau pemecahan pesan untuk saluran keluar
  - Anda menambahkan pemformat saluran baru atau pemetaan gaya
  - Anda sedang men-debug regresi pemformatan lintas saluran
title: "Pemformatan Markdown"
---

# Pemformatan Markdown

OpenClaw memformat Markdown keluar dengan mengubahnya menjadi perwakilan perantara (intermediate representation - IR) bersama sebelum merender output spesifik saluran. IR menjaga teks sumber tetap utuh sambil membawa rentang gaya/tautan sehingga pemecahan pesan (chunking) dan perenderan tetap konsisten di semua saluran.

## Tujuan

- **Konsistensi**: satu tahap penguraian (parse), banyak perender (renderer).
- **Pemecahan Aman**: memecah teks sebelum perenderan sehingga pemformatan inline tidak pernah terputus di antara potongan pesan.
- **Kesesuaian Saluran**: memetakan IR yang sama ke mrkdwn Slack, HTML Telegram, dan rentang gaya Signal tanpa mengurai ulang Markdown.

## Jalur Proses (Pipeline)

1.  **Urai Markdown -> IR**
    - IR adalah teks polos ditambah rentang gaya (bold/italic/strike/code/spoiler) dan rentang tautan.
    - Offset menggunakan unit kode UTF-16 agar rentang gaya Signal selaras dengan API-nya.
    - Tabel diurai hanya jika saluran memilih konversi tabel.
2.  **Pecah IR (Format-Dahulu)**
    - Pemecahan terjadi pada teks IR sebelum perenderan.
    - Pemformatan inline tidak terpisah di antara potongan pesan; rentang gaya diiris per potongan.
3.  **Render per Saluran**
    - **Slack**: token mrkdwn (bold/italic/strike/code), tautan sebagai `<url|label>`.
    - **Telegram**: tag HTML (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
    - **Signal**: teks polos + rentang `text-style`; tautan menjadi `label (url)` jika label berbeda.

## Contoh IR

Input Markdown:

```markdown
Hello **world** — lihat [docs](https://docs.openclaw.ai).
```

IR (skema):

```json
{
  "text": "Hello world — lihat docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 21, "end": 25, "href": "https://docs.openclaw.ai" }]
}
```

## Di Mana Ini Digunakan

- Adaptor keluar Slack, Telegram, dan Signal merender dari IR.
- Saluran lain (WhatsApp, iMessage, MS Teams, Discord) masih menggunakan teks polos atau aturan pemformatan mereka sendiri, dengan konversi tabel Markdown yang diterapkan sebelum pemecahan pesan jika diaktifkan.

## Penanganan Tabel

Tabel Markdown tidak didukung secara konsisten di berbagai klien obrolan. Gunakan `markdown.tables` untuk mengontrol konversi per saluran (dan per akun).

- `code`: merender tabel sebagai blok kode (default untuk sebagian besar saluran).
- `bullets`: mengubah setiap baris menjadi poin-poin (bullet points) (default untuk Signal + WhatsApp).
- `off`: menonaktifkan penguraian dan konversi tabel; teks tabel mentah akan diteruskan apa adanya.

Kunci konfigurasi:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## Aturan Pemecahan (Chunking)

- Batas potongan pesan berasal dari adaptor/konfigurasi saluran dan diterapkan pada teks IR.
- Blok kode (fences) dipertahankan sebagai satu blok dengan baris baru di akhir agar saluran merendernya dengan benar.
- Awalan daftar (list) dan awalan kutipan blok (blockquote) adalah bagian dari teks IR, sehingga pemecahan tidak terjadi di tengah awalan.
- Gaya inline (bold/italic/strike/inline-code/spoiler) tidak pernah terpisah di antara potongan pesan; perender membuka kembali gaya di dalam setiap potongan.

Jika Anda membutuhkan lebih banyak informasi tentang perilaku pemecahan pesan di berbagai saluran, lihat [Streaming + Pemecahan Pesan](/id-ID/concepts/streaming).

## Kebijakan Tautan

- **Slack**: `[label](url)` -> `<url|label>`; URL polos tetap polos. Autolink dinonaktifkan selama penguraian untuk menghindari tautan ganda.
- **Telegram**: `[label](url)` -> `<a href="url">label</a>` (mode urai HTML).
- **Signal**: `[label](url)` -> `label (url)` kecuali jika label cocok dengan URL-nya.

## Spoiler

Penanda spoiler (`||spoiler||`) hanya diurai untuk Signal, di mana mereka dipetakan ke rentang gaya SPOILER. Saluran lain memperlakukannya sebagai teks polos.

## Cara Menambah atau Memperbarui Pemformat Saluran

1.  **Urai Sekali**: gunakan pembantu `markdownToIR(...)` bersama dengan opsi yang sesuai untuk saluran (autolink, gaya tajuk, awalan kutipan).
2.  **Render**: terapkan perender dengan `renderMarkdownWithMarkers(...)` dan peta penanda gaya.
3.  **Pecah**: panggil `chunkMarkdownIR(...)` sebelum perenderan; render setiap potongan.
4.  **Hubungkan Adaptor**: perbarui adaptor keluar saluran untuk menggunakan pemecah pesan dan perender baru.
5.  **Tes**: tambah atau perbarui tes format dan tes pengiriman keluar jika saluran menggunakan pemecahan pesan.

## Hal yang Perlu Diperhatikan

- Token kurung sudut Slack (`<@U123>`, `<#C123>`, `<https://...>`) harus dipertahankan; lepaskan (escape) HTML mentah dengan aman.
- HTML Telegram memerlukan pelepasan teks di luar tag untuk menghindari markup yang rusak.
- Rentang gaya Signal bergantung pada offset UTF-16; jangan gunakan offset titik kode (code point).
- Pertahankan baris baru di akhir untuk blok kode agar penanda penutup berada di barisnya sendiri.
