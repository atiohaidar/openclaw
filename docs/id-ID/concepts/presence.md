---
summary: "Bagaimana entri kehadiran (presence) OpenClaw dihasilkan, digabungkan, dan ditampilkan"
read_when:
  - Mendebug tab Instances (Instansi)
  - Menyelidiki baris instansi yang duplikat atau basi
  - Mengubah suar (beacon) koneksi WS gateway atau event sistem
title: "Kehadiran (Presence)"
---

# Kehadiran (Presence)

"Kehadiran" atau "Presence" di OpenClaw adalah tampilan ringan dan bersifat *best-effort* (upaya terbaik) dari:

- **Gateway** itu sendiri, dan
- **Klien yang terhubung ke Gateway** (aplikasi macOS, WebChat, CLI, dll.)

Kehadiran digunakan terutama untuk merender tab **Instances** pada aplikasi macOS dan untuk memberikan visibilitas cepat bagi operator.

## Bidang Kehadiran (Apa yang muncul)

Entri kehadiran adalah objek terstruktur dengan bidang seperti:

- `instanceId` (opsional tetapi sangat direkomendasikan): identitas klien yang stabil (biasanya `connect.client.instanceId`).
- `host`: nama host yang mudah dibaca manusia.
- `ip`: alamat IP *best-effort*.
- `version`: string versi klien.
- `deviceFamily` / `modelIdentifier`: informasi perangkat keras.
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: "detik sejak input pengguna terakhir" (jika diketahui).
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: stempel waktu pembaruan terakhir (milidetik sejak epoch).

## Produsen (Dari mana kehadiran berasal)

Entri kehadiran dihasilkan oleh beberapa sumber dan **digabungkan**.

### 1) Entri Mandiri Gateway (Self)

Gateway selalu menanamkan entri "self" saat startup sehingga UI akan menampilkan host gateway bahkan sebelum ada klien yang terhubung.

### 2) Koneksi WebSocket

Setiap klien WS dimulai dengan permintaan `connect`. Setelah jabat tangan (handshake) berhasil, Gateway akan menambahkan atau memperbarui (upsert) entri kehadiran untuk koneksi tersebut.

#### Mengapa perintah CLI sekali jalan tidak muncul

CLI sering terhubung untuk perintah yang singkat dan sekali jalan. Untuk menghindari "spam" pada daftar Instansi, `client.mode === "cli"` **tidak** diubah menjadi entri kehadiran.

### 3) Suar (Beacon) `system-event`

Klien dapat mengirim suar periodik yang lebih kaya melalui metode `system-event`. Aplikasi macOS menggunakan ini untuk melaporkan nama host, IP, dan `lastInputSeconds`.

### 4) Koneksi Node (role: node)

Ketika sebuah node terhubung melalui WebSocket Gateway dengan `role: node`, Gateway akan menambahkan atau memperbarui entri kehadiran untuk node tersebut (alur yang sama dengan klien WS lainnya).

## Aturan Penggabungan + Dedup (Mengapa `instanceId` penting)

Entri kehadiran disimpan dalam satu peta (map) di dalam memori:

- Entri diidentifikasi dengan **presence key**.
- Kunci terbaik adalah `instanceId` yang stabil (dari `connect.client.instanceId`) yang tetap ada bahkan setelah restart.
- Kunci bersifat tidak peka huruf besar-kecil (case-insensitive).

Jika klien terhubung kembali tanpa `instanceId` yang stabil, ia mungkin muncul sebagai baris **duplikat**.

## TTL dan Batas Ukuran

Kehadiran sengaja dibuat bersifat fana (ephemeral):

- **TTL:** entri yang lebih lama dari 5 menit akan dihapus (pruned).
- **Entri Maksimal:** 200 (yang tertua akan dihapus terlebih dahulu).

Hal ini menjaga daftar tetap segar dan menghindari penggunaan memori yang tidak terbatas.

## Catatan Remote/Tunnel (IP Loopback)

Ketika klien terhubung melalui terowongan (tunnel) SSH / penerusan port lokal, Gateway mungkin melihat alamat remote sebagai `127.0.0.1`. Untuk menghindari penindihan alamat IP yang dilaporkan klien yang sudah benar, alamat remote loopback akan diabaikan.

## Konsumen

### Tab Instances macOS

Aplikasi macOS merender output dari `system-presence` dan menerapkan indikator status kecil (Active/Idle/Stale) berdasarkan usia pembaruan terakhir.

## Tip Debugging

- Untuk melihat daftar mentah, panggil `system-presence` ke arah Gateway.
- Jika Anda melihat duplikat:
  - Pastikan klien mengirimkan `client.instanceId` yang stabil saat jabat tangan.
  - Pastikan suar periodik menggunakan `instanceId` yang sama.
  - Periksa apakah entri dari koneksi kehilangan `instanceId` (duplikasi memang diharapkan dalam kasus ini).
