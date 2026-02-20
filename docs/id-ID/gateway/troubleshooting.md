---
summary: "Runbook pemecahan masalah mendalam untuk gateway, saluran, otomatisasi, node, dan browser"
read_when:
  - Pusat pemecahan masalah mengarahkan Anda ke sini untuk diagnosis lebih dalam
  - Anda memerlukan bagian runbook berbasis gejala yang stabil dengan perintah yang tepat
title: "Pemecahan Masalah"
---

# Pemecahan masalah Gateway

Halaman ini adalah runbook mendalam.
Mulailah dari [/help/troubleshooting](/help/troubleshooting) jika Anda menginginkan alur triase cepat terlebih dahulu.

## Urutan perintah (Command ladder)

Jalankan perintah ini terlebih dahulu, sesuai urutan:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Sinyal sehat yang diharapkan:

- `openclaw gateway status` menunjukkan `Runtime: running` dan `RPC probe: ok`.
- `openclaw doctor` melaporkan tidak ada masalah konfigurasi/layanan yang memblokir.
- `openclaw channels status --probe` menunjukkan saluran yang terhubung/siap.

## Tidak ada balasan (No replies)

Jika saluran aktif tetapi tidak ada yang menjawab, periksa perutean dan kebijakan sebelum menghubungkan ulang apa pun.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list <channel>
openclaw config get channels
openclaw logs --follow
```

Periksa hal berikut:

- Pairing tertunda untuk pengirim DM.
- Penyaringan sebutan grup (`requireMention`, `mentionPatterns`).
- Ketidakcocokan daftar izinkan (allowlist) saluran/grup.

Tanda (signature) umum:

- `drop guild message (mention required` → pesan grup diabaikan karena tidak ada sebutan (mention).
- `pairing request` → pengirim membutuhkan persetujuan.
- `blocked` / `allowlist` → pengirim/saluran difilter oleh kebijakan.

Terkait:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/pairing](/channels/pairing)
- [/channels/groups](/channels/groups)

## Konektivitas Dashboard Control UI

Ketika dashboard/control UI tidak dapat terhubung, validasi URL, mode autentikasi, dan asumsi konteks aman (secure context).

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Periksa hal berikut:

- URL probe dan URL dashboard yang benar.
- Ketidakcocokan mode autentikasi/token antara klien dan gateway.
- Penggunaan HTTP di mana identitas perangkat (device identity) diperlukan.

Tanda (signature) umum:

- `device identity required` → konteks tidak aman (non-secure context) atau autentikasi perangkat hilang.
- `unauthorized` / loop penyambungan ulang → ketidakcocokan token/kata sandi.
- `gateway connect failed:` → target host/port/url salah.

Terkait:

- [/web/control-ui](/web/control-ui)
- [/id-ID/gateway/authentication](/id-ID/gateway/authentication)
- [/gateway/remote](/gateway/remote)

## Layanan Gateway tidak berjalan

Gunakan ini jika layanan sudah terinstal tetapi prosesnya tidak tetap berjalan (mati).

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Periksa hal berikut:

- `Runtime: stopped` dengan petunjuk alasan keluar.
- Ketidakcocokan konfigurasi layanan (`Config (cli)` vs `Config (service)`).
- Konflik port/listener.

Tanda (signature) umum:

- `Gateway start blocked: set gateway.mode=local` → mode gateway lokal tidak diaktifkan. Perbaikan: atur `gateway.mode="local"` di konfigurasi Anda (atau jalankan `openclaw configure`). Jika Anda menjalankan OpenClaw via Podman menggunakan pengguna `openclaw` khusus, konfigurasinya berada di `~openclaw/.openclaw/openclaw.json`.
- `refusing to bind gateway ... without auth` → bind bukan loopback tanpa token/kata sandi.
- `another gateway instance is already listening` / `EADDRINUSE` → konflik port.

Terkait:

- [/gateway/background-process](/gateway/background-process)
- [/id-ID/gateway/configuration](/id-ID/gateway/configuration)
- [/id-ID/gateway/doctor](/id-ID/gateway/doctor)

## Saluran terhubung tetapi pesan tidak mengalir

Jika status saluran menunjukkan terhubung (connected) tetapi aliran pesan mati, fokuslah pada kebijakan, izin, dan aturan pengiriman khusus saluran.

```bash
openclaw channels status --probe
openclaw pairing list <channel>
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Periksa hal berikut:

- Kebijakan DM (`pairing`, `allowlist`, `open`, `disabled`).
- Daftar izinkan grup dan persyaratan sebutan (mention).
- Izin/cakupan (scope) API saluran yang hilang.

Tanda (signature) umum:

- `mention required` → pesan diabaikan oleh kebijakan sebutan grup.
- `pairing` / jejak persetujuan tertunda → pengirim belum disetujui.
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → masalah autentikasi/izin saluran.

Terkait:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/whatsapp](/channels/whatsapp)
- [/channels/telegram](/channels/telegram)
- [/channels/discord](/channels/discord)

