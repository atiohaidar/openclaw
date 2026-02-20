---
summary: "Dukungan Signal melalui signal-cli (JSON-RPC + SSE), jalur penyiapan, dan model penomoran"
read_when:
  - Mengatur dukungan Signal
  - Mendebug pengiriman/penerimaan pesan Signal
title: "Signal"
---

# Signal (signal-cli)

Status: Integrasi CLI eksternal. Gateway berkomunikasi dengan `signal-cli` melalui HTTP JSON-RPC + SSE.

## Penyiapan Cepat (Pemula)

1. **Gunakan Nomor Terpisah**: Sangat disarankan menggunakan nomor telepon khusus untuk bot.
2. **Instal signal-cli**: Pastikan JRE (Java) sudah terpasang jika menggunakan build JVM.
3. **Pilih Jalur Penyiapan**:
   - **Jalur A (Tautan QR)**: Jalankan `signal-cli link -n "OpenClaw"` dan pindai kode QR menggunakan aplikasi Signal di ponsel Anda.
   - **Jalur B (Registrasi SMS)**: Daftarkan nomor khusus menggunakan verifikasi SMS dan captcha.
4. **Konfigurasi OpenClaw**: Masukkan nomor akun dan path `signal-cli` ke `openclaw.json`.
5. **Setujui Pairing**: Gunakan kode pairing untuk menyetujui pesan masuk pertama.

Contoh konfigurasi minimal:
```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+6281234567890",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
    },
  },
}
```

## Model Penomoran (Penting)

Jika Anda menjalankan bot pada **akun Signal pribadi Anda**, bot akan mengabaikan pesan dari Anda sendiri untuk mencegah loop (balasan berulang). Gunakan **nomor telepon bot yang terpisah** jika Anda ingin bisa mengobrol dengan bot tersebut dari akun pribadi Anda.

## Kontrol Akses

### Pesan Langsung (DM)
- Default menggunakan mode **pairing**. Pengirim baru akan menerima kode unik yang harus disetujui admin melalui `openclaw pairing approve signal <KODE>`.
- Daftar izinkan (*allowlist*) dapat menggunakan nomor telepon (E.164) atau format `uuid:<id>`.

### Grup
- Mendukung fitur sebutan (@mention) secara manual (regex).
- Kebijakan grup dapat diatur ke `open` (semua anggota), `allowlist` (hanya orang tertentu), atau `disabled`.

## Kemampuan Saluran

- **Indikator Mengetik**: Bot akan mengirimkan sinyal "sedang mengetik" saat sedang memproses balasan.
- **Laporan Dibaca (Read Receipts)**: Mendukung pengiriman konfirmasi pesan telah dibaca.
- **Media**: Mendukung lampiran gambar dan file (batas default 8 MB).
- **Reaksi**: Agen dapat memberikan reaksi emoji pada pesan.

## Pemecahan Masalah

- **Pesan tidak masuk**: Pastikan daemon `signal-cli` berjalan dan nomor telepon akun sudah diverifikasi.
- **Pesan DM diabaikan**: Cek status pairing dengan `openclaw pairing list signal`.
- **Gagal mengirim lampiran**: Periksa apakah `signal-cli` memiliki izin untuk membaca file atau tingkatkan batas `mediaMaxMb`.

Untuk rujukan lengkap dan detail CLI `signal-cli`, lihat [Dokumentasi signal-cli](https://github.com/AsamK/signal-cli).



