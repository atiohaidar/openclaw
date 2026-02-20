---
title: "Deteksi Loop Alat"
description: "Konfigurasikan pembatas keamanan opsional untuk mencegah loop pemanggilan alat yang berulang atau macet"
read_when:
  - Pengguna melaporkan agen terjebak dalam pengulangan pemanggilan alat
  - Anda perlu menyesuaikan perlindungan pemanggilan berulang
  - Anda sedang mengedit kebijakan alat/runtime agen
---

# Deteksi Loop Alat (Tool-loop detection)

OpenClaw dapat mencegah agen terjebak dalam pola pemanggilan alat yang berulang-ulang. Fitur pelindung ini **dinonaktifkan secara default**.

Aktifkan hanya jika diperlukan, karena pengaturan yang terlalu ketat dapat memblokir pemanggilan berulang yang sebenarnya sah.

## Mengapa Ini Ada

- Mendeteksi urutan pemanggilan berulang yang tidak menghasilkan kemajuan.
- Mendeteksi loop tanpa hasil dengan frekuensi tinggi (alat sama, input sama, kesalahan berulang).
- Mendeteksi pola pemanggilan berulang pada alat polling tertentu.

## Konfigurasi

Default Global:

```json5
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 20,
      detectorCooldownMs: 12000,
      repeatThreshold: 3,
      criticalThreshold: 6,
      detectors: {
        repeatedFailure: true,
        knownPollLoop: true,
        repeatingNoProgress: true,
      },
    },
  },
}
```

## Perilaku Bidang

- `enabled`: Sakelar utama. `false` berarti tidak ada deteksi loop yang dilakukan.
- `historySize`: Jumlah pemanggilan alat terakhir yang disimpan untuk analisis.
- `repeatThreshold`: Batas minimal pengulangan sebelum peringatan atau pemblokiran dimulai.
- `criticalThreshold`: Ambang batas yang lebih kritis yang dapat memicu penanganan yang lebih ketat.
- `detectors.repeatedFailure`: Mendeteksi kegagalan berulang pada jalur panggilan yang sama.
- `detectors.knownPollLoop`: Mendeteksi loop serupa polling yang sudah dikenal.
- `detectors.repeatingNoProgress`: Mendeteksi pemanggilan berulang frekuensi tinggi tanpa adanya perubahan status.

## Penyiapan yang Direkomendasikan

- Mulailah dengan `enabled: true` dan biarkan nilai default lainnya tetap.
- Jika terjadi positif palsu (false positives):
  - Naikkan nilai `repeatThreshold` atau `criticalThreshold`.
  - Nonaktifkan hanya detektor yang menyebabkan masalah.
  - Kurangi `historySize` agar analisis sejarah pemanggilan tidak terlalu ketat.

## Log dan Perilaku yang Diharapkan

Ketika loop terdeteksi, OpenClaw akan melaporkan kejadian loop dan memblokir atau meredam siklus alat berikutnya tergantung pada tingkat keparahannya. Ini melindungi pengguna dari pemborosan token dan kemacetan, sambil tetap menjaga akses alat yang normal.


