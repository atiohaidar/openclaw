---
title: "Referensi Konfigurasi"
description: "Referensi lengkap bidang demi bidang (field-by-field) untuk ~/.openclaw/openclaw.json"
---

# Referensi Konfigurasi

Setiap bidang (field) yang tersedia di `~/.openclaw/openclaw.json`. Untuk ringkasan yang berorientasi pada tugas, lihat [Konfigurasi](/id-ID/gateway/configuration).

Format konfigurasi adalah **JSON5** (komentar + koma terakhir diperbolehkan). Semua bidang bersifat opsional — OpenClaw menggunakan default yang aman jika dikosongkan.

---

## Channels (Saluran)

Setiap saluran dimulai secara otomatis ketika bagian konfigurasinya ada (kecuali `enabled: false`).

### Akses DM dan grup

Semua saluran mendukung kebijakan DM dan kebijakan grup:

| Kebijakan DM        | Perilaku                                                                                      |
| ------------------- | --------------------------------------------------------------------------------------------- |
| `pairing` (default) | Pengirim yang tidak dikenal mendapatkan kode pairing sekali pakai; pemilik harus menyetujui   |
| `allowlist`         | Hanya pengirim yang ada di `allowFrom` (atau penyimpanan allow yang sudah dipasangkan/paired) |
| `open`              | Izinkan semua DM masuk (memerlukan `allowFrom: ["*"]`)                                        |
| `disabled`          | Abaikan semua DM masuk                                                                        |

| Kebijakan Grup        | Perilaku                                                                   |
| --------------------- | -------------------------------------------------------------------------- |
| `allowlist` (default) | Hanya grup yang cocok dengan allowlist yang dikonfigurasi                  |
| `open`                | Melewati allowlist grup (penyaringan sebutan/mention-gating tetap berlaku) |
| `disabled`            | Blokir semua pesan grup/ruangan                                            |

<Note>
`channels.defaults.groupPolicy` menetapkan nilai default ketika `groupPolicy` penyedia tidak diatur.
Kode pairing akan kedaluwarsa setelah 1 jam. Permintaan pairing DM yang tertunda dibatasi hingga **3 per saluran**.
Slack/Discord memiliki cadangan (fallback) khusus: jika bagian penyedianya hilang sama sekali, kebijakan grup saat runtime dapat diselesaikan menjadi `open` (dengan peringatan saat startup).
</Note>

### WhatsApp

WhatsApp berjalan melalui saluran web gateway (Baileys Web). Saluran ini dimulai secara otomatis ketika ada sesi yang terhubung.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // centang biru (false dalam mode self-chat)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-akun WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Perintah keluar (outbound) secara default menggunakan akun `default` jika ada; jika tidak, menggunakan ID akun pertama yang dikonfigurasi (berdasarkan abjad).
- Direktori autentikasi Baileys akun tunggal yang lama akan dimigrasikan oleh `openclaw doctor` ke `whatsapp/default`.
- Override per akun: `channels.whatsapp.accounts.<id>.sendReadReceipts`, `channels.whatsapp.accounts.<id>.dmPolicy`, `channels.whatsapp.accounts.<id>.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot token: `channels.telegram.botToken` atau `channels.telegram.tokenFile`, dengan `TELEGRAM_BOT_TOKEN` sebagai cadangan (fallback) untuk akun default.
- `configWrites: false` memblokir penulisan konfigurasi yang dimulai dari Telegram (migrasi ID supergroup, `/config set|unset`).
- Pratinjau aliran (stream) Telegram menggunakan `sendMessage` + `editMessageText` (berfungsi di obrolan langsung dan grup).
- Kebijakan coba ulang (retry policy): lihat [Kebijakan coba ulang](/id-ID/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- Token: `channels.discord.token`, dengan `DISCORD_BOT_TOKEN` sebagai cadangan untuk akun default.
- Gunakan `user:<id>` (DM) atau `channel:<id>` (guild channel) untuk target pengiriman; ID numerik saja akan ditolak.
- Slug guild ditulis dengan huruf kecil dan spasi ganti dengan `-`; kunci saluran menggunakan nama yang di-slug-kan (tanpa `#`). Lebih disarankan menggunakan ID guild.
- Pesan yang ditulis oleh bot diabaikan secara default. `allowBots: true` mengaktifkannya (pesan bot sendiri tetap difilter).
- `maxLinesPerMessage` (default 17) memisahkan pesan yang tinggi meskipun di bawah 2000 karakter.
- `channels.discord.ui.components.accentColor` menetapkan warna aksen untuk kontainer komponen Discord v2.

