---
summary: "Pesan polling Heartbeat dan aturan notifikasi"
read_when:
  - Mengatur ritme atau pesan heartbeat
  - Memutuskan antara heartbeat dan cron untuk tugas terjadwal
title: "Heartbeat"
---

# Heartbeat (Gateway)

> **Heartbeat vs Cron?** Lihat [Cron vs Heartbeat](/id-ID/automation/cron-vs-heartbeat) untuk panduan kapan harus menggunakan masing-masing.

Heartbeat menjalankan **putaran agen secara periodik** di sesi utama sehingga model dapat memunculkan apa pun yang membutuhkan perhatian Anda tanpa harus terus-menerus mengirim pesan sampah.

## Panduan Cepat

1. Biarkan heartbeat menyala (default: `30m`).
2. Buat file kecil `HEARTBEAT.md` di ruang kerja agen sebagai daftar periksa (checklist).
3. Tentukan ke mana pesan heartbeat harus dikirim (default: `target: "last"` / saluran terakhir digunakan).
4. Opsional: batasi heartbeat hanya pada jam aktif (waktu lokal) melalui `activeHours`.

Contoh konfigurasi sederhana:
```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        activeHours: { start: "08:00", end: "22:00" }
      },
    },
  },
}
```

## Cara Kerja Heartbeat

- **Interval**: Defaultnya adalah setiap 30 menit. Gunakan `0m` untuk mematikan.
- **Prompt**: Secara default, bot diperintahkan untuk membaca `HEARTBEAT.md` dan membalas dengan **`HEARTBEAT_OK`** jika tidak ada hal mendesak yang perlu dilaporkan.
- **HEARTBEAT_OK**: Jika agen membalas dengan kata ini, OpenClaw akan menganggap semuanya baik-baik saja dan tidak akan mengirimkan pesan apa pun ke aplikasi chat Anda (kecuali jika mode `showOk` dinyalakan).

## Kegunaan Utama

- **Tugas Latar Belakang**: Mengingatkan agen untuk memeriksa tugas yang tertunda (inbox, kalender, pengingat).
- **Pemeriksaan Manusia**: Memberi kesempatan agen untuk sesekali menyapa Anda ("ada yang bisa saya bantu?") pada jam-jam aktif, namun tetap diam saat Anda tidur.

## Pemecahan Masalah

Jika heartbeat tidak berjalan:
- Pastikan Gateway sedang aktif.
- Periksa `activeHours` di konfigurasi Anda.
- Jika antrean utama sedang sibuk, heartbeat akan dilewati dan dicoba lagi nanti.

## File HEARTBEAT.md (Opsional)

Jika file `HEARTBEAT.md` ada di ruang kerja, bot akan membacanya setiap kali heartbeat berjalan. Isi file ini dengan daftar periksa singkat, misalnya:
- Cek email penting yang masuk.
- Berikan ringkasan berita harian jika sudah siang.
- Ingatkan jadwal rapat besok.

**Tips**: Jaga agar isi file ini tetap singkat agar tidak menghabiskan banyak token API.

## Terkait
- [Cron vs Heartbeat](/id-ID/automation/cron-vs-heartbeat)
- [Pekerjaan Cron](/id-ID/automation/cron-jobs)
- [Zona Waktu](/id-ID/concepts/timezone)



