---
summary: "Status dukungan bot Microsoft Teams, kemampuan, dan konfigurasi"
read_when:
  - Mengonfigurasi saluran MS Teams
title: "Microsoft Teams"
---

# Microsoft Teams (Plugin)

Status: Mendukung pengiriman teks dan lampiran di DM. Pengiriman file di saluran/grup memerlukan `sharePointSiteId` dan izin Graph API. Poll dikirim melalui Adaptive Cards.

## Instalasi Plugin

Saluran Microsoft Teams tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/msteams
```

## Penyiapan Cepat (Pemula)

1. **Instal Plugin** Microsoft Teams.
2. **Buat Azure Bot**: Dapatkan App ID, client secret, dan tenant ID dari [Azure Portal](https://portal.azure.com).
3. **Konfigurasi OpenClaw**: Masukkan kredensial tersebut ke dalam file `openclaw.json`.
4. **Ekspos Webhook**: Ekspos port `3978` (default) ke internet menggunakan tunnel (seperti Tailscale Funnel atau ngrok).
5. **Instal Paket Aplikasi Teams**: Unggah paket aplikasi (.zip) ke Microsoft Teams dan hubungkan ke bot Anda.

Contoh konfigurasi minimal:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

## Kontrol Akses (DM + Grup)

- **Akses DM**: Secara default menggunakan mode **pairing**. Pengirim yang tidak dikenal akan diabaikan sampai disetujui melalui perintah `openclaw pairing approve msteams <KODE>`.
- **Akses Grup/Saluran**: Secara default menggunakan mode **allowlist**. Anda harus menambahkan ID grup atau pengirim ke dalam `groupAllowFrom`.
- **Sebutan (Mention)**: Di dalam grup atau saluran, bot hanya akan merespons jika di-mention (@mention), kecuali diatur sebaliknya.

## Cara Kerja Webhook

Bot Framework mengirimkan pesan ke gateway Anda melalui jalur `/api/messages`. Gateway akan merespons pesan tersebut dan mengirimkan balasan kembali ke Teams menggunakan perutean deterministik.

## Pengiriman File di Grup

Berbeda dengan DM, pengiriman file di grup atau saluran Teams mengharuskan bot untuk mengunggah file ke **SharePoint** terlebih dahulu, kemudian membagikan tautannya. Anda perlu menyetel `sharePointSiteId` dalam konfigurasi agar fitur ini berfungsi.

## Adaptive Cards

OpenClaw mendukung pengiriman **Adaptive Cards** untuk polling atau tampilan kustom lainnya. Anda dapat mengirimkan JSON Adaptive Card langsung melalui alat perpesanan atau CLI.

## Pemecahan Masalah

- **Gambar tidak muncul di saluran**: Pastikan izin Microsoft Graph (`ChannelMessage.Read.All`) sudah diberikan dan admin sudah menyetujuinya.
- **Bot tidak merespons**: Periksa apakah URL webhook di Azure Bot sudah benar dan mengarah ke alamat publik gateway Anda.
- **Error 401 saat tes manual**: Hal ini wajar jika Anda ngetes secara manual tanpa JWT dari Azure. Gunakan fitur "Test in Web Chat" di Azure Portal untuk pengujian yang akurat.

Untuk panduan lebih lanjut, jalankan `openclaw status --all` atau hubungi komunitas pengembang.



