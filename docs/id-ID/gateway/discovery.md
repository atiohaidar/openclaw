---
summary: "Penemuan penemuan node dan transport (Bonjour, Tailscale, SSH) untuk menemukan gateway"
read_when:
  - Mengimplementasikan atau mengubah penemuan/iklan Bonjour
  - Menyesuaikan mode koneksi jarak jauh (langsung vs SSH)
title: "Penemuan dan Transport"
---

# Penemuan & Transport

OpenClaw memiliki dua kebutuhan utama terkait penemuan jaringan:
1. **Kontrol Jarak Jauh Operator**: Aplikasi bilah menu macOS yang mengontrol gateway yang berjalan di tempat lain.
2. **Pairing Node**: Perangkat iOS/Android yang mencari gateway dan melakukan pairing secara aman.

Tujuan desainnya adalah menjaga semua penemuan jaringan di dalam **Node Gateway** (`openclaw gateway`) dan membiarkan klien (app Mac, iOS) sebagai konsumennya.

## Istilah

- **Gateway**: Proses tunggal yang menangani status (sesi, pairing, registri node) dan saluran chat.
- **Gateway WS**: Endpoint WebSocket (default: `127.0.0.1:18789`) untuk kontrol.
- **Transport Langsung (Direct WS)**: Endpoint Gateway yang menghadap LAN/Tailscale (tanpa SSH).
- **Transport SSH**: Kontrol jarak jauh dengan meneruskan port gateway melalui SSH.

## Mengapa Tetap Menggunakan "Langsung" dan SSH?

- **Direct WS** memberikan pengalaman pengguna terbaik di jaringan yang sama atau dalam tailnet:
  - Penemuan otomatis (auto-discovery) di LAN via Bonjour.
  - Token pairing + ACL dikelola oleh gateway.
  - Tidak memerlukan akses shell (SSH).
- **SSH** tetap menjadi cadangan universal:
  - Berfungsi di mana saja selama ada akses SSH (bahkan lintas jaringan).
  - Tetap bekerja saat mDNS/multicast bermasalah.
  - Tidak memerlukan port masuk baru selain SSH.

## Cara Klien Menemukan Gateway

### 1) Bonjour / mDNS (Hanya LAN)
Bonjour digunakan untuk kenyamanan di dalam jaringan lokal yang sama.
- **Gateway** mengiklankan endpoint WS miliknya melalui Bonjour.
- **Klien** menelusuri dan menampilkan daftar "pilih gateway".

### 2) Tailnet (Lintas Jaringan)
Untuk koneksi lintas wilayah (misal: antar kota), Bonjour tidak akan berfungsi. Target "langsung" yang disarankan adalah:
- Nama **Tailscale MagicDNS** (paling disarankan) atau IP Tailscale yang stabil.

### 3) Manual / SSH
Jika tidak ada rute langsung (atau dinonaktifkan), klien selalu bisa terhubung melalui SSH dengan meneruskan (forwarding) port gateway lokal.

## Prioritas Pemilihan Transport
1. Jika endpoint langsung (direct) sudah dikonfigurasi dan dapat dijangkau, gunakan itu.
2. Jika Bonjour menemukan gateway di LAN, tawarkan pilihan ke pengguna.
3. Coba melalui Tailscale jika ada.
4. Gunakan SSH sebagai pilihan terakhir.

## Keamanan
- Gateway adalah penentu utama izin masuk klien.
- Permintaan pairing dibuat dan disetujui di dalam gateway.
- Gateway memberlakukan autentikasi (token) dan pembatasan cakupan (ACL).



