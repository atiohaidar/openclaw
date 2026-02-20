---
summary: "Referensi CLI untuk `openclaw security` (audit dan perbaikan masalah keamanan umum)"
read_when:
  - Anda ingin menjalankan audit keamanan cepat pada konfigurasi/status
  - Anda ingin menerapkan perbaikan yang disarankan (chmod, pengaturan default yang lebih ketat)
title: "security"
---

# `openclaw security`

Alat keamanan untuk melakukan audit dan perbaikan (fixes) opsional.

## Audit Keamanan

```bash
# Jalankan audit keamanan standar
openclaw security audit

# Jalankan audit mendalam (deep scan)
openclaw security audit --deep

# Jalankan audit dan terapkan perbaikan otomatis
openclaw security audit --fix

# Output dalam format JSON
openclaw security audit --json
```

Audit ini akan memberikan peringatan jika:
- Beberapa pengirim DM berbagi sesi utama yang sama (disarankan menggunakan mode DM aman).
- Model kecil digunakan tanpa sandboxing sementara alat web/browser diaktifkan.
- Ingress webhook tidak aman (misal: `sessionKey` dapat ditimpa tanpa batasan awalan).
- Mode autentikasi gateway disetel ke `none`.

## Apa yang diubah oleh `--fix`?

Flag `--fix` menerapkan perbaikan yang aman dan deterministik, seperti:
- Mengubah `groupPolicy="open"` menjadi `groupPolicy="allowlist"`.
- Mengatur `logging.redactSensitive` dari `off` ke `tools`.
- Memperketat izin file (permissions) untuk file konfigurasi dan kredensial sensitif.

Flag `--fix` **tidak akan**:
- Merotasi token, kata sandi, atau kunci API.
- Menonaktifkan alat (gateway, cron, exec, dll).
- Mengubah pengaturan jaringan atau paparan (exposure) gateway.



