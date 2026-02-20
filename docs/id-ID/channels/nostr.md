---
summary: "Saluran Nostr DM melalui pesan terenkripsi NIP-04"
read_when:
  - Anda ingin OpenClaw menerima DM melalui Nostr
  - Anda sedang menyiapkan perpesanan terdesentralisasi
title: "Nostr"
---

# Nostr

**Status:** Plugin opsional (dinonaktifkan secara default).

Nostr adalah protokol terdesentralisasi untuk jejaring sosial. Saluran ini memungkinkan OpenClaw untuk menerima dan menanggapi pesan langsung (DM) terenkripsi melalui NIP-04.

## Instalasi (Sesuai Permintaan)

### Melalui Onboarding (Direkomendasikan)
Gunakan instruksi di bawah ini untuk memasang plugin Nostr:
```bash
openclaw onboard
# atau
openclaw channels add
```
Pilih **Nostr** dari daftar saluran opsional.

### Instalasi Manual
```bash
openclaw plugins install @openclaw/nostr
```

## Penyiapan Cepat (Quick Setup)

1. **Buat Pasangan Kunci Nostr** (jika belum punya):
   Anda bisa menggunakan alat seperti `nak` untuk menghasilkan kunci baru.
2. **Tambahkan ke Konfigurasi**:
   Masukkan kunci pribadi (*private key*) Anda ke dalam file `openclaw.json`. Disarankan menggunakan variabel lingkungan.
3. **Ekspor Kunci**:
   ```bash
   export NOSTR_PRIVATE_KEY="nsec1..."
   ```
4. **Mulai Gateway**: Jalankan gateway dan bot akan mulai terhubung ke relai (*relays*) Nostr.

## Referensi Konfigurasi

| Kunci        | Tipe     | Default      | Deskripsi                                  |
| :----------- | :------- | :----------- | :----------------------------------------- |
| `privateKey` | string   | wajib        | Kunci pribadi dalam format `nsec` atau hex |
| `relays`     | string[] | [damus, lol] | URL relai (WebSocket)                      |
| `dmPolicy`   | string   | `pairing`    | Kebijakan akses pesan langsung             |
| `profile`    | object   | -            | Metadata profil NIP-01                     |

## Kontrol Akses

- **Kebijakan DM**: Defaultnya adalah **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui perintah `openclaw pairing approve nostr <KODE>`.
- **Daftar Izinkan (Allowlist)**: Hanya pubkey yang terdaftar di `allowFrom` yang dapat mengirim pesan ke bot.

## Relai (Relays)

Secara default, bot terhubung ke `relay.damus.io` dan `nos.lol`. Disarankan menggunakan 2-3 relai untuk redundansi.

## Pemecahan Masalah

- **Pesan tidak masuk**: Pastikan kunci pribadi valid dan URL relai dapat diakses. Gunakan `wss://` untuk koneksi aman.
- **Bot tidak membalas**: Periksa apakah relai mengizinkan penulisan (*writes*) dan pastikan konektivitas keluar tidak terhalang.
- **Balasan ganda**: Hal ini wajar jika menggunakan banyak relai; OpenClaw melakukan deduplikasi berdasarkan ID event.

## Keamanan

- **Jangan pernah membagikan kunci pribadi Anda**.
- Gunakan variabel lingkungan untuk menyimpan kredensial.
- Gunakan kebijakan **allowlist** untuk bot yang digunakan di lingkungan produksi.



