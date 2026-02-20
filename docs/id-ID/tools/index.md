---
summary: "Permukaan alat agen untuk OpenClaw (browser, canvas, node, pesan, cron) menggantikan skill `openclaw-*` lama"
read_when:
  - Menambah atau memodifikasi alat agen
  - Menghentikan atau mengubah penggunaan skill `openclaw-*`
title: "Alat (Tools)"
---

# Alat (Tools - OpenClaw)

OpenClaw menyediakan **alat agen utama (first-class agent tools)** untuk browser, canvas, node, dan cron.
Alat-alat ini menggantikan skill `openclaw-*` yang lama: alat-alat ini bertipe (typed), tidak memerlukan perintah shell langsung (shelling), dan agen harus mengandalkannya secara langsung.

## Menonaktifkan alat

Anda dapat mengizinkan/menolak alat secara global melalui `tools.allow` / `tools.deny` di `openclaw.json` (kebijakan tolak/deny selalu menang). Ini mencegah alat yang tidak diizinkan dikirim ke penyedia model (model providers).

```json5
{
  tools: { deny: ["browser"] },
}
```

Catatan:

- Pencocokan tidak peka huruf besar-kecil (case-insensitive).
- Karakter wildcard `*` didukung (`"*"` berarti semua alat).
- Jika `tools.allow` hanya merujuk pada alat plugin yang tidak dikenal atau tidak dimuat, OpenClaw akan mencatat peringatan dan mengabaikan daftar izinkan (allowlist) tersebut agar alat inti tetap tersedia.

## Profil alat (profil allowlist dasar)

`tools.profile` menetapkan **daftar izinkan alat dasar** sebelum `tools.allow`/`tools.deny`.
Override per agen: `agents.list[].tools.profile`.

Profil:

- `minimal`: hanya `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: tidak ada batasan (sama seperti jika tidak diatur)

Contoh (hanya pesan secara default, izinkan juga alat Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

## Kebijakan alat spesifik penyedia (provider-specific)

Gunakan `tools.byProvider` untuk **membatasi lebih lanjut** alat bagi penyedia tertentu (atau satu `provider/model`) tanpa mengubah default global Anda.
Override per agen: `agents.list[].tools.byProvider`.

Ini diterapkan **setelah** profil alat dasar dan **sebelum** daftar izinkan/tolak, sehingga hanya dapat mempersempit kumpulan alat.

Contoh (pertahankan profil coding global, tapi berikan alat minimal untuk Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

## Grup alat (singkatan)

Kebijakan alat (global, agen, sandbox) mendukung entri `group:*` yang mencakup beberapa alat sekaligus.
Gunakan ini di `tools.allow` / `tools.deny`.

Grup yang tersedia:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: semua alat bawaan OpenClaw (tidak termasuk plugin penyedia)

## Plugin + alat

Plugin dapat mendaftarkan **alat tambahan** (dan perintah CLI) di luar kumpulan inti.
Lihat [Plugin](/id-ID/tools/plugin) untuk instalasi + konfigurasi, dan [Skill](/id-ID/tools/skills) untuk melihat bagaimana panduan penggunaan alat dimasukkan ke dalam prompt. Beberapa plugin menyertakan skill mereka sendiri bersama alat (misalnya, plugin voice-call).

Alat plugin opsional:

- [Lobster](/id-ID/tools/lobster): runtime alur kerja bertipe dengan persetujuan yang dapat dilanjutkan (memerlukan CLI Lobster pada host gateway).
- [LLM Task](/id-ID/tools/llm-task): langkah LLM khusus JSON untuk output alur kerja terstruktur (opsional validasi skema).

## Inventaris alat

### `apply_patch`

Terapkan patch terstruktur pada satu atau lebih file. Gunakan untuk pengeditan multi-hunk.
Eksperimental: aktifkan melalui `tools.exec.applyPatch.enabled` (hanya untuk model OpenAI).

### `exec`

Jalankan perintah shell di workspace.

Parameter inti:

- `command` (wajib)
- `yieldMs` (otomatis ke latar belakang setelah batas waktu, default 10000)
- `background` (segera ke latar belakang)
- `timeout` (detik; mematikan proses jika terlampaui, default 1800)
- `elevated` (boolean; jalankan di host jika mode elevasi diaktifkan/diizinkan)
- `host` (`sandbox | gateway | node`)
- `security` (`deny | allowlist | full`)
- `ask` (`off | on-miss | always`)
- Butuh TTY asli? Atur `pty: true`.

Catatan:

- Mengembalikan `status: "running"` dengan `sessionId` saat berjalan di latar belakang.
- Gunakan `process` untuk memantau/mencatat/menulis/mematikan/membersihkan sesi latar belakang.
- `elevated` dibatasi oleh `tools.elevated` dan override `agents.list[].tools.elevated`.
- `host=node` dapat menargetkan aplikasi pendamping macOS atau host node headless.
- Persetujuan gateway/node dan daftar izinkan: [Persetujuan Exec](/id-ID/tools/exec-approvals).

### `process`

Kelola sesi eksekusi (`exec`) latar belakang.

Tindakan inti:

- `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

