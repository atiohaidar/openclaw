---
summary: "Referensi CLI untuk `openclaw plugins` (daftar, instal, hapus, aktifkan/nonaktifkan)"
read_when:
  - Anda ingin menginstal atau mengelola plugin Gateway
  - Anda ingin mendebug kegagalan pembuatan plugin
title: "plugins"
---

# `openclaw plugins`

Mengelola plugin dan ekstensi Gateway yang dimuat ke dalam proses server.

## Terkait:
- Sistem Plugin: [Plugins](/id-ID/tools/plugin)
- Manifest Plugin: [Plugin manifest](/id-ID/plugins/manifest)

## Perintah-perintah

```bash
# Menampilkan daftar semua plugin
openclaw plugins list

# Menampilkan informasi detail plugin tertentu
openclaw plugins info <id>

# Mengaktifkan plugin yang sudah diinstal
openclaw plugins enable <id>

# Menonaktifkan plugin tanpa menghapusnya
openclaw plugins disable <id>

# Menghapus instalasi plugin
openclaw plugins uninstall <id>

# Memperbarui plugin (khusus instalasi npm)
openclaw plugins update --all
```

## Menginstal Plugin Baru

Plugin dapat diinstal dari folder lokal, file arsip, atau paket npm:

```bash
# Instal dari folder lokal (copy)
openclaw plugins install ./folder-plugin

# Instal dari folder lokal (link/tautan - berguna untuk pengembangan)
openclaw plugins install --link ./folder-plugin

# Instal dari npm
openclaw plugins install nama-paket-npm
```

**Catatan Keamanan:** Anggap instalasi plugin sama seperti menjalankan kode asing. Gunakan versi yang disematkan (`--pin`) jika memungkinkan untuk menghindari perubahan tak terduga dalam pembaruan otomatis.



