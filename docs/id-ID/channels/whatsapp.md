---
summary: "Dukungan saluran WhatsApp, kontrol akses, perilaku pengiriman, dan operasional"
read_when:
  - Mengonfigurasi perilaku saluran WhatsApp atau perutean pesan
title: "WhatsApp"
---

# WhatsApp (Saluran Web)

Status: Siap digunakan (production-ready) melalui WhatsApp Web (Baileys). Gateway menangani sesi yang terhubung.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/id-ID/channels/pairing">
    Kebijakan DM default menggunakan mode pairing untuk pengirim yang tidak dikenal.
  </Card>
  <Card title="Troubleshooting" icon="wrench" href="/id-ID/channels/troubleshooting">
    Panduan diagnosa dan perbaikan saluran.
  </Card>
  <Card title="Konfigurasi Gateway" icon="settings" href="/id-ID/gateway/configuration">
    Contoh dan pola konfigurasi saluran lengkap.
  </Card>
</CardGroup>

## Menghubungkan WhatsApp

<Steps>
  <Step title="Konfigurasi kebijakan akses">
```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+628123456789"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+628123456789"],
    },
  },
}
```
  </Step>

  <Step title="Login WhatsApp (QR)">
```bash
openclaw channels login --channel whatsapp
```
  </Step>

  <Step title="Mulai Gateway">
```bash
openclaw gateway
```
  </Step>

  <Step title="Setujui permintaan pairing">
```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <KODE>
```
  </Step>
</Steps>

<Note>
OpenClaw menyarankan penggunaan nomor terpisah untuk bot jika memungkinkan, meskipun penggunaan nomor pribadi tetap didukung.
</Note>

## Kebijakan Akses

- **dmPolicy**: Mengontrol akses pesan langsung (DM). Pilihan: `pairing`, `allowlist`, `open`, `disabled`.
- **groupPolicy**: Mengontrol akses di dalam grup. Pilihan: `open`, `allowlist`, `disabled`.
- **allowFrom**: Daftar nomor telepon (E.164) yang diizinkan untuk berinteraksi.

### Grup dan Sebutan (Mentions)
Secara default, bot hanya akan merespons di grup jika:
- Bot disebut (mention) secara eksplisit.
- Pesan bot dibalas (reply).
- Pola sebutan kustom cocok dengan isi pesan.

## Perilaku Pesan

### Reaksi Tanda Terima (Ack Reactions)
Bot dapat memberikan reaksi emoji secara otomatis saat menerima pesan:
```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions"
      },
    },
  },
}
```

### Media dan Ukuran
- Mendukung gambar, video, audio (Pesan Suara), dan dokumen.
- Batas default ukuran media: 5MB untuk balasan otomatis.
- Gambar akan otomatis dioptimalkan ukurannya jika melebihi batas.

## Pemecahan Masalah Umum

- **Status: Not Linked**: Gunakan `openclaw channels login --channel whatsapp` untuk memindai QR ulang.
- **Pesan Grup Diabaikan**: Periksa `groupPolicy`, `groupAllowFrom`, dan pastikan bot sudah disebut (mention) jika diperlukan.
- **Error Bun**: Disarankan menjalankan Gateway WhatsApp menggunakan Node.js, karena Bun saat ini belum sepenuhnya stabil untuk operasi WhatsApp.

## Terkait
- [Pairing](/id-ID/channels/pairing)
- [Perutean Saluran](/id-ID/channels/channel-routing)
- [Troubleshooting](/id-ID/channels/troubleshooting)