## Pengiriman Cron dan Heartbeat

Jika cron atau heartbeat tidak berjalan atau tidak terkirim, verifikasi status penjadwal (scheduler) terlebih dahulu, lalu target pengiriman.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Periksa hal berikut:

- Cron diaktifkan dan waktu bangun berikutnya ada.
- Status riwayat jalannya pekerjaan (`ok`, `skipped`, `error`).
- Alasan skip heartbeat (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Tanda (signature) umum:

- `cron: scheduler disabled; jobs will not run automatically` → cron dinonaktifkan.
- `cron: timer tick failed` → detak penjadwal (scheduler tick) gagal; periksa kesalahan file/log/runtime.
- `heartbeat skipped` dengan `reason=quiet-hours` → di luar jendela jam aktif.
- `heartbeat: unknown accountId` → ID akun tidak valid untuk target pengiriman heartbeat.

Terkait:

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Alat yang terhubung ke Node gagal

Jika node terpasang (paired) tetapi alat (tools) gagal, isolasi status latar depan (foreground), izin, dan persetujuan.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Periksa hal berikut:

- Node online dengan kapabilitas yang diharapkan.
- Pemberian izin OS untuk kamera/mikrofon/lokasi/layar.
- Persetujuan eksekusi (exec) dan status daftar izinkan (allowlist).

Tanda (signature) umum:

- `NODE_BACKGROUND_UNAVAILABLE` → aplikasi node harus berada di latar depan (foreground).
- `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → izin OS hilang.
- `SYSTEM_RUN_DENIED: approval required` → persetujuan eksekusi tertunda.
- `SYSTEM_RUN_DENIED: allowlist miss` → perintah diblokir oleh daftar izinkan.

Terkait:

- [/nodes/troubleshooting](/nodes/troubleshooting)
- [/nodes/index](/nodes/index)
- [/tools/exec-approvals](/tools/exec-approvals)

## Alat Browser gagal

Gunakan ini jika tindakan alat browser gagal meskipun gateway itu sendiri sehat.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Periksa hal berikut:

- Jalur executable browser yang valid.
- Aksesibilitas profil CDP.
- Lampiran tab relay ekstensi untuk `profile="chrome"`.

Tanda (signature) umum:

- `Failed to start Chrome CDP on port` → proses browser gagal diluncurkan.
- `browser.executablePath not found` → jalur yang dikonfigurasi tidak valid.
- `Chrome extension relay is running, but no tab is connected` → relay ekstensi tidak terpasang.
- `Browser attachOnly is enabled ... not reachable` → profil attach-only tidak memiliki target yang dapat dijangkau.

Terkait:

- [/tools/browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
- [/tools/chrome-extension](/tools/chrome-extension)
- [/tools/browser](/tools/browser)

## Jika Anda melakukan upgrade dan terjadi masalah tiba-tiba

Sebagian besar kerusakan setelah upgrade disebabkan oleh penyimpangan (drift) konfigurasi atau penegakan nilai default yang sekarang lebih ketat.

### 1) Perilaku override Auth dan URL berubah

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Yang perlu diperiksa:

- Jika `gateway.mode=remote`, panggilan CLI mungkin menargetkan remote padahal layanan lokal Anda baik-baik saja.
- Panggilan eksplisit dengan `--url` tidak menggunakan kredensial yang tersimpan sebagai cadangan.

Tanda (signature) umum:

- `gateway connect failed:` → target URL salah.
- `unauthorized` → titik akhir (endpoint) dapat dijangkau tetapi autentikasi salah.

### 2) Pembatasan (Guardrail) Bind dan Auth lebih ketat

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Yang perlu diperiksa:

- Bind selain loopback (`lan`, `tailnet`, `custom`) memerlukan autentikasi yang dikonfigurasi.
- Kunci lama seperti `gateway.token` tidak menggantikan `gateway.auth.token`.

Tanda (signature) umum:

- `refusing to bind gateway ... without auth` → ketidakcocokan bind+auth.
- `RPC probe: failed` saat runtime sedang berjalan → gateway aktif tetapi tidak dapat diakses dengan autentikasi/URL saat ini.

### 3) Status Pairing dan Identitas Perangkat berubah

```bash
openclaw devices list
openclaw pairing list <channel>
openclaw logs --follow
openclaw doctor
```

Yang perlu diperiksa:

- Persetujuan perangkat yang tertunda untuk dashboard/node.
- Persetujuan pairing DM yang tertunda setelah perubahan kebijakan atau identitas.

Tanda (signature) umum:

- `device identity required` → autentikasi perangkat tidak terpenuhi.
- `pairing required` → pengirim/perangkat harus disetujui.

Jika konfigurasi layanan dan runtime tetap tidak sesuai setelah pemeriksaan, instal ulang metadata layanan dari profil/direktori state yang sama:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Terkait:

- [/gateway/pairing](/gateway/pairing)
- [/id-ID/gateway/authentication](/id-ID/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)


