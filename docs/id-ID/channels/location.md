---
summary: "Parsing lokasi saluran masuk (Telegram + WhatsApp) dan kolom konteks"
read_when:
  - Menambah atau mengubah parsing lokasi saluran
  - Menggunakan kolom konteks lokasi dalam prompt agen atau alat
title: "Parsing Lokasi Saluran"
---

# Parsing Lokasi Saluran

OpenClaw menormalisasi lokasi yang dibagikan dari berbagai saluran percakapan menjadi:

- Teks yang mudah dibaca manusia yang ditambahkan ke badan pesan masuk, dan
- Kolom terstruktur dalam payload konteks balasan otomatis.

Saluran yang saat ini didukung:
- **Telegram** (pin lokasi + tempat/venues + lokasi langsung)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` dengan `geo_uri`)

## Pemformatan Teks

Lokasi akan ditampilkan secara ramah tanpa tanda kurung:

- **Pin Lokasi**: `📍 48.858844, 2.294351 ±12m`
- **Tempat Bernama**: `📍 Menara Eiffel — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- **Lokasi Langsung**: `🛰 Lokasi langsung: 48.858844, 2.294351 ±12m`

Jika pesan menyertakan keterangan (caption) atau komentar, teks tersebut akan ditambahkan pada baris berikutnya.

## Kolom Konteks (Context Fields)

Ketika lokasi tersedia, kolom berikut akan ditambahkan ke objek `ctx`:

- `LocationLat` (garis lintang/latitude)
- `LocationLon` (garis bujur/longitude)
- `LocationAccuracy` (akurasi dalam meter; opsional)
- `LocationName` (nama lokasi; opsional)
- `LocationAddress` (alamat lokasi; opsional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean; apakah lokasi tersebut *live*)

## Catatan Saluran

- **Telegram**: Tempat (*venues*) akan dipetakan ke `LocationName` dan `LocationAddress`; lokasi langsung menggunakan `live_period`.
- **WhatsApp**: Komentar dari pesan lokasi atau keterangan dari lokasi langsung akan ditambahkan sebagai baris teks keterangan.
- **Matrix**: `geo_uri` akan diproses sebagai pin lokasi; ketinggian (altitude) diabaikan dan `LocationIsLive` akan selalu bernilai *false*.



