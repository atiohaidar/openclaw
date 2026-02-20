---
summary: "Status dukungan bot Zalo, kemampuan, dan konfigurasi"
read_when:
  - Bekerja pada fitur Zalo atau webhook
title: "Zalo"
---

# Zalo (Bot API)

Status: Eksperimental. Saat ini hanya mendukung pesan langsung (DM); dukungan grup akan segera hadir sesuai dokumentasi Zalo.

## Instalasi Plugin

Saluran Zalo tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/zalo
```

## Penyiapan Cepat (Quick Setup)

1. **Instal Plugin** Zalo.
2. **Setel Token**: Dapatkan token bot dari [Zalo Bot Platform](https://bot.zaloplatforms.com).
3. **Konfigurasi OpenClaw**: Masukkan token tersebut ke dalam file `openclaw.json`.
4. **Mulai Gateway**: Bot akan mulai mendengarkan pesan dari Zalo.
5. **Setujui Pairing**: Secara default, akses DM menggunakan mode **pairing**. Setujui kode pairing saat pertama kali bot dihubungi.

Contoh konfigurasi minimal:
```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## Cara Kerja

Zalo adalah aplikasi perpesanan yang populer di Vietnam. Bot API-nya memungkinkan Gateway menjalankan bot untuk percakapan 1-lawan-1. Secara default bot menggunakan mode **long-polling** (tidak memerlukan URL publik), namun Anda bisa beralih ke mode **webhook** jika memiliki URL HTTPS.

## Kontrol Akses

- **Kebijakan DM**: Defaultnya adalah **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui `openclaw pairing approve zalo <KODE>`.
- **Daftar Izinkan (Allowlist)**: Hanya ID pengguna numerik yang ada di `allowFrom` yang diperbolehkan mengirim pesan ke bot.

## Kemampuan Saluran

- **Pesan Teks**: Dukungan penuh dengan pemotongan otomatis setiap 2000 karakter (batas API Zalo).
- **Gambar**: Mendukung pengiriman dan penerimaan gambar.
- **Stiker**: Pesan stiker dicatat namun tidak diproses oleh agen.
- **Streaming**: Dinonaktifkan karena batas karakter yang kecil membuat fitur streaming kurang optimal.

## Pemecahan Masalah

- **Bot tidak membalas**: Pastikan token bot valid dan pengirim sudah disetujui dalam daftar pairing atau allowlist.
- **Webhook tidak menerima event**: Pastikan URL webhook menggunakan HTTPS, rahasia (*secret*) antara 8-256 karakter, dan pastikan mode polling tidak sedang berjalan (dua mode ini tidak bisa berjalan bersamaan).

Untuk referensi lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