### `loop-detection` (pelindung loop pemanggilan alat)

OpenClaw melacak riwayat pemanggilan alat terbaru dan memblokir atau memperingatkan ketika mendeteksi loop berulang tanpa kemajuan.
Aktifkan dengan `tools.loopDetection.enabled: true` (default adalah `false`).

### `web_search`

Cari di web menggunakan Brave Search API.
Aktifkan melalui `tools.web.search.enabled`. Memerlukan kunci API Brave.

### `web_fetch`

Ambil dan ekstrak konten yang dapat dibaca dari sebuah URL (HTML → markdown/teks).
Aktifkan melalui `tools.web.fetch.enabled`. Untuk situs yang berat JavaScript, lebih baik gunakan alat `browser`.

### `browser`

Kontrol browser khusus yang dikelola oleh OpenClaw.

Tindakan inti:

- `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
- `snapshot` (pohon aksesibilitas atau AI)
- `screenshot` (mengembalikan blok gambar + `MEDIA:<path>`)
- `act` (klik/ketik/tekan/hover/seret/pilih/isi/atur ukuran/tunggu/evaluasi)
- `navigate`, `console`, `pdf`, `upload`, `dialog`

Manajemen profil:

- `profiles` — daftar semua profil browser beserta statusnya
- `create-profile` — buat profil baru
- `delete-profile` — hapus profil (hanya lokal)

Catatan:

- Memerlukan `browser.enabled=true`.
- Jika node yang mendukung browser terhubung, alat dapat merutekannya secara otomatis.
- `snapshot` menyediakan referensi (seperti `e12`) yang dapat digunakan oleh `act`.

### `canvas`

Jalankan Canvas pada node (tampilkan, evaluasi, snapshot, A2UI).

Tindakan inti:

- `present`, `hide`, `navigate`, `eval`
- `snapshot` (mengembalikan blok gambar + `MEDIA:<path>`)
- `a2ui_push`, `a2ui_reset`

### `nodes`

Temukan dan targetkan node yang telah dipasangkan (paired); kirim notifikasi; tangkap kamera/layar.

Tindakan inti:

- `status`, `describe`
- `pending`, `approve`, `reject` (pairing)
- `notify` (notifikasi sistem macOS)
- `run` (eksekusi macOS)
- `camera_snap`, `camera_clip`, `screen_record`
- `location_get`

### `image`

Analisis gambar dengan model gambar yang dikonfigurasi.
Menggunakan model gambar secara langsung, terlepas dari model chat utama.

### `message`

Kirim pesan dan tindakan saluran di Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Tindakan inti:

- `send` (teks + media opsional)
- `poll` (jajak pendapat)
- `react` / `reactions` / `read` / `edit` / `delete`
- `pin` / `unpin` / `list-pins`
- `thread-create` / `thread-reply`
- `sticker`
- `member-info` / `role-info`
- `timeout` / `kick` / `ban`

### `cron`

Kelola tugas cron dan wakeup Gateway.

Tindakan inti:

- `status`, `list`
- `add`, `update`, `remove`, `run`, `runs`
- `wake` (masukkan event sistem ke antrean)

### `gateway`

Mulai ulang atau terapkan pembaruan pada proses Gateway yang sedang berjalan.

Tindakan inti:

- `restart`
- `config.get` / `config.apply` / `config.patch`
- `update.run` (jalankan pembaruan + muat ulang)

### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`

Daftar sesi, periksa riwayat transkrip, atau kirim pesan ke sesi lain.

Catatan:

- `sessions_spawn` memulai agen baru (sub-agen) dan memposting balasan pengumuman ke chat peminta.
- Perutean sesi dikontrol oleh `tools.sessions.visibility`.

### `agents_list`

Daftar ID agen yang dapat ditargetkan oleh sesi saat ini dengan `sessions_spawn`.

## Parameter (umum)

Alat yang didukung oleh gateway (`canvas`, `nodes`, `cron`):

- `gatewayUrl` (default `ws://127.0.0.1:18789`)
- `gatewayToken` (jika autentikasi diaktifkan)

## Alur agen yang direkomendasikan

Otomasi Browser:

1. `browser` → `status` / `start`
2. `snapshot` (ai atau aria)
3. `act` (klik/ketik/tekan)
4. `screenshot` jika butuh konfirmasi visual

Targeting Node:

1. `nodes` → `status`
2. `describe` pada node yang dipilih
3. `notify` / `run` / `camera_snap` / `screen_record`

## Keamanan

- Hindari `system.run` langsung; gunakan `nodes` → `run` hanya dengan persetujuan pengguna yang eksplisit.
- Hormati persetujuan pengguna untuk pengambilan kamera/layar.

## Bagaimana alat ditampilkan ke agen

Alat diekspos melalui dua saluran paralel:

1. **Teks prompt sistem**: daftar yang dapat dibaca manusia + panduan.
2. **Skema alat**: definisi fungsi terstruktur yang dikirim ke API model.

Ini berarti agen melihat "alat apa saja yang ada" dan "bagaimana cara memanggilnya".


