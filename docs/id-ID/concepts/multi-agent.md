---
summary: "Perutean multi-agen: agen terisolasi, akun saluran, dan binding"
title: Perutean Multi-Agen
read_when: "Anda menginginkan beberapa agen terisolasi (workspace + autentikasi) dalam satu proses gateway."
status: active
---

# Perutean Multi-Agen

Tujuan: menjalankan beberapa agen yang **terisolasi** (workspace + `agentDir` + sesi terpisah), serta beberapa akun saluran (misalnya dua akun WhatsApp) dalam satu Gateway yang sedang berjalan. Pesan masuk dirutekan ke agen tertentu melalui **bindings**.

## Apa itu "Satu Agen"?

Sebuah **agen** adalah otak yang sepenuhnya memiliki cakupannya sendiri, dengan:

- **Workspace** (file, AGENTS.md/SOUL.md/USER.md, catatan lokal, aturan persona).
- **Direktori status** (`agentDir`) untuk profil autentikasi, registri model, dan konfigurasi per agen.
- **Penyimpanan sesi** (riwayat obrolan + status perutean) di bawah `~/.openclaw/agents/<agentId>/sessions`.

Profil autentikasi bersifat **per agen**. Setiap agen membaca dari filenya sendiri:

```text
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Kredensial agen utama **tidak** dibagikan secara otomatis. Jangan pernah menggunakan kembali `agentDir` yang sama untuk agen yang berbeda (itu akan menyebabkan tabrakan autentikasi/sesi). Jika Anda ingin berbagi kredensial, salin `auth-profiles.json` ke dalam `agentDir` agen tujuan.

Skill bersifat per agen melalui folder `skills/` di masing-masing workspace, dengan skill bersama yang tersedia di `~/.openclaw/skills`. Lihat [Skill: per agen vs bersama](/id-ID/tools/skills#per-agent-vs-shared-skills).

Gateway dapat menampung **satu agen** (default) atau **banyak agen** secara berdampingan.

**Catatan Workspace:** workspace setiap agen adalah **cwd (current working directory) default**, bukan sandbox yang keras. Jalur relatif akan diselesaikan di dalam workspace, tetapi jalur absolut dapat menjangkau lokasi lain di host kecuali jika sandboxing diaktifkan. Lihat [Sandboxing](/id-ID/gateway/sandboxing).

## Jalur (Peta Cepat)

- Konfigurasi: `~/.openclaw/openclaw.json` (atau `OPENCLAW_CONFIG_PATH`)
- Direktori Status: `~/.openclaw` (atau `OPENCLAW_STATE_DIR`)
- Workspace: `~/.openclaw/workspace` (atau `~/.openclaw/workspace-<agentId>`)
- Direktori Agen: `~/.openclaw/agents/<agentId>/agent` (atau `agents.list[].agentDir`)
- Sesi: `~/.openclaw/agents/<agentId>/sessions`

### Mode Agen Tunggal (Default)

Jika Anda tidak melakukan apa pun, OpenClaw menjalankan agen tunggal:

- `agentId` default-nya adalah **`main`**.
- Sesi diberi kunci sebagai `agent:main:<mainKey>`.
- Workspace default ke `~/.openclaw/workspace` (atau `~/.openclaw/workspace-<profile>` jika `OPENCLAW_PROFILE` disetel).
- Status default ke `~/.openclaw/agents/main/agent`.

## Pembantu Agen (Agent Helper)

Gunakan wizard agen untuk menambahkan agen terisolasi baru:

```bash
openclaw agents add work
```

Kemudian tambahkan `bindings` (atau biarkan wizard yang melakukannya) untuk merutekan pesan masuk.

Verifikasi dengan:

```bash
openclaw agents list --bindings
```

## Mulai Cepat

<Steps>
  <Step title="Buat workspace untuk setiap agen">

Gunakan wizard atau buat workspace secara manual:

```bash
openclaw agents add coding
openclaw agents add social
```

Setiap agen mendapatkan workspace sendiri dengan `SOUL.md`, `AGENTS.md`, dan `USER.md` (opsional), serta `agentDir` dan penyimpanan sesi khusus di bawah `~/.openclaw/agents/<agentId>`.

  </Step>

  <Step title="Buat akun saluran">

Buat satu akun per agen pada saluran pilihan Anda:

- Discord: satu bot per agen, aktifkan Message Content Intent, salin setiap token.
- Telegram: satu bot per agen melalui BotFather, salin setiap token.
- WhatsApp: tautkan setiap nomor telepon per akun.

```bash
openclaw channels login --channel whatsapp --account work
```

Lihat panduan saluran: [Discord](/id-ID/channels/discord), [Telegram](/id-ID/channels/telegram), [WhatsApp](/id-ID/channels/whatsapp).

  </Step>

  <Step title="Tambah agen, akun, dan binding">

Tambahkan agen di bawah `agents.list`, akun saluran di bawah `channels.<channel>.accounts`, dan hubungkan mereka dengan `bindings` (contoh di bawah).

  </Step>

  <Step title="Mulai ulang dan verifikasi">

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

  </Step>
</Steps>

## Banyak Agen = Banyak Orang, Banyak Kepribadian

Dengan **banyak agen**, setiap `agentId` menjadi **persona yang terisolasi sepenuhnya**:

- **Nomor telepon/akun yang berbeda** (per saluran `accountId`).
- **Kepribadian yang berbeda** (per file workspace agen seperti `AGENTS.md` dan `SOUL.md`).
- **Autentikasi + sesi terpisah** (tidak ada campur tangan kecuali diaktifkan secara eksplisit).

Ini memungkinkan **banyak orang** berbagi satu server Gateway sambil menjaga "otak" AI dan data mereka tetap terisolasi.

## Satu Nomor WhatsApp, Banyak Orang (Pemisah DM)

Anda dapat merutekan **DM WhatsApp yang berbeda** ke agen yang berbeda meskipun menggunakan **satu akun WhatsApp**. Pencocokan dilakukan berdasarkan E.164 pengirim (seperti `+628123456789`) dengan `peer.kind: "direct"`. Balasan tetap berasal dari nomor WhatsApp yang sama (tidak ada identitas pengirim per agen).

Poin penting: obrolan langsung digabungkan ke **kunci sesi utama** agen, sehingga isolasi sejati memerlukan **satu agen per orang**.

Contoh:

```json5
{
  agents: {
    list: [
      { id: "andi", workspace: "~/.openclaw/workspace-andi" },
      { id: "budi", workspace: "~/.openclaw/workspace-budi" },
    ],
  },
  bindings: [
    {
      agentId: "andi",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+62811111111" } },
    },
    {
      agentId: "budi",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+62822222222" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+62811111111", "+62822222222"],
    },
  },
}
```

Catatan:

- Kontrol akses DM bersifat **global per akun WhatsApp** (pairing/allowlist), bukan per agen.
- Untuk grup bersama, ikat grup tersebut ke satu agen atau gunakan [Grup Siaran](/id-ID/channels/broadcast-groups).

## Aturan Perutean (Bagaimana pesan memilih agen)

Binding bersifat **deterministik** dan aturan yang **paling spesifik yang akan menang**:

1.  Kecocokan `peer` (ID DM/grup/saluran yang tepat).
2.  Kecocokan `parentPeer` (pewarisan utas/THREAD).
3.  `guildId + roles` (perutean peran Discord).
4.  `guildId` (Discord).
5.  `teamId` (Slack).
6.  Kecocokan `accountId` untuk sebuah saluran.
7.  Kecocokan tingkat saluran (`accountId: "*"`).
8.  Fallback ke agen default (`agents.list[].default`, jika tidak ada maka entri daftar pertama, default: `main`).

Jika beberapa binding cocok di tingkat yang sama, yang pertama dalam urutan konfigurasi yang menang. Jika binding menyetel beberapa bidang pencocokan (misalnya `peer` + `guildId`), semua bidang yang ditentukan harus cocok (semantik `AND`).

## Banyak Akun / Nomor Telepon

Saluran yang mendukung **banyak akun** (misalnya WhatsApp) menggunakan `accountId` untuk mengidentifikasi setiap login. Setiap `accountId` dapat dirutekan ke agen yang berbeda, sehingga satu server dapat menampung banyak nomor telepon tanpa mencampur sesi.

## Konsep Utama

- `agentId`: satu "otak" (workspace, autentikasi per agen, penyimpanan sesi per agen).
- `accountId`: satu instansi akun saluran (misalnya akun WhatsApp `"pribadi"` vs `"bisnis"`).
- `binding`: merutekan pesan masuk ke `agentId` berdasarkan `(channel, accountId, peer)` dan opsional ID guild/team.
- Obrolan langsung digabungkan ke `agent:<agentId>:<mainKey>` ("utama" per agen; `session.mainKey`).

## Contoh Platform

### Bot Discord per Agen

Setiap akun bot Discord dipetakan ke `accountId` yang unik. Ikat setiap akun ke agen dan kelola daftar izinkan (allowlist) per bot.

```json5
{
  agents: {
    list: [
      { id: "utama", workspace: "~/.openclaw/workspace-utama" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "utama", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_UTAMA",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    }
  }
}
```

Catatan:

- Undang setiap bot ke guild dan aktifkan Message Content Intent.
- Token berada di `channels.discord.accounts.<id>.token` (akun default dapat menggunakan `DISCORD_BOT_TOKEN`).

### Bot Telegram per Agen

```json5
{
  agents: {
    list: [
      { id: "utama", workspace: "~/.openclaw/workspace-utama" },
      { id: "peringatan", workspace: "~/.openclaw/workspace-peringatan" },
    ],
  },
  bindings: [
    { agentId: "utama", match: { channel: "telegram", accountId: "default" } },
    { agentId: "peringatan", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

Catatan:

- Buat satu bot per agen melalui BotFather dan salin setiap tokennya.
- Token berada di `channels.telegram.accounts.<id>.botToken` (akun default dapat menggunakan `TELEGRAM_BOT_TOKEN`).

### Nomor WhatsApp per Agen

Tautkan setiap akun sebelum memulai gateway:

```bash
openclaw channels login --channel whatsapp --account pribadi
openclaw channels login --channel whatsapp --account bisnis
```

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "rumah",
        default: true,
        name: "Rumah",
        workspace: "~/.openclaw/workspace-rumah",
        agentDir: "~/.openclaw/agents/rumah/agent",
      },
      {
        id: "kerja",
        name: "Kerja",
        workspace: "~/.openclaw/workspace-kerja",
        agentDir: "~/.openclaw/agents/kerja/agent",
      },
    ],
  },

  // Perutean deterministik: pencocokan pertama menang (paling spesifik di atas).
  bindings: [
    { agentId: "rumah", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "kerja", match: { channel: "whatsapp", accountId: "biz" } },

    // Opsional: penimpa per-peer (contoh: kirim grup spesifik ke agen kerja).
    {
      agentId: "kerja",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Nonaktif secara default: pesan antar-agen harus diaktifkan eksplisit + masuk allowlist.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["rumah", "kerja"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Opsional penimpa jalur. Default: ~/.openclaw/credentials/whatsapp/personal
        },
        biz: {
          // Opsional penimpa jalur. Default: ~/.openclaw/credentials/whatsapp/biz
        },
      },
    },
  },
}
```

## Contoh: Obrolan Harian WhatsApp + Kerja Mendalam Telegram

Pemisahan berdasarkan saluran: rute WhatsApp ke agen harian yang cepat dan Telegram ke agen Opus yang cerdas.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Harian",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Catatan:

- Jika Anda memiliki banyak akun untuk satu saluran, tambahkan `accountId` ke dalam binding (misalnya `{ channel: "whatsapp", accountId: "personal" }`).
- Untuk merutekan satu DM/grup tertentu ke Opus sementara sisanya di agen chat, tambahkan binding `match.peer`; pencocokan peer selalu menang atas aturan di tingkat saluran.

## Contoh: Saluran Sama, Satu Orang ke Opus

Gunakan agen cepat untuk WhatsApp, tetapi rute satu DM ke Opus:

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Harian",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+628123456789" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Binding peer selalu menang, jadi letakkan di atas aturan tingkat saluran.

## Agen Keluarga yang Terikat ke Grup WhatsApp

Ikat agen khusus keluarga ke satu grup WhatsApp, dengan gerbang sebutan (mention) dan kebijakan alat yang lebih ketat:

```json5
{
  agents: {
    list: [
      {
        id: "keluarga",
        name: "Keluarga",
        workspace: "~/.openclaw/workspace-keluarga",
        identity: { name: "Bot Keluarga" },
        groupChat: {
          mentionPatterns: ["@keluarga", "@botkeluarga", "@Bot Keluarga"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "keluarga",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Catatan:

- Daftar izinkan/tolak alat (tool allow/deny) adalah untuk **alat**, bukan skill. Jika sebuah skill perlu menjalankan biner, pastikan `exec` diizinkan dan biner tersebut ada di dalam sandbox.
- Untuk penyaringan yang lebih ketat, setel `agents.list[].groupChat.mentionPatterns` dan pastikan daftar izinkan grup diaktifkan untuk saluran tersebut.

## Konfigurasi Sandbox dan Alat per Agen

Mulai v2026.1.6, setiap agen dapat memiliki batasan sandbox dan alatnya sendiri:

```js
{
  agents: {
    list: [
      {
        id: "pribadi",
        workspace: "~/.openclaw/workspace-pribadi",
        sandbox: {
          mode: "off",  // Tanpa sandbox untuk agen pribadi
        },
        // Tanpa batasan alat - semua alat tersedia
      },
      {
        id: "keluarga",
        workspace: "~/.openclaw/workspace-keluarga",
        sandbox: {
          mode: "all",     // Selalu di dalam sandbox
          scope: "agent",  // Satu kontainer per agen
          docker: {
            // Penyiapan opsional setelah pembuatan kontainer
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Hanya alat baca
          deny: ["exec", "write", "edit", "apply_patch"],    // Tolak yang lain
        },
      },
    ],
  },
}
```

Catatan: `setupCommand` berada di bawah `sandbox.docker` dan berjalan satu kali saat pembuatan kontainer. Penimpaan `sandbox.docker.*` per agen akan diabaikan jika scope yang diselesaikan adalah `"shared"`.

**Manfaat:**

- **Isolasi Keamanan**: Membatasi alat untuk agen yang kurang dipercaya.
- **Kontrol Sumber Daya**: Menjalankan sandbox untuk agen tertentu sambil membiarkan yang lain tetap di host.
- **Kebijakan Fleksibel**: Izin yang berbeda untuk setiap agen.

Catatan: `tools.elevated` bersifat **global** dan berbasis pengirim; ini tidak dapat dikonfigurasi per agen. Jika Anda memerlukan batasan per agen, gunakan `agents.list[].tools` untuk menolak `exec`. Untuk grup, gunakan `agents.list[].groupChat.mentionPatterns` agar sebutan @mentions dipetakan dengan tepat ke agen yang dimaksud.

Lihat [Sandbox & Alat Multi-Agen](/id-ID/tools/multi-agent-sandbox-tools) untuk contoh yang lebih detail.
