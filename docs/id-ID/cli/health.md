---
summary: "Referensi CLI untuk `openclaw health` (endpoint kesehatan gateway melalui RPC)"
read_when:
  - Anda ingin memeriksa kesehatan Gateway yang sedang berjalan dengan cepat
title: "health"
---

# `openclaw health`

Mengambil data kesehatan dari Gateway yang sedang berjalan.

```bash
# Pemeriksaan standar
openclaw health

# Output format JSON (untuk skrip)
openclaw health --json

# Pemeriksaan detail dengan pencatatan waktu per akun
openclaw health --verbose
```

**Catatan:** Flag `--verbose` akan menjalankan pemeriksaan langsung (live probes) dan mencetak waktu respons per akun jika terdapat banyak akun yang dikonfigurasi.


