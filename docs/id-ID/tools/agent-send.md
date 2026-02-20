---
summary: "Eksekusi CLI `openclaw agent` secara langsung (dengan pengiriman opsional)"
read_when:
  - Menambah atau memodifikasi titik masuk CLI agen
title: "Pengiriman Agen (Agent Send)"
---

# `openclaw agent` (Eksekusi Agen Langsung)

Perintah `openclaw agent` menjalankan satu putaran agen tanpa memerlukan pesan chat masuk. Secara default, perintah ini berjalan **melalui Gateway**; tambahkan `--local` untuk memaksa penggunaan runtime tersemat pada mesin saat ini.

## Perilaku

- Wajib: `--message <teks>`
- Pemilihan sesi:
  - `--to <tujuan>` menentukan kunci sesi.
  - `--session-id <id>` menggunakan kembali sesi yang sudah ada berdasarkan ID.
  - `--agent <id>` menargetkan agen yang dikonfigurasi secara langsung.
- Menjalankan runtime agen yang sama dengan balasan chat normal.
- Bendera (flag) thinking/verbose akan disimpan ke dalam sesi.
- Output:
  - Default: mencetak teks balasan (beserta baris `MEDIA:<url>`).
  - `--json`: mencetak payload terstruktur + metadata.
- Pengiriman kembali ke saluran bersifat opsional dengan `--deliver` + `--channel`.

Jika Gateway tidak dapat dijangkau, CLI akan **beralih (fallback)** ke eksekusi lokal tersemat.

## Contoh

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Ringkas log"
openclaw agent --session-id 1234 --message "Ringkas inbox" --thinking medium
openclaw agent --to +15555550123 --message "Lacak log" --verbose on --json
openclaw agent --to +15555550123 --message "Kirim balasan" --deliver
```

## Bendera (Flags)

- `--local`: jalankan secara lokal (memerlukan kunci API penyedia model di shell Anda).
- `--deliver`: kirim balasan ke saluran yang dipilih.
- `--channel`: saluran pengiriman (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, default: `whatsapp`).
- `--reply-to`: override target pengiriman.
- `--thinking <off|minimal|low|medium|high|xhigh>`: tetapkan tingkat pemikiran.
- `--verbose <on|full|off>`: tetapkan tingkat verbose.
- `--timeout <detik>`: override batas waktu agen.
- `--json`: output JSON terstruktur.



