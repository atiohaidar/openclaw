---
summary: "Permukaan pencatatan log, log file, gaya log WS, dan pemformatan konsol"
read_when:
  - Mengubah output atau format log
  - Mendebug output CLI atau gateway
title: "Pencatatan Log (Logging)"
---

# Pencatatan Log (Logging)

OpenClaw memiliki dua permukaan log utama:
- **Output Konsol**: Apa yang Anda lihat di terminal atau UI Debug.
- **Log File**: File berbasis JSON (JSON lines) yang ditulis oleh pencatat log gateway.

## Pencatat Log Berbasis File

- File log default disimpan di bawah `/tmp/openclaw/` (satu file per hari): `openclaw-YYYY-MM-DD.log`.
- Jalur file dan level log dapat dikonfigurasi melalui `~/.openclaw/openclaw.json`:
  - `logging.file`
  - `logging.level` (default: `info`, bisa diatur ke `debug` atau `trace`).

UI Kontrol dan CLI dapat memantau log ini secara langsung:
```bash
openclaw logs --follow
```

## Penangkapan Konsol (Console Capture)

CLI menangkap semua output dari `console.log/info/warn/error` dan menulisnya ke log file sambil tetap menampilkannya di layar.

Anda dapat mengatur gaya tampilan konsol secara mandiri:
- `logging.consoleLevel` (default: `info`)
- `logging.consoleStyle` (`pretty`, `compact`, atau `json`)

## Redaksi Informasi Sensitif

Ringkasan alat (seperti `🛠️ Exec: ...`) dapat menyamarkan token sensitif sebelum ditampilkan di konsol.
- `logging.redactSensitive`: Diatur ke `tools` (default) untuk menyamarkan token.
- `logging.redactPatterns`: Pola regex kustom untuk menyamarkan teks tertentu.

## Log WebSocket Gateway

Gateway mencetak log protokol WebSocket dalam dua mode:
- **Mode Normal (tanpa --verbose)**: Hanya mencetak hasil RPC yang "menarik" (error, panggilan lambat >= 50ms, atau error parsing).
- **Mode Verbose (--verbose)**: Mencetak seluruh lalu lintas permintaan/tanggapan WebSocket.

Gunakan flag berikut untuk mengatur detailnya:
```bash
# Tampilkan semua transmisi WS (format ringkas)
openclaw gateway --verbose --ws-log compact

# Tampilkan metadata lengkap per frame
openclaw gateway --verbose --ws-log full
```

## Pemformatan Konsol

Format konsol dibuat untuk kenyamanan pembacaan manual:
- **Awalan Sub-sistem**: Setiap baris memiliki awalan (misal: `[gateway]`, `[canal]`, `[tailscale]`).
- **Warna Sub-sistem**: Membantu membedakan sumber log secara visual.
- **Warna TTY**: Otomatis mendeteksi kemampuan terminal dan menyesuaikan penggunaan warna.
- **Redaksi**: Menyamarkan token sensitif dalam output interaktif namun tetap mencatat detail lengkap di log file (jika level diatur ke debug/trace).



