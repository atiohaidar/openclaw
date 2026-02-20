---
summary: "Menyiarkan pesan WhatsApp ke beberapa agen sekaligus"
read_when:
  - Mengonfigurasi grup siaran
  - Mendebug balasan multi-agen di WhatsApp
title: "Grup Siaran (Broadcast Groups)"
---

# Grup Siaran (Broadcast Groups)

**Status:** Eksperimental  
**Versi:** Ditambahkan pada 2026.1.9

## Ringkasan

Grup siaran memungkinkan beberapa agen untuk memproses dan menanggapi pesan yang sama secara bersamaan. Hal ini memungkinkan Anda membuat tim agen khusus yang bekerja sama dalam satu grup WhatsApp atau DM — semuanya menggunakan satu nomor telepon yang sama.

Ruang lingkup saat ini: **Hanya WhatsApp** (Web Channel).

## Kasus Penggunaan (Use Cases)

1.  **Tim Spesialis**: Misalnya, satu agen bertugas meninjau kode, agen lain membuat dokumentasi, dan agen lainnya memeriksa celah keamanan dalam satu grup pengembangan.
2.  **Dukungan Multi-Bahasa**: Agen yang berbeda membalas dalam bahasa Inggris, Jerman, atau Spanyol secara bersamaan.
3.  **Alur Kerja Jaminan Kualitas (QA)**: Satu agen memberikan jawaban dukungan, sementara agen QA meninjau kualitas jawaban tersebut.

## Konfigurasi

### Penyiapan Dasar
Tambahkan bagian `broadcast` di dalam file konfigurasi Anda. Kunci (*keys*) yang digunakan adalah ID peer WhatsApp (JID grup atau nomor E.164).

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "asisten3"]
  }
}
```

### Strategi Pemrosesan
- **Parallel (Default)**: Semua agen memproses pesan secara bersamaan.
- **Sequential**: Agen memproses pesan secara berurutan; satu agen menunggu agen sebelumnya selesai.

## Cara Kerja

1.  **Pesan Masuk**: Sistem memeriksa apakah ID pengirim/grup ada dalam daftar `broadcast`.
2.  **Isolasi Sesi**: Setiap agen memiliki kunci sesi, riwayat percakapan, ruang kerja, dan memori yang terisolasi sepenuhnya. Mereka tidak melihat balasan satu sama lain secara default.
3.  **Konteks Bersama**: Meskipun terisolasi, semua agen melihat konteks pesan grup yang sama saat dipicu.

## Praktik Terbaik

-   **Fokuskan Agen**: Beri setiap agen satu tanggung jawab yang jelas.
-   **Nama Deskriptif**: Gunakan nama yang mencerminkan fungsi agen (misal: "Security Scanner").
-   **Batasi Jumlah Agen**: Disarankan menggunakan 5-10 agen saja per grup untuk menjaga performa.

## Pemecahan Masalah

-   **Agen tidak merespons**: Pastikan ID agen ada di `agents.list` dan format ID peer sudah benar.
-   **Hanya satu agen yang membalas**: Periksa apakah ID peer mungkin hanya tercatat di `bindings` tapi tidak di `broadcast`.

Untuk rujukan lebih lanjut, lihat [Manajemen Sesi](/id-ID/concepts/sessions).



