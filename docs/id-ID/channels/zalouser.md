---
summary: "Dukungan akun pribadi Zalo melalui zca-cli (Login QR), kemampuan, dan konfigurasi"
read_when:
  - Menyiapkan Zalo Personal untuk OpenClaw
  - Mendebug aliran pesan atau login Zalo Personal
title: "Zalo Personal"
---

# Zalo Personal (Tidak Resmi)

Status: Eksperimental. Integrasi ini mengotomatiskan **akun Zalo pribadi** menggunakan `zca-cli`.

> **Peringatan:** Ini adalah integrasi tidak resmi dan dapat mengakibatkan penangguhan atau pemblokiran (*banned*) akun. Gunakan dengan risiko Anda sendiri.

## Instalasi Plugin

Saluran Zalo Personal tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/zalouser
```

## Prasyarat: zca-cli

Mesin Gateway harus memiliki biner `zca` yang tersedia di dalam `PATH`. Pastikan Anda dapat menjalankan perintah `zca --version` di terminal.

## Penyiapan Cepat (Quick Setup)

1. **Instal Plugin** Zalo Personal.
2. **Login (QR)**: Jalankan perintah ini di mesin gateway dan pindai kode QR yang muncul menggunakan aplikasi Zalo di ponsel Anda.
   ```bash
   openclaw channels login --channel zalouser
   ```
3. **Konfigurasi OpenClaw**: Aktifkan saluran di file `openclaw.json`.
4. **Mulai Gateway**: Bot akan mulai mendengarkan pesan dari akun Zalo Anda.

## Kontrol Akses

- **Kebijakan DM**: Defaultnya adalah **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui `openclaw pairing approve zalouser <KODE>`.
- **Akses Grup**: Secara default diperbolehkan (`open`). Anda dapat membatasinya menjadi `allowlist` untuk menentukan grup spesifik mana saja yang bisa diakses oleh bot.

## Kemampuan Saluran

- **Pesan Teks**: Mendukung pengiriman teks dengan pemotongan otomatis setiap ~2000 karakter.
- **Pencarian Direktori**: Anda dapat menggunakan perintah CLI `openclaw directory` untuk mencari ID teman atau grup.
- **Profil Ganda**: Mendukung penggunaan lebih dari satu akun Zalo (menggunakan profil zca yang berbeda).

## Pemecahan Masalah

- **`zca` tidak ditemukan**: Pastikan `zca-cli` sudah terpasang dan berada dalam `PATH` sistem Anda.
- **Login tidak bertahan**: Gunakan perintah `openclaw channels logout --channel zalouser` kemudian login kembali jika sesi terputus.

Untuk rujukan lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



