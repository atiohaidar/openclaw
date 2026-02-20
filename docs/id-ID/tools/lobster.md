---
title: Lobster
summary: "Runtime alur kerja bertipe untuk OpenClaw dengan gerbang persetujuan yang dapat dilanjutkan."
description: Runtime alur kerja bertipe untuk OpenClaw — pipeline yang dapat disusun dengan gerbang persetujuan.
read_when:
  - Anda ingin alur kerja multi-langkah yang deterministik dengan persetujuan eksplisit
  - Anda perlu melanjutkan alur kerja tanpa menjalankan ulang langkah sebelumnya
---

# Lobster

Lobster adalah shell alur kerja yang memungkinkan OpenClaw menjalankan urutan alat multi-langkah sebagai satu operasi deterministik dengan pos pemeriksaan persetujuan yang eksplisit.

## Mengapa Menggunakan Lobster?

Saat ini, alur kerja yang kompleks memerlukan banyak pemanggilan alat bolak-balik. Setiap panggilan memakan biaya token, dan LLM harus mengatur setiap langkahnya. Lobster memindahkan pengaturan tersebut ke dalam runtime bertipe:

- **Satu panggilan, bukan banyak**: OpenClaw menjalankan satu panggilan alat Lobster dan mendapatkan hasil yang terstruktur.
- **Persetujuan terintegrasi**: Efek samping (seperti mengirim email atau memposting komentar) akan menghentikan alur kerja hingga disetujui secara eksplisit.
- **Dapat dilanjutkan**: Alur kerja yang terhenti akan mengembalikan sebuah token; setujui dan lanjutkan tanpa menjalankan ulang semuanya dari awal.

## Mengapa DSL alih-alih program biasa?

Lobster sengaja dibuat kecil. Tujuannya bukan untuk menjadi "bahasa baru," melainkan spesifikasi pipeline yang dapat diprediksi dan ramah AI dengan fitur persetujuan dan token resume yang utama.

- **Setujui/lanjutkan sudah terintegrasi**: Program normal mungkin bisa meminta input manusia, tetapi tidak bisa *berhenti dan berlanjut* dengan token yang tahan lama tanpa Anda membuat runtime-nya sendiri.
- **Deterministik + dapat diaudit**: Pipeline adalah data, sehingga mudah untuk dicatat, dibandingkan (diff), diputar ulang, dan ditinjau.
- **Aman**: Batas waktu (timeout), batas output, pemeriksaan sandbox, dan daftar izinkan diterapkan oleh runtime, bukan oleh setiap skrip.

## Cara kerjanya

OpenClaw meluncurkan CLI `lobster` lokal dalam **mode alat** dan mengurai amplop JSON dari stdout.
Jika pipeline berhenti untuk persetujuan, alat tersebut mengembalikan `resumeToken` sehingga Anda dapat melanjutkannya nanti.

## Pola: CLI Kecil + Pipa JSON + Persetujuan

Buatlah perintah kecil yang menghasilkan JSON, lalu hubungkan dalam satu panggilan Lobster.

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | approve --prompt 'Terapkan perubahan?'",
  "timeoutMs": 30000
}
```

Jika pipeline meminta persetujuan, lanjutkan dengan token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

## Langkah LLM khusus JSON (llm-task)

Untuk alur kerja yang membutuhkan **langkah LLM terstruktur**, aktifkan plugin `llm-task` yang opsional dan panggil dari Lobster. Ini menjaga alur kerja tetap deterministik sambil tetap memungkinkan Anda mengklasifikasi/meringkas/menulis draf dengan sebuah model.

Lihat [LLM Task](/id-ID/tools/llm-task) untuk detail dan opsi konfigurasi.

## File Alur Kerja (.lobster)

Lobster dapat menjalankan file alur kerja YAML/JSON dengan bidang `name`, `args`, `steps`, `env`, `condition`, dan `approval`.

```yaml
name: triase-inbox
steps:
  - id: collect
    command: inbox list --json
  - id: approve
    command: inbox apply --approve
    stdin: $collect.stdout
    approval: required
```

## Cara Instalasi

Instal CLI Lobster pada **host yang sama** yang menjalankan OpenClaw Gateway. Pastikan `lobster` ada di dalam `PATH` Anda.

## Mengaktifkan Alat

Lobster adalah alat plugin **opsional** (tidak diaktifkan secara default).

Direkomendasikan (aman):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

## Contoh: Triase Email

Tanpa Lobster:
Agen memanggil API email berkali-kali, meminta persetujuan untuk setiap draf, dan sering lupa apa yang sudah diproses jika sesi terputus.

Dengan Lobster:
Satu panggilan pipeline menjalankan seluruh proses triase. Jika draf sudah siap, bot meminta satu persetujuan untuk semuanya. Jika disetujui, semua email dikirim sekaligus.

## Parameter Alat

### `run`
Menjalankan pipeline dalam mode alat.

### `resume`
Melanjutkan alur kerja yang terhenti setelah mendapatkan persetujuan.

## Amplop Output

Lobster mengembalikan amplop JSON dengan salah satu dari tiga status:
- `ok` → selesai dengan sukses
- `needs_approval` → tertunda; memerlukan `resumeToken` untuk berlanjut
- `cancelled` → ditolak atau dibatalkan secara eksplisit

## Persetujuan

Jika `requiresApproval` muncul, periksa prompt-nya dan putuskan:
- `approve: true` → lanjutkan dan terapkan efek samping
- `approve: false` → batalkan alur kerja

## Keamanan

- **Subproses lokal saja** — tidak ada panggilan jaringan langsung dari plugin itu sendiri.
- **Tanpa rahasia (secrets)** — Lobster tidak mengelola OAuth; ia memanggil alat OpenClaw yang menanganinya.
- **Sadar-sandbox** — dinonaktifkan jika konteks alat berada di dalam sandbox.
- **Diperkuat** — nama eksekutabel tetap (`lobster`) di dalam `PATH`; batas waktu dan batas output diterapkan secara ketat.

## Pemecahan Masalah

- **`lobster subprocess timed out`** → tingkatkan `timeoutMs`.
- **`lobster output exceeded maxStdoutBytes`** → tingkatkan `maxStdoutBytes` atau kurangi ukuran output.
- **`lobster returned invalid JSON`** → pastikan pipeline hanya mencetak JSON.


