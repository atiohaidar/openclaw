---
summary: "Referensi CLI untuk `openclaw directory` (diri sendiri, kontak, grup)"
read_when:
  - Anda ingin mencari ID kontak/grup untuk suatu saluran
  - Anda sedang mengembangkan adapter direktori saluran
title: "directory"
---

# `openclaw directory`

Pencarian direktori untuk saluran yang mendukungnya (kontak, grup, dan profil "diri sendiri").

## Flag Umum
- `--channel <nama>`: ID/alias saluran (wajib jika ada banyak saluran).
- `--account <id>`: ID akun.
- `--json`: Output formal JSON.

## Catatan
- Perintah ini membantu Anda menemukan ID untuk digunakan di perintah lain (terutama `openclaw message send --target ...`).
- Hasil pencarian seringkali berasal dari konfigurasi (allowlist) atau grup yang sudah terdaftar.

## Contoh Penggunaan

### Mencari ID pengguna di Slack
```bash
openclaw directory peers list --channel slack --query "Budi"
```

### Mengambil informasi diri sendiri
```bash
openclaw directory self --channel telegram
```

### Menampilkan daftar grup
```bash
openclaw directory groups list --channel whatsapp
```

### Menampilkan anggota grup
```bash
openclaw directory groups members --channel whatsapp --group-id <id-grup>
```

**ID Format (per Saluran):**
- WhatsApp: `+628...` (DM), `...@g.us` (grup).
- Telegram: `@username` atau ID numerik.
- Slack/Discord: `user:<id>` atau `channel:<id>`.
- MS Teams: `user:<id>` atau `conversation:<id>`.



