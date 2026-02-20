---
summary: "Mode eksekusi (exec) elevasi dan arahan /elevated"
read_when:
  - Menyesuaikan default mode elevasi, daftar izinkan, atau perilaku perintah slash
title: "Mode Elevasi"
---

# Mode Elevasi (Arahan /elevated)

## Apa fungsinya?

- `/elevated on`: berjalan di host gateway dan tetap memerlukan persetujuan eksekusi (sama dengan `/elevated ask`).
- `/elevated full`: berjalan di host gateway **dan** menyetujui eksekusi secara otomatis (melewati persetujuan).
- `/elevated ask`: berjalan di host gateway namun tetap memerlukan persetujuan eksekusi (sama dengan `/elevated on`).
- Hanya mengubah perilaku ketika agen berada di dalam **sandbox** (jika tidak, eksekusi sudah berjalan di host secara default).
- Bentuk arahan: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.

## Apa yang dikontrol (dan apa yang tidak)

- **Gating Ketersediaan**: `tools.elevated` adalah dasar global. `agents.list[].tools.elevated` dapat membatasi elevasi lebih lanjut per agen.
- **Status Per Sesi**: `/elevated ...` menetapkan tingkat elevasi untuk kunci sesi saat ini.
- **Arahan Inline**: `/elevated ...` di dalam pesan hanya berlaku untuk pesan tersebut.
- **Eksekusi Host**: elevasi memaksa `exec` berjalan ke host gateway; `full` juga menyetel `security=full`.
- **Persetujuan**: `full` melewati persetujuan eksekusi; `on`/`ask` tetap mematuhinya jika aturan memerlukan.

## Urutan Resolusi

1. Arahan inline pada pesan (hanya berlaku untuk pesan tersebut).
2. Override sesi (diatur dengan mengirim pesan yang hanya berisi arahan).
3. Default global (`agents.defaults.elevatedDefault` dalam konfigurasi).

## Mengatur Default Sesi

- Kirim pesan yang **hanya** berisi arahan (boleh ada spasi), misalnya `/elevated full`.
- Balasan konfirmasi akan dikirim (misal: `Elevated mode set to full...`).
- Jika akses elevasi dinonaktifkan atau pengirim tidak ada dalam daftar izinkan, arahan akan membalas dengan kesalahan dan tidak mengubah status sesi.
- Kirim `/elevated` tanpa argumen untuk melihat tingkat elevasi saat ini.

## Ketersediaan + Daftar Izinkan (Allowlists)

- Fitur ini dapat diaktifkan/dinonaktifkan melalui `tools.elevated.enabled`.
- Daftar izinkan pengirim: `tools.elevated.allowFrom` dengan daftar per penyedia (misal: `discord`, `whatsapp`).
- Gerbang per agen: `agents.list[].tools.elevated.enabled` (opsional; hanya bisa membatasi lebih lanjut).

## Pencatatan (Logging) + Status

- Pemanggilan eksekusi terelevasi dicatat pada tingkat log `info`.
- Status sesi mencakup mode elevasi (misal: `elevated=ask`, `elevated=full`).


