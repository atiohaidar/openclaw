---
title: CLI Sandbox
summary: "Mengelola kontainer sandbox dan memeriksa kebijakan sandbox yang efektif"
read_when: "Anda sedang mengelola kontainer sandbox atau mendebug perilaku kebijakan alat/sandbox."
---

# CLI Sandbox

Mengelola kontainer sandbox berbasis Docker untuk eksekusi agen yang terisolasi.

## Ringkasan
OpenClaw dapat menjalankan agen di dalam kontainer Docker yang terisolasi demi keamanan. Perintah `sandbox` membantu Anda mengelola kontainer ini, terutama setelah pembaruan atau perubahan konfigurasi.

## Perintah-perintah

### `openclaw sandbox explain`
Memeriksa mode sandbox yang **efektif**, cakupan (scope), akses ruang kerja, kebijakan alat sandbox, dan konfigurasi terkait.
```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
```

### `openclaw sandbox list`
Menampilkan semua kontainer sandbox beserta status dan konfigurasinya.
```bash
openclaw sandbox list
```
**Output mencakup:**
- Nama kontainer dan status (berjalan/berhenti).
- Gambar Docker yang digunakan.
- Umur kontainer (sejak dibuat) dan waktu pasif (idle).

### `openclaw sandbox recreate`
Menghapus kontainer sandbox lama untuk memaksa pembuatan ulang dengan gambar atau konfigurasi terbaru.
```bash
# Menghapus semua kontainer
openclaw sandbox recreate --all

# Menghapus kontainer untuk sesi tertentu
openclaw sandbox recreate --session main

# Menyeimbangkan pembaruan tanpa konfirmasi
openclaw sandbox recreate --all --force
```

**Penting:** Kontainer akan otomatis dibuat ulang saat agen digunakan kembali di waktu mendatang.

## Terkait:
- Dokumentasi Sandbox: [Sandboxing](/id-ID/gateway/sandboxing)
- Konfigurasi Agen: [Agent Configuration](/id-ID/concepts/agent-workspace)



