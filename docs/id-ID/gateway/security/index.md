---
summary: "Pertimbangan keamanan dan model ancaman untuk menjalankan gateway AI dengan akses shell"
read_when:
  - Menambahkan fitur yang memperluas akses atau otomatisasi
title: "Keamanan"
---

# Keamanan 🔒

OpenClaw menghubungkan model AI ke aplikasi chat dan peralatan (tools) nyata. **Tidak ada sistem yang benar-benar sempurna**, oleh karena itu penting untuk memahami siapa yang bisa mengakses bot Anda dan apa yang bisa dilakukan bot tersebut.

## Audit Keamanan: `openclaw security audit`

Jalankan perintah ini secara berkala untuk mengecek kerentanan pada konfigurasi Anda:

```bash
openclaw security audit --fix
```

Perintah ini akan memeriksa:
- Paparan autentikasi gateway.
- Kebijakan DM (Direct Message) dan Grup.
- Izin file sistem.
- Paparan kontrol browser.

## Dasar Keamanan dalam 60 Detik

Gunakan konfigurasi dasar ini untuk tingkat keamanan yang kuat:

```json5
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "ganti-dengan-token-acak-panjang" },
  },
  tools: {
    profile: "messaging", // Hanya izinkan alat perpesanan dasar
    deny: ["group:automation", "group:runtime", "group:fs"], // Blokir akses file dan sistem
    exec: { security: "deny", ask: "always" }, // Selalu minta izin untuk eksekusi perintah
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

## Model Ancaman

Asisten AI Anda memiliki kemampuan untuk:
- Menjalankan perintah shell.
- Membaca/menulis file.
- Mengirim pesan ke siapa pun.

Sedangkan orang lain yang mengirim pesan ke bot Anda mungkin mencoba:
- Menipu AI untuk melakukan hal berbahaya (Prompt Injection).
- Mencoba mengakses data pribadi Anda.

## Strategi Pertahanan

1. **Identitas Pertama**: Tentukan siapa yang boleh bicara dengan bot (pairing/allowlist).
2. **Cakupan Selanjutnya**: Tentukan apa yang boleh dilakukan bot (kebijakan alat/sandboxing).
3. **Model Terakhir**: Asumsikan model dapat dimanipulasi; desain sistem agar manipulasi tersebut memiliki dampak yang terbatas.

## Tips Tambahan

- **Gunakan Sandboxing**: Selalu jalankan alat yang berisiko di dalam Docker (sandbox).
- **Gunakan Pairing**: Jangan biarkan bot terbuka untuk umum kecuali Anda benar-benar yakin.
- **Batasi Hak Akses File**: Jangan biarkan bot membaca folder sensitif seperti SSH keys atau folder kredensial.

## Terkait
- [Sandboxing](/id-ID/gateway/sandboxing)
- [Pairing](/id-ID/channels/pairing)
- [Troubleshooting](/id-ID/gateway/troubleshooting)



