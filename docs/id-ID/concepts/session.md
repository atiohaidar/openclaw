---
summary: "Aturan manajemen sesi, kunci, dan persistensi untuk obrolan"
read_when:
  - Memodifikasi penanganan atau penyimpanan sesi
title: "Manajemen Sesi"
---

# Manajemen Sesi

OpenClaw memperlakukan **satu sesi obrolan langsung (DM) per agen** sebagai yang utama. Obrolan langsung digabungkan ke `agent:<agentId>:<mainKey>` (default `main`), sementara obrolan grup/saluran mendapatkan kunci mereka sendiri. `session.mainKey` dihormati.

Gunakan `session.dmScope` untuk mengontrol bagaimana **pesan langsung** dikelompokkan:

- `main` (default): semua DM berbagi sesi utama yang sama untuk kesinambungan konteks.
- `per-peer`: isolasi berdasarkan ID pengirim lintas saluran.
- `per-channel-peer`: isolasi berdasarkan saluran + pengirim (direkomendasikan untuk kotak masuk multi-pengguna).
- `per-account-channel-peer`: isolasi berdasarkan akun + saluran + pengirim (direkomendasikan untuk kotak masuk multi-akun).
  Gunakan `session.identityLinks` untuk memetakan ID pengirim berawalan penyedia ke identitas kanonik sehingga orang yang sama berbagi sesi DM lintas saluran saat menggunakan mode `per-peer`, `per-channel-peer`, atau `per-account-channel-peer`.

## Mode DM Aman (Direkomendasikan untuk pengaturan multi-pengguna)

> **Peringatan Keamanan:** Jika agen Anda dapat menerima DM dari **banyak orang**, Anda sangat disarankan untuk mengaktifkan mode DM aman. Tanpa mode ini, semua pengguna berbagi konteks percakapan yang sama, yang dapat membocorkan informasi pribadi antar pengguna.

**Contoh masalah dengan pengaturan default:**

- Alice (`<PENGIRIM_A>`) mengirim pesan ke agen Anda tentang topik pribadi (misalnya, janji temu medis).
- Bob (`<PENGIRIM_B>`) mengirim pesan ke agen Anda dan bertanya "Apa yang kita bicarakan tadi?"
- Karena kedua DM tersebut berbagi sesi yang sama, model mungkin menjawab Bob menggunakan konteks sebelumnya dari Alice.

