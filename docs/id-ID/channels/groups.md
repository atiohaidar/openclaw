---
summary: "Perilaku obrolan grup di berbagai platform (WhatsApp/Telegram/Discord/dll)"
read_when:
  - Mengubah perilaku grup atau pengaturan sebutan (mention)
title: "Grup"
---

# Grup

OpenClaw menangani obrolan grup secara konsisten di semua platform: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, dan Microsoft Teams.

## Ringkasan Perilaku

Secara default:
- Akses grup dibatasi (`groupPolicy: "allowlist"`).
- Balasan memerlukan sebutan (@mention) kecuali jika fitur penahan sebutan dimatikan.

| Tujuan                                               | Pengaturan                                                       |
| :--------------------------------------------------- | :--------------------------------------------------------------- |
| Izinkan semua grup tapi hanya balas jika di-@mention | `groups: { "*": { requireMention: true } }`                    |
| Matikan semua balasan grup                           | `groupPolicy: "disabled"`                                      |
| Hanya grup tertentu                                  | `groups: { "<id-grup>": { ... } }`                             |
| Hanya Anda yang bisa memicu bot di grup              | `groupPolicy: "allowlist"`, `groupAllowFrom: ["nomor_anda"]` |

## Alur Pesan Grup

1. **groupPolicy**: Apakah grup diizinkan secara umum? (open/disabled/allowlist)
2. **Daftar Izinkan (Allowlist)**: Apakah ID grup atau pengirim terdaftar?
3. **Penahan Sebutan (Mention Gating)**: Apakah bot disebut (@mention)?

## Sesi Grup

Setiap grup memiliki kunci sesi (session key) yang terisolasi sendiri. Artinya, riwayat percakapan di satu grup tidak akan bercampur dengan grup lain atau DM pribadi Anda. 

## Kasus Penggunaan: DM Pribadi vs Grup Publik

Ini adalah pola yang sangat disarankan:
- **DM**: Akses penuh ke alat di mesin host (tanpa sandbox).
- **Grup**: Menjalankan bot di dalam **Sandbox (Docker)** dengan alat yang dibatasi (hanya bisa kirim pesan, tidak bisa akses file host).

## Dokumentasi Terkait
- [Saluran WhatsApp](/id-ID/channels/whatsapp)
- [Saluran Telegram](/id-ID/channels/telegram)
- [Saluran Discord](/id-ID/channels/discord)
- [Sandboxing](/id-ID/gateway/sandboxing)



