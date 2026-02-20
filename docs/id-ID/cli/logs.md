---
summary: "Referensi CLI untuk `openclaw logs` (pantau log gateway melalui RPC)"
read_when:
  - Anda perlu memantau log Gateway secara jarak jauh (tanpa SSH)
  - Anda menginginkan baris log dalam format JSON untuk kebutuhan alat bantu
title: "logs"
---

# `openclaw logs`

Memantau log file Gateway melalui RPC (bekerja dalam mode jarak jauh).

## Terkait:
- Ringkasan Logging: [Logging](/id-ID/logging)

## Contoh

```bash
# Menampilkan log terbaru
openclaw logs

# Memantau log secara real-time (terus-menerus)
openclaw logs --follow

# Menampilkan 500 baris log terakhir
openclaw logs --limit 500

# Menampilkan log dalam format JSON
openclaw logs --json

# Menampilkan waktu dalam zona waktu lokal Anda
openclaw logs --local-time
```

**Catatan:** Gunakan flag `--local-time` agar stempel waktu (timestamps) dirender sesuai dengan zona waktu lokal Anda untuk memudahkan pembacaan.


