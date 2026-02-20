---
summary: "Status dukungan bot Telegram, kemampuan, dan konfigurasi"
read_when:
  - Mengerjakan fitur Telegram atau webhook
title: "Telegram"
---

# Telegram (Bot API)

Status: Siap digunakan (production-ready) untuk DM bot dan grup melalui grammY. Secara default menggunakan mode *long polling*; mode *webhook* bersifat opsional.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/id-ID/channels/pairing">
    Kebijakan DM default untuk Telegram adalah pairing.
  </Card>
  <Card title="Troubleshooting" icon="wrench" href="/id-ID/channels/troubleshooting">
    Panduan diagnosa dan perbaikan saluran.
  </Card>
  <Card title="Konfigurasi Gateway" icon="settings" href="/id-ID/gateway/configuration">
    Contoh dan pola konfigurasi saluran lengkap.
  </Card>
</CardGroup>

## Penyiapan Cepat

<Steps>
  <Step title="Buat bot di BotFather">
    Buka Telegram dan chat dengan **@BotFather**. Jalankan `/newbot`, ikuti petunjuknya, dan simpan **token bot** yang diberikan.
  </Step>

  <Step title="Konfigurasi token dan kebijakan DM">
```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc", // Masukkan token dari BotFather
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```
  </Step>

  <Step title="Mulai gateway dan setujui DM pertama">
```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <KODE>
```
  </Step>
</Steps>

## Pengaturan Sisi Telegram

- **Mode Privasi**: Secara default, bot dalam grup hanya menerima pesan yang menyebutnya (mention). Jika ingin bot melihat semua pesan, matikan mode privasi via `/setprivacy` di @BotFather atau jadikan bot sebagai admin grup.
- **Admin Grup**: Memberikan status admin memungkinkan bot melihat semua riwayat pesan, sangat berguna untuk fitur grup yang selalu aktif.

## Kontrol Akses

- **dmPolicy**: Mengontrol akses pesan langsung (`pairing`, `allowlist`, `open`, `disabled`).
- **allowFrom**: Daftar ID numerik pengguna Telegram yang diizinkan.
- **groupPolicy**: Mengontrol siapa yang boleh berinteraksi dengan bot di dalam grup (`open`, `allowlist`, `disabled`).

### Mencari ID Pengguna Telegram
1. Kirim pesan ke bot Anda.
2. Jalankan `openclaw logs --follow`.
3. Cari nilai `from.id` di dalam log.

## Fitur Unggulan

- **Pratinjau Langsung (Live Stream)**: OpenClaw dapat menampilkan teks balasan saat sedang dibuat (streaming) di Telegram. Atur `channels.telegram.streamMode` ke `partial`.
- **Tombol Inline**: Mendukung tombol di bawah pesan yang dapat diklik pengguna untuk memberikan masukan ke bot.
- **Topik Forum**: Mendukung grup bertipe forum (supergroups) dengan isolasi sesi per topik.
- **Pesan Suara & Stiker**: Bot dapat mengirimkan pesan suara (`audio_as_voice`) dan mencerna pesan stiker (melalui cache pengenalan visual).

## Pemecahan Masalah
- **Bot Tidak Merespons di Grup**: Pastikan bot sudah disebut (mention) atau matikan Mode Privasi.
- **Command Tidak Muncul**: Tunggu beberapa saat agar Telegram memperbarui menu bot setelah konfigurasi `customCommands`.
- **Error Jaringan**: Periksa apakah `api.telegram.org` dapat dijangkau dari server Anda.

## Terkait
- [Pairing](/id-ID/channels/pairing)
- [Perutean Saluran](/id-ID/channels/channel-routing)
- [Troubleshooting](/id-ID/channels/troubleshooting)



