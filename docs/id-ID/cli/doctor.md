---
summary: "Referensi CLI untuk `openclaw doctor` (pemeriksaan kesehatan + perbaikan terpandu)"
read_when:
  - Anda mengalami masalah konektivitas/autentikasi dan ingin perbaikan terpandu
  - Anda baru saja memperbarui aplikasi dan ingin melakukan pengecekan kesehatan
title: "doctor"
---

# `openclaw doctor`

Pemeriksaan kesehatan + perbaikan cepat untuk gateway dan saluran (channels).

## Terkait:
- Pemecahan Masalah: [Troubleshooting](/id-ID/gateway/troubleshooting)
- Audit Keamanan: [Security](/id-ID/gateway/security)

## Contoh

```bash
# Jalankan pemeriksaan kesehatan
openclaw doctor

# Jalankan dengan perbaikan otomatis
openclaw doctor --repair

# Jalankan pemeriksaan mendalam
openclaw doctor --deep
```

## Catatan:
- Prompt interaktif (seperti perbaikan gantungan kunci/keychain atau OAuth) hanya berjalan jika terminal mendukung TTY dan flag `--non-interactive` **tidak** diaktifkan.
- Flag `--fix` (alias dari `--repair`) akan membuat cadangan konfigurasi di `~/.openclaw/openclaw.json.bak` sebelum melakukan perubahan.

### Khusus macOS: Override lingkungan `launchctl`
Jika Anda pernah menjalankan perintah `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`, nilai tersebut akan menimpa file konfigurasi Anda dan mungkin menyebabkan kesalahan "unauthorized" yang terus-menerus.
Gunakan `launchctl unsetenv` untuk menghapusnya jika diperlukan.



