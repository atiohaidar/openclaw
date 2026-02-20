---
summary: "Referensi CLI untuk `openclaw reset` (mereset status/konfigurasi lokal)"
read_when:
  - Anda ingin menghapus data lokal tetapi tetap menjaga CLI terinstal
  - Anda ingin melihat simulasi (dry-run) apa saja yang akan dihapus
title: "reset"
---

# `openclaw reset`

Mereset konfigurasi dan status lokal (tetapi tetap menjaga CLI terinstal).

```bash
# Mereset secara interaktif
openclaw reset

# Melihat simulasi penghapusan tanpa benar-benar menghapusnya
openclaw reset --dry-run

# Mereset konfigurasi, kredensial, dan sesi secara non-interaktif
openclaw reset --scope config+creds+sessions --yes --non-interactive
```



