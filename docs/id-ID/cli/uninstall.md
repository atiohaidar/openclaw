---
summary: "Referensi CLI untuk `openclaw uninstall` (menghapus layanan gateway + data lokal)"
read_when:
  - Anda ingin menghapus layanan gateway dan/atau status lokal
  - Anda ingin melihat simulasi (dry-run) terlebih dahulu
title: "uninstall"
---

# `openclaw uninstall`

Menghapus instalasi layanan gateway dan data lokal (aplikasi CLI sendiri akan tetap ada).

```bash
# Menghapus secara interaktif
openclaw uninstall

# Menghapus semua data dan layanan tanpa konfirmasi (berbahaya)
openclaw uninstall --all --yes

# Melihat simulasi penghapusan
openclaw uninstall --dry-run
```



