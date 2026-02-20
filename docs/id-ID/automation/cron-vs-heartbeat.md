---
summary: "Panduan memilih antara heartbeat dan pekerjaan cron untuk otomatisasi"
read_when:
  - Memutuskan cara menjadwalkan tugas berulang
  - Menyiapkan pemantauan latar belakang atau notifikasi
  - Mengoptimalkan penggunaan token untuk pemeriksaan berkala
title: "Cron vs Heartbeat"
---

# Cron vs Heartbeat: Kapan Menggunakan Masing-masing

Baik heartbeat maupun pekerjaan cron memungkinkan Anda menjalankan tugas sesuai jadwal. Panduan ini membantu Anda memilih mekanisme yang tepat untuk kebutuhan Anda.

## Panduan Keputusan Cepat

| Kebutuhan                             | Direkomendasikan     | Mengapa                                    |
| :------------------------------------ | :------------------- | :----------------------------------------- |
| Cek inbox setiap 30 menit             | Heartbeat            | Digabung dengan cek lainnya, sadar konteks |
| Kirim laporan harian tepat jam 9 pagi | Cron (terisolasi)    | Memerlukan waktu yang presisi              |
| Pantau kalender untuk event mendatang | Heartbeat            | Cocok untuk pemantauan berkala             |
| Analisis mendalam mingguan            | Cron (terisolasi)    | Tugas mandiri, bisa pakai model berbeda    |
| Ingatkan saya dalam 20 menit          | Cron (utama, `--at`) | Sekali jalan dengan waktu presisi          |

## Heartbeat: Pemantauan Berkala

Heartbeat berjalan di **sesi utama (main session)** dengan interval tertentu (default: 30 menit). Fitur ini dirancang agar agen dapat memeriksa berbagai hal dan memunculkan apa pun yang penting.

### Kapan menggunakan heartbeat

- **Banyak pemeriksaan berkala**: Daripada membuat 5 pekerjaan cron terpisah untuk cek inbox, kalender, cuaca, notifikasi, dan status proyek, satu heartbeat dapat merangkum semuanya sekaligus.
- **Keputusan sadar konteks**: Agen memiliki konteks sesi utama yang lengkap, sehingga dapat membuat keputusan cerdas tentang apa yang mendesak vs apa yang bisa menunggu.
- **Kontinuitas percakapan**: Jalannya heartbeat berbagi sesi yang sama, sehingga agen mengingat percakapan terakhir dan dapat menindaklanjutinya secara alami.

## Cron: Penjadwalan Presisi

Pekerjaan cron berjalan pada waktu yang tepat dan dapat berjalan dalam sesi terisolasi tanpa memengaruhi konteks utama.

### Kapan menggunakan cron

- **Waktu yang tepat diperlukan**: "Kirim ini setiap Senin jam 9:00 pagi" (bukan "sekitar jam 9").
- **Tugas mandiri**: Tugas yang tidak memerlukan konteks percakapan.
- **Model/pemikiran berbeda**: Analisis berat yang membutuhkan model yang lebih kuat.
- **Pengingat sekali jalan**: "Ingatkan saya dalam 20 menit" dengan flag `--at`.
- **Tugas yang berisik/sering**: Tugas yang dapat mengotori riwayat sesi utama.

## Alur Keputusan

1. Apakah tugas harus berjalan pada waktu yang **SANGAT TEPAT**?
   - YA -> Gunakan cron
2. Apakah tugas memerlukan **isolasi** dari sesi utama?
   - YA -> Gunakan cron (terisolasi)
3. Apakah tugas ini bisa **digabung** dengan pemeriksaan berkala lainnya?
   - YA -> Gunakan heartbeat (tambahkan ke `HEARTBEAT.md`)
   - TIDAK -> Gunakan cron
4. Apakah ini pengingat **sekali jalan**?
   - YA -> Gunakan cron dengan flag `--at`

## Menggabungkan Keduanya

Pengaturan yang paling efisien adalah menggunakan **keduanya**:

1. **Heartbeat** menangani pemantauan rutin (inbox, kalender, notifikasi) dalam satu putaran gabungan setiap 30 menit.
2. **Cron** menangani jadwal yang presisi (laporan harian, tinjauan mingguan) dan pengingat sekali jalan.

## Lobster: Alur Kerja Deterministik dengan Persetujuan

Lobster adalah runtime alur kerja untuk **pipelining alat multi-langkah** yang membutuhkan eksekusi deterministik dan persetujuan eksplisit. Gunakan saat tugas lebih dari sekadar satu putaran agen, dan Anda menginginkan alur kerja yang dapat dilanjutkan dengan poin pemeriksaan manusia (human checkpoints).

Lihat [Lobster](/id-ID/tools/lobster) untuk penggunaan lengkap.
