---
summary: "Pengaturan opsional berbasis Docker dan onboarding untuk OpenClaw"
read_when:
  - Anda menginginkan gateway dalam kontainer alih-alih instalasi lokal
  - Anda sedang melakukan validasi alur Docker
title: "Docker"
---

# Docker (opsional)

Docker bersifat **opsional**. Gunakan ini hanya jika Anda menginginkan gateway dalam kontainer atau untuk memvalidasi alur Docker.

## Apakah Docker tepat untuk saya?

- **Ya**: jika Anda menginginkan lingkungan gateway yang terisolasi dan sekali pakai, atau ingin menjalankan OpenClaw di host tanpa instalasi lokal.
- **Tidak**: jika Anda menjalankannya di mesin Anda sendiri dan hanya menginginkan loop pengembangan tercepat. Gunakan alur instalasi normal.
- **Catatan sandboxing**: sandboxing agen juga menggunakan Docker, tetapi hal itu **tidak** mengharuskan gateway penuh berjalan di dalam Docker. Lihat [Sandboxing](/id-ID/gateway/sandboxing).

Panduan ini mencakup:

- Gateway dalam Kontainer (OpenClaw penuh dalam Docker)
- Sandbox Agen Per-sesi (gateway host + alat agen yang terisolasi Docker)

Detail sandboxing: [Sandboxing](/id-ID/gateway/sandboxing)

## Persyaratan

- Docker Desktop (atau Docker Engine) + Docker Compose v2
- Ruang disk yang cukup untuk citra (image) + log

## Gateway dalam Kontainer (Docker Compose)

### Mulai cepat (direkomendasikan)

Dari root repositori:

```bash
./docker-setup.sh
```

Skrip ini akan:

- membangun citra (image) gateway
- menjalankan wizard onboarding
- mencetak petunjuk pengaturan penyedia opsional
- memulai gateway melalui Docker Compose
- membuat token gateway dan menulisnya ke `.env`

Variabel lingkungan (env var) opsional:

- `OPENCLAW_DOCKER_APT_PACKAGES` — menginstal paket apt tambahan selama proses build
- `OPENCLAW_EXTRA_MOUNTS` — menambahkan bind mount host tambahan
- `OPENCLAW_HOME_VOLUME` — mempertahankan `/home/node` dalam volume bernama (named volume)

Setelah selesai:

- Buka `http://127.0.0.1:18789/` di browser Anda.
- Tempelkan token ke Control UI (Settings → token).
- Butuh URL-nya lagi? Jalankan `docker compose run --rm openclaw-cli dashboard --no-open`.

Skrip ini menulis konfigurasi/workspace di host:

- `~/.openclaw/`
- `~/.openclaw/workspace`

Menjalankan di VPS? Lihat [Hetzner (Docker VPS)](/id-ID/install/hetzner).

### Pembantu Shell (opsional)

