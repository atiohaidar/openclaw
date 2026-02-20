---
summary: "Login manual untuk otomasi browser + posting X/Twitter"
read_when:
  - Anda perlu login ke situs untuk otomasi browser
  - Anda ingin memposting pembaruan ke X/Twitter
title: "Login Browser"
---

# Login Browser + Posting X/Twitter

## Login Manual (Direkomendasikan)

Jika sebuah situs memerlukan login, **masuklah secara manual** di profil browser **host** (browser yang dikelola openclaw).

**Jangan** berikan kredensial Anda kepada model. Login otomatis sering kali memicu sistem pertahanan anti-bot dan dapat menyebabkan akun terkunci.

Dokumentasi utama browser: [Browser](/id-ID/tools/browser).

## Profil Chrome Mana yang Digunakan?

OpenClaw mengontrol **profil Chrome khusus** (bernama `openclaw`, dengan UI berwarna oranye). Profil ini terpisah dari profil browser harian Anda.

Dua cara mudah untuk mengaksesnya:

1. **Minta agen untuk membuka browser**, lalu masuklah (login) secara manual.
2. **Buka melalui CLI**:
   ```bash
   openclaw browser start
   openclaw browser open https://x.com
   ```

Jika Anda memiliki beberapa profil, gunakan flag `--browser-profile <nama>` (defaultnya adalah `openclaw`).

## X/Twitter: Alur yang Direkomendasikan

- **Membaca/Mencari/Thread:** gunakan browser **host** (login manual).
- **Posting Pembaruan:** gunakan browser **host** (login manual).

## Sandboxing + Akses Browser Host

Sesi browser di dalam sandbox **lebih mungkin** memicu deteksi bot. Untuk X/Twitter (dan situs ketat lainnya), lebih baik gunakan browser **host**.

Jika agen berada di dalam sandbox, alat browser secara default akan menggunakan sandbox tersebut. Untuk mengizinkan kontrol host:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Kemudian targetkan browser hostnya:
```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Atau nonaktifkan sandboxing untuk agen yang bertugas memposting pembaruan.


