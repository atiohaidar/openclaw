---
summary: "Referensi CLI untuk `openclaw config` (ambil/atur/hapus nilai konfigurasi)"
read_when:
  - Anda ingin membaca atau mengubah konfigurasi secara non-interaktif
title: "config"
---

# `openclaw config`

Pembantu konfigurasi: mengambil (`get`), mengatur (`set`), atau menghapus (`unset`) nilai berdasarkan jalurnya (path). Jalankan tanpa sub-perintah untuk membuka panduan konfigurasi (sama seperti `openclaw configure`).

## Contoh

```bash
# Mengambil nilai konfigurasi
openclaw config get browser.executablePath

# Mengatur nilai konfigurasi
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"

# Mengatur nilai pada elemen array (indeks ke-0)
openclaw config set agents.list[0].tools.exec.node "nama-node"

# Menghapus nilai konfigurasi
openclaw config unset tools.web.search.apiKey
```

## Jalur (Paths)

Jalur menggunakan notasi titik (dot) atau kurung siku (bracket):
```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

## Nilai (Values)

Nilai akan diparsing sebagai JSON5 jika memungkinkan; jika tidak, akan dianggap sebagai string biasa. Gunakan flag `--json` untuk mewajibkan parsing format JSON5.

```bash
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

**Catatan:** Mulai ulang gateway setelah melakukan pengeditan agar perubahan diterapkan.



