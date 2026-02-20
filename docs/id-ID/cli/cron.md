---
summary: "Referensi CLI untuk `openclaw cron` (menjadwalkan dan menjalankan pekerjaan latar belakang)"
read_when:
  - Anda ingin menjadwalkan pekerjaan rutin atau pengaktifan otomatis
  - Anda sedang mendebug eksekusi cron dan log terkait
title: "cron"
---

# `openclaw cron`

Mengelola pekerjaan cron (cron jobs) untuk penjadwal (scheduler) Gateway.

## Terkait:
- Pekerjaan Cron: [Cron jobs](/id-ID/automation/cron-jobs)

## Perintah Umum

```bash
# Menampilkan status cron
openclaw cron status

# Menampilkan daftar semua pekerjaan cron
openclaw cron list

# Menambah pekerjaan baru yang berjalan setiap jam
openclaw cron add --name "cek-laporan" --every "1h" --message "Ringkas laporan harian"

# Mengaktifkan/menonaktifkan pekerjaan
openclaw cron enable <id>
openclaw cron disable <id>

# Menghapus pekerjaan cron
openclaw cron rm <id>
```

## Pengeditan Umum

Mengubah pengaturan pengiriman tanpa mengubah isi pesan:
```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

**Catatan:** Pekerjaan berulang kini menggunakan mekanisme *retry backoff* eksponensial (30 dtk → 1 mnt → 5 mnt → ...) jika terjadi kesalahan berturut-turut.



