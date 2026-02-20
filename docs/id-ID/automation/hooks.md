---
summary: "Hook: otomatisasi berbasis kejadian untuk perintah dan peristiwa siklus hidup"
read_when:
  - Anda menginginkan otomatisasi berbasis kejadian untuk /new, /reset, /stop, dan siklus hidup agen
  - Anda ingin membangun, menginstal, atau mendebug hook
title: "Hook"
---

# Hook

Hook menyediakan sistem berbasis kejadian (event-driven) yang dapat diperluas untuk mengotomatiskan tindakan sebagai respons terhadap perintah dan kejadian agen. Hook ditemukan secara otomatis dari direktori dan dapat dikelola melalui perintah CLI, mirip dengan cara kerja skill di OpenClaw.

## Mengenal Hook

Hook adalah skrip kecil yang berjalan saat sesuatu terjadi. Ada dua jenis:

- **Hook** (halaman ini): berjalan di dalam Gateway saat kejadian agen dipicu, seperti `/new`, `/reset`, `/stop`, atau kejadian siklus hidup (lifecycle events).
- **Webhooks**: webhook HTTP eksternal yang memungkinkan sistem lain memicu pekerjaan di OpenClaw. Lihat [Webhook Hooks](/id-ID/automation/webhook).

Hook juga dapat dibundel di dalam plugin; lihat [Plugin](/id-ID/tools/plugin#plugin-hooks).

### Kegunaan Umum:
- Menyimpan snapshot memori saat Anda mereset sesi.
- Mencatat jejak audit perintah untuk pemecahan masalah.
- Memicu otomatisasi lanjutan saat sesi dimulai atau berakhir.
- Menulis file ke dalam ruang kerja agen atau memanggil API eksternal.

## Memulai

### Hook Bawaan (Bundled Hooks)

OpenClaw menyertakan empat hook bawaan yang otomatis ditemukan:

- **💾 session-memory**: Menyimpan konteks sesi ke ruang kerja agen Anda (default `~/.openclaw/workspace/memory/`) saat Anda menjalankan `/new`.
- **📎 bootstrap-extra-files**: Menyuntikkan file bootstrap ruang kerja tambahan selama `agent:bootstrap`.
- **📝 command-logger**: Mencatat semua kejadian perintah ke `~/.openclaw/logs/commands.log`.
- **🚀 boot-md**: Menjalankan `BOOT.md` saat gateway dimulai.

Lihat daftar hook yang tersedia:
```bash
openclaw hooks list
```

Aktifkan sebuah hook:
```bash
openclaw hooks enable session-memory
```

## Penemuan Hook (Hook Discovery)

Hook ditemukan secara otomatis dari tiga direktori (berdasarkan urutan prioritas):

1. **Workspace hooks**: `<workspace>/hooks/` (per-agen, prioritas tertinggi).
2. **Managed hooks**: `~/.openclaw/hooks/` (terinstal oleh pengguna, dibagikan antar ruang kerja).
3. **Bundled hooks**: `<openclaw>/dist/hooks/bundled/` (bawaan OpenClaw).

Setiap hook adalah sebuah direktori yang berisi:
```
my-hook/
├── HOOK.md          # Metadata + dokumentasi
└── handler.ts       # Implementasi handler
```

## Jenis Kejadian (Event Types)

### Kejadian Perintah (Command Events)
Dipicu saat perintah agen dijalankan:
- `command:new`: Saat perintah `/new` dijalankan.
- `command:reset`: Saat perintah `/reset` dijalankan.
- `command:stop`: Saat perintah `/stop` dijalankan.

### Kejadian Pesan (Message Events)
Dipicu saat pesan diterima atau dikirim:
- `message:received`: Saat pesan masuk diterima dari saluran mana pun.
- `message:sent`: Saat pesan keluar berhasil dikirim.

## Membuat Hook Kustom

1. **Pilih Lokasi**: Buat folder di `~/.openclaw/hooks/my-hook`.
2. **Buat HOOK.md**: Isi dengan metadata nama, deskripsi, dan kejadian yang dipantau.
3. **Buat handler.ts**: Tulis fungsi TypeScript untuk menangani kejadian tersebut.
4. **Aktifkan**: Jalankan `openclaw hooks enable my-hook`.

## Praktik Terbaik

- **Jaga agar Handler Cepat**: Hook berjalan selama pemrosesan perintah. Pastikan kodenya ringan dan gunakan operasi asinkron jika melakukan tugas berat.
- **Tangani Kesalahan dengan Anggun**: Selalu gunakan `try/catch` agar kesalahan pada satu hook tidak menghentikan jalannya sistem atau hook lainnya.
- **Filter Kejadian Sejak Awal**: Segera kembali (return) jika kejadian yang diterima tidak relevan dengan hook Anda.
