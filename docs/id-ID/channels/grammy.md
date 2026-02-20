---
summary: "Integrasi Telegram Bot API melalui grammY dengan catatan penyiapan"
read_when:
  - Bekerja pada jalur Telegram atau grammY
title: grammY
---

# Integrasi grammY (Telegram Bot API)

## Mengapa Menggunakan grammY?

- Klien Bot API berbasis TypeScript dengan bantuan *long-poll* dan *webhook* bawaan, serta penanganan kesalahan dan pembatas kecepatan (*rate limiter*).
- Penanganan media yang lebih bersih; mendukung semua metode Bot API.
- Dapat diperluas: mendukung proxy kustom dan middleware sesi.

## Apa yang Tersedia?

- **Satu Jalur Klien**: grammY sekarang menjadi satu-satunya klien Telegram (pengiriman + gateway) dengan fitur pembatas kecepatan grammY yang diaktifkan secara default.
- **Gateway**: Membangun bot grammY, menghubungkan penahan sebutan (*mention gating*) dan daftar izinkan (*allowlist*), serta mengunduh media. Mendukung mode *long-poll* atau *webhook*.
- **Proxy**: Dukungan opsional untuk proxy melalui `undici.ProxyAgent`.
- **Dukungan Webhook**: Gateway mengaktifkan mode webhook jika `webhookUrl` dan `webhookSecret` dikonfigurasi; jika tidak, ia akan menggunakan mode *long-poll*.
- **Sesi**: Chat langsung digabungkan ke sesi utama agen; grup menggunakan sesi terisolasi per ID chat.
- **Konfigurasi**: Mendukung `botToken`, `dmPolicy`, `groups`, `mediaMaxMb`, `proxy`, dan berbagai pengaturan webhook lainnya.
- **Pratinjau Live Stream**: Fitur opsional `streamMode` mengirimkan pesan sementara dan memperbaruinya secara berkala menggunakan `editMessageText`.

## Hal-hal Lain

- Penggunaan plugin tambahan grammY jika terkena limit Bot API (error 429).
- Penambahan tes terstruktur untuk stiker dan pesan suara.
- Jalur port webhook yang dapat dikonfigurasi.



