---
summary: "Referensi CLI untuk `openclaw setup` (inisialisasi konfigurasi + ruang kerja)"
read_when:
  - Anda melakukan penyiapan pertama kali tanpa panduan onboarding lengkap
  - Anda ingin mengatur jalur ruang kerja (workspace) default
title: "setup"
---

# `openclaw setup`

Menginisialisasi file `~/.openclaw/openclaw.json` dan ruang kerja (workspace) agen.

## Terkait:
- Panduan memulai: [Memulai](/id-ID/start/getting-started)
- Wizard: [Onboarding](/id-ID/start/onboarding)

## Contoh

```bash
# Penyiapan dasar
openclaw setup

# Penyiapan dengan menentukan folder ruang kerja kustom
openclaw setup --workspace ~/.openclaw/workspace
```

Untuk menjalankan wizard melalui perintah setup:
```bash
openclaw setup --wizard
```