**Mode notifikasi reaksi:** `off` (tidak ada), `own` (pesan bot, default), `all` (semua pesan), `allowlist` (dari `guilds.<id>.users` pada semua pesan).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- JSON service account: inline (`serviceAccount`) atau berbasis file (`serviceAccountFile`).
- Cadangan variabel lingkungan: `GOOGLE_CHAT_SERVICE_ACCOUNT` atau `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Gunakan `spaces/<spaceId>` atau `users/<userId|email>` untuk target pengiriman.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Mode Socket** memerlukan `botToken` dan `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` sebagai cadangan akun default).
- **Mode HTTP** memerlukan `botToken` plus `signingSecret` (di tingkat root atau per akun).
- `configWrites: false` memblokir penulisan konfigurasi yang dimulai dari Slack.
- Gunakan `user:<id>` (DM) atau `channel:<id>` untuk target pengiriman.

**Mode notifikasi reaksi:** `off`, `own` (default), `all`, `allowlist` (dari `reactionAllowlist`).

**Isolasi sesi thread:** `thread.historyScope` adalah per-thread (default) atau dibagikan di seluruh saluran (shared across channel). `thread.inheritParent` menyalin transkrip saluran induk ke thread baru.

| Grup Tindakan | Default    | Catatan                        |
| ------------- | ---------- | ------------------------------ |
| reactions     | diaktifkan | Bereaksi + daftar reaksi       |
| messages      | diaktifkan | Baca/kirim/edit/hapus          |
| pins          | diaktifkan | Sematkan/lepas sematan/perinci |
| memberInfo    | diaktifkan | Informasi anggota              |
| emojiList     | diaktifkan | Daftar emoji kustom            |

### Mattermost

Mattermost dikirimkan sebagai plugin: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Mode obrolan: `oncall` (respons saat disebutkan/@-mention, default), `onmessage` (setiap pesan), `onchar` (pesan yang dimulai dengan awalan pemicu).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Mode notifikasi reaksi:** `off`, `own` (default), `all`, `allowlist` (dari `reactionAllowlist`).

### iMessage

OpenClaw meluncurkan `imsg rpc` (JSON-RPC melalui stdio). Tidak diperlukan daemon atau port.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Memerlukan akses Full Disk Access ke DB Pesan (Messages DB).
- Lebih disarankan menggunakan target `chat_id:<id>`. Gunakan `imsg chats --limit 20` untuk melihat daftar obrolan.
- `cliPath` dapat mengarah ke pembungkus (wrapper) SSH; atur `remoteHost` (`host` atau `user@host`) untuk pengambilan lampiran SCP.
- `attachmentRoots` dan `remoteAttachmentRoots` membatasi jalur lampiran masuk (default: `/Users/*/Library/Messages/Attachments`).
- SCP menggunakan pemeriksaan host-key yang ketat, jadi pastikan kunci relai host sudah ada di `~/.ssh/known_hosts`.

<Accordion title="Contoh wrapper SSH iMessage">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Multi-akun (semua saluran)

Jalankan banyak akun per saluran (masing-masing dengan `accountId` sendiri):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `default` digunakan ketika `accountId` dikosongkan (CLI + perutean).
- Token variabel lingkungan hanya berlaku untuk akun **default**.
- Pengaturan saluran dasar berlaku untuk semua akun kecuali jika ditimpa (overridden) per akun.
- Gunakan `bindings[].match.accountId` untuk mengarahkan setiap akun ke agen yang berbeda.

### Penyaringan sebutan (mention gating) obrolan grup

Pesan grup secara default **memerlukan sebutan** (sebutan metadata atau pola regex). Berlaku untuk obrolan grup WhatsApp, Telegram, Discord, Google Chat, dan iMessage.

**Tipe sebutan (mention):**

- **Sebutan metadata**: Sebutan asli platform @-mentions. Diabaikan dalam mode self-chat WhatsApp.
- **Pola teks**: Pola regex dalam `agents.list[].groupChat.mentionPatterns`. Selalu diperiksa.
- Penyaringan sebutan hanya ditegakkan ketika deteksi memungkinkan (sebutan asli atau setidaknya satu pola).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` menetapkan nilai default global. Saluran dapat menimpanya dengan `channels.<channel>.historyLimit` (atau per akun). Atur ke `0` untuk menonaktifkan.

#### Batas riwayat DM

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Penyelesaian (Resolution): override per-DM → default penyedia → tidak ada batas (semua disimpan).

Didukung: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Mode self-chat

Masukkan nomor Anda sendiri dalam `allowFrom` untuk mengaktifkan mode self-chat (mengabaikan @-mention asli, hanya merespons pola teks):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["balas", "@openclaw"] },
      },
    ],
  },
}
```

### Perintah (Chat command handling)

```json5
{
  commands: {
    native: "auto", // daftarkan perintah asli jika didukung
    text: true, // uraikan /perintah dalam pesan obrolan
    bash: false, // izinkan ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // izinkan /config
    debug: false, // izinkan /debug
    restart: false, // izinkan /restart + alat restart gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Detail perintah">

- Perintah teks harus berupa pesan **mandiri** dengan awalan `/`.
- `native: "auto"` mengaktifkan perintah asli untuk Discord/Telegram, membiarkan Slack mati.
- Override per saluran: `channels.discord.commands.native` (boolean atau `"auto"`). `false` menghapus perintah yang terdaftar sebelumnya.
- `channels.telegram.customCommands` menambahkan entri menu bot Telegram tambahan.
- `bash: true` mengaktifkan `! <cmd>` untuk host shell. Memerlukan `tools.elevated.enabled` dan pengirim berada di `tools.elevated.allowFrom.<channel>`.
- `config: true` mengaktifkan `/config` (membaca/menulis `openclaw.json`).
- `channels.<provider>.configWrites` membatasi mutasi konfigurasi per saluran (default: true).
- `allowFrom` adalah per-penyedia. Jika diatur, ini menjadi **satu-satunya** sumber otorisasi (allowlist/pairing saluran dan `useAccessGroups` akan diabaikan).
- `useAccessGroups: false` memungkinkan perintah melewati kebijakan grup-akses (access-group) ketika `allowFrom` tidak diatur.

</Accordion>

---

## Agent defaults (Default agen)

### `agents.defaults.workspace`

Default: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Opsional, root repositori yang ditampilkan pada baris Runtime di prompt sistem. Jika tidak diatur, OpenClaw akan mendeteksi secara otomatis dengan menelusuri folder ke atas dari workspace.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Menonaktifkan pembuatan otomatis file bootstrap workspace (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Jumlah karakter maksimum per file bootstrap workspace sebelum dipotong (truncation). Default: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Total karakter maksimum yang dimasukkan di seluruh file bootstrap workspace. Default: `150000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 150000 } },
}
```

### `agents.defaults.imageMaxDimensionPx`

Ukuran piksel maksimum untuk sisi terpanjang gambar dalam blok gambar transkrip/alat sebelum panggilan ke penyedia.
Default: `1200`.

Nilai yang lebih rendah biasanya mengurangi penggunaan vision-token dan ukuran muatan permintaan untuk sesi yang memiliki banyak tangkapan layar.
Nilai yang lebih tinggi menjaga detail visual lebih baik.

```json5
{
  agents: { defaults: { imageMaxDimensionPx: 1200 } },
}
```

### `agents.defaults.userTimezone`

Zona waktu untuk konteks prompt sistem (bukan stempel waktu pesan). Menggunakan zona waktu host sebagai cadangan.

```json5
{
  agents: { defaults: { userTimezone: "Asia/Jakarta" } },
}
```

### `agents.defaults.timeFormat`

Format waktu dalam prompt sistem. Default: `auto` (preferensi OS).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: format `provider/model` (cth. `anthropic/claude-opus-4-6`). Jika Anda mengabaikan penyedia, OpenClaw mengasumsikan `anthropic` (usang).
- `models`: katalog model yang dikonfigurasi dan allowlist untuk `/model`. Setiap entri dapat menyertakan `alias` (singkatan) dan `params` (khusus penyedia: `temperature`, `maxTokens`).
- `imageModel`: hanya digunakan jika model utama tidak mendukung input gambar.
- `maxConcurrent`: jumlah maksimum jalannya agen secara paralel di seluruh sesi (setiap sesi tetap diserialkan). Default: 1.

**Singkatan alias bawaan** (hanya berlaku jika model ada di `agents.defaults.models`):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Alias yang Anda konfigurasi akan selalu diutamakan daripada default.

Model Z.AI GLM-4.x secara otomatis mengaktifkan mode berpikir (thinking mode) kecuali Anda mengatur `--thinking off` atau menentukan sendiri `agents.defaults.models["zai/<model>"].params.thinking`.
Model Z.AI mengaktifkan `tool_stream` secara default untuk streaming panggilan alat. Atur `agents.defaults.models["zai/<model>"].params.tool_stream` ke `false` untuk menonaktifkannya.

### `agents.defaults.cliBackends`

Backend CLI opsional untuk jalannya teks saja (tanpa panggilan alat). Berguna sebagai cadangan ketika penyedia API gagal.

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- Backend CLI mendahulukan teks; alat selalu dinonaktifkan.
- Sesi didukung ketika `sessionArg` diatur.
- Terusan gambar (image pass-through) didukung ketika `imageArg` menerima jalur file.

### `agents.defaults.heartbeat`

Jalannya heartbeat secara periodik.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m menonaktifkan
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Baca HEARTBEAT.md jika ada...",
        ackMaxChars: 300,
        suppressToolErrorWarnings: false,
      },
    },
  },
}
```

- `every`: string durasi (ms/s/m/h). Default: `30m`.
- `suppressToolErrorWarnings`: jika true, menyembunyikan payload peringatan kesalahan alat selama jalannya heartbeat.
- Per-agen: atur `agents.list[].heartbeat`. Ketika ada agen yang menentukan `heartbeat`, **hanya agen tersebut** yang menjalankan heartbeat.
- Heartbeat menjalankan giliran agen penuh — interval yang lebih pendek menghabiskan lebih banyak token.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Sesi mendekati kompaksi (compaction). Simpan memori tahan lama sekarang.",
          prompt: "Tulis catatan penting apa pun ke memory/YYYY-MM-DD.md; balas dengan NO_REPLY jika tidak ada yang perlu disimpan.",
        },
      },
    },
  },
}
```

- `mode`: `default` atau `safeguard` (peringkasan terpotong-potong/chunked untuk riwayat yang panjang). Lihat [Kompaksi](/id-ID/concepts/compaction).
- `memoryFlush`: giliran agen secara diam-diam (silent agentic turn) sebelum kompaksi otomatis untuk menyimpan memori yang tahan lama (durable memories). Dilewati (skipped) ketika workspace bersifat baca-saja.

### `agents.defaults.contextPruning`

Memangkas **hasil alat yang lama** dari konteks dalam-memori sebelum dikirim ke LLM. **Tidak** memodifikasi riwayat sesi di disk.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // durasi (ms/s/m/h), unit default: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Konten hasil alat lama dibersihkan]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="Perilaku mode cache-ttl">

- `mode: "cache-ttl"` mengaktifkan pemangkasan (pruning passes).
- `ttl` mengontrol seberapa sering pemangkasan dapat dijalankan kembali (setelah sentuhan cache terakhir).
- Pemangkasan melakukan pemangkasan halus (soft-trim) pada hasil alat yang terlalu besar terlebih dahulu, kemudian membersihkan secara tuntas (hard-clear) hasil alat yang lebih lama jika diperlukan.

**Soft-trim** menyisakan bagian awal + akhir dan memasukkan `...` di tengah.

**Hard-clear** mengganti seluruh hasil alat dengan placeholder.

Catatan:

- Blok gambar tidak pernah dipangkas/dibersihkan.
- Rasio didasarkan pada jumlah karakter (perkiraan), bukan jumlah token yang tepat.
- Jika terdapat kurang dari `keepLastAssistants` pesan asisten, pemangkasan akan dilewati.

</Accordion>

Lihat [Pemangkasan Sesi](/id-ID/concepts/session-pruning) untuk detail perilaku.

### Block streaming (Streaming blok)

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (gunakan minMs/maxMs)
    },
  },
}
```

- Saluran non-Telegram memerlukan `*.blockStreaming: true` yang eksplisit untuk mengaktifkan balasan blok.
- Override saluran: `channels.<channel>.blockStreamingCoalesce` (dan varian per akun). Signal/Slack/Discord/Google Chat defaultnya adalah `minChars: 1500`.
- `humanDelay`: jeda acak di antara balasan blok. `natural` = 800–2500ms. Override per agen: `agents.list[].humanDelay`.

Lihat [Streaming](/id-ID/concepts/streaming) untuk perilaku + detail chunking.

### Indikator mengetik (Typing indicators)

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- Default: `instant` untuk obrolan langsung/sebutan, `message` untuk obrolan grup yang tidak menyebutkan agen.
- Override per sesi: `session.typingMode`, `session.typingIntervalSeconds`.

Lihat [Indikator Mengetik](/id-ID/concepts/typing-indicators).

### `agents.defaults.sandbox`

Opsional, **Docker sandboxing** untuk agen yang tertanam. Lihat [Sandboxing](/id-ID/gateway/sandboxing) untuk panduan lengkap.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
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
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
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
          "apply_patch",
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

<Accordion title="Detail Sandbox">

**Akses Workspace:**

- `none`: workspace sandbox per cakupan (per-scope) di bawah `~/.openclaw/sandboxes`
- `ro`: workspace sandbox di `/workspace`, workspace agen di-mount baca-saja di `/agent`
- `rw`: workspace agen di-mount baca/tulis di `/workspace`

**Cakupan (Scope):**

- `session`: kontainer + workspace per sesi
- `agent`: satu kontainer + workspace per agen (default)
- `shared`: kontainer dan workspace bersama (tidak ada isolasi lintas-sesi)

**`setupCommand`** berjalan sekali setelah pembuatan kontainer (via `sh -lc`). Membutuhkan akses internet (network egress), root yang dapat ditulis, pengguna root.

**Kontainer defaultnya menggunakan `network: "none"`** — atur ke `"bridge"` jika agen membutuhkan akses keluar.

**Lampiran masuk (Inbound attachments)** diletakkan (staged) ke `media/inbound/*` di workspace aktif.

**`docker.binds`** melakukan mount direktori host tambahan; bind global dan per-agen akan digabungkan.

**Browser sandbox** (`sandbox.browser.enabled`): Chromium + CDP dalam kontainer. URL noVNC dimasukkan ke dalam prompt sistem. Tidak memerlukan `browser.enabled` di konfigurasi utama.

- `allowHostControl: false` (default) memblokir sesi sandbox agar tidak menargetkan browser host.
- `sandbox.browser.binds` melakukan mount direktori host tambahan hanya ke dalam kontainer browser sandbox. Jika diatur (termasuk `[]`), ini menggantikan `docker.binds` untuk kontainer browser.

</Accordion>

Bangun citra (images):

```bash
scripts/sandbox-setup.sh           # citra sandbox utama
scripts/sandbox-browser-setup.sh   # citra browser opsional
```

### `agents.list` (override per agen)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // atau { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: ID agen yang stabil (wajib).
- `default`: jika lebih dari satu yang diatur, yang pertama menang (peringatan akan dicatat). Jika tidak ada yang diatur, entri pertama adalah default.
- `model`: bentuk string hanya menimpa `primary`; bentuk objek `{ primary, fallbacks }` menimpa keduanya (`[]` menonaktifkan cadangan global). Pekerjaan Cron yang hanya menimpa `primary` tetap mewarisi cadangan default kecuali Anda mengatur `fallbacks: []`.
- `identity.avatar`: jalur relatif terhadap workspace, URL `http(s)`, atau URI `data:`.
- `identity` menurunkan nilai default: `ackReaction` dari `emoji`, `mentionPatterns` dari `name`/`emoji`.
- `subagents.allowAgents`: allowlist ID agen untuk `sessions_spawn` (`["*"]` = mana saja; default: hanya agen yang sama).

---

## Multi-agent routing (Perutean multi-agen)

Menjalankan beberapa agen terisolasi di dalam satu Gateway. Lihat [Multi-Agen](/id-ID/concepts/multi-agent).

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

### Bidang pencocokan binding (Binding match fields)

- `match.channel` (wajib)
- `match.accountId` (opsional; `*` = semua akun; dikosongkan = akun default)
- `match.peer` (opsional; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (opsional; khusus saluran)

**Urutan pencocokan deterministik:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (tepat, tanpa peer/guild/team)
5. `match.accountId: "*"` (seluruh saluran)
6. Agen default

Di dalam setiap tingkatan, entri `bindings` pertama yang cocok akan menang.

### Profil akses per agen

<Accordion title="Akses penuh (tanpa sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Alat baca-saja (read-only) + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Tanpa akses sistem file (hanya pesan)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

Lihat [Sandbox & Alat Multi-Agen](/id-ID/tools/multi-agent-sandbox-tools) untuk detail prioritas.

---

## Session (Sesi)

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // usang (runtime selalu menggunakan "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Detail bidang Sesi">

- **`dmScope`**: cara pengelompokan DM.
  - `main`: semua DM berbagi sesi utama.
  - `per-peer`: isolasi berdasarkan ID pengirim lintas saluran.
  - `per-channel-peer`: isolasi per saluran + pengirim (disarankan untuk inbox multi-pengguna).
  - `per-account-channel-peer`: isolasi per akun + saluran + pengirim (disarankan untuk multi-akun).
- **`identityLinks`**: memetakan ID kanonik ke peer berawalan-penyedia untuk berbagi sesi lintas saluran.
- **`reset`**: kebijakan reset utama. `daily` melakukan reset pada `atHour` waktu lokal; `idle` melakukan reset setelah `idleMinutes`. Jika keduanya dikonfigurasi, mana pun yang kedaluwarsa lebih dulu akan menang.
- **`resetByType`**: override per tipe (`direct`, `group`, `thread`). Nama lama `dm` diterima sebagai alias untuk `direct`.
- **`mainKey`**: bidang usang. Runtime sekarang selalu menggunakan `"main"` untuk bucket obrolan langsung utama.
- **`sendPolicy`**: pencocokan berdasarkan `channel`, `chatType` (`direct|group|channel`, dengan alias lama `dm`), `keyPrefix`, atau `rawKeyPrefix`. Deny pertama akan menang.
- **`maintenance`**: `warn` memperingatkan sesi aktif saat ada pengeluaran (eviction); `enforce` menerapkan pemangkasan dan rotasi.

</Accordion>

---

## Messages (Pesan)

```json5
{
  messages: {
    responsePrefix: "🦞", // atau "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 menonaktifkan
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Response prefix (Awalan respons)

Override per saluran/akun: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`.

Penyelesaian (Paling spesifik yang menang): akun → saluran → global. `""` menonaktifkan dan menghentikan bertingkat (cascade). `"auto"` mengambil dari `[{identity.name}]`.

**Variabel template:**

| Variabel          | Deskripsi                  | Contoh                      |
| ----------------- | -------------------------- | --------------------------- |
| `{model}`         | Nama pendek model          | `claude-opus-4-6`           |
| `{modelFull}`     | Identifikasi model lengkap | `anthropic/claude-opus-4-6` |
| `{provider}`      | Nama penyedia (provider)   | `anthropic`                 |
| `{thinkingLevel}` | Tingkat berpikir saat ini  | `high`, `low`, `off`        |
| `{identity.name}` | Nama identitas agen        | (sama dengan `"auto"`)      |

Variabel tidak peka huruf besar-kecil (case-insensitive). `{think}` adalah alias untuk `{thinkingLevel}`.

### Ack reaction (Reaksi pengakuan)

- Default ke `identity.emoji` agen aktif, jika tidak ada maka `"👀"`. Setel `""` untuk menonaktifkan.
- Override per saluran: `channels.<channel>.ackReaction`, `channels.<channel>.accounts.<id>.ackReaction`.
- Urutan penyelesaian: akun → saluran → `messages.ackReaction` → cadangan identitas.
- Cakupan (Scope): `group-mentions` (default), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: menghapus reaksi ack setelah membalas (khusus Slack/Discord/Telegram/Google Chat).

### Inbound debounce (Batasan masuk)

Mengelompokkan pesan teks saja yang cepat dari pengirim yang sama ke dalam satu giliran agen. Media/lampiran akan dikirimkan segera. Perintah kontrol mengabaikan batasan (debouncing).

### TTS (text-to-speech)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` mengontrol auto-TTS. `/tts off|always|inbound|tagged` menimpa per sesi.
- `summaryModel` menimpa `agents.defaults.model.primary` untuk auto-summary.
- Kunci API menggunakan cadangan `ELEVENLABS_API_KEY`/`XI_API_KEY` dan `OPENAI_API_KEY`.

---

## Talk

Default untuk mode Talk (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- ID Suara menggunakan cadangan `ELEVENLABS_VOICE_ID` atau `SAG_VOICE_ID`.
- `apiKey` menggunakan cadangan `ELEVENLABS_API_KEY`.
- `voiceAliases` memungkinkan arahan Talk menggunakan nama yang lebih ramah.

---

## Tools (Alat)

### Tool profiles (Profil alat)

`tools.profile` menetapkan allowlist dasar sebelum `tools.allow`/`tools.deny`:

| Profil      | Termasuk                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | hanya `session_status`                                                                    |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Tidak ada batasan (sama seperti tidak diatur)                                             |

### Tool groups (Grup alat)

| Grup               | Alat                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` diterima sebagai alias untuk `exec`)                           |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Semua alat bawaan (tidak termasuk plugin penyedia)                                       |

### `tools.allow` / `tools.deny`

Kebijakan global izinkan/tolak alat (tolak menang). Tidak peka huruf besar-kecil, mendukung wildcard `*`. Diterapkan bahkan ketika Docker sandbox dimatikan.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Membatasi alat lebih lanjut untuk penyedia atau model tertentu. Urutan: profil dasar → profil penyedia → izinkan/tolak.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Mengontrol akses jalan host yang ditingkatkan (elevated):

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- Override per agen (`agents.list[].tools.elevated`) hanya dapat membatasi lebih lanjut.
- `/elevated on|off|ask|full` menyimpan status per sesi; arahan inline berlaku untuk satu pesan saja.
- `exec` yang ditingkatkan berjalan di host, melewati sandboxing.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.loopDetection` (Deteksi loop)

Pemeriksaan keamanan loop-alat **dinonaktifkan secara default**. Atur `enabled: true` untuk mengaktifkan deteksi.
Pengaturan dapat ditentukan secara global di `tools.loopDetection` dan ditimpa per agen di `agents.list[].tools.loopDetection`.

```json5
{
  tools: {
    loopDetection: {
      enabled: true,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

- `historySize`: riwayat panggilan alat maksimum yang disimpan untuk analisis loop.
- `warningThreshold`: ambang batas pola pengulangan tanpa-kemajuan untuk peringatan.
- `criticalThreshold`: ambang batas pengulangan yang lebih tinggi untuk memblokir loop kritis.
- `globalCircuitBreakerThreshold`: ambang batas penghentian keras untuk semua jalan tanpa-kemajuan.
- `detectors.genericRepeat`: peringatan pada panggilan alat yang sama/argumen yang sama berulang kali.
- `detectors.knownPollNoProgress`: peringatan/blokir pada alat jajak (poll) yang dikenal (`process.poll`, `command_status`, dst.).
- `detectors.pingPong`: peringatan/blokir pada pola pasangan tanpa-kemajuan yang bergantian.
- Jika `warningThreshold >= criticalThreshold` atau `criticalThreshold >= globalCircuitBreakerThreshold`, validasi gagal.

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // atau env BRAVE_API_KEY
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Konfigurasi pemahaman media masuk (gambar/audio/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Bidang entri model media">

**Entri Provider** (`type: "provider"` atau dikosongkan):

- `provider`: ID penyedia API (`openai`, `anthropic`, `google`/`gemini`, `groq`, dst.)
- `model`: override ID model
- `profile` / `preferredProfile`: pemilihan profil autentikasi

**Entri CLI** (`type: "cli"`):

- `command`: executable yang akan dijalankan
- `args`: argumen dengan template (mendukung `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, dst.)

**Bidang umum:**

- `capabilities`: daftar opsional (`image`, `audio`, `video`). Default: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: override per entri.
- Kegagalan akan beralih (fallback) ke entri berikutnya.

Autentikasi penyedia mengikuti urutan standar: profil autentikasi → variabel lingkungan → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.sessions`

Mengontrol sesi mana yang dapat ditargetkan oleh alat sesi (`sessions_list`, `sessions_history`, `sessions_send`).

Default: `tree` (sesi saat ini + sesi yang dihasilkan darinya, seperti sub-agen).

```json5
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      visibility: "tree",
    },
  },
}
```

Catatan:

- `self`: hanya kunci sesi saat ini.
- `tree`: sesi saat ini + sesi yang dihasilkan oleh sesi saat ini (sub-agen).
- `agent`: sesi apa pun yang dimiliki oleh ID agen saat ini (dapat mencakup pengguna lain jika Anda menjalankan sesi per-pengirim di bawah ID agen yang sama).
- `all`: sesi apa pun. Penargetan lintas-agen tetap memerlukan `tools.agentToAgent`.
- Sandbox clamp: ketika sesi saat ini di-sandbox dan `agents.defaults.sandbox.sessionToolsVisibility="spawned"`, visibilitas dipaksa ke `tree` meskipun `tools.sessions.visibility="all"`.

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: model default untuk sub-agen yang dihasilkan. Jika dikosongkan, sub-agen mewarisi model pemanggil.
- Kebijakan alat per sub-agen: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Custom providers and base URLs (Penyedia kustom dan URL dasar)

OpenClaw menggunakan katalog model pi-coding-agent. Tambahkan penyedia kustom melalui `models.providers` di konfigurasi atau `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- Gunakan `authHeader: true` + `headers` untuk kebutuhan autentikasi kustom.
- Override root konfigurasi agen dengan `OPENCLAW_AGENT_DIR` (atau `PI_CODING_AGENT_DIR`).

### Contoh penyedia

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Gunakan `cerebras/zai-glm-4.7` untuk Cerebras; `zai/glm-4.7` untuk akses langsung Z.AI.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

Atur `OPENCODE_API_KEY` (atau `OPENCODE_ZEN_API_KEY`). Singkatan: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Atur `ZAI_API_KEY`. `z.ai/*` dan `z-ai/*` adalah alias yang diterima. Singkatan: `openclaw onboard --auth-choice zai-api-key`.

- Endpoint umum: `https://api.z.ai/api/paas/v4`
- Endpoint koding (default): `https://api.z.ai/api/coding/paas/v4`
- Untuk endpoint umum, tentukan penyedia kustom dengan override URL dasar.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Untuk endpoint China: `baseUrl: "https://api.moonshot.cn/v1"` atau `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Kompatibel dengan Anthropic, penyedia bawaan. Singkatan: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (kompatibel dengan Anthropic)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

URL dasar harus menghilangkan `/v1` (klien Anthropic akan menambahkannya sendiri). Singkatan: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (langsung)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Atur `MINIMAX_API_KEY`. Singkatan: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Model lokal (LM Studio)">

Lihat [Model Lokal](/gateway/local-models). Intinya: jalankan MiniMax M2.1 melalui LM Studio Responses API pada perangkat keras yang mumpuni; tetap gunakan model berbayar (hosted) sebagai cadangan (fallback).

</Accordion>

---

## Skills (Keahlian)

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: allowlist opsional hanya untuk keahlian bawaan (keahlian managed/workspace tidak terpengaruh).
- `entries.<skillKey>.enabled: false` menonaktifkan keahlian meskipun disertakan/diinstal.
- `entries.<skillKey>.apiKey`: kemudahan untuk keahlian yang mendeklarasikan variabel lingkungan utama.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- Dimuat dari `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, ditambah `plugins.load.paths`.
- **Perubahan konfigurasi memerlukan restart gateway.**
- `allow`: allowlist opsional (hanya plugin yang terdaftar yang akan dimuat). `deny` akan menang.

Lihat [Plugins](/id-ID/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` menonaktifkan `act:evaluate` dan `wait --fn`.
- Profil jarak jauh (remote profiles) adalah attach-only (start/stop/reset dinonaktifkan).
- Urutan deteksi otomatis: browser default jika berbasis Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Layanan kontrol: hanya loopback (port diturunkan dari `gateway.port`, default `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, teks pendek, URL gambar, atau URI data
    },
  },
}
```

- `seamColor`: warna aksen untuk UI platform asli (tint gelembung Mode Talk, dst.).
- `assistant`: override identitas UI Kontrol. Cadangan mengikuti identitas agen aktif.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // none | token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // atau OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // untuk mode=trusted-proxy; lihat /id-ID/gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Penolakan HTTP /tools/invoke tambahan
      deny: ["browser"],
      // Hapus alat dari daftar penolakan HTTP default
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Detail bidang Gateway">

- `mode`: `local` (jalankan gateway) atau `remote` (hubungkan ke gateway jarak jauh). Gateway menolak untuk dimulai kecuali `local`.
- `port`: port multiplexing tunggal untuk WS + HTTP. Prioritas: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (default), `lan` (`0.0.0.0`), `tailnet` (hanya IP Tailscale), atau `custom`.
- **Autentikasi**: wajib secara default. Bind non-loopback memerlukan token/kata sandi bersama. Wizard onboarding menghasilkan token secara default.
- `auth.mode: "none"`: mode tanpa autentikasi eksplisit. Hanya gunakan untuk pengaturan loopback lokal yang tepercaya; ini sengaja tidak ditawarkan oleh prompt onboarding.
- `auth.mode: "trusted-proxy"`: menyerahkan autentikasi ke reverse proxy yang sadar-identitas dan mempercayai header identitas dari `gateway.trustedProxies` (lihat [Trusted Proxy Auth](/id-ID/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: ketika `true`, header identitas Tailscale Serve memenuhi syarat autentikasi (diverifikasi via `tailscale whois`). Defaultnya adalah `true` ketika `tailscale.mode = "serve"`.
- `auth.rateLimit`: pembatas autentikasi-gagal opsional. Berlaku per IP klien dan per cakupan autentikasi (secret-bersama dan token-perangkat dilacak secara independen). Upaya yang diblokir mengembalikan `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` defaultnya adalah `true`; atur `false` jika Anda sengaja ingin lalu lintas localhost juga dibatasi kecepatannya (untuk pengaturan pengujian atau penerapan proksi yang ketat).
- `tailscale.mode`: `serve` (hanya tailnet, bind loopback) atau `funnel` (publik, memerlukan autentikasi).
- `remote.transport`: `ssh` (default) atau `direct` (ws/wss). Untuk `direct`, `remote.url` harus `ws://` atau `wss://`.
- `gateway.remote.token` adalah hanya untuk panggilan CLI jarak jauh; tidak mengaktifkan autentikasi gateway lokal.
- `trustedProxies`: IP reverse proxy yang mengakhiri (terminate) TLS. Hanya cantumkan proksi yang Anda kontrol.
- `gateway.tools.deny`: nama alat tambahan yang diblokir untuk HTTP `POST /tools/invoke` (memperluas daftar penolakan default).
- `gateway.tools.allow`: menghapus nama alat dari daftar penolakan HTTP default.

</Accordion>

### Endpoint yang kompatibel dengan OpenAI

- Chat Completions: dinonaktifkan secara default. Aktifkan dengan `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Pengerasan input URL Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Isolasi multi-instance (Multiple Gateways)

Jalankan beberapa gateway pada satu host dengan port dan direktori status yang unik:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Bendera kemudahan: `--dev` (menggunakan `~/.openclaw-dev` + port `19001`), `--profile <nama>` (menggunakan `~/.openclaw-<nama>`).

Lihat [Multiple Gateways](/gateway/multiple-gateways).

---

## Hooks (Kait)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

Autentikasi: `Authorization: Bearer <token>` atau `x-openclaw-token: <token>`.

**Endpoint:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` dari muatan permintaan hanya diterima jika `hooks.allowRequestSessionKey=true` (default: `false`).
- `POST /hooks/<name>` → diselesaikan melalui `hooks.mappings`

<Accordion title="Detail Pemetaan">

- `match.path` mencocokkan sub-jalur setelah `/hooks` (cth. `/hooks/gmail` → `gmail`).
- `match.source` mencocokkan bidang muatan untuk jalur generik.
- Template seperti `{{messages[0].subject}}` membaca dari muatan (payload).
- `transform` dapat mengarah ke modul JS/TS yang mengembalikan tindakan hook.
- `transform.module` harus berupa jalur relatif dan tetap berada di dalam `hooks.transformsDir` (jalur absolut dan traversal ditolak).
- `agentId` mengarahkan ke agen tertentu; ID yang tidak dikenal akan dialihkan ke default.
- `allowedAgentIds`: membatasi perutean eksplisit (`*` atau dikosongkan = izinkan semua, `[]` = tolak semua).
- `defaultSessionKey`: kunci sesi tetap opsional untuk eksekusi agen hook tanpa `sessionKey` eksplisit.
- `allowRequestSessionKey`: mengizinkan pemanggil `/hooks/agent` untuk mengatur `sessionKey` (default: `false`).
- `allowedSessionKeyPrefixes`: allowlist awalan opsional untuk nilai `sessionKey` eksplisit (permintaan + pemetaan), cth. `["hook:"]`.
- `deliver: true` mengirimkan balasan akhir ke saluran; `channel` defaultnya adalah `last`.
- `model` menimpa LLM untuk jalan hook ini (harus diizinkan jika katalog model diatur).

</Accordion>

### Integrasi Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Gateway secara otomatis memulai `gog gmail watch serve` saat booting jika dikonfigurasi. Atur `OPENCLAW_SKIP_GMAIL_WATCHER=1` untuk menonaktifkan.
- Jangan jalankan `gog gmail watch serve` secara terpisah bersamaan dengan Gateway.

---

## Host Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // atau OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Melayani HTML/CSS/JS yang dapat diedit agen dan A2UI melalui HTTP di bawah port Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Khusus lokal (Local-only): pertahankan `gateway.bind: "loopback"` (default).
- Bind non-loopback: rute kanvas memerlukan autentikasi Gateway (token/kata sandi/trusted-proxy), sama seperti HTTP surface Gateway lainnya.
- Node WebViews biasanya tidak mengirim header autentikasi; setelah node dipasangkan dan terhubung, Gateway mengiklankan URL kemampuan cakupan-node (node-scoped capability URLs) untuk akses kanvas/A2UI.
- URL kemampuan terikat pada sesi WS node aktif dan cepat kedaluwarsa. Cadangan berbasis IP tidak digunakan.
- Memasukkan klien live-reload ke dalam HTML yang dilayani.
- Secara otomatis membuat starter `index.html` jika kosong.
- Juga melayani A2UI di `/__openclaw__/a2ui/`.
- Perubahan memerlukan restart gateway.
- Nonaktifkan live reload untuk direktori besar atau kesalahan `EMFILE`.

---

## Discovery (Penemuan)

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (default): menghilangkan `cliPath` + `sshPort` dari rekaman TXT.
- `full`: menyertakan `cliPath` + `sshPort`.
- Hostname defaultnya adalah `openclaw`. Timpa dengan `OPENCLAW_MDNS_HOSTNAME`.

### Area luas (Wide-area / DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Menulis zona DNS-SD unicast di bawah `~/.openclaw/dns/`. Untuk penemuan lintas jaringan, pasangkan dengan server DNS (disarankan CoreDNS) + Tailscale split DNS.

Pengaturan: `openclaw dns setup --apply`.

---

## Environment (Lingkungan)

### `env` (variabel lingkungan inline)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Variabel lingkungan inline hanya diterapkan jika variabel lingkungan proses tidak memiliki kunci tersebut.
- File `.env`: CWD `.env` + `~/.openclaw/.env` (keduanya tidak menimpa variabel yang ada).
- `shellEnv`: mengimpor kunci yang hilang yang diharapkan dari profil login shell Anda.
- Lihat [Environment](/id-ID/help/environment) untuk urutan prioritas selengkapnya.

### Substitusi variabel lingkungan

Gunakan referensi variabel lingkungan di string konfigurasi mana pun dengan `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Hanya nama huruf besar yang dicocokkan: `[A-Z_][A-Z0-9_]*`.
- Variabel yang hilang/kosong akan memicu kesalahan saat pemuatan konfigurasi.
- Gunakan `$${VAR}` untuk karakter literal `${VAR}`.
- Berfungsi dengan `$include`.

---

## Penyimpanan autentikasi (Auth storage)

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- Profil autentikasi per-agen disimpan di `<agentDir>/auth-profiles.json`.
- Impor OAuth lama dari `~/.openclaw/credentials/oauth.json`.
- Lihat [OAuth](/id-ID/concepts/oauth).

---

## Logging (Pencatatan)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*(["']?)([^\\s"']+)\\1"],
  },
}
```

- File log default: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Atur `logging.file` untuk jalur yang stabil.
- `consoleLevel` meningkat ke `debug` saat menggunakan `--verbose`.

---

## Wizard

Metadata yang ditulis oleh wizard CLI (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Identity (Identitas)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

Ditulis oleh asisten onboarding macOS. Menurunkan default:

- `messages.ackReaction` dari `identity.emoji` (cadangan ke 👀)
- `mentionPatterns` dari `identity.name`/`identity.emoji`
- `avatar` menerima: jalur relatif terhadap workspace, URL `http(s)`, atau URI `data:`

---

## Bridge (Usang, dihapus)

Build saat ini tidak lagi menyertakan bridge TCP. Node terhubung melalui WebSocket Gateway. Kunci `bridge.*` bukan lagi bagian dari skema konfigurasi (validasi gagal hingga dihapus; `openclaw doctor --fix` dapat menghapus kunci yang tidak dikenal).

<Accordion title="Konfigurasi Bridge lama (referensi sejarah)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    webhook: "https://example.invalid/legacy", // fallback usang untuk pekerjaan yang disimpan dengan notify:true
    webhookToken: "ganti-dengan-token-khusus", // bearer token opsional untuk autentikasi webhook keluar
    sessionRetention: "24h", // string durasi atau false
  },
}
```

- `sessionRetention`: berapa lama sesi cron yang sudah selesai disimpan sebelum dipangkas. Default: `24h`.
- `webhookToken`: bearer token yang digunakan untuk pengiriman POST webhook cron (`delivery.mode = "webhook"`), jika dikosongkan tidak ada header autentikasi yang dikirim.
- `webhook`: URL webhook cadangan lama yang usang (http/https) hanya digunakan untuk pekerjaan tersimpan yang masih memiliki `notify: true`.

Lihat [Pekerjaan Cron](/automation/cron-jobs).

---

## Variabel template model media

Placeholder template yang dikembangkan di `tools.media.*.models[].args`:

| Variabel          | Deskripsi                                         |
| ----------------- | ------------------------------------------------- |
| `{{Body}}`        | Isi pesan masuk lengkap                           |
| `{{ContentType}}` | Tipe MIME lampiran                                |
| `{{MediaPath}}`   | Jalur lokal ke file media                         |
| `{{Prompt}}`      | Prompt pemrosesan (jika ada)                      |
| `{{MaxChars}}`    | Batas karakter untuk model teks                   |
| `{{MaxBytes}}`    | Batas ukuran file dalam byte                      |
| `{{TimeoutMs}}`   | Batas waktu eksekusi dalam milidetik              |
| `{{Language}}`    | Kode bahasa target (cth. `id`, `en`)              |
| `{{Format}}`      | Format keluaran yang diminta (cth. `json`, `raw`) |
| `{{BaseName}}`    | Nama file tanpa ekstensi                          |
| `{{Extension}}`   | Ekstensi file dengan titik                        |
| `{{MediaDir}}`    | Direktori yang berisi file media                  |
| `{{ChatType}}`    | Tipe obrolan (`direct`, `group`, dst.)            |
| `{{PeerId}}`      | ID unik pengirim/obrolan lintas saluran           |
| `{{AccountId}}`   | ID akun saluran                                   |
| `{{Channel}}`     | Nama saluran (`whatsapp`, `telegram`, dst.)       |
| `{{AgentId}}`     | ID agen yang memproses                            |
| `{{SessionKey}}`  | Kunci sesi aktif                                  |



