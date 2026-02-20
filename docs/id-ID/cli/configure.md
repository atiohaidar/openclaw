---
summary: "Referensi CLI untuk `openclaw configure` (prompt konfigurasi interaktif)"
read_when:
  - Anda ingin mengubah kredensial, perangkat, atau default agen secara interaktif
title: "configure"
---

# `openclaw configure`

Prompt interaktif untuk mengatur kredensial, perangkat, dan pengaturan default agen.

**Catatan:** Bagian **Model** kini menyertakan seleksi ganda untuk daftar izinkan (allowlist) `agents.defaults.models` (apa yang muncul di perintah `/model` dan pemilih model).

**Tips:** Perintah `openclaw config` tanpa sub-perintah akan membuka panduan yang sama. Gunakan `openclaw config get|set|unset` untuk pengeditan non-interaktif.

## Terkait:
- Referensi konfigurasi Gateway: [Konfigurasi](/id-ID/gateway/configuration)
- CLI Konfigurasi: [Config](/id-ID/cli/config)

## Catatan Tambahan:
- Memilih lokasi jalannya Gateway akan selalu memperbarui `gateway.mode`.
- Layanan berbasis saluran (seperti Slack, Discord, MS Teams) akan menanyakan daftar izinkan saluran/ruangan selama penyiapan.

## Contoh

```bash
# Menjalankan panduan konfigurasi lengkap
openclaw configure

# Hanya menjalankan bagian model dan saluran
openclaw configure --section models --section channels
```


