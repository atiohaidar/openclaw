---
summary: "Penyiapan Slack dan perilaku runtime (Mode Socket + HTTP Events API)"
read_when:
  - Mengonfigurasi Slack atau mendebug mode socket/HTTP
title: "Slack"
---

# Slack

Status: Siap untuk produksi untuk Pesan Langsung (DM) dan Saluran (Channels) melalui integrasi aplikasi Slack. Secara default menggunakan **Mode Socket**, namun Mode HTTP juga didukung.

## Penyiapan Cepat

### Mode Socket (Default)
1. **Buat Aplikasi Slack**: Aktifkan **Socket Mode** di pengaturan aplikasi Slack Anda.
2. **Dapatkan Token**:
   - Buat **App Token** (`xapp-...`) dengan izin `connections:write`.
   - Instal aplikasi ke workspace Anda dan salin **Bot Token** (`xoxb-...`).
3. **Konfigurasi OpenClaw**:
```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```
4. **Langganan Event**: Tambahkan event seperti `app_mention`, `message.channels`, `message.im`, dan reaksi di menu **Event Subscriptions**.

### Mode HTTP (Events API)
Gunakan mode ini jika Anda ingin menggunakan webhook HTTPS tradisional alih-alih koneksi websocket persisten. Berikan **Signing Secret** dan setel URL Request ke endpoint gateway Anda (default: `/slack/events`).

## Kontrol Akses

### Pesan Langsung (DM)
- Default menggunakan mode **pairing**. Pengirim baru harus disetujui melalui perintah `openclaw pairing approve slack <KODE>`.
- Anda dapat mengubah kebijakan menjadi `allowlist` atau `open`.

### Saluran (Channels)
- Secara default, bot hanya akan merespons di saluran jika di-mention (@mention) atau berada dalam utasan (thread) yang aktif.
- Gunakan konfigurasi `channels.slack.channels` untuk memberikan instruksi khusus per saluran.

## Perilaku Utasan (Threading)

OpenClaw mendukung percakapan dalam utasan. Secara default, balasan bot tetap berada di dalam utasan yang sama. Anda dapat mengatur apakah bot harus mengambil riwayat pesan sebelumnya dari utasan tersebut melalui pengaturan `historyLimit`.

## Kemampuan Saluran

- **Pesan Streaming**: Teks yang dihasilkan bot dikirim secara bertahap sehingga pengguna tidak perlu menunggu seluruh balasan selesai.
- **Lampiran**: Mendukung pengunduhan gambar dan file dari Slack (batas default 20 MB).
- **Perintah Slash**: Anda dapat menggunakan `/openclaw` (atau nama lain yang dikonfigurasi) untuk berinteraksi dengan asisten.
- **Reaksi**: Bot dapat memberikan reaksi emoji sebagai konfirmasi bahwa pesan sedang diproses.

## Pemecahan Masalah

- **Bot tidak membalas di saluran**: Pastikan bot sudah diundang ke saluran tersebut dan kebijakan `groupPolicy` memperbolehkan akses.
- **Mode Socket tidak terhubung**: Periksa apakah App Token dan Bot Token sudah benar dan Socket Mode diaktifkan di dasbor Slack.
- **Pesan DM diabaikan**: Cek status pairing dengan `openclaw pairing list slack`.

Untuk detail izin (scopes) dan manifest aplikasi yang lengkap, lihat [Referensi Konfigurasi Slack](/id-ID/gateway/configuration-reference#slack).



