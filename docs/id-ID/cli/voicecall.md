---
summary: "Referensi CLI untuk `openclaw voicecall` (permukaan perintah plugin voice-call)"
read_when:
  - Anda menggunakan plugin voice-call dan menginginkan titik masuk CLI
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` adalah perintah yang disediakan oleh plugin. Perintah ini hanya muncul jika plugin `voice-call` sudah diinstal dan diaktifkan.

## Dokumentasi Utama:
- Plugin Voice-call: [Voice Call](/id-ID/plugins/voice-call)

## Perintah Umum

```bash
# Menampilkan status panggilan tertentu
openclaw voicecall status --call-id <id>

# Melakukan panggilan keluar
openclaw voicecall call --to "+628123456789" --message "Halo" --mode notify

# Melanjutkan percakapan pada panggilan yang sedang berlangsung
openclaw voicecall continue --call-id <id> --message "Ada pertanyaan lain?"

# Mengakhiri panggilan
openclaw voicecall end --call-id <id>
```

## Mengekspos Webhook (Tailscale)
```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

**Catatan Keamanan:** Hanya ekspos endpoint webhook ke jaringan yang Anda percayai. Sangat disarankan untuk menggunakan Tailscale Serve daripada Funnel jika memungkinkan.



