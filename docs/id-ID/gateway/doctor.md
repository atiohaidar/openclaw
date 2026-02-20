---
summary: "Perintah Doctor: pemeriksaan kesehatan, migrasi konfigurasi, dan langkah perbaikan"
read_when:
  - Menambahkan atau memodifikasi migrasi doctor
  - Memperkenalkan perubahan konfigurasi yang bersifat memutus (breaking)
title: "Doctor"
---

# Doctor

`openclaw doctor` adalah alat perbaikan + migrasi untuk OpenClaw. Perintah ini memperbaiki konfigurasi/state yang usang (stale), memeriksa kesehatan, dan memberikan langkah perbaikan yang dapat langsung dilakukan.

## Mulai cepat

```bash
openclaw doctor
```

### Headless / otomatisasi

```bash
openclaw doctor --yes
```

Menerima nilai default tanpa bertanya (termasuk langkah perbaikan restart/layanan/sandbox jika berlaku).

```bash
openclaw doctor --repair
```

Terapkan perbaikan yang direkomendasikan tanpa bertanya (perbaikan + restart jika aman).

```bash
openclaw doctor --repair --force
```

Terapkan juga perbaikan yang agresif (menimpa konfigurasi supervisor kustom).

```bash
openclaw doctor --non-interactive
```

Berjalan tanpa prompt dan hanya menerapkan migrasi yang aman (normalisasi konfigurasi + pemindahan state di disk). Melewatkan tindakan restart/layanan/sandbox yang memerlukan konfirmasi manusia. Migrasi state lama akan berjalan otomatis jika terdeteksi.

```bash
openclaw doctor --deep
```

Pindai layanan sistem untuk mencari instalasi gateway tambahan (launchd/systemd/schtasks).

Jika Anda ingin meninjau perubahan sebelum ditulis ke file, buka file konfigurasi terlebih dahulu:

```bash
cat ~/.openclaw/openclaw.json
```

## Apa yang dilakukan (ringkasan)

- Pembaruan pre-flight opsional untuk instalasi git (hanya mode interaktif).
- Pemeriksaan kesegaran protokol UI (membangun ulang Control UI jika skema protokol lebih baru).
- Pemeriksaan kesehatan + prompt restart.
- Ringkasan status Skills (yang memenuhi syarat/hilang/diblokir).
- Normalisasi konfigurasi untuk nilai-nilai lama (legacy).
- Peringatan override penyedia OpenCode Zen (`models.providers.opencode`).
- Migrasi state lama di disk (sesi/direktori agen/autentikasi WhatsApp).
- Pemeriksaan integritas state dan izin akses (sesi, transkrip, direktori state).
- Pemeriksaan izin file konfigurasi (chmod 600) saat dijalankan secara lokal.
- Kesehatan autentikasi model: memeriksa kedaluwarsa OAuth, dapat menyegarkan token yang hampir habis, dan melaporkan status cooldown/nonaktif profil autentikasi (auth-profile).
- Deteksi direktori workspace ekstra (`~/openclaw`).
- Perbaikan citra (image) sandbox saat sandboxing diaktifkan.
- Migrasi layanan lama (legacy) dan deteksi gateway tambahan.
- Pemeriksaan runtime gateway (layanan terinstal tetapi tidak berjalan; label launchd yang tersimpan di cache).
- Peringatan status saluran (dipantau dari gateway yang sedang berjalan).
- Audit konfigurasi supervisor (launchd/systemd/schtasks) dengan perbaikan opsional.
- Pemeriksaan praktik terbaik runtime gateway (perbandingan Node vs Bun, jalur pengelola versi).
- Diagnostik konflik port gateway (default `18789`).
- Peringatan keamanan untuk kebijakan DM yang terbuka (open).
- Peringatan autentikasi gateway jika tidak ada `gateway.auth.token` yang diatur (mode lokal; menawarkan pembuatan token).
- Pemeriksaan linger systemd pada Linux.
- Pemeriksaan instalasi sumber (ketidakcocokan pnpm workspace, aset UI hilang, biner tsx hilang).
- Menulis konfigurasi yang diperbarui + metadata wizard.

## Perilaku detail dan alasan (rationale)

### 0) Pembaruan opsional (instalasi git)

Jika ini adalah hasil git checkout dan doctor dijalankan secara interaktif, perintah ini akan menawarkan untuk memperbarui (fetch/rebase/build) sebelum menjalankan tugas doctor lainnya.

### 1) Normalisasi konfigurasi

Jika konfigurasi mengandung bentuk nilai lama (misalnya `messages.ackReaction` tanpa override khusus saluran), doctor akan menormalisasinya ke dalam skema saat ini.

### 2) Migrasi kunci konfigurasi lama (legacy)