Untuk manajemen Docker harian yang lebih mudah, instal `ClawDock`:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Tambahkan ke konfigurasi shell Anda (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Kemudian gunakan perintah `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`, dll. Jalankan `clawdock-help` untuk melihat semua perintah.

Lihat [README Pembantu `ClawDock`](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) untuk detailnya.

### Alur manual (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Catatan: jalankan `docker compose ...` dari root repositori. Jika Anda mengaktifkan
`OPENCLAW_EXTRA_MOUNTS` atau `OPENCLAW_HOME_VOLUME`, skrip pengaturan akan menulis
`docker-compose.extra.yml`; sertakan file ini saat menjalankan Compose di tempat lain:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Token Control UI + pairing (Docker)

Jika Anda melihat pesan “unauthorized” atau “disconnected (1008): pairing required”, ambil
tautan dashboard baru dan setujui perangkat browser tersebut:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Detail lebih lanjut: [Dashboard](/id-ID/web/dashboard), [Perangkat (Devices)](/id-ID/cli/devices).

### Mount tambahan (opsional)

Jika Anda ingin melakukan mount direktori host tambahan ke dalam kontainer, atur
`OPENCLAW_EXTRA_MOUNTS` sebelum menjalankan `docker-setup.sh`. Ini menerima
daftar bind mount Docker yang dipisahkan koma dan menerapkannya ke
`openclaw-gateway` maupun `openclaw-cli` dengan membuat `docker-compose.extra.yml`.

Contoh:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Catatan:

- Jalur (path) harus dibagikan dengan Docker Desktop di macOS/Windows.
- Setiap entri harus berupa `source:target[:options]` tanpa spasi, tab, atau baris baru.
- Jika Anda mengedit `OPENCLAW_EXTRA_MOUNTS`, jalankan ulang `docker-setup.sh` untuk membuat ulang
  file compose ekstra.
- `docker-compose.extra.yml` dibuat secara otomatis. Jangan mengeditnya secara manual.

### Mempertahankan seluruh home kontainer (opsional)

Jika Anda ingin `/home/node` tetap ada meskipun kontainer dibuat ulang, atur
volume bernama melalui `OPENCLAW_HOME_VOLUME`. Ini akan membuat volume Docker dan melakukan mount pada
`/home/node`, sambil tetap menggunakan bind mount standar untuk konfigurasi/workspace. Gunakan
volume bernama di sini (bukan jalur bind); untuk bind mount, gunakan
`OPENCLAW_EXTRA_MOUNTS`.

Contoh:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Anda dapat menggabungkan ini dengan mount tambahan:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Catatan:

- Nama volume harus sesuai dengan pola `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
- Jika Anda mengubah `OPENCLAW_HOME_VOLUME`, jalankan ulang `docker-setup.sh` untuk membuat ulang
  file compose ekstra.
- Volume bernama akan tetap ada sampai dihapus dengan `docker volume rm <nama>`.

### Instal paket apt tambahan (opsional)

Jika Anda membutuhkan paket sistem di dalam citra (misalnya, alat build atau pustaka
media), atur `OPENCLAW_DOCKER_APT_PACKAGES` sebelum menjalankan `docker-setup.sh`.
Ini akan menginstal paket selama proses pembangunan citra (image build), sehingga tetap ada meskipun
kontainer dihapus.

Contoh:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Catatan:

- Ini menerima daftar nama paket apt yang dipisahkan spasi.
- Jika Anda mengubah `OPENCLAW_DOCKER_APT_PACKAGES`, jalankan ulang `docker-setup.sh` untuk membangun ulang
  citra tersebut.

### Kontainer untuk Power-user / Fitur Lengkap (opsional)

Citra Docker default dirancang dengan prinsip **keamanan utama (security-first)** dan berjalan sebagai pengguna non-root bernama `node`. Ini meminimalkan permukaan serangan (attack surface), namun artinya:

- tidak ada instalasi paket sistem saat runtime
- tidak ada Homebrew secara default
- tidak menyertakan browser Chromium/Playwright bawaan

Jika Anda menginginkan kontainer dengan fitur yang lebih lengkap, gunakan opsi berikut:

1. **Pertahankan `/home/node`** agar unduhan browser dan cache alat tidak hilang:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2. **Sertakan dependensi sistem ke dalam citra** (dapat diulang + permanen):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3. **Instal browser Playwright tanpa `npx`** (menghindari konflik penimpaan npm):

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Jika Anda memerlukan Playwright untuk menginstal dependensi sistem (system deps), bangun ulang citra dengan
`OPENCLAW_DOCKER_APT_PACKAGES` alih-alih menggunakan `--with-deps` saat runtime.

4. **Pertahankan unduhan browser Playwright**:

- Atur `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` di
  `docker-compose.yml`.
- Pastikan `/home/node` tetap ada melalui `OPENCLAW_HOME_VOLUME`, atau mount
  `/home/node/.cache/ms-playwright` via `OPENCLAW_EXTRA_MOUNTS`.

### Izin (Permissions) + EACCES

Citra berjalan sebagai pengguna `node` (uid 1000). Jika Anda melihat error izin pada
`/home/node/.openclaw`, pastikan bind mount host Anda dimiliki oleh uid 1000.

Contoh (host Linux):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

Jika Anda memilih untuk menjalankan sebagai root demi kenyamanan, Anda menerima konsekuensi keamanannya.

### Pembangunan ulang lebih cepat (direkomendasikan)

Untuk mempercepat pembangunan ulang, urutkan Dockerfile Anda agar layer dependensi masuk ke dalam cache.
Ini menghindari menjalankan ulang `pnpm install` kecuali file lock berubah:

```dockerfile
FROM node:22-bookworm

# Instal Bun (diperlukan untuk skrip build)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Cache dependensi kecuali metadata paket berubah
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### Pengaturan saluran (opsional)

Gunakan kontainer CLI untuk mengonfigurasi saluran, lalu restart gateway jika diperlukan.

WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (token bot):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (token bot):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Dokumentasi: [WhatsApp](/id-ID/channels/whatsapp), [Telegram](/id-ID/channels/telegram), [Discord](/id-ID/channels/discord)

### OpenAI Codex OAuth (Docker headless)

Jika Anda memilih OpenAI Codex OAuth di wizard, ia akan membuka URL browser dan mencoba
menangkap callback di `http://127.0.0.1:1455/auth/callback`. Di Docker atau
pengaturan headless, callback tersebut mungkin menunjukkan error browser. Salin URL pengalihan (redirect URL) lengkap tempat Anda mendarat dan tempelkan kembali ke wizard untuk menyelesaikan autentikasi.

### Pemeriksaan kesehatan (Health check)

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### Tes smoke E2E (Docker)

```bash
scripts/e2e/onboard-docker.sh
```

### Tes smoke impor QR (Docker)

```bash
pnpm test:docker:qr
```

### Catatan

- Bind gateway default ke `lan` untuk penggunaan kontainer.
- Perintah CMD Dockerfile menggunakan `--allow-unconfigured`; konfigurasi yang di-mount dengan `gateway.mode` bukan `local` tetap akan berjalan. Timpa CMD untuk menegakkan batasan tersebut.
- Kontainer gateway adalah sumber kebenaran (source of truth) untuk sesi (`~/.openclaw/agents/<agentId>/sessions/`).

## Sandbox Agen (gateway host + alat Docker)

Pembahasan mendalam: [Sandboxing](/id-ID/gateway/sandboxing)

### Apa fungsinya

Ketika `agents.defaults.sandbox` diaktifkan, **sesi non-utama** akan menjalankan alat di dalam kontainer Docker. Gateway tetap berada di host Anda, tetapi eksekusi alat terisolasi:

- cakupan (scope): `"agent"` secara default (satu kontainer + workspace per agen)
- cakupan (scope): `"session"` untuk isolasi per-sesi
- folder workspace per-cakupan di-mount di `/workspace`
- akses workspace agen opsional (`agents.defaults.sandbox.workspaceAccess`)
- kebijakan alat izinkan/tolak (tolak menang)
- media masuk disalin ke dalam workspace sandbox yang aktif (`media/inbound/*`) agar alat dapat membacanya (dengan `workspaceAccess: "rw"`, ini mendarat di workspace agen)

Peringatan: `scope: "shared"` menonaktifkan isolasi lintas-sesi. Semua sesi berbagi
satu kontainer dan satu workspace.

### Profil sandbox per-agen (multi-agen)

Jika Anda menggunakan perutean multi-agen, setiap agen dapat menimpa pengaturan sandbox + alat:
`agents.list[].sandbox` dan `agents.list[].tools` (ditambah `agents.list[].tools.sandbox.tools`). Ini memungkinkan Anda menjalankan
berbagai tingkat akses dalam satu gateway:

- Akses penuh (agen pribadi)
- Alat baca-saja + workspace baca-saja (agen keluarga/kerja)
- Tanpa alat sistem file/shell (agen publik)

Lihat [Multi-Agent Sandbox & Tools](/id-ID/tools/multi-agent-sandbox-tools) untuk contoh,
aturan prioritas, dan pemecahan masalah.

### Perilaku default

- Citra (image): `openclaw-sandbox:bookworm-slim`
- Satu kontainer per agen
- Akses workspace agen: `workspaceAccess: "none"` (default) menggunakan `~/.openclaw/sandboxes`
  - `"ro"` menjaga workspace sandbox di `/workspace` dan melakukan mount workspace agen secara baca-saja di `/agent` (menonaktifkan `write`/`edit`/`apply_patch`)
  - `"rw"` melakukan mount workspace agen secara baca/tulis di `/workspace`
- Bersihkan otomatis (Auto-prune): idle > 24 jam ATAU umur > 7 hari
- Jaringan: `none` secara default (pilih secara eksplisit jika Anda membutuhkan koneksi keluar/egress)
- Diizinkan secara default: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- Ditolak secara default: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Mengaktifkan sandboxing

Jika Anda berencana menginstal paket di `setupCommand`, perhatikan:

- Nilai default `docker.network` adalah `"none"` (tidak ada koneksi keluar/egress).
- `readOnlyRoot: true` memblokir instalasi paket.
- Pengguna `user` haruslah root untuk `apt-get` (kosongkan `user` atau atur `user: "0:0"`).
  OpenClaw secara otomatis membuat ulang kontainer saat `setupCommand` (atau konfigurasi docker) berubah
  kecuali kontainer tersebut **baru saja digunakan** (dalam durasi ~5 menit). Kontainer yang masih panas
  akan mencatat peringatan dengan perintah `openclaw sandbox recreate ...` yang tepat.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (defaultnya adalah agent)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/jalur/ke/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 untuk menonaktifkan pembersihan idle
          maxAgeDays: 7, // 0 untuk menonaktifkan pembersihan umur maks
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

Pengaturan pengerasan keamanan (hardening) berada di bawah `agents.defaults.sandbox.docker`:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`.

Multi-agen: timpa `agents.defaults.sandbox.{docker,browser,prune}.*` per agen melalui `agents.list[].sandbox.{docker,browser,prune}.*`
(diabaikan jika `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` bernilai `"shared"`).

### Membangun citra sandbox default

```bash
scripts/sandbox-setup.sh
```

Ini akan membangun `openclaw-sandbox:bookworm-slim` menggunakan `Dockerfile.sandbox`.

### Citra sandbox umum (opsional)

Jika Anda menginginkan citra sandbox dengan alat build umum (Node, Go, Rust, dll.), bangun citra umum:

```bash
scripts/sandbox-common-setup.sh
```

Ini akan membangun `openclaw-sandbox-common:bookworm-slim`. Untuk menggunakannya:

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } },
    },
  },
}
```

### Citra sandbox browser

Untuk menjalankan alat browser di dalam sandbox, bangun citra browser:

```bash
scripts/sandbox-browser-setup.sh
```

Ini akan membangun `openclaw-sandbox-browser:bookworm-slim` menggunakan
`Dockerfile.sandbox-browser`. Kontainer ini menjalankan Chromium dengan CDP diaktifkan dan
pengamat noVNC opsional (headful via Xvfb).

Catatan:

- Mode headful (Xvfb) mengurangi pemblokiran bot dibandingkan mode headless.
- Mode headless masih dapat digunakan dengan mengatur `agents.defaults.sandbox.browser.headless=true`.
- Tidak diperlukan lingkungan desktop penuh (GNOME); Xvfb sudah menyediakan tampilannya.

Gunakan konfigurasi:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true },
      },
    },
  },
}
```

Citra browser kustom:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } },
    },
  },
}
```

