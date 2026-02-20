---
summary: "Konfigurasi dan penyiapan bot chat Twitch"
read_when:
  - Menyiapkan integrasi chat Twitch untuk OpenClaw
title: "Twitch"
---

# Twitch (Plugin)

Dukungan chat Twitch melalui koneksi IRC. OpenClaw terhubung sebagai pengguna Twitch (akun bot) untuk menerima dan mengirim pesan di saluran chat.

## Instalasi Plugin

Saluran Twitch tersedia sebagai plugin terpisah.

```bash
openclaw plugins install @openclaw/twitch
```

## Penyiapan Cepat (Pemula)

1. **Buat Akun Bot**: Disarankan menggunakan akun Twitch terpisah untuk bot Anda.
2. **Dapatkan Token Akses**:
   Buka [Twitch Token Generator](https://twitchtokengenerator.com/), pilih **Bot Token**, dan pastikan izin (*scopes*) `chat:read` serta `chat:write` sudah dicentang.
3. **Dapatkan User ID Anda**: Konversi nama pengguna (username) Twitch Anda menjadi ID angka melalui alat pencarian online (seperti StreamWeasels).
4. **Konfigurasi OpenClaw**: Tambahkan kredensial ke file `openclaw.json`.
5. **Mulai Gateway**: Bot akan otomatis bergabung ke saluran yang Anda tentukan.

Contoh konfigurasi minimal:
```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "nama_bot_anda",
      accessToken: "oauth:abc123...",
      clientId: "id_klien_anda",
      channel: "nama_channel_tujuan",
      allowFrom: ["123456789"], // Sangat Disarankan: Hanya ID Anda yang bisa memicu bot
    },
  },
}
```

## Kontrol Akses

- **Daftar Izinkan (Allowlist)**: Gunakan ID pengguna (angka) untuk keamanan maksimal, karena nama pengguna bisa berubah.
- **Peran (Roles)**: Anda bisa membatasi penggunaan bot hanya untuk moderator, VIP, atau pelanggan (*subscriber*) menggunakan parameter `allowedRoles`.
- **Sebutan (Mention)**: Secara default, bot hanya merespons jika di-@mention di dalam chat.

## Penyegaran Token (Refresh Token)

Token dari Twitch Token Generator biasanya kadaluwarsa dalam beberapa jam. Untuk bot yang berjalan terus-menerus, disarankan menggunakan aplikasi Twitch Anda sendiri melalui [Twitch Developer Console](https://dev.twitch.tv/console) agar bot dapat melakukan penyegaran token otomatis (refresh token).

## Kemampuan Saluran

- **Batas Karakter**: Pesan dibatasi maksimal 500 characters dan akan dipotong otomatis pada batas kata jika melebihi.
- **Dukungan Markdown**: Pemformatan Markdown akan dihapus sebelum pesan dikirim.
- **Aksi Agen**: Agen dapat mengirim pesan ke saluran mana pun yang ia ikuti menggunakan perintah `send`.

## Pemecahan Masalah

- **Bot tidak merespons**: Pastikan ID pengguna Anda sudah masuk dalam `allowFrom` atau hapus batasan tersebut sementara untuk pengujian.
- **Gagal autentikasi**: Periksa apakah token akses diawali dengan `oauth:` dan pastikan izin chat sudah benar.
- **Error pemuatan plugin**: Pastikan plugin `@openclaw/twitch` sudah terpasang dengan benar.

Untuk referensi lengkap, lihat [Konfigurasi Gateway](/id-ID/gateway/configuration).



