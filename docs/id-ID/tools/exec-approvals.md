---
summary: "Persetujuan eksekusi, daftar izinkan, dan prompt pelarian sandbox"
read_when:
  - Mengonfigurasi persetujuan eksekusi atau daftar izinkan
  - Mengimplementasikan UX persetujuan eksekusi di aplikasi macOS
  - Meninjau prompt pelarian sandbox dan implikasinya
title: "Persetujuan Eksekusi"
---

# Persetujuan Eksekusi (Exec Approvals)

Persetujuan eksekusi (exec approvals) adalah **pembatas keamanan (guardrail) aplikasi pendamping / host node** untuk mengizinkan agen yang berada dalam sandbox menjalankan perintah pada host nyata (`gateway` atau `node`). Anggap ini sebagai interlock keamanan: perintah hanya diizinkan jika kebijakan + daftar izinkan + persetujuan pengguna (opsional) semuanya sepakat.

Persetujuan eksekusi adalah **tambahan** dari kebijakan alat dan penyaringan elevasi (kecuali elevasi disetel ke `full`, yang akan melewati persetujuan).

## Di Mana Ini Berlaku

Persetujuan eksekusi ditegakkan secara lokal pada host eksekusi:
- **host gateway** → proses `openclaw` pada mesin gateway.
- **host node** → runner node (aplikasi pendamping macOS atau host node headless).

## Pengaturan Kebijakan

### Keamanan (`exec.security`)
- **deny**: blokir semua permintaan eksekusi host.
- **allowlist**: hanya izinkan perintah yang terdaftar dalam daftar izinkan (allowlist).
- **full**: izinkan semuanya (setara dengan mode elevasi).

### Tanya (`exec.ask`)
- **off**: jangan pernah memunculkan prompt.
- **on-miss**: munculkan prompt hanya jika perintah tidak cocok dengan daftar izinkan.
- **always**: munculkan prompt untuk setiap perintah.

### Fallback Tanya (`askFallback`)
Jika prompt diperlukan tetapi UI tidak terjangkau:
- **deny**: blokir.
- **allowlist**: izinkan hanya jika cocok dengan daftar izinkan.
- **full**: izinkan.

## Daftar Izinkan (Per Agen)

Daftar izinkan (allowlist) bersifat **per agen**. Pola pencocokan menggunakan glob yang **tidak sensitif huruf besar/kecil**. Pola harus mengarah ke **jalur biner** lengkap.

Contoh:
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

Setiap entri dalam daftar izinkan mencatat kapan terakhir kali digunakan dan perintah apa yang dijalankan.

## Bin Aman (Hanya stdin-only)

`tools.exec.safeBins` mendefinisikan daftar biner **khusus stdin** (misalnya `jq`) yang dapat berjalan dalam mode allowlist **tanpa** entri daftar izinkan eksplisit. Bin aman menolak argumen file posisional dan hanya beroperasi pada aliran data (stream) yang masuk.

Bin aman default: `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`.

## Alur Persetujuan

Ketika prompt diperlukan, gateway akan menyiarkan `exec.approval.requested` ke klien operator. UI Kontrol atau aplikasi macOS akan menyelesaikannya. Dialog konfirmasi mencakup perintah, argumen, direktori kerja (cwd), dan ID agen.

Tindakan yang tersedia:
- **Izinkan sekali (Allow once)** → jalankan sekarang.
- **Selalu izinkan (Always allow)** → tambahkan ke daftar izinkan + jalankan.
- **Tolak (Deny)** → blokir eksekusi.

## Meneruskan Persetujuan ke Saluran Chat

Anda dapat meneruskan prompt persetujuan eksekusi ke saluran chat apa pun (termasuk saluran plugin) dan menyetujuinya dengan perintah `/approve`.

Balas di chat dengan:
- `/approve <id> allow-once`
- `/approve <id> allow-always`
- `/approve <id> deny`

## Implikasi

- Mode **full** sangat kuat; prioritaskan penggunaan daftar izinkan (allowlist) jika memungkinkan.
- Daftar izinkan per agen mencegah persetujuan satu agen "bocor" ke agen lainnya.
- Persetujuan hanya berlaku untuk permintaan eksekusi host dari **pengirim resmi**. Pengirim yang tidak sah tidak dapat menggunakan `/exec`.


