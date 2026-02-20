---
summary: "Runbook untuk layanan Gateway, siklus hidup, dan operasi"
read_when:
  - Menjalankan atau mendebug proses gateway
title: "Runbook Gateway"
---

# Runbook Gateway

Gunakan halaman ini untuk startup hari pertama dan operasi hari kedua dari layanan Gateway.

<CardGroup cols={2}>
  <Card title="Pemecahan Masalah Mendalam" icon="siren" href="/id-ID/gateway/troubleshooting">
    Diagnostik berbasis gejala dengan urutan perintah yang tepat dan tanda tangan log.
  </Card>
  <Card title="Konfigurasi" icon="sliders" href="/id-ID/gateway/configuration">
    Panduan pengaturan berbasis tugas + referensi konfigurasi lengkap.
  </Card>
</CardGroup>

## Startup lokal 5 menit

<Steps>
  <Step title="Mulai Gateway">

```bash
openclaw gateway --port 18789
# debug/trace ditampilkan ke stdio
openclaw gateway --port 18789 --verbose
# paksa hentikan listener pada port yang dipilih, lalu mulai
openclaw gateway --force
```

  </Step>

  <Step title="Verifikasi kesehatan layanan">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Baseline sehat: `Runtime: running` dan `RPC probe: ok`.

  </Step>

  <Step title="Validasi kesiapan saluran">

```bash
openclaw channels status --probe
```

  </Step>
</Steps>

<Note>
Muat ulang konfigurasi Gateway memantau jalur file konfigurasi aktif (diselesaikan dari default profil/state, atau `OPENCLAW_CONFIG_PATH` jika diatur).
Mode default adalah `gateway.reload.mode="hybrid"`.
</Note>

## Model runtime

- Satu proses yang selalu menyala untuk perutean, control plane, dan koneksi saluran.
- Port tunggal yang multiplexed untuk:
  - Kontrol WebSocket/RPC
  - API HTTP (kompatibel dengan OpenAI, Responses, pemanggilan alat)
  - Control UI dan hook
- Mode bind default: `loopback`.
- Autentikasi diperlukan secara default (`gateway.auth.token` / `gateway.auth.password`, atau `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Prioritas port dan bind

| Pengaturan   | Urutan penyelesaian                                           |
| ------------ | ------------------------------------------------------------- |
| Port Gateway | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Mode Bind    | CLI/override → `gateway.bind` → `loopback`                    |

### Mode hot reload

| `gateway.reload.mode` | Perilaku                                                   |
| --------------------- | ---------------------------------------------------------- |
| `off`                 | Tidak ada muat ulang konfigurasi                           |
| `hot`                 | Hanya terapkan perubahan yang aman bagi runtime (hot-safe) |
| `restart`             | Restart pada perubahan yang memerlukan muat ulang penuh    |
| `hybrid` (default)    | Terapkan langsung jika aman, restart jika diperlukan       |

## Kumpulan perintah operator

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## Akses jarak jauh (Remote access)

Disukai: Tailscale/VPN.
Cadangan: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Kemudian hubungkan klien ke `ws://127.0.0.1:18789` secara lokal.

<Warning>
Jika autentikasi gateway dikonfigurasi, klien tetap harus mengirimkan autentikasi (`token`/`password`) bahkan melalui SSH tunnel.
</Warning>

Lihat: [Gateway Jarak Jauh](/gateway/remote), [Autentikasi](/id-ID/gateway/authentication), [Tailscale](/gateway/tailscale).

## Pengawasan dan siklus hidup layanan

Gunakan proses yang diawasi (supervised) untuk keandalan seperti lingkungan produksi.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

Label LaunchAgent adalah `ai.openclaw.gateway` (default) atau `ai.openclaw.<profile>` (profil bernama). `openclaw doctor` mengaudit dan memperbaiki penyimpangan konfigurasi layanan.

  </Tab>

  <Tab title="Linux (user systemd)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Agar tetap berjalan setelah logout, aktifkan lingering:

```bash
sudo loginctl enable-linger <user>
```

  </Tab>

  <Tab title="Linux (layanan sistem)">

Gunakan unit sistem untuk host multi-pengguna/selalu-nyala.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  </Tab>
</Tabs>

## Beberapa gateway pada satu host

Sebagian besar pengaturan seharusnya hanya menjalankan **satu** Gateway.
Gunakan lebih dari satu hanya untuk isolasi ketat/redundansi (misalnya profil penyelamatan).

Daftar periksa per instance:

- `gateway.port` unik
- `OPENCLAW_CONFIG_PATH` unik
- `OPENCLAW_STATE_DIR` unik
- `agents.defaults.workspace` unik

Contoh:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Lihat: [Beberapa gateway](/gateway/multiple-gateways).

### Jalur cepat profil dev

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Default mencakup state/konfigurasi yang terisolasi dan port gateway dasar `19001`.

## Referensi cepat protokol (sudut pandang operator)

- Frame klien pertama harus berupa `connect`.
- Gateway mengembalikan snapshot `hello-ok` (mencakup `presence`, `health`, `stateVersion`, `uptimeMs`, batas/kebijakan).
- Permintaan (Request): `req(method, params)` → `res(ok/payload|error)`.
- Event umum: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Proses agen melalui dua tahap:

1. Ack diterima segera (`status:"accepted"`)
2. Respons penyelesaian akhir (`status:"ok"|"error"`), dengan event `agent` yang di-stream di antaranya.

Lihat dokumen protokol lengkap: [Protokol Gateway](/gateway/protocol).

## Pemeriksaan operasional

### Liveness

- Buka WS dan kirim `connect`.
- Harapkan respons `hello-ok` dengan snapshot.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Pemulihan celah (Gap recovery)

Event tidak diputar ulang. Jika terjadi celah urutan, segarkan status (`health`, `system-presence`) sebelum melanjutkan.

## Tanda-tanda kegagalan umum

| Tanda (Signature)                                              | Kemungkinan masalah                                 |
| -------------------------------------------------------------- | --------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Bind bukan loopback tanpa token/password            |
| `another gateway instance is already listening` / `EADDRINUSE` | Konflik port                                        |
| `Gateway start blocked: set gateway.mode=local`                | Konfigurasi diatur ke mode remote                   |
| `unauthorized` saat koneksi                                    | Ketidakcocokan autentikasi antara klien dan gateway |

Untuk tahapan diagnosis lengkap, gunakan [Pemecahan Masalah Gateway](/id-ID/gateway/troubleshooting).

## Jaminan keamanan

- Klien protokol Gateway gagal dengan cepat saat Gateway tidak tersedia (tidak ada fallback saluran langsung yang implisit).
- Frame pertama yang tidak valid/bukan connect akan ditolak dan ditutup.
- Shutdown yang anggun (graceful) memancarkan event `shutdown` sebelum socket ditutup.

---

Terkait:

- [Pemecahan Masalah](/id-ID/gateway/troubleshooting)
- [Proses Latar Belakang](/gateway/background-process)
- [Konfigurasi](/id-ID/gateway/configuration)
- [Kesehatan (Health)](/gateway/health)
- [Doctor](/id-ID/gateway/doctor)
- [Autentikasi](/id-ID/gateway/authentication)

