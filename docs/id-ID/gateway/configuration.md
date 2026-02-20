---
summary: "Ringkasan konfigurasi: tugas umum, pengaturan cepat, dan tautan ke referensi lengkap"
read_when:
  - Menyiapkan OpenClaw untuk pertama kalinya
  - Mencari pola konfigurasi yang umum
  - Mencari bagian konfigurasi tertentu
title: "Konfigurasi"
---

# Konfigurasi

OpenClaw membaca file konfigurasi opsional berformat <Tooltip tip="JSON5 mendukung komentar dan koma di akhir">**JSON5**</Tooltip> dari `~/.openclaw/openclaw.json`.

Jika file tersebut tidak ada, OpenClaw akan menggunakan nilai default yang aman. Alasan umum untuk menambahkan konfigurasi adalah:

- Menghubungkan saluran dan mengontrol siapa yang dapat mengirim pesan ke bot
- Mengatur model, alat, sandboxing, atau otomatisasi (cron, hook)
- Menyetel sesi, media, jaringan, atau UI

Lihat [referensi lengkap](/id-ID/gateway/configuration-reference) untuk setiap bidang yang tersedia.

<Tip>
**Baru pertama kali mengonfigurasi?** Mulailah dengan `openclaw onboard` untuk pengaturan interaktif, atau lihat panduan [Contoh Konfigurasi](/gateway/configuration-examples) untuk konfigurasi yang bisa langsung disalin.
</Tip>

## Konfigurasi minimal

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Mengedit konfigurasi

