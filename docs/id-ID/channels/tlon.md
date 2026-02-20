---
summary: "Status dukungan Tlon/Urbit, kemampuan, dan konfigurasi"
read_when:
  - Mengonfigurasi saluran Tlon/Urbit
title: "Tlon"
---

# Tlon (Plugin)

Tlon adalah pengirim pesan terdesentralisasi yang dibangun di atas Urbit. OpenClaw terhubung ke "kapapl" (ship) Urbit Anda serta dapat menanggapi pesan DM dan chat grup. Balasan grup secara default membutuhkan sebutan (@mention).

Status: Didukung melalui plugin. Mendukung DM, sebutan grup, balasan utasan (thread), dan teks-only media fallback (URL ditambahkan ke keterangan). Reaksi, polling, dan unggahan media asli belum didukung.

## Instalasi Plugin

Saluran Tlon tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/tlon
```

## Penyiapan Cepat (Setup)

1. **Instal Plugin** Tlon.
2. **Kumpulkan Kredensial**: Dapatkan URL ship dan kode login Anda.
3. **Konfigurasi OpenClaw**: Masukkan detail ship ke dalam `openclaw.json`.
4. **Mulai Gateway**: Bot akan mulai terhubung ke ship Urbit Anda.

Contoh konfigurasi minimal:
```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://host-ship-anda",
      code: "kode-login-anda",
    },
  },
}
```

## Saluran Grup

Penemuan saluran secara otomatis (*auto-discovery*) diaktifkan secara default. Anda juga dapat menentukan saluran secara manual menggunakan parameter `groupChannels`. Jika ship Anda berada di jaringan pribadi/lokal, pastikan untuk menyetel `allowPrivateNetwork: true` untuk alasan keamanan (SSRF hardening).

## Kontrol Akses

- **Daftar Izinkan DM**: Secara default bot mengizinkan semua DM jika daftar ini kosong. Anda dapat membatasinya dengan `dmAllowlist`.
- **Otorisasi Grup**: Secara default dibatasi. Anda dapat mengatur aturan per saluran (misal: `restricted` atau `open`) menggunakan `authorization.channelRules`.

## Catatan Penting

- **Sebutan (Mention)**: Balasan di dalam grup mengharuskan bot di-mention agar ia merespons.
- **Utasan (Threads)**: Jika pesan masuk berada dalam sebuah utasan, OpenClaw akan membalas di dalam utasan tersebut.
- **Media**: Pengiriman media saat ini akan dikirim sebagai teks yang berisi URL tautan ke media tersebut.

Untuk referensi lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



