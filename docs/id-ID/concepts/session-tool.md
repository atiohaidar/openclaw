---
summary: "Alat sesi agen untuk mencantumkan sesi, mengambil riwayat, dan mengirim pesan lintas sesi"
read_when:
  - Menambah atau memodifikasi alat sesi
title: "Alat Sesi"
---

# Alat Sesi

Tujuan: menyediakan set alat yang kecil dan sulit disalahgunakan agar agen dapat mencantumkan sesi, mengambil riwayat, dan mengirim pesan ke sesi lain.

## Nama Alat

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

## Model Kunci (Key Model)

- Wadah obrolan langsung utama selalu menggunakan kunci literal `"main"` (diselesaikan sesuai kunci utama agen saat ini).
- Obrolan grup menggunakan `agent:<agentId>:<channel>:group:<id>` atau `agent:<agentId>:<channel>:channel:<id>` (teruskan kunci lengkapnya).
- Pekerjaan Cron menggunakan `cron:<job.id>`.
- Hook menggunakan `hook:<uuid>` kecuali jika disetel secara eksplisit.
- Sesi node menggunakan `node-<nodeId>` kecuali jika disetel secara eksplisit.

`global` dan `unknown` adalah nilai yang dicadangkan dan tidak pernah dicantumkan. Jika `session.scope = "global"`, kami mengalihkannya (alias) ke `main` untuk semua alat sehingga pemanggil tidak pernah melihat `global`.

## sessions_list

Mencantumkan sesi dalam bentuk array (baris).

Parameter:

- `kinds?: string[]` filter: salah satu dari `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` baris maksimum (default: default server, misal: dibatasi 200)
- `activeMinutes?: number` hanya sesi yang diperbarui dalam N menit terakhir.
- `messageLimit?: number` 0 = tanpa pesan (default 0); >0 = sertakan N pesan terakhir.

Perilaku:

- `messageLimit > 0` mengambil `chat.history` per sesi dan menyertakan N pesan terakhir.
- Hasil alat (tool results) difilter dari output daftar; gunakan `sessions_history` untuk melihat pesan alat.
- Saat berjalan di sesi agen yang berada di dalam **sandbox**, alat sesi secara default hanya memiliki **visibilitas sesi yang lahir (spawned) dari sesi tersebut** (lihat di bawah).

Bentuk Baris (JSON):

- `key`: kunci sesi (string).
- `kind`: `main | group | cron | hook | node | other`.
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`.
- `displayName`: (label tampilan grup jika tersedia).
- `updatedAt`: (ms).
- `sessionId`.
- `model`, `contextTokens`, `totalTokens`.
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`.
- `sendPolicy`: (override sesi jika disetel).
- `lastChannel`, `lastTo`.
- `deliveryContext`: yang dinormalisasi `{ channel, to, accountId }` jika tersedia.
- `transcriptPath`: jalur file transkrip (hasil estimasi terbaik dari direktori store + sessionId).
- `messages?`: (hanya jika `messageLimit > 0`).

## sessions_history

Mengambil transkrip untuk satu sesi.

Parameter:

- `sessionKey` (wajib; menerima kunci sesi atau `sessionId` dari `sessions_list`).
- `limit?: number` pesan maksimum (dibatasi server).
- `includeTools?: boolean` (default false).

Perilaku:

- `includeTools=false` memfilter pesan dengan `role: "toolResult"`.
- Mengembalikan array pesan dalam format transkrip mentah.
- Jika diberikan `sessionId`, OpenClaw akan menyelesaikannya ke kunci sesi yang sesuai (error jika ID tidak ditemukan).

## sessions_send

Mengirim pesan ke sesi lain.

Parameter:

- `sessionKey` (wajib; menerima kunci sesi atau `sessionId` dari `sessions_list`).
- `message` (wajib).
- `timeoutSeconds?: number` (default >0; 0 = kirim dan lupakan/fire-and-forget).

Perilaku:

- `timeoutSeconds = 0`: masukkan ke antrean dan kembalikan `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0`: tunggu hingga N detik untuk penyelesaian, lalu kembalikan `{ runId, status: "ok", reply }`.
- Jika waktu tunggu habis: `{ runId, status: "timeout", error }`. Proses berlanjut; panggil `sessions_history` nanti.
- Jika proses gagal: `{ runId, status: "error", error }`.
- Pengiriman pengumuman (announce delivery) berjalan setelah proses utama selesai dan bersifat best-effort; `status: "ok"` tidak menjamin pengumuman telah terkirim.
- Menunggu melalui `agent.wait` di gateway (sisi server) sehingga koneksi ulang tidak memutus penungguan.
- Konteks pesan agen-ke-agen disuntikkan untuk proses utama.
- Pesan antar-sesi disimpan dengan `message.provenance.kind = "inter_session"` agar pembaca transkrip dapat membedakan instruksi agen yang dirutekan dari input pengguna eksternal.
- Setelah proses utama selesai, OpenClaw menjalankan **loop balas-kembali (reply-back loop)**:
    - Putaran 2+ bergantian antara agen peminta dan target.
    - Balas tepat dengan `REPLY_SKIP` untuk menghentikan ping-pong.
    - Putaran maksimum adalah `session.agentToAgent.maxPingPongTurns` (0–5, default 5).
