---
summary: "Pekerjaan Cron + bangun (wakeups) untuk penjadwal Gateway"
read_when:
  - Menjadwalkan pekerjaan latar belakang atau membangunkan agen
  - Menghubungkan otomatisasi yang harus berjalan bersamaan dengan heartbeat
  - Memutuskan antara heartbeat dan cron untuk tugas terjadwal
title: "Pekerjaan Cron (Cron Jobs)"
---

# Pekerjaan Cron (Penjadwal Gateway)

> **Cron vs Heartbeat?** Lihat [Cron vs Heartbeat](/id-ID/automation/cron-vs-heartbeat) untuk panduan kapan menggunakan masing-masing fitur ini.

Cron adalah penjadwal bawaan Gateway. Fitur ini menyimpan daftar pekerjaan, membangunkan agen pada waktu yang tepat, dan secara opsional mengirimkan output kembali ke obrolan (chat).

Jika Anda ingin *"jalankan ini setiap pagi"* atau *"ingatkan agen dalam 20 menit"*, cron adalah mekanismenya.

Pemecahan Masalah: [/id-ID/automation/troubleshooting](/id-ID/automation/troubleshooting)

## Ringkasan Cepat

- Cron berjalan **di dalam Gateway** (bukan di dalam model).
- Pekerjaan disimpan di bawah `~/.openclaw/cron/` sehingga jadwal tidak hilang saat Gateway dimulai ulang.
- Dua gaya eksekusi:
  - **Sesi Utama (Main session)**: memasukkan kejadian sistem (system event) ke antrean, lalu dijalankan pada heartbeat berikutnya.
  - **Terisolasi (Isolated)**: menjalankan putaran agen khusus dalam sesi `cron:<jobId>`, dengan pengiriman hasil (defaultnya diumumkan atau tidak sama sekali).
- "Wakeup" (membangunkan) adalah fitur utama: sebuah pekerjaan dapat meminta "bangun sekarang" vs "heartbeat berikutnya".

## Mulai Cepat

Buat pengingat sekali jalan, verifikasi keberadaannya, dan jalankan segera:

```bash
openclaw cron add \
  --name "Pengingat" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: periksa draf dok cron" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
```

Jadwalkan pekerjaan terisolasi yang berulang dengan pengiriman hasil:

```bash
openclaw cron add \
  --name "Ringkasan pagi" \
  --cron "0 7 * * *" \
  --tz "Asia/Jakarta" \
  --session isolated \
  --message "Ringkas pembaruan semalam." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## Di Mana Pekerjaan Cron Disimpan

Pekerjaan cron disimpan di host Gateway pada file `~/.openclaw/cron/jobs.json`. Gunakan perintah `openclaw cron add/edit` untuk melakukan perubahan.

## Konsep Dasar

Anggap pekerjaan cron sebagai: **kapan** harus berjalan + **apa** yang harus dilakukan.

1. **Pilih jadwal**
   - Pengingat sekali jalan → `schedule.kind = "at"` (CLI: `--at`)
   - Pekerjaan berulang → `schedule.kind = "every"` atau `schedule.kind = "cron"`

2. **Pilih lokasi eksekusi**
   - `sessionTarget: "main"` → berjalan saat heartbeat berikutnya dengan konteks sesi utama.
   - `sessionTarget: "isolated"` → berjalan sebagai putaran agen khusus dalam sesi `cron:<jobId>`.

3. **Pilih muatan (payload)**
   - Sesi utama → `payload.kind = "systemEvent"`
   - Sesi terisolasi → `payload.kind = "agentTurn"`

### Jadwal (Schedules)

Cron mendukung tiga jenis jadwal:
- `at`: stempel waktu sekali jalan (ISO 8601).
- `every`: interval tetap (dalam milidetik).
- `cron`: ekspresi cron 5-bidang (atau 6-bidang dengan detik) dengan zona waktu IANA opsional.

### Eksekusi Utama vs Terisolasi

#### Pekerjaan Sesi Utama (System Events)
Pekerjaan utama memasukkan kejadian sistem ke antrean dan secara opsional membangunkan pelaksana heartbeat. Sangat cocok jika Anda ingin prompt heartbeat normal dan konteks sesi utama. Lihat [Heartbeat](/id-ID/gateway/heartbeat).

#### Pekerjaan Terisolasi (Sesi Cron Khusus)
Pekerjaan terisolasi berjalan sebagai putaran agen khusus. Setiap eksekusi dimulai dengan **ID sesi baru (fresh session id)** (tidak membawa konteks percakapan sebelumnya). Sangat berguna untuk tugas-tugas latar belakang yang sering atau berisik agar tidak memenuhi riwayat obrolan utama Anda.

## Contoh CLI

Pengingat sekali jalan (dalam 20 menit, sesi utama, bangun segera):

```bash
openclaw cron add \
  --name "Cek Kalender" \
  --at "20m" \
  --session main \
  --system-event "Heartbeat berikutnya: cek kalender." \
  --wake now
```

Pekerjaan terisolasi berulang (umumkan ke WhatsApp setiap jam 7 pagi):

```bash
openclaw cron add \
  --name "Status Pagi" \
  --cron "0 7 * * *" \
  --tz "Asia/Jakarta" \
  --session isolated \
  --message "Ringkas inbox + kalender hari ini." \
  --announce \
  --channel whatsapp \
  --to "+628123456789"
```

Menjalankan secara manual:

```bash
openclaw cron run <jobId>
```

Melihat riwayat jalannya pekerjaan:

```bash
openclaw cron runs --id <jobId> --limit 50
```

## Pemecahan Masalah

### "Tidak ada yang berjalan"
- Pastikan cron diaktifkan: cek `cron.enabled` di konfigurasi atau variabel lingkungan `OPENCLAW_SKIP_CRON`.
- Pastikan Gateway berjalan terus-menerus (cron berjalan di dalam proses Gateway).
- Untuk jadwal `cron`: pastikan zona waktu (`--tz`) sudah tepat.

### Pekerjaan berulang terus tertunda setelah gagal
OpenClaw menerapkan penundaan ulang (backoff) eksponensial untuk pekerjaan berulang setelah terjadi kesalahan berturut-turut: 30 detik, 1 menit, 5 menit, 15 menit, lalu 60 menit di antara percobaan ulang.



