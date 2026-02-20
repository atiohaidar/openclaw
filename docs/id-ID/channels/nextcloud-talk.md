---
summary: "Status dukungan Nextcloud Talk, kemampuan, dan konfigurasi"
read_when:
  - Mengonfigurasi saluran Nextcloud Talk
title: "Nextcloud Talk"
---

# Nextcloud Talk (Plugin)

Status: Didukung melalui plugin (bot webhook). Mendukung pesan langsung (DM), ruang (rooms), reaksi, dan pesan markdown.

## Instalasi Plugin

Saluran Nextcloud Talk tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

## Penyiapan Cepat (Pemula)

1. **Instal Plugin** Nextcloud Talk.
2. **Buat Bot di Server Nextcloud**:
   Jalankan perintah `occ` berikut di server Nextcloud Anda:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. **Aktifkan Bot**: Nyalakan bot di dalam pengaturan ruang (room) yang diinginkan.
4. **Konfigurasi OpenClaw**: Tambahkan `baseUrl` dan `botSecret` ke dalam `openclaw.json`.
5. **Mulai Gateway**: Bot akan mulai mendengarkan event dari Nextcloud.

## Kontrol Akses

- **Pesan Langsung (DM)**: Secara default menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui perintah `openclaw pairing approve nextcloud-talk <KODE>`.
- **Daftar Izinkan (Allowlist)**: Gunakan ID pengguna Nextcloud asli untuk `allowFrom`. Nama tampilan (display name) akan diabaikan.

## Kemampuan Saluran

| Fitur            | Status           |
| :--------------- | :--------------- |
| Pesan Langsung   | ✅ Didukung       |
| Ruang (Groups)   | ✅ Didukung       |
| Utasan (Threads) | ❌ Tidak didukung |
| Media            | Hanya URL        |
| Reaksi           | ✅ Didukung       |

## Catatan Penting

- **Inisiasi Pesan**: Bot tidak dapat memulai DM; pengguna harus mengirim pesan ke bot terlebih dahulu.
- **Media**: Pengunggahan file langsung belum didukung oleh API bot; media dikirim sebagai tautan URL.
- **Deteksi DM**: Untuk membedakan DM dan Ruang secara akurat, disarankan menyetel `apiUser` dan `apiPassword` di konfigurasi.

Untuk referensi lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