Jika konfigurasi mengandung kunci yang sudah usang (deprecated), perintah lain akan menolak untuk berjalan dan meminta Anda menjalankan `openclaw doctor`.

Doctor akan:

- Menjelaskan kunci lama mana yang ditemukan.
- Menunjukkan migrasi yang diterapkan.
- Menulis ulang `~/.openclaw/openclaw.json` dengan skema terbaru.

Gateway juga secara otomatis menjalankan migrasi doctor saat startup jika mendeteksi format konfigurasi lama, sehingga konfigurasi yang usang diperbaiki tanpa campur tangan manual.

Migrasi saat ini meliputi:

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → `bindings` tingkat root
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

### 2b) Override penyedia OpenCode Zen

Jika Anda menambahkan `models.providers.opencode` (atau `opencode-zen`) secara manual, tindakan tersebut akan menimpa katalog bawaan OpenCode Zen dari `@mariozechner/pi-ai`. Hal ini dapat memaksa setiap model menggunakan satu API saja atau meniadakan perhitungan biaya. Doctor akan memberikan peringatan agar Anda dapat menghapus override tersebut dan mengembalikan perutean API serta perhitungan biaya per model.

### 3) Migrasi state lama (tata letak disk)

Doctor dapat memigrasikan tata letak file di disk yang lama ke dalam struktur saat ini:

- Penyimpanan sesi + transkrip:
  - dari `~/.openclaw/sessions/` ke `~/.openclaw/agents/<agentId>/sessions/`
- Direktori agen:
  - dari `~/.openclaw/agent/` ke `~/.openclaw/agents/<agentId>/agent/`
- Status autentikasi WhatsApp (Baileys):
  - dari `~/.openclaw/credentials/*.json` lama (kecuali `oauth.json`)
  - ke `~/.openclaw/credentials/whatsapp/<accountId>/...` (ID akun default: `default`)

Migrasi ini bersifat usaha terbaik (best-effort) dan idempoten; doctor akan mengeluarkan peringatan jika meninggalkan folder lama sebagai cadangan. Gateway/CLI juga secara otomatis memigrasikan sesi + direktori agen lama saat startup agar riwayat/autentikasi/model mendarat di jalur per-agen tanpa menjalankan doctor secara manual. Autentikasi WhatsApp sengaja hanya dimigrasikan melalui `openclaw doctor`.

### 4) Pemeriksaan integritas state (persistensi sesi, perutean, dan keamanan)

Direktori state adalah pusat operasional. Jika hilang, Anda akan kehilangan sesi, kredensial, log, dan konfigurasi (kecuali Anda memiliki cadangan di tempat lain).

Pemeriksaan yang dilakukan Doctor:

- **Direktori state hilang**: memperingatkan tentang kehilangan data yang fatal, meminta pembuatan ulang direktori, dan mengingatkan bahwa data yang hilang tidak dapat dipulihkan.
- **Izin direktori state**: memverifikasi izin penulisan; menawarkan perbaikan izin (dan memberikan petunjuk `chown` jika terdeteksi ketidakcocokan pemilik/grup).
- **Direktori sesi hilang**: folder `sessions/` dan direktori penyimpanan sesi diperlukan untuk menyimpan riwayat dan menghindari crash `ENOENT`.
- **Ketidakcocokan transkrip**: memperingatkan jika entri sesi terbaru kehilangan file transkrip.
- **Transkrip utama "1-line JSONL"**: menandai jika transkrip utama hanya berisi satu baris (riwayat tidak terakumulasi).
- **Beberapa direktori state**: memperingatkan jika terdapat beberapa folder `~/.openclaw` di berbagai direktori home atau jika `OPENCLAW_STATE_DIR` mengarah ke tempat lain (riwayat bisa terpisah di antara berbagai instalasi).
- **Pengingat mode remote**: jika `gateway.mode=remote`, doctor mengingatkan Anda untuk menjalankannya di host remote (karena state berada di sana).
- **Izin file konfigurasi**: memperingatkan jika `~/.openclaw/openclaw.json` dapat dibaca oleh grup/publik dan menawarkan untuk memperketat izinnya ke `600`.

### 5) Kesehatan autentikasi model (kedaluwarsa OAuth)

Doctor memeriksa profil OAuth di penyimpanan autentikasi, memperingatkan saat token hampir habis atau sudah kedaluwarsa, dan dapat menyegarkannya jika aman. Jika profil Anthropic Claude Code sudah usang, doctor menyarankan menjalankan `claude setup-token` (atau menempelkan setup-token). Perintah penyegaran hanya muncul saat dijalankan secara interaktif (TTY); mode `--non-interactive` akan melewatkan upaya penyegaran.

Doctor juga melaporkan profil autentikasi yang sementara tidak dapat digunakan karena:

- cooldown singkat (rate limit/timeout/kegagalan autentikasi)
- penonaktifan lebih lama (kegagalan penagihan/kredit habis)

### 6) Validasi model Hook

Jika `hooks.gmail.model` diatur, doctor memvalidasi referensi model terhadap katalog dan daftar izinkan, serta memperingatkan jika tidak dapat diselesaikan atau tidak diizinkan.

### 7) Perbaikan citra (image) Sandbox

Saat sandboxing diaktifkan, doctor memeriksa citra Docker dan menawarkan untuk membangun citra tersebut atau beralih ke nama lama jika citra saat ini tidak ditemukan.

### 8) Migrasi layanan Gateway dan petunjuk pembersihan

Doctor mendeteksi layanan gateway lama (launchd/systemd/schtasks) dan menawarkan untuk menghapusnya serta menginstal layanan OpenClaw menggunakan port gateway saat ini. Doctor juga dapat memindai layanan serupa gateway lainnya dan mencetak petunjuk pembersihan. Layanan gateway OpenClaw yang diberi nama profil dianggap sebagai layanan utama dan tidak ditandai sebagai "ekstra."

### 9) Peringatan keamanan

Doctor mengeluarkan peringatan ketika sebuah penyedia terbuka untuk DM tanpa daftar izinkan, atau ketika sebuah kebijakan dikonfigurasi dengan cara yang berbahaya.

### 10) systemd linger (Linux)

Jika berjalan sebagai layanan pengguna systemd, doctor memastikan fitur lingering diaktifkan agar gateway tetap hidup setelah logout.

### 11) Status Skills

Doctor mencetak ringkasan singkat tentang skill yang memenuhi syarat/hilang/diblokir untuk workspace saat ini.

### 12) Pemeriksaan autentikasi Gateway (token lokal)

Doctor memperingatkan ketika `gateway.auth` hilang pada gateway lokal dan menawarkan untuk membuat token. Gunakan `openclaw doctor --generate-gateway-token` untuk memaksa pembuatan token dalam otomatisasi.

### 13) Pemeriksaan kesehatan Gateway + restart

Doctor menjalankan pemeriksaan kesehatan dan menawarkan untuk merestart gateway jika terlihat tidak sehat.

### 14) Peringatan status saluran

Jika gateway sehat, doctor menjalankan pemeriksaan status saluran dan melaporkan peringatan beserta saran perbaikannya.

### 15) Audit + perbaikan konfigurasi Supervisor

Doctor memeriksa konfigurasi supervisor yang terinstal (launchd/systemd/schtasks) untuk mencari default yang hilang atau usang (misalnya: dependensi jaringan online systemd dan jeda restart). Jika ditemukan ketidakcocokan, doctor merekomendasikan pembaruan dan dapat menulis ulang file layanan/tugas ke nilai default terbaru.

Catatan:

- `openclaw doctor` meminta konfirmasi sebelum menulis ulang konfigurasi supervisor.
- `openclaw doctor --yes` menerima perintah perbaikan default.
- `openclaw doctor --repair` menerapkan perbaikan yang direkomendasikan tanpa prompt.
- `openclaw doctor --repair --force` menimpa konfigurasi supervisor kustom.
- Anda selalu dapat memaksa penulisan ulang penuh melalui `openclaw gateway install --force`.

### 16) Diagnostik runtime + port Gateway

Doctor memeriksa runtime layanan (PID, status keluar terakhir) dan memperingatkan ketika layanan terinstal tetapi tidak benar-benar berjalan. Doctor juga memeriksa konflik port pada port gateway (default `18789`) dan melaporkan kemungkinan penyebabnya (gateway sudah berjalan, SSH tunnel).

### 17) Praktik terbaik runtime Gateway

Doctor memberikan peringatan ketika layanan gateway berjalan di Bun atau jalur Node yang dikelola versi (`nvm`, `fnm`, `volta`, `asdf`, dll.). Saluran WhatsApp + Telegram memerlukan Node, dan jalur pengelola versi dapat rusak setelah upgrade karena layanan tidak memuat inisialisasi shell Anda. Doctor menawarkan untuk bermigrasi ke instalasi Node sistem jika tersedia (Homebrew/apt/choco).

### 18) Penulisan konfigurasi + metadata Wizard

Doctor menyimpan setiap perubahan konfigurasi dan mencantumkan metadata wizard untuk mencatat jalannya perintah doctor tersebut.

### 19) Tips Workspace (cadangan + sistem memori)

Doctor menyarankan sistem memori workspace jika belum ada dan mencetak tips cadangan jika workspace belum menggunakan git.

Lihat [/id-ID/concepts/agent-workspace](/id-ID/concepts/agent-workspace) untuk panduan lengkap struktur workspace dan cadangan git (disarankan menggunakan GitHub atau GitLab privat).


