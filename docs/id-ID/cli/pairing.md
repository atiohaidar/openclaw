---
summary: "Referensi CLI untuk `openclaw pairing` (menyetujui/menampilkan daftar permintaan pemasangan)"
read_when:
  - Anda menggunakan mode pemasangan (pairing-mode) untuk DM dan perlu menyetujui pengirim
title: "pairing"
---

# `openclaw pairing`

Menyetujui atau memeriksa permintaan pemasangan (pairing) DM untuk saluran yang mendukung fitur ini.

## Terkait:
- Alur Pemasangan: [Pairing](/id-ID/channels/pairing)

## Perintah-perintah

```bash
# Menampilkan daftar permintaan pemasangan (misal: WhatsApp)
openclaw pairing list whatsapp

# Menyetujui permintaan pemasangan dengan kode tertentu
openclaw pairing approve whatsapp <kode> --notify
```



