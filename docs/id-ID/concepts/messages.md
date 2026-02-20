---
summary: "Alur pesan, sesi, antrean, dan visibilitas penalaran"
read_when:
  - Menjelaskan bagaimana pesan masuk menjadi balasan
  - Mengklarifikasi sesi, mode antrean, atau perilaku streaming
  - Mendokumentasikan visibilitas penalaran dan implikasi penggunaan
title: "Pesan"
---

# Pesan

Halaman ini merangkum bagaimana OpenClaw menangani pesan masuk, sesi, antrean, streaming, dan visibilitas penalaran.

## Alur Pesan (Tingkat Tinggi)

```
Pesan Masuk
  -> perutean/binding -> kunci sesi
  -> antrean (jika ada run yang aktif)
  -> proses agen (streaming + alat)
  -> balasan keluar (batas saluran + pemotongan)
```

Pengaturan utama ada di konfigurasi:

- `messages.*` untuk awalan (prefix), antrean, dan perilaku grup.
- `agents.defaults.*` untuk streaming blok dan default pemotongan (chunking).
- Override saluran (`channels.whatsapp.*`, `channels.telegram.*`, dll.) untuk batas dan peralihan streaming.

Lihat [Konfigurasi](/id-ID/gateway/configuration) untuk skema lengkap.

## Dedulpikasi Pesan Masuk

Saluran dapat mengirim ulang pesan yang sama setelah koneksi ulang. OpenClaw menyimpan cache jangka pendek berdasarkan saluran/akun/peer/sesi/ID pesan sehingga pengiriman duplikat tidak memicu proses agen berulang kali.

## Debouncing Pesan Masuk

Pesan beruntun yang cepat dari **pengirim yang sama** dapat digabungkan menjadi satu giliran agen melalui `messages.inbound`. Debouncing dibatasi per saluran + percakapan dan menggunakan pesan terbaru untuk threading/ID balasan.

Konfigurasi (default global + override per saluran):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Catatan:

- Debounce hanya berlaku untuk pesan **teks saja**; media/lampiran akan langsung diproses.
- Perintah kontrol melewati proses debouncing agar tetap berdiri sendiri.

## Sesi dan Perangkat

Sesi dimiliki oleh gateway, bukan oleh klien.

- Obrolan langsung (DM) digabungkan ke dalam kunci sesi utama agen.
- Grup/saluran mendapatkan kunci sesi mereka sendiri.
- Penyimpanan sesi dan transkrip berada di host gateway.

Beberapa perangkat/saluran dapat dipetakan ke sesi yang sama, tetapi riwayat tidak sepenuhnya disinkronkan kembali ke setiap klien. Rekomendasi: gunakan satu perangkat utama untuk percakapan panjang guna menghindari konteks yang berbeda. UI Kontrol dan TUI selalu menunjukkan transkrip sesi yang didukung gateway, sehingga keduanya adalah sumber kebenaran.

Detail: [Manajemen Sesi](/id-ID/concepts/session).

## Isi Pesan Masuk dan Konteks Riwayat

OpenClaw memisahkan **body prompt** dari **body perintah**:

- `Body`: teks prompt yang dikirim ke agen. Ini dapat mencakup amplop saluran dan pembungkus riwayat opsional.
- `CommandBody`: teks asli pengguna untuk penguraian arahan/perintah.
- `RawBody`: alias lama untuk `CommandBody` (dipertahankan untuk kompatibilitas).

Ketika sebuah saluran menyediakan riwayat, ia menggunakan pembungkus bersama:

- `[Pesan obrolan sejak balasan terakhir Anda - untuk konteks]`
- `[Pesan saat ini - tanggapi ini]`

Untuk **obrolan non-langsung** (grup/saluran/ruang), **isi pesan saat ini** diawali dengan label pengirim (gaya yang sama digunakan untuk entri riwayat). Ini menjaga konsistensi antara pesan real-time dan pesan yang antre/riwayat dalam prompt agen.

Buffer riwayat hanya berisi pesan **pending**: ini mencakup pesan grup yang *tidak* memicu proses agen (misalnya, pesan tanpa mention) dan **mengecualikan** pesan yang sudah ada dalam transkrip sesi.

Penghapusan arahan (directive stripping) hanya berlaku pada bagian **pesan saat ini** sehingga riwayat tetap utuh. Saluran yang membungkus riwayat harus menyetel `CommandBody` (atau `RawBody`) ke teks pesan asli dan membiarkan `Body` sebagai prompt gabungan.

Buffer riwayat dapat dikonfigurasi melalui `messages.groupChat.historyLimit` (default global) dan override per saluran seperti `channels.slack.historyLimit` atau `channels.telegram.accounts.<id>.historyLimit` (setel `0` untuk menonaktifkan).

## Antrean dan Followup

Jika sebuah proses (run) sudah aktif, pesan masuk dapat dimasukkan ke antrean, diarahkan ke proses saat ini, atau dikumpulkan untuk giliran tindak lanjut (followup).

- Konfigurasi melalui `messages.queue` (dan `messages.queue.byChannel`).
- Mode: `interrupt`, `steer`, `followup`, `collect`, ditambah varian backlog.

Detail: [Antrean](/id-ID/concepts/queue).

## Streaming, Chunking, dan Batching

Streaming blok mengirimkan balasan parsial saat model menghasilkan blok teks. Chunking menghormati batas teks saluran dan menghindari pemotongan blok kode.

Pengaturan utama:

- `agents.defaults.blockStreamingDefault` (`on|off`, default off)
- `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
- `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
- `agents.defaults.blockStreamingCoalesce` (batching berdasarkan waktu idle)
- `agents.defaults.humanDelay` (jeda seperti manusia di antara balasan blok)
- Override saluran: `*.blockStreaming` dan `*.blockStreamingCoalesce` (saluran non-Telegram memerlukan `*.blockStreaming: true` secara eksplisit)

Detail: [Streaming + chunking](/id-ID/concepts/streaming).

## Visibilitas Penalaran (Reasoning) dan Token

OpenClaw dapat menampilkan atau menyembunyikan penalaran model:

- `/reasoning on|off|stream` mengontrol visibilitas.
- Konten penalaran tetap dihitung dalam penggunaan token saat dihasilkan oleh model.
- Telegram mendukung aliran penalaran ke dalam gelembung draf.

Detail: [Arahan Berpikir + Penalaran](/id-ID/tools/thinking) dan [Penggunaan Token](/id-ID/reference/token-use).

## Awalan (Prefix), Threading, dan Balasan

Pemformatan pesan keluar disentralisasi di `messages`:

- `messages.responsePrefix`, `channels.<channel>.responsePrefix`, dan `channels.<channel>.accounts.<id>.responsePrefix` (kaskade awalan keluar), ditambah `channels.whatsapp.messagePrefix` (awalan masuk WhatsApp)
- Threading balasan melalui `replyToMode` dan default per saluran.

Detail: [Konfigurasi](/id-ID/gateway/configuration#messages) dan dokumentasi saluran.
