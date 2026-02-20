---
summary: "Referensi CLI untuk `openclaw node` (headless node host)"
read_when:
  - Menjalankan host node tanpa antarmuka (headless)
  - Memasangkan node non-macOS untuk kemampuan system.run
title: "node"
---

# `openclaw node`

Menjalankan **headless node host** yang terhubung ke Gateway OpenClaw dan mengekspos kemampuan `system.run` (eksekusi perintah) pada mesin ini.

## Mengapa menggunakan host node?
Gunakan host node jika Anda ingin agen dapat **menjalankan perintah di komputer lain** dalam jaringan Anda tanpa harus menginstal aplikasi OpenClaw lengkap di sana.

Kegunaan umum:
- Menjalankan perintah pada server Linux/Windows jarak jauh.
- Melakukan eksekusi dalam **sandbox** di gateway, tetapi mendelegasikan perintah yang disetujui ke host lain.
- Menyediakan target eksekusi yang ringan untuk otomatisasi atau node CI.

Eksekusi tetap dijaga oleh **persetujuan eksekusi (exec approvals)** dan daftar izinkan per-agen.

## Jalankan (Latar Depan / Foreground)

```bash
openclaw node run --host <alamat-gateway> --port 18789
```

## Layanan (Latar Belakang / Background)

Menginstal host node sebagai layanan sistem (user service).

```bash
# Menginstal layanan
openclaw node install --host <alamat-gateway> --port 18789

# Mengelola layanan
openclaw node status   # Cek status
openclaw node stop     # Hentikan
openclaw node restart  # Mulai ulang
openclaw node uninstall # Hapus instalasi layanan
```

## Pemasangan (Pairing)
Koneksi pertama akan membuat permintaan pemasangan node di Gateway. Anda harus menyetujuinya di sisi Gateway melalui:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Host node menyimpan ID node, token, dan info koneksi gateway di file `~/.openclaw/node.json`.