**Solusinya:** Setel `dmScope` untuk mengisolasi sesi per pengguna:

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    // Mode DM aman: isolasi konteks DM per saluran + pengirim.
    dmScope: "per-channel-peer",
  },
}
```

**Kapan harus mengaktifkan ini:**

- Anda memiliki persetujuan pairing untuk lebih dari satu pengirim.
- Anda menggunakan daftar izinkan (allowlist) DM dengan banyak entri.
- Anda menyetel `dmPolicy: "open"`.
- Beberapa nomor telepon atau akun dapat mengirim pesan ke agen Anda.

Catatan:

- Defaultnya adalah `dmScope: "main"` untuk kesinambungan (semua DM berbagi sesi utama). Ini cocok untuk pengaturan satu pengguna.
- Untuk kotak masuk multi-akun pada saluran yang sama, lebih disarankan menggunakan `per-account-channel-peer`.
- Jika orang yang sama menghubungi Anda di beberapa saluran, gunakan `session.identityLinks` untuk menggabungkan sesi DM mereka menjadi satu identitas kanonik.
- Anda dapat memverifikasi pengaturan DM Anda dengan `openclaw security audit` (lihat [keamanan](/id-ID/cli/security)).

## Gateway adalah Sumber Kebenaran

Semua status sesi **dimiliki oleh gateway** (OpenClaw "induk"). Klien UI (aplikasi macOS, WebChat, dll.) harus menanyakan gateway untuk daftar sesi dan jumlah token, bukannya membaca file lokal.

- Dalam **mode jarak jauh (remote)**, penyimpanan sesi yang Anda kelola berada di host gateway jarak jauh, bukan di Mac Anda.
- Jumlah token yang ditampilkan di UI berasal dari bidang penyimpanan gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Klien tidak mengurai transkrip JSONL untuk "memperbaiki" total tersebut.

## Lokasi Penyimpanan Status

- Di **host gateway**:
  - File store: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (per agen).
- Transkrip: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (sesi topik Telegram menggunakan `.../<SessionId>-topic-<threadId>.jsonl`).
- Store adalah peta (map) `sessionKey -> { sessionId, updatedAt, ... }`. Menghapus entri aman untuk dilakukan; entri tersebut akan dibuat ulang saat dibutuhkan.
- Entri grup mungkin menyertakan `displayName`, `channel`, `subject`, `room`, dan `space` untuk melabeli sesi di UI.
- Entri sesi menyertakan metadata `origin` (label + petunjuk perutean) sehingga UI dapat menjelaskan dari mana sesi berasal.
- OpenClaw **tidak** membaca folder sesi Pi/Tau versi lama.

## Pemangkasan Sesi (Pruning)

OpenClaw memangkas **hasil alat lama** dari konteks dalam memori tepat sebelum panggilan LLM secara default. Ini **tidak** menulis ulang riwayat JSONL. Lihat [/id-ID/concepts/session-pruning](/id-ID/concepts/session-pruning).

## Flush Memori Pra-pemadatan

Saat sesi mendekati pemadatan otomatis, OpenClaw dapat menjalankan **giliran flush memori senyap** yang mengingatkan model untuk menulis catatan penting ke disk. Ini hanya berjalan jika workspace dapat ditulis. Lihat [Memori](/id-ID/concepts/memory) dan [Pemadatan](/id-ID/concepts/compaction).

## Pemetaan Transport → Kunci Sesi

- Obrolan langsung mengikuti `session.dmScope` (default `main`).
  - `main`: `agent:<agentId>:<mainKey>` (kesinambungan lintas perangkat/saluran).
    - Beberapa nomor telepon dan saluran dapat dipetakan ke kunci utama agen yang sama; mereka bertindak sebagai transport masuk ke satu percakapan.
  - `per-peer`: `agent:<agentId>:dm:<peerId>`.
  - `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  - `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId default ke `default`).
  - Jika `session.identityLinks` cocok dengan ID pengirim berawalan penyedia (misalnya `telegram:123`), kunci kanonik akan menggantikan `<peerId>` sehingga orang yang sama berbagi sesi lintas saluran.
- Obrolan grup mengisolasi status: `agent:<agentId>:<channel>:group:<id>` (ruang/saluran menggunakan `agent:<agentId>:<channel>:channel:<id>`).
  - Forum topik Telegram menambahkan `:topic:<threadId>` ke ID grup untuk isolasi.
  - Kunci lama `group:<id>` masih dikenali untuk keperluan migrasi.
- Konteks masuk mungkin masih menggunakan `group:<id>`; saluran akan disimpulkan dari `Provider` dan dinormalisasi ke bentuk kanonik `agent:<agentId>:<channel>:group:<id>`.
- Sumber lainnya:
  - Pekerjaan Cron: `cron:<job.id>`
  - Webhook: `hook:<uuid>` (kecuali jika diatur secara eksplisit oleh webhook)
  - Proses Node: `node-<nodeId>`

## Siklus Hidup (Lifecycle)

- Kebijakan Reset: sesi digunakan kembali sampai kedaluwarsa, dan kedaluwarsa dievaluasi pada pesan masuk berikutnya.
- Reset Harian: default ke **jam 4:00 pagi waktu lokal pada host gateway**. Sebuah sesi dianggap basi jika pembaruan terakhirnya lebih lama dari waktu reset harian terbaru.
- Reset Idle (Opsional): `idleMinutes` menambahkan jendela menganggur (idle). Ketika reset harian dan idle dikonfigurasi, **mana yang kedaluwarsa lebih dulu** akan memaksa sesi baru.
- Mode Idle-Saja (Lama): jika Anda menyetel `session.idleMinutes` tanpa konfigurasi `session.reset`/`resetByType`, OpenClaw tetap berada dalam mode idle-saja untuk kompatibilitas ke belakang.
- Override Per Tipe (Opsional): `resetByType` memungkinkan Anda menimpa kebijakan untuk sesi `direct`, `group`, dan `thread` (thread = utas Slack/Discord, topik Telegram, utas Matrix jika disediakan oleh konektor).
- Override Per Saluran (Opsional): `resetByChannel` menimpa kebijakan reset untuk suatu saluran (berlaku untuk semua tipe sesi pada saluran tersebut dan didahulukan di atas `reset`/`resetByType`).
- Pemicu Reset: perintah persis `/new` atau `/reset` (ditambah apa pun dalam `resetTriggers`) memulai ID sesi baru dan meneruskan sisa pesan. `/new <model>` menerima alias model, `provider/model`, atau nama penyedia (pencocokan samar) untuk menyetel model sesi baru. Jika `/new` atau `/reset` dikirim sendiri, OpenClaw menjalankan "giliran salam" singkat untuk mengonfirmasi reset.
- Reset Manual: hapus kunci tertentu dari store atau hapus transkrip JSONL; pesan berikutnya akan membuatnya kembali.
- Pekerjaan cron yang terisolasi selalu membuat `sessionId` baru per eksekusi (tidak ada penggunaan kembali sesi idle).

## Kebijakan Pengiriman (Send Policy - Opsional)

Memblokir pengiriman untuk tipe sesi tertentu tanpa mencantumkan ID individu.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Mencocokkan kunci sesi asli (termasuk awalan `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Override saat aplikasi berjalan (hanya pemilik):

