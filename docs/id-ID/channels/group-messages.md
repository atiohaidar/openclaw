---
summary: "Perilaku dan konfigurasi penanganan pesan grup WhatsApp (pola sebutan/mentionPatterns dibagikan di berbagai platform)"
read_when:
  - Mengubah aturan pesan grup atau sebutan (mention)
title: "Pesan Grup"
---

# Pesan Grup (WhatsApp)

Tujuan: Memungkinkan asisten untuk berada di dalam grup WhatsApp, bangun hanya ketika dipanggil (@mention), dan menjaga riwayat percakapan tersebut terpisah dari sesi pesan langsung (DM) pribadi Anda.

Catatan: `mentionPatterns` sekarang juga digunakan oleh Telegram, Discord, Slack, dan iMessage. Dokumen ini berfokus pada perilaku spesifik di WhatsApp.

## Fitur yang Tersedia

- **Mode Aktivasi**:
  - `mention` (default): Memerlukan @mention eksplisit, pola regex, atau nomor telepon bot di dalam teks.
  - `always`: Bot bangun pada setiap pesan, namun hanya akan membalas jika ia merasa bisa memberikan nilai tambah (jika tidak, ia akan diam menggunakan token `NO_REPLY`).
- **Kebijakan Grup**: Dikendalikan oleh `groupPolicy` (`open`, `disabled`, atau `allowlist`). Secara default adalah `allowlist` (diblokir sampai Anda menambahkan pengirim ke dalam daftar).
- **Sesi Terisolasi**: Kunci sesi untuk grup berbeda dengan DM (misal: `agent:asisten:whatsapp:group:<jid>`). Perintah seperti `/verbose on` atau `/reset` yang dikirim di grup tidak akan memengaruhi sesi DM pribadi Anda.
- **Injeksi Konteks**: Pesan grup yang muncul sebelum bot dipanggil akan dimasukkan sebagai konteks tambahan agar bot paham alur pembicaraan sebelumnya.
- **Identitas Pengirim**: Setiap pesan grup akan menyertakan penanda `[from: Nama Pengirim (+nomor)]` agar asisten tahu siapa yang sedang berbicara.
- **Pesan Sekali Lihat (View-once)**: Bot akan membuka pesan ini sebelum mengekstrak teks atau sebutan di dalamnya.
- **Prompt Sistem Grup**: Bot akan diberitahu konteks bahwa ia berada di dalam grup tertentu beserta daftar anggotanya.

## Contoh Konfigurasi (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\\\+?6281234567890"],
        },
      },
    ],
  },
}
```

## Perintah Aktivasi (Hanya Pemilik)

Anda dapat mengubah mode aktivasi langsung dari dalam chat grup:
- `/activation mention`
- `/activation always`

Gunakan perintah `/status` di dalam grup untuk melihat mode aktivasi yang sedang aktif. Sesi grup bersifat independen, sehingga perintah manajemen sesi hanya berlaku untuk grup tersebut.

## Praktik Terbaik

1. **Undang Bot ke Grup**: Pastikan akun WhatsApp yang menjalankan OpenClaw sudah masuk ke grup target.
2. **Gunakan @mention**: Panggil bot dengan nama atau nomornya.
3. **Konteks Riwayat**: Bot akan melihat pesan-pesan sebelumnya untuk memahami diskusi sebelum memberikan jawaban.
4. **Keamanan**: Gunakan `groupPolicy: "allowlist"` untuk membatasi siapa saja yang bisa memicu bot di dalam grup publik.

Untuk rujukan teknis lebih lanjut, lihat [Pemecahan Masalah Saluran](/id-ID/channels/troubleshooting).