Jika diaktifkan, agen akan menerima:

- URL kontrol browser sandbox (untuk alat `browser`)
- URL noVNC (jika diaktifkan dan headless=false)

Ingat: jika Anda menggunakan daftar izinkan (allowlist) untuk alat, tambahkan `browser` (dan hapus dari
daftar tolak/deny) atau alat tersebut akan tetap terblokir.
Aturan pembersihan (`agents.defaults.sandbox.prune`) juga berlaku untuk kontainer browser.

### Citra sandbox kustom

Bangun citra Anda sendiri dan arahkan konfigurasi ke citra tersebut:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } },
    },
  },
}
```

### Kebijakan alat (izinkan/tolak)

- `deny` (tolak) menang atas `allow` (izinkan).
- Jika `allow` kosong: semua alat (kecuali yang ditolak) tersedia.
- Jika `allow` tidak kosong: hanya alat di dalam `allow` yang tersedia (dikurangi yang ditolak).

### Strategi pembersihan (pruning)

Dua pengaturan:

- `prune.idleHours`: hapus kontainer yang tidak digunakan dalam X jam (0 = nonaktif)
- `prune.maxAgeDays`: hapus kontainer yang umurnya lebih dari X hari (0 = nonaktif)

Contoh:

- Pertahankan sesi sibuk tetapi batasi masa hidupnya:
  `idleHours: 24`, `maxAgeDays: 7`
- Jangan pernah dihapus:
  `idleHours: 0`, `maxAgeDays: 0`

### Catatan keamanan

- Isolasi keras hanya berlaku untuk **alat** (exec/read/write/edit/apply_patch).
- Alat khusus host seperti browser/kamera/canvas diblokir secara default.
- Mengizinkan `browser` di sandbox **merusak isolasi** (browser berjalan di host).

## Pemecahan Masalah

- Citra hilang: bangun dengan [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) atau atur `agents.defaults.sandbox.docker.image`.
- Kontainer tidak berjalan: kontainer akan dibuat otomatis per sesi sesuai permintaan.
- Error izin di sandbox: atur `docker.user` ke UID:GID yang sesuai dengan kepemilikan
  workspace yang di-mount (atau ubah kepemilikan folder workspace).
- Alat kustom tidak ditemukan: OpenClaw menjalankan perintah dengan `sh -lc` (login shell), yang
  mengambil sumber dari `/etc/profile` dan mungkin mengatur ulang PATH. Atur `docker.env.PATH` untuk menambahkan jalur
  alat kustom Anda (misalnya, `/custom/bin:/usr/local/share/npm-global/bin`), atau tambahkan
  skrip di bawah `/etc/profile.d/` dalam Dockerfile Anda.