- `/send on` → izinkan pengiriman untuk sesi ini.
- `/send off` → tolak pengiriman untuk sesi ini.
- `/send inherit` → hapus override dan gunakan aturan konfigurasi.
  Kirimkan perintah ini sebagai pesan mandiri agar terdaftar.

## Konfigurasi (Contoh Penggantian Nama Opsional)

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // jaga agar kunci grup terpisah
    dmScope: "main", // kesinambungan DM (setel per-channel-peer/per-account-channel-peer untuk kotak masuk bersama)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Default: mode=daily, atHour=4 (waktu lokal host gateway).
      // Jika Anda juga menyetel idleMinutes, mana yang kedaluwarsa lebih dulu yang menang.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Memeriksa Sesi

- `openclaw status` — menampilkan jalur store dan sesi terbaru.
- `openclaw sessions --json` — membuang setiap entri (filter dengan `--active <minutes>`).
- `openclaw gateway call sessions.list --params '{}'` — mengambil sesi dari gateway yang sedang berjalan (gunakan `--url`/`--token` untuk akses gateway jarak jauh).
- Kirim `/status` sebagai pesan mandiri dalam obrolan untuk melihat apakah agen dapat dihubungi, seberapa banyak konteks sesi yang digunakan, peralihan thinking/verbose saat ini, dan kapan kredensial WhatsApp web Anda terakhir kali disegarkan.
- Kirim `/context list` atau `/context detail` untuk melihat apa yang ada dalam prompt sistem dan file workspace yang disuntikkan.
- Kirim `/stop` sebagai pesan mandiri untuk membatalkan proses saat ini, menghapus antrean tindak lanjut untuk sesi tersebut, dan menghentikan semua sub-agen yang lahir darinya.
- Kirim `/compact` (instruksi opsional) sebagai pesan mandiri untuk meringkas konteks lama dan membebaskan ruang jendela konteks. Lihat [/id-ID/concepts/compaction](/id-ID/concepts/compaction).
- Transkrip JSONL dapat dibuka langsung untuk meninjau secara lengkap.

## Tip

- Biarkan kunci utama didedikasikan untuk lalu lintas 1:1; biarkan grup memiliki kunci mereka sendiri.
- Saat mengotomatiskan pembersihan, hapus kunci individu alih-alih seluruh store untuk menjaga konteks di tempat lain.

## Metadata Asal Sesi (Session Origin)

Setiap entri sesi mencatat dari mana ia berasal (best-effort) di bagian `origin`:

- `label`: label manusia (diambil dari label percakapan + subjek grup/saluran).
- `provider`: ID saluran yang dinormalisasi (termasuk ekstensi).
- `from`/`to`: ID perutean asli dari amplop pesan masuk.
- `accountId`: ID akun penyedia (jika multi-akun).
- `threadId`: ID utas/topik jika saluran mendukungnya.
  Bidang asal ini diisi untuk pesan langsung, saluran, dan grup. Jika konektor hanya memperbarui perutean pengiriman (misalnya, untuk menjaga kesegaran sesi utama DM), ia tetap harus menyediakan konteks masuk agar sesi tetap memiliki metadata penjelasnya. Ekstensi dapat melakukan ini dengan mengirimkan `ConversationLabel`, `GroupSubject`, `GroupChannel`, `GroupSpace`, dan `SenderName` dalam konteks masuk dan memanggil `recordSessionMetaFromInbound`.
