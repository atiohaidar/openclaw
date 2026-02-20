---
summary: "Ringkasan bot Feishu, fitur, dan konfigurasi"
read_when:
  - Anda ingin menghubungkan bot Feishu/Lark
  - Anda sedang mengonfigurasi saluran Feishu
title: Feishu
---

# Bot Feishu

Feishu (Lark) adalah platform obrolan tim yang digunakan perusahaan untuk kolaborasi. Plugin ini menghubungkan OpenClaw ke bot Feishu/Lark menggunakan langganan event WebSocket sehingga pesan dapat diterima tanpa harus mengekspos URL webhook publik.

## Instalasi Plugin

Instal plugin Feishu terlebih dahulu:

```bash
openclaw plugins install @openclaw/feishu
```

## Mulai Cepat (Quickstart)

### Metode 1: Wizard Onboarding (Direkomendasikan)
Jalankan perintah berikut dan pilih Feishu:
```bash
openclaw onboard
```

### Metode 2: Setup CLI Manual
Jika Anda sudah melewati tahap awal, gunakan:
```bash
openclaw channels add
```
Pilih **Feishu**, lalu masukkan App ID dan App Secret Anda.

## Langkah 1: Buat Aplikasi Feishu

1. **Buka Feishu Open Platform**: Masuk ke [open.feishu.cn](https://open.feishu.cn/app). Untuk Lark (versi global), gunakan [open.larksuite.com](https://open.larksuite.com/app).
2. **Buat Aplikasi**: Klik **Create enterprise app**, isi nama dan deskripsi bot Anda.
3. **Salin Kredensial**: Dari menu **Credentials & Basic Info**, salin **App ID** dan **App Secret**.
4. **Konfigurasi Izin (Permissions)**: Masuk ke bagian Permissions dan tambahkan izin `im:message`, `im:chat`, dan lainnya yang diperlukan agar bot bisa mengirim dan menerima pesan.
5. **Aktifkan Kemampuan Bot**: Di menu **App Capability > Bot**, aktifkan fitur bot.
6. **Berlangganan Event**: Di menu **Event Subscription**, pilih **Use long connection to receive events** (WebSocket) dan tambahkan event `im.message.receive_v1`.
7. **Terbitkan Aplikasi**: Buat versi aplikasi di **Version Management** dan ajukan rilis.

## Konfigurasi Lanjutan

Anda dapat mengatur perilaku bot melalui file `openclaw.json`:

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing", // Opsi: pairing, allowlist, open, disabled
      streaming: true, // Menggunakan kartu interaktif untuk streaming teks
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

## Kontrol Akses

- **Pesan Langsung (DM)**: Secara default menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui `openclaw pairing approve feishu <KODE>`.
- **Grup**: Secara default membutuhkan **@mention** agar bot merespons. Anda bisa mengubahnya di pengaturan `groups`.

## Pemecahan Masalah

- **Bot tidak merespons di grup**: Pastikan bot sudah ditambahkan ke grup dan Anda melakukan @mention (jika fitur tersebut aktif).
- **Pesan tidak diterima**: Pastikan **Event Subscription** sudah menggunakan mode **Long Connection** (WebSocket) dan aplikasi sudah dalam status diterbitkan (Published).
- **Gagal mengirim pesan**: Cek apakah izin `im:message:send_as_bot` sudah diberikan.

Untuk detail konfigurasi lebih lengkap, lihat [Referensi Konfigurasi](/id-ID/gateway/configuration).



