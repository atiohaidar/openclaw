---
summary: "Status dukungan Matrix, kemampuan, dan konfigurasi"
read_when:
  - Mengatur saluran Matrix atau menggunakan fitur E2EE
title: "Matrix"
---

# Matrix (Plugin)

Matrix adalah protokol perpesanan terbuka dan terdesentralisasi. OpenClaw terhubung sebagai **pengguna** Matrix di homeserver mana pun, jadi Anda memerlukan akun Matrix untuk bot ini.

Status: Didukung melalui plugin. Mendukung pesan langsung (DM), ruang (rooms), utasan (threads), media, reaksi, jajak pendapat (polls), lokasi, dan enkripsi E2EE.

## Instalasi Plugin

Saluran Matrix memerlukan instalasi plugin secara terpisah:

```bash
openclaw plugins install @openclaw/matrix
```

## Penyiapan (Setup)

1. **Instal Plugin Matrix** via CLI.
2. **Buat Akun Matrix**: Gunakan homeserver publik (seperti `matrix.org`) atau host sendiri.
3. **Dapatkan Token Akses**:
   OpenClaw dapat melakukan login otomatis jika Anda memberikan `username` dan `password`, atau Anda bisa memasukkan `accessToken` secara manual di konfigurasi.
4. **Konfigurasi Kredensial**:
   Gunakan file `openclaw.json` untuk mengatur `homeserver`, `accessToken`, dan kebijakan DM.
5. **Mulai Gateway**: Bot akan mulai sinkronisasi dengan Matrix.

## Enkripsi End-to-End (E2EE)

E2EE didukung melalui SDK kripto Rust:
- Aktifkan dengan menyetel `channels.matrix.encryption: true`.
- Pesan di ruang terenkripsi akan didekripsi secara otomatis.
- Pada awal koneksi, Anda perlu melakukan **Verifikasi Perangkat** melalui klien Matrix lain (seperti Element) untuk menyetujui bot.

## Kontrol Akses

### Pesan Langsung (DM)
- Default menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui perintah `openclaw pairing approve matrix <KODE>`.
- Daftar izinkan harus menggunakan ID Matrix lengkap (misal: `@user:server.org`).

### Ruang (Rooms/Grup)
- Default menggunakan mode **allowlist**. Anda harus memasukkan ID Ruang atau alias ke dalam konfigurasi `groups` agar bot merespons.
- Anda dapat mengatur apakah bot membutuhkan `@mention` atau tidak di setiap ruang.

## Kemampuan Saluran

| Fitur            | Status                          |
| :--------------- | :------------------------------ |
| Pesan Langsung   | ✅ Didukung                      |
| Ruang (Groups)   | ✅ Didukung                      |
| Utasan (Threads) | ✅ Didukung                      |
| Media            | ✅ Didukung                      |
| E2EE             | ✅ Didukung (butuh modul kripto) |
| Reaksi           | ✅ Didukung                      |
| Lokasi           | ✅ Didukung                      |

## Pemecahan Masalah

- **Pesan di ruang diabaikan**: Pastikan ID ruang sudah masuk dalam `allowlist` atau `groupPolicy` diatur ke `open`.
- **E2EE gagal**: Pastikan modul kripto terpasang dengan benar. Jalankan `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` jika terjadi error modul.
- **Pesan DM diabaikan**: Cek status pairing dengan `openclaw pairing list matrix`.

Untuk rujukan lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).

