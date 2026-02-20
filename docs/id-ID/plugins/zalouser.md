---
summary: "Plugin Zalo Personal: login QR + perpesanan melalui zca-cli (instalasi plugin + konfigurasi saluran + CLI + alat)"
read_when:
  - Anda ingin dukungan Zalo Personal (tidak resmi) di OpenClaw
  - Anda sedang mengonfigurasi atau mengembangkan plugin zalouser
title: "Plugin Zalo Personal"
---

# Zalo Personal (plugin)

Dukungan Zalo Personal untuk OpenClaw melalui sebuah plugin, menggunakan `zca-cli` untuk mengotomatisasi akun pengguna Zalo normal.

> **Peringatan:** Otomatisasi tidak resmi dapat menyebabkan penangguhan/pemblokiran akun. Gunakan dengan risiko Anda sendiri.

## Penamaan

ID saluran adalah `zalouser` untuk memperjelas bahwa ini mengotomatisasi **akun pengguna Zalo pribadi** (tidak resmi). Kami mencadangkan nama `zalo` untuk integrasi API Zalo resmi di masa mendatang jika memungkinkan.

## Lokasi Berjalan

Plugin ini berjalan **di dalam proses Gateway**.
Jika Anda menggunakan Gateway jarak jauh, instal dan konfigurasi pada **mesin yang menjalankan Gateway**, lalu mulai ulang Gateway.

## Instalasi

### Opsi A: Instal dari npm
```bash
openclaw plugins install @openclaw/zalouser
```
Mulai ulang Gateway setelahnya.

### Opsi B: Instal dari folder lokal (Pengembangan)
```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```
Mulai ulang Gateway setelahnya.

## Prasyarat: zca-cli

Mesin Gateway harus memiliki `zca` di dalam `PATH`:
```bash
zca --version
```

## Konfigurasi

Konfigurasi saluran berada di bawah `channels.zalouser` (bukan di `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Halo dari OpenClaw"
openclaw directory peers list --channel zalouser --query "nama"
```

## Alat Agen

Nama alat: `zalouser`

Tindakan: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`


