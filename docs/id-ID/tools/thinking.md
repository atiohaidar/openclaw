---
summary: "Sintaks arahan untuk /think + /verbose dan bagaimana pengaruhnya terhadap penalaran model"
read_when:
  - Mengatur pemrosesan arahan atau default thinking/verbose
title: "Tingkat Pemikiran (Thinking Levels)"
---

# Tingkat Pemikiran (Arahan /think)

## Apa fungsinya?

- Arahan inline dalam pesan masuk: `/t <level>`, `/think:<level>`, atau `/thinking <level>`.
- Tingkat (Level): `off | minimal | low | medium | high | xhigh` (khusus model GPT-5.2 + Codex)
  - minimal → “pikirkan (think)”
  - low → “pikirkan baik-baik (think hard)”
  - medium → “pikirkan lebih keras (think harder)”
  - high → “pikirkan sangat dalam (ultrathink)” (anggaran maksimal)
  - xhigh → “pikirkan sangat dalam+ (ultrathink+)” (khusus GPT-5.2 + Codex)
- Catatan Penyedia:
  - Z.AI (`zai/*`) hanya mendukung pemikiran biner (aktif/nonaktif). Level apa pun selain `off` diperlakukan sebagai aktif.

## Urutan Resolusi

1. Arahan inline pada pesan (hanya berlaku untuk pesan tersebut).
2. Override sesi (diatur dengan mengirim pesan yang hanya berisi arahan).
3. Default global (`agents.defaults.thinkingDefault` dalam konfigurasi).
4. Cadangan (fallback): `low` untuk model yang mampu menalar; `off` jika tidak.

## Mengatur Default Sesi

- Kirim pesan yang **hanya** berisi arahan (boleh ada spasi), misalnya `/think:medium` atau `/t high`.
- Aturan tersebut akan menetap untuk sesi saat ini (per-pengirim secara default); dihapus dengan `/think:off` atau pengaturan ulang sesi saat tidak aktif.
- Balasan konfirmasi akan dikirim (misal: `Thinking level set to high.`). Jika level tidak valid, permintaan akan ditolak dengan petunjuk.
- Kirim `/think` (atau `/think:`) tanpa argumen untuk melihat tingkat pemikiran saat ini.

## Arahan Verbose (/verbose atau /v)

- Level: `on` (minimal) | `full` | `off` (default).
- Pesan yang berisi arahan verbose saja akan mengubah status verbose sesi dan memberikan tanggapan `Verbose logging enabled.` / `Verbose logging disabled.`.
- Arahan inline hanya memengaruhi pesan tersebut; selain itu, default sesi/global tetap berlaku.
- Kirim `/verbose` tanpa argumen untuk melihat level verbose saat ini.
- Saat verbose aktif, agen yang menghasilkan hasil alat terstruktur (seperti agen Pi atau agen JSON lainnya) akan mengirim setiap pemanggilan alat kembali sebagai pesan khusus metadata terpisah.
- Saat verbose disetel ke `full`, output dari alat juga akan diteruskan setelah selesai.

## Visibilitas Penalaran (/reasoning)

- Level: `on|off|stream`.
- Pesan yang berisi arahan ini saja akan menentukan apakah blok pemikiran ditunjukkan dalam balasan.
- Saat diaktifkan, penalaran dikirim sebagai **pesan terpisah** dengan awalan `Reasoning:`.
- `stream` (khusus Telegram): menampilkan penalaran ke dalam bubble draf Telegram saat balasan sedang dibuat, lalu mengirim jawaban akhir tanpa penalaran tersebut.
- Alias: `/reason`.
- Kirim `/reasoning` tanpa argumen untuk melihat level penalaran saat ini.

## Terkait

- Dokumentasi mode elevasi dapat ditemukan di [Mode Elevasi](/id-ID/tools/elevated).

## Heartbeat

- Pesan heartbeat secara default hanya mengirimkan payload final. Untuk menyertakan pesan `Reasoning:` yang terpisah (bila tersedia), atur `agents.defaults.heartbeat.includeReasoning: true` di konfigurasi.

## UI Chat Web

- Pemilih (selector) pemikiran di chat web mencerminkan level simpanan sesi saat halaman dimuat.
- Memilih level lain hanya berlaku untuk satu pesan berikutnya; setelah pengiriman, pemilih akan kembali ke level sesi yang tersimpan.
- Untuk mengubah default sesi, kirim arahan `/think:<level>` seperti biasa.