- Setelah loop berakhir, OpenClaw menjalankan **tahap pengumuman agen-ke-agen** (hanya agen target):
    - Balas tepat dengan `ANNOUNCE_SKIP` untuk tetap diam.
    - Balasan lainnya akan dikirim ke saluran target.
    - Tahap pengumuman menyertakan permintaan asli + balasan putaran-1 + balasan ping-pong terbaru.

## Bidang Saluran (Channel Field)

- Untuk grup, `channel` adalah saluran yang tercatat pada entri sesi.
- Untuk obrolan langsung, `channel` dipetakan dari `lastChannel`.
- Untuk cron/hook/node, `channel` adalah `internal`.
- Jika hilang, `channel` adalah `unknown`.

## Keamanan / Kebijakan Pengiriman (Send Policy)

Pemblokiran berbasis kebijakan berdasarkan saluran/tipe obrolan (bukan per ID sesi).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Penimpa (override) saat aplikasi berjalan (per entri sesi):

- `sendPolicy: "allow" | "deny"` (kosong = mewarisi konfigurasi).
- Dapat disetel melalui `sessions.patch` atau perintah mandiri `/send on|off|inherit` oleh pemilik.

Titik penegakan aturan:

- `chat.send` / `agent` (gateway).
- Logika pengiriman balas-otomatis (auto-reply).

## sessions_spawn

Menjalankan sub-agen dalam sesi terisolasi dan mengumumkan hasilnya kembali ke saluran obrolan peminta.

Parameter:

- `task` (wajib).
- `label?` (opsional; digunakan untuk log/UI).
- `agentId?` (opsional; jalankan di bawah ID agen lain jika diizinkan).
- `model?` (opsional; menimpa model sub-agen).
- `runTimeoutSeconds?` (default 0; jika disetel, membatalkan proses sub-agen setelah N detik).
- `cleanup?` (`delete|keep`, default `keep`).

Daftar Izinkan (Allowlist):

- `agents.list[].subagents.allowAgents`: daftar ID agen yang diizinkan melalui `agentId` (`["*"]` untuk mengizinkan semua). Default: hanya agen peminta.

Penemuan (Discovery):

- Gunakan `agents_list` untuk mengetahui ID agen mana yang diizinkan untuk `sessions_spawn`.

Perilaku:

- Memulai sesi `agent:<agentId>:subagent:<uuid>` baru dengan `deliver: false`.
- Sub-agen mendapatkan set alat lengkap **kecuali alat sesi** (dapat dikonfigurasi melalui `tools.subagents.tools`).
- Sub-agen tidak diizinkan memanggil `sessions_spawn` (tidak ada sub-agen menjalankan sub-agen).
- Selalu bersifat tidak memblokir (non-blocking): segera mengembalikan `{ status: "accepted", runId, childSessionKey }`.
- Setelah selesai, OpenClaw menjalankan **tahap pengumuman** sub-agen dan mengirimkan hasilnya ke saluran obrolan peminta.
    - Jika balasan akhir asisten kosong, `toolResult` terbaru dari riwayat sub-agen akan disertakan sebagai `Result`.
- Balas tepat dengan `ANNOUNCE_SKIP` selama tahap pengumuman untuk tetap diam.
- Balasan pengumuman dinormalisasi menjadi `Status`/`Result`/`Notes`; `Status` berasal dari hasil runtime (bukan teks model).
- Sesi sub-agen akan diarsipkan otomatis setelah `agents.defaults.subagents.archiveAfterMinutes` (default: 60).
- Balasan pengumuman menyertakan baris statistik (waktu proses, token, sessionKey/sessionId, jalur transkrip, dan biaya opsional).

## Visibilitas Sesi Sandbox

Alat sesi dapat dibatasi cakupannya untuk mengurangi akses lintas sesi.

Perilaku Default:

- `tools.sessions.visibility` default-nya adalah `tree` (sesi saat ini + sesi sub-agen yang lahir darinya).
- Untuk sesi sandbox, `agents.defaults.sandbox.sessionToolsVisibility` dapat membatasi visibilitas secara ketat.

Konfigurasi:

```json5
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // default: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned", // atau "all"
      },
    },
  },
}
```

Catatan:

- `self`: hanya kunci sesi saat ini.
- `tree`: sesi saat ini + sesi yang lahir (spawned) dari sesi ini.
- `agent`: sesi apa pun yang dimiliki oleh ID agen saat ini.
- `all`: sesi apa pun (akses lintas agen tetap memerlukan `tools.agentToAgent`).
- Jika sebuah sesi berada dalam sandbox dan `sessionToolsVisibility="spawned"`, OpenClaw akan membatasi visibilitas ke `tree` meskipun Anda menyetel `tools.sessions.visibility="all"`.
