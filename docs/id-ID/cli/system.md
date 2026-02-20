---
summary: "Referensi CLI untuk `openclaw system` (kejadian sistem, heartbeat, kehadiran)"
read_when:
  - Anda ingin memasukkan kejadian sistem tanpa membuat pekerjaan cron
  - Anda perlu mengaktifkan atau menonaktifkan heartbeat
  - Anda ingin memeriksa entri kehadiran sistem
title: "system"
---

# `openclaw system`

Pembantu tingkat sistem untuk Gateway: memasukkan kejadian sistem, mengontrol heartbeat, dan melihat status kehadiran (presence).

## Perintah Umum

```bash
# Menambahkan kejadian sistem dan segera memicu heartbeat
openclaw system event --text "Cek tindak lanjut mendesak" --mode now

# Mengaktifkan heartbeat
openclaw system heartbeat enable

# Melihat heartbeat terakhir
openclaw system heartbeat last

# Melihat entri kehadiran sistem
openclaw system presence
```

## `system event`
Memasukkan kejadian sistem pada sesi **utama**. Heartbeat berikutnya akan memasukkannya sebagai baris `System:` dalam prompt. Gunakan `--mode now` untuk segera memicu heartbeat; `next-heartbeat` akan menunggu jadwal berikutnya.

## `system heartbeat`
- `last`: Menampilkan kejadian heartbeat terakhir.
- `enable`: Mengaktifkan kembali heartbeat.
- `disable`: Menjeda heartbeat.

## `system presence`
Menampilkan daftar entri kehadiran sistem saat ini yang diketahui oleh Gateway (node, instansi, dan baris status serupa).

## Catatan
- Memerlukan Gateway yang sedang berjalan.
- Kejadian sistem bersifat sementara (ephemeral) dan tidak disimpan setelah Gateway dimulai ulang.



