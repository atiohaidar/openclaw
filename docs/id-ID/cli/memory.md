---
summary: "Referensi CLI untuk `openclaw memory` (status/indeks/cari)"
read_when:
  - Anda ingin membuat indeks atau melakukan pencarian semantik pada memori
  - Anda sedang mendebug ketersediaan memori atau proses pengindeksan
title: "memory"
---

# `openclaw memory`

Mengelola pengindeksan dan pencarian memori semantik. Fitur ini disediakan oleh plugin memori yang aktif (default: `memory-core`).

## Terkait:
- Konsep Memori: [Memory](/id-ID/concepts/memory)
- Plugin: [Plugins](/id-ID/tools/plugin)

## Contoh

```bash
# Menampilkan status indeks memori
openclaw memory status

# Menampilkan status mendalam (deep)
openclaw memory status --deep

# Melakukan pengindeksan ulang memori
openclaw memory index

# Melakukan pencarian semantik dalam memori
openclaw memory search "checklist rilis"

# Menentukan agen spesifik
openclaw memory status --agent main
```

## Opsi
- `--agent <id>`: Membatasi cakupan pada agen tunggal (default: semua agen yang dikonfigurasi).
- `--verbose`: Menampilkan log detail selama proses pemeriksaan dan pengindeksan.

**Catatan:** `memory status --deep` akan memeriksa ketersediaan vektor dan embedding. Gunakan `memory status --deep --index` untuk secara otomatis melakukan indeks ulang jika data dianggap kotor (dirty).


