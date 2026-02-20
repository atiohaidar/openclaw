---
summary: "Runtime agen (embedded pi-mono), kontrak workspace, dan bootstrap sesi"
read_when:
  - Mengubah runtime agen, bootstrap workspace, atau perilaku sesi
title: "Runtime Agen"
---

# Runtime Agen 🤖

OpenClaw menjalankan runtime agen tunggal (embedded) yang diturunkan dari **pi-mono**.

## Workspace (wajib)

OpenClaw menggunakan satu direktori workspace agen (`agents.defaults.workspace`) sebagai **satu-satunya** direktori kerja (`cwd`) agen untuk alat dan konteks.

Direkomendasikan: gunakan `openclaw setup` untuk membuat `~/.openclaw/openclaw.json` jika hilang dan menginisialisasi file workspace.

Tata letak workspace lengkap + panduan cadangan: [Workspace agen](/id-ID/concepts/agent-workspace)

Jika `agents.defaults.sandbox` diaktifkan, sesi non-utama dapat menimpanya dengan
workspace per-sesi di bawah `agents.defaults.sandbox.workspaceRoot` (lihat
[Konfigurasi Gateway](/id-ID/gateway/configuration)).

## File bootstrap (disuntikkan)

Di dalam `agents.defaults.workspace`, OpenClaw mengharapkan file yang dapat diedit pengguna berikut ini:

- `AGENTS.md` — instruksi operasi + “memori”
- `SOUL.md` — persona, batasan, nada bicara
- `TOOLS.md` — catatan alat yang dikelola pengguna (misalnya `imsg`, `sag`, konvensi)
- `BOOTSTRAP.md` — ritual penggunaan pertama kali (dihapus setelah selesai)
- `IDENTITY.md` — nama agen/vibe/emoji
- `USER.md` — profil pengguna + panggilan pilihan

Pada giliran pertama dari sesi baru, OpenClaw menyuntikkan isi file-file ini langsung ke dalam konteks agen.

File kosong akan dilewati. File besar akan dipangkas dan dipotong dengan penanda agar prompt tetap ramping (baca file untuk konten lengkap).

Jika file hilang, OpenClaw menyuntikkan satu baris penanda “file hilang” (dan `openclaw setup` akan membuat template default yang aman).

`BOOTSTRAP.md` hanya dibuat untuk **workspace yang benar-benar baru** (tidak ada file bootstrap lain). Jika Anda menghapusnya setelah menyelesaikan ritual, file tersebut tidak seharusnya dibuat ulang pada restart berikutnya.

Untuk menonaktifkan pembuatan file bootstrap sepenuhnya (untuk workspace yang sudah diisi sebelumnya), atur:

```json5
{ agent: { skipBootstrap: true } }
```

## Alat bawaan (Built-in tools)

Alat inti (baca/eksekusi/edit/tulis dan alat sistem terkait) selalu tersedia,
tunduk pada kebijakan alat. `apply_patch` bersifat opsional dan dibatasi oleh
`tools.exec.applyPatch`. `TOOLS.md` **tidak** mengontrol alat apa yang ada; itu adalah
panduan tentang bagaimana _Anda_ ingin alat tersebut digunakan.

## Skill

OpenClaw memuat skill dari tiga lokasi (workspace menang jika ada konflik nama):

- Bundled (disertakan saat instalasi)
- Dikelola/lokal: `~/.openclaw/skills`
- Workspace: `<workspace>/skills`

Skill dapat dibatasi oleh konfigurasi/env (lihat `skills` di [Konfigurasi Gateway](/id-ID/gateway/configuration)).

## Integrasi pi-mono

OpenClaw menggunakan kembali bagian dari basis kode pi-mono (model/alat), tetapi **manajemen sesi, penemuan (discovery), dan penyambungan alat adalah milik OpenClaw**.

- Tidak ada runtime agen pi-coding.
- Tidak ada pengaturan `~/.pi/agent` atau `<workspace>/.pi` yang dikonsultasikan.

## Sesi

Transkrip sesi disimpan sebagai JSONL di:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

ID sesi bersifat stabil dan dipilih oleh OpenClaw.
Folder sesi Pi/Tau lama (legacy) **tidak** dibaca.

## Mengendalikan saat streaming (Steering while streaming)

Saat mode antrean adalah `steer`, pesan masuk disuntikkan ke dalam proses yang sedang berjalan.
Antrean diperiksa **setelah setiap pemanggilan alat**; jika ada pesan yang antre,
pemanggilan alat yang tersisa dari pesan asisten saat ini akan dilewati (hasil alat error
dengan "Skipped due to queued user message."), kemudian pesan pengguna yang antre
disuntikkan sebelum respons asisten berikutnya.

Saat mode antrean adalah `followup` atau `collect`, pesan masuk ditahan sampai
giliran saat ini berakhir, kemudian giliran agen baru dimulai dengan payload yang antre. Lihat
[Antrean](/id-ID/concepts/queue) untuk perilaku mode + debounce/cap.

Block streaming mengirimkan blok asisten yang telah selesai segera setelah blok tersebut selesai; ini
**mati secara default** (`agents.defaults.blockStreamingDefault: "off"`).
Atur batas melalui `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; default ke text_end).
Kontrol pemotongan blok lunak dengan `agents.defaults.blockStreamingChunk` (default ke
800–1200 karakter; lebih memilih pemisahan paragraf, kemudian baris baru; kalimat terakhir).
Gabungkan potongan yang dialirkan dengan `agents.defaults.blockStreamingCoalesce` untuk mengurangi
spam satu baris (penggabungan berbasis idle sebelum kirim). Saluran Non-Telegram memerlukan
`*.blockStreaming: true` eksplisit untuk mengaktifkan balasan blok.
Ringkasan alat verbose dipancarkan pada saat alat dimulai (tanpa debounce); Control UI
mengalirkan output alat melalui event agen jika tersedia.
Lebih detail: [Streaming + chunking](/id-ID/concepts/streaming).

## Referensi model (Model refs)

Referensi model dalam konfigurasi (misalnya `agents.defaults.model` dan `agents.defaults.models`) diparsing dengan memisahkan pada `/` **pertama**.

- Gunakan `provider/model` saat mengonfigurasi model.
- Jika ID model itu sendiri mengandung `/` (gaya OpenRouter), sertakan awalan provider (contoh: `openrouter/moonshotai/kimi-k2`).
- Jika Anda menghilangkan provider, OpenClaw memperlakukan input sebagai alias atau model untuk **provider default** (hanya berfungsi jika tidak ada `/` dalam ID model).

## Konfigurasi (minimal)

Minimal, atur:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (sangat direkomendasikan)

---

_Selanjutnya: [Chat Grup](/id-ID/channels/group-messages)_ 🦞


