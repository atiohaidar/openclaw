---
summary: "Kapan OpenClaw menunjukkan indikator mengetik dan cara mengaturnya"
read_when:
  - Mengubah perilaku atau default indikator mengetik
title: "Indikator Mengetik"
---

# Indikator mengetik

Indikator mengetik dikirim ke saluran obrolan saat proses agen sedang aktif. Gunakan `agents.defaults.typingMode` untuk mengontrol **kapan** indikator mengetik dimulai dan `typingIntervalSeconds` untuk mengontrol **seberapa sering** indikator tersebut disegarkan.

## Default

Bila `agents.defaults.typingMode` **tidak disetel**, OpenClaw tetap menggunakan perilaku lama:

-   **Obrolan langsung (DM)**: indikator mengetik dimulai segera setelah loop model dimulai.
-   **Obrolan grup dengan sebutan (mention)**: indikator mengetik dimulai segera.
-   **Obrolan grup tanpa sebutan**: indikator mengetik dimulai hanya saat teks pesan mulai distreaming.
-   **Proses Heartbeat**: indikator mengetik dinonaktifkan.

## Mode

Setel `agents.defaults.typingMode` ke salah satu dari:

-   `never` — jangan pernah tampilkan indikator mengetik.
-   `instant` — mulai mengetik **segera setelah loop model dimulai**, bahkan jika proses tersebut nantinya hanya mengembalikan token balasan senyap.
-   `thinking` — mulai mengetik pada **delta penalaran (reasoning) pertama** (memerlukan `reasoningLevel: "stream"` untuk proses tersebut).
-   `message` — mulai mengetik pada **delta teks non-senyap pertama** (mengabaikan token senyap `NO_REPLY`).

Urutan "seberapa cepat indikator menyala":
`never` → `message` → `thinking` → `instant`

## Konfigurasi

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

Anda dapat menimpa mode atau ritme per sesi:

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```

## Catatan

-   Mode `message` tidak akan menunjukkan indikator mengetik untuk balasan yang hanya berupa token senyap (misalnya token `NO_REPLY` yang digunakan untuk menekan output).
-   Mode `thinking` hanya akan menyala jika proses men-streaming penalaran (`reasoningLevel: "stream"`). Jika model tidak mengeluarkan delta penalaran, indikator mengetik tidak akan dimulai.
-   Heartbeat tidak pernah menunjukkan indikator mengetik, apa pun modenya.
-   `typingIntervalSeconds` mengontrol **ritme penyegaran**, bukan waktu mulai. Defaultnya adalah 6 detik.
