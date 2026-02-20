---
summary: "Aturan perutean per saluran (WhatsApp, Telegram, Discord, Slack) dan konteks bersama"
read_when:
  - Mengubah perutean saluran atau perilaku kotak masuk (inbox)
title: "Perutean Saluran"
---

# Saluran & Perutean

OpenClaw merutekan balasan **kembali ke saluran tempat pesan berasal**. Model AI tidak memilih saluran secara mandiri; perutean bersifat deterministik dan dikendalikan oleh konfigurasi host.

## Istilah Penting

- **Saluran (Channel)**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
- **AccountId**: Instansi akun per saluran (jika didukung).
- **AgentId**: Ruang kerja (workspace) dan penyimpanan sesi yang terisolasi ("otak" agen).
- **SessionKey**: Kunci yang digunakan untuk menyimpan konteks dan mengontrol eksekusi bersamaan.

## Bentuk Kunci Sesi (Contoh)

Pesan langsung (DM) biasanya akan digabungkan ke sesi **utama (main)** agen:
- `agent:<agentId>:main`

Grup dan saluran tetap terisolasi per saluran:
- Grup: `agent:<agentId>:<channel>:group:<id>`
- Saluran/Ruang: `agent:<agentId>:<channel>:channel:<id>`

Utasan (Threads):
- Utasan Slack/Discord menambahkan `:thread:<threadId>` ke kunci dasar.
- Topik forum Telegram menyematkan `:topic:<topicId>` dalam kunci grup.

## Aturan Perutean

Perutean memilih **satu agen** untuk setiap pesan yang masuk berdasarkan urutan berikut:

1. **Kecocokan Peer Eksak**: Berdasarkan ID pengirim atau ID grup yang terdaftar di `bindings`.
2. **Kecocokan Role/Grup (Discord)**: Berdasarkan `guildId` dan `roles`.
3. **Kecocokan Server (Discord/Slack)**: Berdasarkan `guildId` atau `teamId`.
4. **Kecocokan Akun**: Berdasarkan `accountId` pada saluran.
5. **Kecocokan Saluran**: Berdasarkan jenis saluran (misal: semua pesan Telegram masuk ke agen tertentu).
6. **Agen Default**: Jika tidak ada yang cocok, gunakan agen yang ditandai sebagai `default`.

Agen yang terpilih menentukan ruang kerja (workspace) mana yang akan digunakan.

## Grup Siaran (Broadcast Groups)

Grup siaran memungkinkan Anda menjalankan **beberapa agen sekaligus** untuk pesan yang sama. Ini berguna jika Anda ingin beberapa agen memproses pesan yang masuk ke satu grup WhatsApp atau Telegram.

Contoh konfigurasi:
```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
  },
}
```

## Penyimpanan Sesi

Semua riwayat sesi disimpan di direktori status (default: `~/.openclaw`):
- `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- File transcript dalam format JSONL disimpan di sebelah file store tersebut.

## Perilaku WebChat

WebChat terhubung langsung ke **agen yang dipilih** dan menggunakan sesi utama agen tersebut secara default. Ini memungkinkan Anda melihat konteks lintas saluran untuk agen tersebut dalam satu tempat di browser.

## Konteks Balasan

Setiap pesan yang masuk menyertakan detail balasan (reply context) jika tersedia:
- Mencakup ID pesan yang dibalas, isi pesan, dan pengirimnya.
- Konteks kutipan (quoted context) akan ditambahkan ke bagian atas pesan sebagai blok `[Membalas ...]`.
- Perilaku ini konsisten di semua saluran chat.



