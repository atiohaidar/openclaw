---
title: Sandbox vs Kebijakan Alat vs Elevated
summary: "Mengapa sebuah alat (tool) diblokir: runtime sandbox, kebijakan allow/deny alat, dan gerbang eksekusi elevated"
read_when: "Anda merasa terjebak di dalam sandbox atau melihat penolakan alat/elevated dan ingin tahu kunci konfigurasi mana yang harus diubah."
---

# Sandbox vs Kebijakan Alat vs Elevated

OpenClaw memiliki tiga kontrol keamanan yang saling berkaitan namun berbeda:

1. **Sandbox** (`sandbox.*`): Menentukan **di mana alat dijalankan** (di dalam Docker atau langsung di mesin host).
2. **Kebijakan Alat** (`tools.*` / `allow` / `deny`): Menentukan **alat mana yang boleh digunakan/tersedia**.
3. **Elevated** (`elevated.*`): Adalah **jalur keluar khusus eksekusi (exec-only)** untuk menjalankan perintah di mesin host meskipun Anda sedang berada di dalam sandbox (Docker).

## Debug Cepat

Gunakan perintah inspeksi untuk melihat apa yang *sebenarnya* terjadi pada sistem Anda:

```bash
openclaw sandbox explain
openclaw sandbox explain --agent work
```

Perintah ini akan mencetak:
- Mode sandbox yang sedang aktif.
- Daftar alat yang diizinkan (allow) atau dilarang (deny) di dalam sandbox.
- Detail tentang akses ruang kerja (workspace access).

## Sandbox: Lokasi Menjalankan Alat

Mode sandbox diatur oleh `agents.defaults.sandbox.mode`:
- `off`: Semua alat berjalan langsung di mesin host (tanpa isolasi).
- `non-main`: Hanya sesi non-utama (seperti obrolan grup/saluran) yang dijalankan di dalam sandbox.
- `all`: Semua sesi dijalankan di dalam sandbox (isolasi penuh).

## Kebijakan Alat: Alat Mana yang Boleh Dipanggil

Kebijakan ini memiliki prioritas sebagai berikut:
- **Deny (Larang)** selalu menang. Jika sebuah alat ada di daftar `deny`, ia tidak bisa dipanggil sama sekali.
- Jika daftar **Allow (Izinkan)** tidak kosong, maka semua alat lain yang tidak ada di daftar tersebut akan dianggap diblokir.
- Kebijakan alat di dalam sandbox (`tools.sandbox.tools.*`) hanya berlaku saat sesi sedang berjalan di dalam Docker.

### Grup Alat (Shorthands)
Anda dapat menggunakan pintasan grup untuk mengizinkan banyak alat sekaligus:
- `group:runtime`: Mencakup `exec`, `bash`, `process`.
- `group:fs`: Mencakup alat file sistem seperti `read`, `write`, `edit`.
- `group:ui`: Mencakup `browser`, `canvas`.

## Elevated: Jalur Keluar ke Host

Mode Elevated **tidak** memberikan alat tambahan; ia hanya mempengaruhi perintah `exec`.
- Jika Anda sedang di dalam sandbox, menggunakan perintah `/elevated on` memungkinkan Anda menjalankan perintah `exec` langsung di mesin host (tetap memerlukan persetujuan pemilik).
- Gunakan `/elevated full` untuk melewati persetujuan eksekusi selama sesi berlangsung (hanya jika diizinkan di konfigurasi).

## FAQ "Terjebak di Sandbox"

### "Alat X diblokir oleh kebijakan sandbox"
Solusi:
- Matikan sandbox: `agents.defaults.sandbox.mode=off`.
- Tambahkan alat tersebut ke daftar `tools.sandbox.tools.allow` di file konfigurasi Anda.

### "Mengapa sesi saya dijalankan di sandbox?"
Dalam mode `non-main`, obrolan grup atau saluran chat dianggap bukan sesi utama dan akan otomatis dijalankan di dalam sandbox untuk keamanan.

## Terkait
- [Sandboxing](/id-ID/gateway/sandboxing)
- [Mode Elevated](/id-ID/tools/elevated)
- [Keamanan](/id-ID/gateway/security)



