---
summary: "Desain antrean perintah yang melakukan serialisasi proses agen balasan otomatis masuk"
read_when:
  - Mengubah eksekusi balasan otomatis atau konkurensi
title: "Antrean Perintah"
---

# Antrean Perintah (16-01-2026)

Kami melakukan serialisasi proses agen balasan otomatis masuk (dari semua saluran) melalui antrean internal dalam proses (in-process) yang kecil untuk mencegah bentrokan antara beberapa proses agen, sambil tetap mengizinkan paralelisme yang aman antar sesi.

## Mengapa

- Proses balasan otomatis bisa memakan biaya tinggi (panggilan LLM) dan dapat berbenturan ketika beberapa pesan masuk tiba dalam waktu yang hampir bersamaan.
- Serialisasi menghindari persaingan untuk sumber daya bersama (file sesi, log, CLI stdin) dan mengurangi kemungkinan terkena batas tingkat (rate limits) dari hulu.

## Cara Kerja

- Antrean FIFO (First-In, First-Out) yang menyadari "jalur" (lane) mengosongkan setiap jalur dengan batas konkurensi yang dapat dikonfigurasi (default 1 untuk jalur yang tidak dikonfigurasi; jalur `main` defaultnya 4, `subagent` defaultnya 8).
- `runEmbeddedPiAgent` memasukkan ke antrean berdasarkan **kunci sesi** (jalur `session:<key>`) untuk menjamin hanya satu proses aktif per sesi.
- Setiap proses sesi kemudian dimasukkan ke dalam **jalur global** (jalur `main` secara default) sehingga paralelisme keseluruhan dibatasi oleh `agents.defaults.maxConcurrent`.
- Saat logging verbose diaktifkan, proses yang mengantre akan memancarkan pemberitahuan singkat jika mereka menunggu lebih dari ~2 detik sebelum dimulai.
- Indikator mengetik (typing indicators) tetap menyala segera setelah masuk antrean (jika didukung oleh saluran) sehingga pengalaman pengguna tetap lancar saat menunggu giliran.

## Mode Antrean (Per Saluran)

Pesan masuk dapat mengarahkan proses yang sedang berjalan, menunggu giliran tindak lanjut (followup), atau melakukan keduanya:

- `steer`: segera masukkan ke dalam proses yang sedang berjalan (membatalkan panggilan alat yang tertunda setelah batas alat berikutnya). Jika tidak mendukung streaming, akan beralih ke mode `followup`.
- `followup`: masuk antrean untuk giliran agen berikutnya setelah proses saat ini berakhir.
- `collect`: menggabungkan semua pesan yang mengantre menjadi **satu** giliran tindak lanjut (default). Jika pesan menargetkan saluran/utas yang berbeda, mereka akan diproses satu per satu untuk menjaga perutean.
- `steer-backlog` (alias `steer+backlog`): lakukan `steer` sekarang **dan** simpan pesan untuk giliran tindak lanjut.
- `interrupt` (lama): hentikan proses aktif untuk sesi tersebut, lalu jalankan pesan terbaru.
- `queue` (alias lama): sama seperti `steer`.

`steer-backlog` berarti Anda bisa mendapatkan respons tindak lanjut setelah proses yang diarahkan, sehingga permukaan streaming mungkin terlihat seperti duplikat. Gunakan `collect` / `steer` jika Anda menginginkan satu respons per pesan masuk.

Kirim `/queue collect` sebagai perintah mandiri (per sesi) atau setel `messages.queue.byChannel.discord: "collect"`.

Default (jika tidak diatur dalam konfigurasi):

- Semua permukaan → `collect`

Konfigurasi secara global atau per saluran melalui `messages.queue`:

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

## Opsi Antrean

Opsi berlaku untuk mode `followup`, `collect`, dan `steer-backlog` (serta untuk `steer` saat ia beralih ke mode `followup`):

- `debounceMs`: tunggu suasana tenang sebelum memulai giliran tindak lanjut (mencegah "lanjutkan, lanjutkan").
- `cap`: jumlah maksimal pesan yang mengantre per sesi.
- `drop`: kebijakan jika antrean penuh (`old`, `new`, `summarize`).

`summarize` menyimpan daftar poin singkat dari pesan-pesan yang dibuang dan menyajikannya sebagai prompt tindak lanjut sintetis.
Default: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

## Override Per Sesi

- Kirim `/queue <mode>` sebagai perintah mandiri untuk menyimpan mode sesi saat ini.
- Opsi dapat digabungkan: `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` atau `/queue reset` menghapus override sesi.

## Cakupan dan Jaminan

- Berlaku untuk proses agen balasan otomatis di semua saluran masuk yang menggunakan pipa balasan gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, dll.).
- Jalur default (`main`) berlaku untuk seluruh proses untuk balasan masuk + heartbeat utama; setel `agents.defaults.maxConcurrent` untuk mengizinkan beberapa sesi berjalan secara paralel.
- Jalur tambahan mungkin ada (misalnya `cron`, `subagent`) sehingga pekerjaan latar belakang dapat berjalan secara paralel tanpa menghalangi balasan masuk.
- Jalur per sesi menjamin bahwa hanya satu proses agen yang menyentuh sesi tertentu pada satu waktu.
- Tidak ada dependensi eksternal atau thread pekerja latar belakang; murni TypeScript + promises.

## Pemecahan Masalah

- Jika perintah tampak macet, aktifkan log verbose dan cari baris "queued for ...ms" untuk mengonfirmasi bahwa antrean sedang dikosongkan.
- Jika Anda ingin melihat kedalaman antrean, aktifkan log verbose dan perhatikan baris waktu antrean.
