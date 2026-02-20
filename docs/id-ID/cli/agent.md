---
summary: "Referensi CLI untuk `openclaw agent` (menjalankan satu putaran agen melalui Gateway)"
read_when:
  - Anda ingin menjalankan satu putaran (turn) agen dari skrip
title: "agent"
---

# `openclaw agent`

Menjalankan satu putaran (turn) agen melalui Gateway (gunakan `--local` untuk mode tertanam). Gunakan `--agent <id>` untuk menargetkan agen tertentu yang sudah dikonfigurasi.

## Terkait:
- Alat pengiriman agen: [Agent send](/id-ID/tools/agent-send)

## Contoh

```bash
# Mengirim pesan ke agen dan mengirimkan jawabannya ke target
openclaw agent --to +628123456789 --message "update status" --deliver

# Menjalankan agen tertentu (ops) tanpa pengiriman otomatis
openclaw agent --agent ops --message "Ringkas log"

# Menentukan level fitur pemikiran (thinking)
openclaw agent --session-id 1234 --message "Ringkas inbox" --thinking medium

# Menentukan saluran balasan kustom
openclaw agent --agent ops --message "Buat laporan" --deliver --reply-channel slack --reply-to "#reports"
```



