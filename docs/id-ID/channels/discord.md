---
summary: "Status dukungan bot Discord, kemampuan, dan konfigurasi"
read_when:
  - Mengerjakan fitur saluran Discord
title: "Discord"
---

# Discord (Bot API)

Status: Siap digunakan untuk DM dan saluran server (guild) melalui gateway resmi Discord.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/id-ID/channels/pairing">
    DM Discord menggunakan mode pairing secara default.
  </Card>
  <Card title="Slash Commands" icon="terminal" href="/id-ID/tools/slash-commands">
    Perilaku perintah asli dan katalog perintah.
  </Card>
  <Card title="Troubleshooting" icon="wrench" href="/id-ID/channels/troubleshooting">
    Panduan diagnosa dan perbaikan saluran.
  </Card>
</CardGroup>

## Penyiapan Cepat

Anda perlu membuat aplikasi baru dengan bot, menambahkannya ke server, dan menghubungkannya ke OpenClaw.

<Steps>
  <Step title="Buat aplikasi dan bot di Discord Developer Portal">
    Buka [Developer Portal](https://discord.com/developers/applications), buat **New Application**, dan atur bot pada bagian **Bot**.
  </Step>

  <Step title="Aktifkan Privileged Intents">
    Pada halaman **Bot**, aktifkan:
    - **Message Content Intent** (Wajib)
    - **Server Members Intent** (Disarankan)
  </Step>

  <Step title="Salin Token Bot">
    Klik **Reset Token** untuk mendapatkan token pertama Anda. Simpan token ini dengan aman.
  </Step>

  <Step title="Undang bot ke server">
    Gunakan **OAuth2 URL Generator** dengan cakupan `bot` dan `applications.commands`, serta izin dasar (Send Messages, View Channels, Read History). Buka URL yang dihasilkan di browser untuk mengundang bot.
  </Step>

  <Step title="Atur Konfigurasi OpenClaw">
```bash
openclaw config set channels.discord.token "TOKEN_ANDA"
openclaw config set channels.discord.enabled true
openclaw gateway restart
```
  </Step>

  <Step title="Setujui pairing pertama">
    Kirim pesan DM ke bot di Discord, lalu ambil kode pairing yang muncul dan setujui melalui CLI:
```bash
openclaw pairing approve discord <KODE>
```
  </Step>
</Steps>

## Kebijakan Akses

- **dmPolicy**: Mengontrol akses pesan langsung (`pairing`, `allowlist`, `open`, `disabled`).
- **groupPolicy**: Mengontrol akses bot di dalam server (guild). Defaultnya adalah `allowlist` demi keamanan.
- **guilds**: Anda harus mendaftarkan ID Server Anda di dalam konfigurasi agar bot dapat merespons di luar DM.

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "ID_SERVER_ANDA": {
          requireMention: true,
          users: ["ID_USER_ANDA"]
        }
      }
    }
  }
}
```

### Sebutan (Mentions)
Di dalam server, bot biasanya hanya merespons jika disebut (@bot). Anda dapat mengubah ini dengan menyetel `requireMention: false` pada konfigurasi server tertentu.

## Komponen Interaktif

OpenClaw mendukung komponen Discord seperti tombol dan menu pilihan. Ini digunakan untuk berbagai hal, termasuk **persetujuan eksekusi perintah (exec approvals)** secara langsung dari chat.

## Pemecahan Masalah
- **Bot Tidak Melihat Pesan**: Pastikan **Message Content Intent** sudah diaktifkan di Developer Portal.
- **Pesan Server Diabaikan**: Periksa kebijakan `groupPolicy` dan pastikan ID Server sudah terdaftar di `guilds`.
- **Error Pairing**: Pastikan Anda sudah mengaktifkan pengaturan "Allow Direct Messages from server members" di Discord.

## Terkait
- [Pairing](/id-ID/channels/pairing)
- [Perutean Saluran](/id-ID/channels/channel-routing)
- [Troubleshooting](/id-ID/channels/troubleshooting)