<Tabs>
  <Tab title="Wizard interaktif">
    ```bash
    openclaw onboard       # wizard pengaturan penuh
    openclaw configure     # wizard konfigurasi
    ```
  </Tab>
  <Tab title="CLI (perintah singkat)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    Buka [http://127.0.0.1:18789](http://127.0.0.1:18789) dan gunakan tab **Config**.
    Control UI menampilkan formulir berdasarkan skema konfigurasi, dilengkapi dengan editor **Raw JSON** untuk pengeditan manual.
  </Tab>
  <Tab title="Edit langsung">
    Edit `~/.openclaw/openclaw.json` secara langsung. Gateway akan memantau file tersebut dan menerapkan perubahan secara otomatis (lihat [hot reload](#hot-reload-konfigurasi)).
  </Tab>
</Tabs>

## Validasi ketat

<Warning>
OpenClaw hanya menerima konfigurasi yang sepenuhnya cocok dengan skema. Kunci yang tidak dikenal, tipe data yang salah, atau nilai yang tidak valid akan menyebabkan Gateway **menolak untuk dijalankan**. Satu-satunya pengecualian di tingkat root adalah `$schema` (string), agar editor dapat menyertakan metadata JSON Schema.
</Warning>

Jika validasi gagal:

- Gateway tidak akan berjalan (boot)
- Hanya perintah diagnostik yang berfungsi (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Jalankan `openclaw doctor` untuk melihat masalah secara detail
- Jalankan `openclaw doctor --fix` (atau `--yes`) untuk menerapkan perbaikan

## Tugas umum

<AccordionGroup>
  <Accordion title="Menyiapkan saluran (WhatsApp, Telegram, Discord, dll.)">
    Masing-masing saluran memiliki bagian konfigurasi sendiri di bawah `channels.<provider>`. Lihat halaman saluran khusus untuk langkah-langkah pengaturan:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Semua saluran berbagi pola kebijakan DM yang sama:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // hanya untuk allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Memilih dan mengonfigurasi model">
    Atur model utama dan cadangan (fallback) opsional:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` menentukan katalog model dan bertindak sebagai daftar izinkan (allowlist) untuk perintah `/model`.
    - Referensi model menggunakan format `penyedia/model` (misal: `anthropic/claude-opus-4-6`).
    - `agents.defaults.imageMaxDimensionPx` mengontrol pengecilan skala gambar transkrip/alat (default `1200`); nilai yang lebih rendah biasanya mengurangi penggunaan token visi pada proses yang banyak menggunakan screenshot.
    - Lihat [CLI Model](/concepts/models) untuk mengganti model dalam chat dan [Failover Model](/concepts/model-failover) untuk rotasi autentikasi dan perilaku fallback.
    - Untuk penyedia kustom/host mandiri, lihat [Penyedia kustom](/id-ID/gateway/configuration-reference#penyedia-kustom-dan-base-url) di referensi.

  </Accordion>

  <Accordion title="Mengontrol siapa yang dapat mengirim pesan ke bot">
    Akses DM dikontrol per saluran melalui `dmPolicy`:

    - `"pairing"` (default): pengirim tidak dikenal akan mendapatkan kode pairing satu kali untuk disetujui
    - `"allowlist"`: hanya pengirim dalam `allowFrom` (atau penyimpanan pasangan yang diizinkan)
    - `"open"`: izinkan semua DM masuk (memerlukan `allowFrom: ["*"]`)
    - `"disabled"`: abaikan semua DM

    Untuk grup, gunakan `groupPolicy` + `groupAllowFrom` atau daftar izinkan khusus saluran.

    Lihat [referensi lengkap](/id-ID/gateway/configuration-reference#akses-dm-dan-grup) untuk detail per saluran.

  </Accordion>

  <Accordion title="Menyiapkan penyaringan sebutan (mention gating) di grup">
    Pesan grup secara default **memerlukan sebutan (mention)**. Konfigurasikan pola per agen:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Sebutan metadata**: sebutan (mention) asli @-mentions (WhatsApp tap-to-mention, Telegram @bot, dll.)
    - **Pola teks**: pola regex dalam `mentionPatterns`
    - Lihat [referensi lengkap](/id-ID/gateway/configuration-reference#penyaringan-sebutan-mention-gating-grup) untuk override per saluran dan mode chat mandiri (self-chat).

  </Accordion>

  <Accordion title="Mengonfigurasi sesi dan reset">
    Sesi mengontrol kelanjutan dan isolasi percakapan:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // direkomendasikan untuk multi-user
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` (bersama) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Lihat [Manajemen Sesi](/concepts/session) untuk cakupan (scoping), tautan identitas, dan kebijakan pengiriman.
    - Lihat [referensi lengkap](/id-ID/gateway/configuration-reference#session) untuk semua bidang.

  </Accordion>

  <Accordion title="Mengaktifkan sandboxing">
    Jalankan sesi agen dalam kontainer Docker yang terisolasi:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Bangun citra (image) terlebih dahulu: `scripts/sandbox-setup.sh`

    Lihat [Sandboxing](/id-ID/gateway/sandboxing) untuk panduan lengkap dan [referensi lengkap](/id-ID/gateway/configuration-reference#sandbox) untuk semua opsi.

  </Accordion>

  <Accordion title="Menyiapkan heartbeat (pemeriksaan berkala)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: string durasi (`30m`, `2h`). Atur ke `0m` untuk menonaktifkan.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Lihat [Heartbeat](/gateway/heartbeat) untuk panduan lengkap.

  </Accordion>

  <Accordion title="Mengonfigurasi cron job">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    Lihat [Cron job](/id-ID/automation/cron-jobs) untuk ringkasan fitur dan contoh CLI.

  </Accordion>

  <Accordion title="Menyiapkan webhook (hook)">
    Aktifkan titik akhir (endpoint) webhook HTTP pada Gateway:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    Lihat [referensi lengkap](/id-ID/gateway/configuration-reference#hooks) untuk semua opsi pemetaan dan integrasi Gmail.

  </Accordion>

  <Accordion title="Mengonfigurasi perutean multi-agen">
    Jalankan beberapa agen yang terisolasi dengan workspace dan sesi terpisah:

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

    Lihat [Multi-Agen](/concepts/multi-agent) dan [referensi lengkap](/id-ID/gateway/configuration-reference#multi-agent-routing) untuk aturan pengikatan (binding) dan profil akses per-agen.

  </Accordion>

  <Accordion title="Membagi konfigurasi ke dalam beberapa file ($include)">
    Gunakan `$include` untuk mengatur konfigurasi yang besar:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **File tunggal**: menggantikan objek yang menampungnya
    - **Array file**: digabungkan secara mendalam (deep-merge) sesuai urutan (yang terakhir menang)
    - **Kunci selevel (sibling)**: digabungkan setelah include (menimpa nilai yang disertakan)
    - **Include bersarang**: didukung hingga kedalaman 10 tingkat
    - **Jalur relatif**: diselesaikan relatif terhadap file yang menyertakannya
    - **Penanganan error**: pesan error yang jelas untuk file yang hilang, kesalahan parsing, dan include yang melingkar (circular)

  </Accordion>
</AccordionGroup>

## Hot reload konfigurasi

Gateway memantau `~/.openclaw/openclaw.json` dan menerapkan perubahan secara otomatis — sebagian besar pengaturan tidak memerlukan restart manual.

### Mode muat ulang (Reload)

| Mode                   | Perilaku                                                                                             |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | Menerapkan perubahan yang aman secara instan (hot-apply). Otomatis merestart untuk perubahan kritis. |
| **`hot`**              | Hanya menerapkan perubahan yang aman (hot-safe). Mencatat peringatan saat restart diperlukan.        |
| **`restart`**          | Merestart Gateway pada setiap perubahan konfigurasi, baik aman maupun tidak.                         |
| **`off`**              | Menonaktifkan pemantauan file. Perubahan akan berlaku pada restart manual berikutnya.                |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Apa yang bisa di-hot-apply vs apa yang butuh restart

Sebagian besar bidang dapat di-hot-apply tanpa waktu henti. Dalam mode `hybrid`, perubahan yang membutuhkan restart akan ditangani secara otomatis.

| Kategori       | Bidang                                                             | Butuh restart? |
| -------------- | ------------------------------------------------------------------ | -------------- |
| Saluran        | `channels.*`, `web` (WhatsApp) — semua saluran bawaan dan ekstensi | Tidak          |
| Agen & model   | `agent`, `agents`, `models`, `routing`                             | Tidak          |
| Otomatisasi    | `hooks`, `cron`, `agent.heartbeat`                                 | Tidak          |
| Sesi & pesan   | `session`, `messages`                                              | Tidak          |
| Alat & media   | `tools`, `browser`, `skills`, `audio`, `talk`                      | Tidak          |
| UI & lainnya   | `ui`, `logging`, `identity`, `bindings`                            | Tidak          |
| Server Gateway | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)               | **Ya**         |
| Infrastruktur  | `discovery`, `canvasHost`, `plugins`                               | **Ya**         |

<Note>
`gateway.reload` dan `gateway.remote` adalah pengecualian — mengubah keduanya **tidak** memicu restart.
</Note>

## RPC Konfigurasi (pembaruan terprogram)

<Note>
RPC penulisan control-plane (`config.apply`, `config.patch`, `update.run`) dibatasi (rate-limited) sebanyak **3 permintaan per 60 detik** per `deviceId+clientIp`. Saat dibatasi, RPC mengembalikan `UNAVAILABLE` beserta `retryAfterMs`.
</Note>

<AccordionGroup>
  <Accordion title="config.apply (penggantian penuh)">
    Melakukan validasi + menulis konfigurasi penuh dan merestart Gateway dalam satu langkah.

    <Warning>
    `config.apply` menggantikan **seluruh konfigurasi**. Gunakan `config.patch` untuk pembaruan parsial, atau `openclaw config set` untuk kunci tunggal.
    </Warning>

    Parameter:

    - `raw` (string) — payload JSON5 untuk seluruh konfigurasi
    - `baseHash` (opsional) — hash konfigurasi dari `config.get` (diperlukan jika konfigurasi sudah ada)
    - `sessionKey` (opsional) — kunci sesi untuk ping bangun setelah restart
    - `note` (opsional) — catatan untuk sentinel restart
    - `restartDelayMs` (opsional) — jeda sebelum restart (default 2000)

    Permintaan restart akan disatukan (coalesced) jika ada yang masih tertunda, dan berlaku cooldown 30 detik antar siklus restart.

    ```bash
    openclaw gateway call config.get --params '{}'  # dapatkan payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: "~/.openclaw/workspace" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  </Accordion>

  <Accordion title="config.patch (pembaruan parsial)">
    Menggabungkan pembaruan parsial ke dalam konfigurasi yang sudah ada (semantik JSON merge patch):

    - Objek digabungkan secara rekursif
    - `null` menghapus kunci
    - Array akan diganti secara utuh

    Parameter:

    - `raw` (string) — JSON5 berisi hanya kunci yang ingin diubah
    - `baseHash` (diperlukan) — hash konfigurasi dari `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — sama dengan `config.apply`

    Perilaku restart sama dengan `config.apply`.

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { "*": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  </Accordion>
</AccordionGroup>

## Variabel lingkungan (Env var)

OpenClaw membaca variabel lingkungan dari proses induk ditambah:

- `.env` dari direktori kerja saat ini (jika ada)
- `~/.openclaw/.env` (cadangan global)

Kedua file tersebut tidak menimpa variabel lingkungan yang sudah ada. Anda juga dapat mengatur variabel lingkungan secara inline di konfigurasi:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Impor shell env (opsional)">
  Jika diaktifkan dan kunci yang diharapkan belum diatur, OpenClaw akan menjalankan shell login Anda dan hanya mengimpor kunci yang hilang:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Ekuivalen variabel lingkungan: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Substitusi variabel lingkungan dalam nilai konfigurasi">
  Gunakan variabel lingkungan dalam nilai string konfigurasi apa pun dengan `${NAMA_VAR}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Aturan:

- Hanya nama huruf besar yang cocok: `[A-Z_][A-Z0-9_]*`
- Variabel yang hilang/kosong akan memicu error saat dimuat
- Gunakan `$${VAR}` untuk output literal (escape)
- Berfungsi di dalam file `$include`
- Substitusi inline: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Lihat [Lingkungan](/help/environment) untuk urutan prioritas dan sumber selengkapnya.

## Referensi lengkap

Untuk referensi lengkap bidang demi bidang, lihat **[Referensi Konfigurasi](/id-ID/gateway/configuration-reference)**.



