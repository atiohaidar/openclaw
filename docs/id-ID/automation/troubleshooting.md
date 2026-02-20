---
summary: "Pemecahan masalah penjadwalan dan pengiriman cron serta heartbeat"
read_when:
  - Cron tidak berjalan
  - Cron berjalan tapi tidak ada pesan yang terkirim
  - Heartbeat tampak senyap atau terlewat
title: "Pemecahan Masalah Otomatisasi"
---

# Pemecahan Masalah Otomatisasi

Gunakan halaman ini untuk masalah penjadwal dan pengiriman (`cron` + `heartbeat`).

## Perintah Pemeriksaan

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Kemudian jalankan pemeriksaan otomatisasi:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron Tidak Berjalan

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
```

Kondisi yang Baik:
- `cron status` melaporkan aktif (enabled) dan ada waktu `nextWakeAtMs` di masa depan.
- Pekerjaan aktif dan memiliki jadwal/zona waktu yang valid.
- `cron runs` menunjukkan status `ok` atau alasan lewati (skip) yang jelas.

Masalah Umum:
- `cron: scheduler disabled` -> cron dinonaktifkan di konfigurasi.
- `cron: timer tick failed` -> penjadwal mengalami crash; periksa log.
- `reason: not-due` -> pemanggilan manual tanpa `--force` padahal belum waktunya.

## Cron Berjalan Tapi Tidak Ada Pengiriman

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw channels status --probe
```

Kondisi yang Baik:
- Status jalan (run status) adalah `ok`.
- Mode pengiriman (delivery mode) dan target sudah diatur untuk pekerjaan terisolasi.
- Proba saluran melaporkan target saluran terhubung.

Masalah Umum:
- Status sukses tapi mode pengiriman `none` -> memang tidak ada pesan luar yang diharapkan.
- Target pengiriman hilang atau tidak valid (`channel`/`to`).
- Kesalahan autentikasi saluran (`unauthorized`, `Forbidden`) -> pengiriman diblokir oleh kredensial saluran.

## Heartbeat Tertekan atau Terlewat

```bash
openclaw system heartbeat last
openclaw logs --follow
```

Kondisi yang Baik:
- Heartbeat aktif dengan interval yang tidak nol.
- Hasil heartbeat terakhir adalah `ran` (berjalan).

Masalah Umum:
- `heartbeat skipped` dengan `reason=quiet-hours` -> berada di luar jam aktif (`activeHours`).
- `requests-in-flight` -> sesi utama sedang sibuk; heartbeat ditunda.
- `empty-heartbeat-file` -> heartbeat dilewati karena `HEARTBEAT.md` kosong dan tidak ada kejadian cron yang menunggu.
- `no-heartbeat-file` -> heartbeat dilewati karena file `HEARTBEAT.md` tidak ditemukan.



