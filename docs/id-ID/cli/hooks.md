---
summary: "Referensi CLI untuk `openclaw hooks` (hook agen)"
read_when:
  - Anda ingin mengelola hook agen
  - Anda ingin menginstal atau memperbarui hook
title: "hooks"
---

# `openclaw hooks`

Mengelola hook agen (otomatisasi berbasis kejadian untuk perintah seperti `/new`, `/reset`, dan awal mula gateway).

## Terkait:
- Hook: [Hooks](/id-ID/automation/hooks)
- Hook Plugin: [Plugins](/id-ID/tools/plugin#plugin-hooks)

## Menampilkan Semua Hook

```bash
openclaw hooks list
```

Menampilkan semua hook yang ditemukan dari direktori ruang kerja, kelolaan, dan bawaan.

**Opsi:**
- `--eligible`: Hanya tampilkan hook yang memenuhi syarat (persyaratan terpenuhi).
- `--json`: Output dalam format JSON.
- `-v, --verbose`: Tampilkan informasi detail termasuk persyaratan yang kurang.

## Informasi Hook Spesifik

```bash
openclaw hooks info <nama>
```

Menampilkan informasi detail tentang hook tertentu (misal: `session-memory`).

## Mengaktifkan Hook

```bash
openclaw hooks enable <nama>
```

Mengaktifkan hook tertentu dengan menambahkannya ke konfigurasi Anda.

**Catatan:** Hook yang dikelola oleh plugin tidak dapat diaktifkan/dinonaktifkan di sini. Anda harus mengaktifkan/menonaktifkan plugin yang bersangkutan.

## Menonaktifkan Hook

```bash
openclaw hooks disable <nama>
```

## Menginstal Hook Baru

```bash
openclaw hooks install <jalur-atau-spesifikasi>
openclaw hooks install <nama-npm> --pin
```

Menginstal paket hook dari folder lokal, arsip, atau npm.

**Opsi:**
- `-l, --link`: Membuat tautan (link) ke direktori lokal alih-alih menyalinnya.
- `--pin`: Mencatat instalasi npm dengan versi yang tepat.

## Memperbarui Hook

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

## Hook Bawaan (Bundled)

### `session-memory`
Menyimpan konteks sesi ke memori saat Anda menjalankan `/new`.
Output: `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

### `command-logger`
Mencatat semua kejadian perintah ke file audit pusat.
Output: `~/.openclaw/logs/commands.log`

### `boot-md`
Menjalankan file `BOOT.md` saat gateway mulai berjalan.
Kejadian: `gateway:startup`


