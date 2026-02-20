---
title: IRC
description: Hubungkan OpenClaw ke saluran IRC dan pesan langsung.
---

Gunakan IRC jika Anda ingin OpenClaw berada dalam saluran klasik (`#room`) dan pesan langsung (DM). IRC tersedia sebagai plugin ekstensi, namun dikonfigurasi melalui konfigurasi utama di `channels.irc`.

## Mulai Cepat (Quick Start)

1. Aktifkan konfigurasi IRC di `~/.openclaw/openclaw.json`.
2. Setel minimal parameter berikut:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Mulai atau muat ulang gateway: `openclaw gateway run`.

## Keamanan Standar

- **Kebijakan DM**: Defaultnya adalah `pairing`.
- **Kebijakan Grup**: Defaultnya adalah `allowlist`.
- **Enkripsi**: Sangat disarankan untuk selalu menggunakan `tls: true`.

## Kontrol Akses

Terdapat dua tingkat akses untuk saluran IRC:
1. **Akses Saluran** (`groupPolicy` + `groups`): Menentukan apakah bot diizinkan menerima pesan dari saluran tersebut.
2. **Akses Pengirim** (`groupAllowFrom`): Menentukan siapa saja yang diperbolehkan memanggil bot di dalam saluran.

Anda dapat menggunakan nama panggilan (*nick*) saja atau format lengkap `nick!user@host` untuk identitas yang lebih kuat.

## Pemicu Balasan (Sebutan/Mention)

Meskipun pengirim dan saluran sudah diizinkan, bot secara default hanya akan membalas di saluran grup jika ia di-mention (@mention). Jika Anda ingin bot membalas pada **setiap** pesan tanpa perlu di-mention, setel `requireMention: false` pada konfigurasi saluran tersebut.

## Keamanan Saluran Publik

Jika Anda mengizinkan semua orang (`allowFrom: ["*"]`) di saluran publik, disarankan untuk membatasi alat yang bisa digunakan oleh pengguna biasa. Anda dapat menggunakan `toolsBySender` untuk memberikan akses alat yang berbeda antara pemilik bot dan pengguna umum.

## NickServ

Untuk melakukan identifikasi otomatis dengan NickServ setelah terhubung:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "password-nickserv-anda"
      }
    }
  }
}
```

## Pemecahan Masalah

- **Bot terhubung tapi tidak membalas**: Periksa apakah ada pesan `missing-mention` di log. Pastikan bot di-mention atau matikan `requireMention`.
- **Gagal login**: Pastikan nama panggilan (*nick*) tidak sedang digunakan dan kata sandi server sudah benar.
- **Error TLS**: Verifikasi kombinasi host dan port, serta pastikan server tersebut mendukung koneksi terenkripsi.


