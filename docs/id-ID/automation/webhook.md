---
summary: "Ingres Webhook untuk membangunkan agen dan menjalankan agen secara terisolasi"
read_when:
  - Menambah atau mengubah endpoint webhook
  - Menghubungkan sistem eksternal ke OpenClaw
title: "Webhook"
---

# Webhook

Gateway dapat mengekspos endpoint webhook HTTP sederhana untuk pemicu eksternal.

## Mengaktifkan Webhook

```json5
{
  hooks: {
    enabled: true,
    token: "rahasia-bersama",
    path: "/hooks",
    // Opsional: batasi perutean agentId eksplisit ke daftar izinkan ini.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

Catatan:
- `hooks.token` wajib diisi jika `hooks.enabled=true`.
- `hooks.path` standarnya adalah `/hooks`.

## Autentikasi

Setiap permintaan harus menyertakan token hook. Gunakan Header:
- `Authorization: Bearer <token>` (Direkomendasikan)
- `x-openclaw-token: <token>`

## Endpoint

### `POST /hooks/wake`
Digunakan untuk membangunkan agen pada sesi **utama**.

### `POST /hooks/agent`
Menjalankan agen dalam sesi **terisolasi**.

Payload:
```json
{
  "message": "Jalankan ini",
  "name": "Email",
  "agentId": "hooks",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+628123456789",
  "model": "openai/gpt-4o-mini",
  "timeoutSeconds": 120
}
```

- `message` (wajib): Prompt atau pesan untuk diproses agen.
- `deliver`: Jika `true`, respons agen akan dikirim ke saluran perpesanan.
- `wakeMode`: `now` (segera) atau `next-heartbeat` (tunggu jadwal berikutnya).

## Keamanan

- Simpan endpoint webhook di belakang loopback, tailnet (Tailscale), atau reverse proxy tepercaya.
- Gunakan token hook khusus; jangan gunakan kembali token autentikasi gateway.
- Kegagalan autentikasi yang berulang akan dibatasi (rate-limited) per alamat klien.
- Payloads hook dianggap tidak tepercaya dan dibungkus dengan batas keamanan secara default.



