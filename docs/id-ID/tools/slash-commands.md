---
summary: "Perintah slash: teks vs asli (native), konfigurasi, dan perintah yang didukung"
read_when:
  - Menggunakan atau mengonfigurasi perintah chat
  - Menelusuri masalah perutean perintah atau izin
title: "Perintah Slash (Slash Commands)"
---

# Perintah Slash

Perintah ditangani oleh Gateway. Sebagian besar perintah harus dikirim sebagai pesan **mandiri (standalone)** yang dimulai dengan `/`.
Perintah chat bash khusus host menggunakan `! <perintah>` (dengan `/bash <perintah>` sebagai alias).

Ada dua sistem yang berkaitan:

- **Perintah (Commands)**: pesan `/...` mandiri.
- **Arahan (Directives)**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  - Arahan akan dihapus dari pesan sebelum model melihatnya.
  - Pada pesan chat normal (bukan hanya arahan), arahan diperlakukan sebagai “petunjuk inline” dan **tidak** menetapkan pengaturan sesi secara permanen.
  - Pada pesan yang hanya berisi arahan, pengaturan akan disimpan ke sesi dan bot akan membalas dengan konfirmasi.
  - Arahan hanya diterapkan untuk **pengirim terotorisasi**.

Ada juga beberapa **pintasan inline** (hanya untuk pengirim terotorisasi): `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Pintasan ini langsung dijalankan, dihapus sebelum pesan dilihat model, dan sisa teks tetap diproses melalui alur normal.

## Konfigurasi

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

- `commands.text` (default `true`) mengaktifkan parsing `/...` dalam pesan chat.
- `commands.native` (default `"auto"`) mendaftarkan perintah asli (native commands) pada platform yang mendukungnya.
  - Auto: aktif untuk Discord/Telegram; nonaktif untuk Slack; diabaikan untuk penyedia tanpa dukungan perintah asli.
- `commands.nativeSkills` (default `"auto"`) mendaftarkan perintah **skill** secara asli jika didukung.
- `commands.bash` (default `false`) mengaktifkan `! <perintah>` untuk menjalankan perintah shell host.
- `commands.config` (default `false`) mengaktifkan `/config` (membaca/menulis `openclaw.json`).
- `commands.debug` (default `false`) mengaktifkan `/debug` (override runtime saja).
- `commands.allowFrom` (opsional) menetapkan daftar izinkan (allowlist) per penyedia untuk otorisasi perintah.

## Daftar Perintah

Teks + Native (jika diaktifkan):

- `/help`: bantuan.
- `/commands`: daftar perintah.
- `/skill <nama> [input]`: jalankan skill berdasarkan nama.
- `/status`: tampilkan status saat ini, termasuk penggunaan kuota penyedia.
- `/allowlist`: daftar/tambah/hapus entri daftar izinkan.
- `/approve <id> allow-once|allow-always|deny`: selesaikan permintaan persetujuan eksekusi.
- `/context [list|detail|json]`: jelaskan “konteks”; `detail` menunjukkan ukuran per file/alat/skill.
- `/export-session [jalur]`: ekspor sesi saat ini ke HTML.
- `/whoami`: tampilkan ID pengirim Anda (alias: `/id`).
- `/subagents list|kill|log|info|send|steer|spawn`: periksa, kontrol, atau jalankan sub-agen.
- `/kill <id|#|all>`: batalkan segera satu atau semua sub-agen yang sedang berjalan.
- `/config show|get|set|unset`: kelola konfigurasi di disk (memerlukan `commands.config: true`).
- `/debug show|set|unset|reset`: override runtime (memerlukan `commands.debug: true`).
- `/usage off|tokens|full|cost`: informasi penggunaan token atau ringkasan biaya di akhir balasan.
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`: kontrol TTS; lihat [/id-ID/tts](/id-ID/tts).
- `/stop`: hentikan sesi yang sedang berjalan.
- `/restart`: mulai ulang gateway.
- `/dock-telegram` / `/dock-discord` / `/dock-slack`: alihkan balasan ke platform tertentu.
- `/activation mention|always`: khusus grup, atur mode aktivasi bot.
- `/reset` atau `/new [model]`: buat sesi baru, opsional menentukan model.
- `/think <off|minimal|low|medium|high|xhigh>`: atur tingkat pemikiran model.
- `/verbose on|full|off`: aktifkan detail tambahan untuk perbaikan/debugging.
- `/reasoning on|off|stream`: tampilkan proses penalaran model secara terpisah.
- `/elevated on|off|ask|full`: atur mode elevasi untuk eksekusi perintah (full melewati persetujuan).
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`: pengaturan eksekusi.
- `/model <nama>`: ganti model (atau gunakan alias model).
- `/queue <mode>`: pengatura antrean pesan.
- `/bash <perintah>`: jalankan perintah shell host (alias untuk `! <perintah>`).

Hanya Teks:

- `/compact [instruksi]`: jalankan kompaksi (lihat [/id-ID/concepts/compaction](/id-ID/concepts/compaction)).
- `! <perintah>`: jalankan perintah shell host (satu per satu).
- `!poll`: periksa output/status pekerjaan bash yang sedang berjalan.
- `!stop`: hentikan pekerjaan bash yang sedang berjalan.

Catatan:

- **Jalur cepat (Fast path)**: pesan yang hanya berisi perintah dari pengirim terotorisasi ditangani segera tanpa melalui antrean atau model.
- **Penyaringan sebutan grup**: pesan perintah saja akan melewati syarat sebutan (@mention).
- **Perintah Skill**: skill yang dapat dipanggil pengguna (user-invocable) diekspos sebagai perintah slash.
- **Penyelarasan Argument**: Discord menggunakan fitur autocomplete untuk pilihan dinamis; Telegram dan Slack akan menunjukkan menu tombol jika argumen tidak disertakan.

## Pemilihan Model (`/model`)

Perintah `/model` mempermudah penggantian model yang digunakan dalam percakapan.

Contoh:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Catatan:

- `/model` dan `/model list` menampilkan pemilih nomor (picker) model yang tersedia.
- `/model <nomor>` memilih model dari daftar tersebut.
- `/model status` menampilkan detail model, termasuk endpoint penyedia dan mode API.

## Override Debug

`/debug` memungkinkan pengaturan konfigurasi **hanya untuk runtime** (disimpan di memori, bukan di disk). Berguna untuk pengujian sementara. Memerlukan `commands.debug: true`.

## Pembaruan Konfigurasi

`/config` menulis langsung ke file konfigurasi `openclaw.json` Anda. Perubahan ini bersifat permanen. Memerlukan `commands.config: true`.

## Catatan Platform

- **Perintah Teks** berjalan dalam sesi chat normal (DM berbagi `main`, grup memiliki sesinya sendiri).
- **Perintah Asli (Native)** menggunakan sesi terisolasi untuk menghindari tabrakan dengan chat reguler.
- **`/stop`** menargetkan sesi chat aktif untuk membatalkan proses yang sedang berjalan.
- **Slack**: Jika mengaktifkan `commands.native`, Anda harus membuat satu perintah slash Slack untuk setiap perintah bawaan secara manual.


