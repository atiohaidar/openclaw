---
summary: "Jendela konteks + pemadatan (compaction): cara OpenClaw menjaga sesi agar tetap berada dalam batas model"
read_when:
  - Anda ingin memahami pemadatan otomatis dan perintah /compact
  - Anda sedang mendebug sesi panjang yang mencapai batas konteks
title: "Pemadatan"
---

# Jendela Konteks & Pemadatan (Compaction)

Setiap model memiliki **jendela konteks** (maksimal token yang dapat dilihat). Obrolan yang berjalan lama mengumpulkan pesan dan hasil alat; setelah jendela tersebut penuh, OpenClaw akan **memadatkan** riwayat lama agar tetap berada dalam batas.

## Apa itu Pemadatan

Pemadatan **meringkas percakapan lama** menjadi entri ringkasan yang padat dan membiarkan pesan terbaru tetap utuh. Ringkasan ini disimpan dalam riwayat sesi, sehingga permintaan di masa mendatang akan menggunakan:

- Ringkasan pemadatan
- Pesan terbaru setelah titik pemadatan

Pemadatan bersifat **persisten** dalam riwayat JSONL sesi.

## Konfigurasi

Gunakan pengaturan `agents.defaults.compaction` di `openclaw.json` Anda untuk mengonfigurasi perilaku pemadatan (mode, target token, dll.).

## Pemadatan Otomatis (Default Aktif)

Bila sesi mendekati atau melebihi jendela konteks model, OpenClaw memicu pemadatan otomatis dan dapat mencoba ulang permintaan asli menggunakan konteks yang telah dipadatkan.

Anda akan melihat:

- `🧹 Auto-compaction complete` dalam mode verbose
- `/status` menunjukkan `🧹 Compactions: <count>`

Sebelum pemadatan, OpenClaw dapat menjalankan giliran **silent memory flush** untuk menyimpan catatan penting ke disk. Lihat [Memori](/id-ID/concepts/memory) untuk detail dan konfigurasi.

## Pemadatan Manual

Gunakan `/compact` (opsional dengan instruksi) untuk memaksa proses pemadatan:

```
/compact Fokus pada keputusan dan pertanyaan terbuka
```

## Sumber Jendela Konteks

Jendela konteks bersifat spesifik untuk setiap model. OpenClaw menggunakan definisi model dari katalog penyedia yang dikonfigurasi untuk menentukan batas.

## Pemadatan vs Pemangkasan (Pruning)

- **Pemadatan (Compaction)**: meringkas dan bersifat **persisten** di JSONL.
- **Pemangkasan Sesi (Session Pruning)**: memangkas hanya **hasil alat** lama, **dalam memori**, per permintaan.

Lihat [/id-ID/concepts/session-pruning](/id-ID/concepts/session-pruning) untuk detail pemangkasan.

## Tip

- Gunakan `/compact` jika sesi terasa basi atau konteks terasa terlalu besar.
- Output alat yang besar sudah dipotong secara otomatis; pemangkasan dapat lebih lanjut mengurangi penumpukan hasil alat.
- Jika Anda membutuhkan lembaran baru, gunakan `/new` atau `/reset` untuk memulai ID sesi baru.
