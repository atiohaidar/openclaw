---
summary: "Wizard onboarding CLI: pengaturan terpandu untuk gateway, workspace, saluran, dan skill"
read_when:
  - Menjalankan atau mengonfigurasi wizard onboarding
  - Menyiapkan mesin baru
title: "Wizard Onboarding (CLI)"
sidebarTitle: "Onboarding: CLI"
---

# Wizard Onboarding (CLI)

Wizard onboarding adalah cara yang **direkomendasikan** untuk menyiapkan OpenClaw di macOS,
Linux, atau Windows (melalui WSL2; sangat direkomendasikan).
Wizard ini mengonfigurasi koneksi Gateway lokal atau Gateway jarak jauh (remote), ditambah saluran (channel), skill,
dan pengaturan default workspace dalam satu alur yang terpandu.

```bash
openclaw onboard
```

<Info>
Chat pertama tercepat: buka Control UI (tidak perlu pengaturan saluran). Jalankan
`openclaw dashboard` dan chat di browser. Dokumentasi: [Dashboard](/web/dashboard).
</Info>

Untuk mengonfigurasi ulang nanti:

```bash
openclaw configure
openclaw agents add <nama>
```

<Note>
`--json` tidak berarti mode non-interaktif. Untuk skrip, gunakan `--non-interactive`.
</Note>

<Tip>
Direkomendasikan: siapkan kunci API Brave Search agar agen dapat menggunakan `web_search`
(`web_fetch` dapat berfungsi tanpa kunci). Jalur termudah: `openclaw configure --section web`
yang menyimpan `tools.web.search.apiKey`. Dokumentasi: [Alat web](/tools/web).
</Tip>

## QuickStart vs Tingkat Lanjut (Advanced)

Wizard dimulai dengan pilihan **QuickStart** (default) vs **Advanced** (kontrol penuh).

<Tabs>
  <Tab title="QuickStart (default)">
    - Gateway lokal (loopback)
    - Default workspace (atau workspace yang sudah ada)
    - Port Gateway **18789**
    - Autentikasi Gateway **Token** (dibuat otomatis, bahkan pada loopback)
    - Eksposur Tailscale **Mati (Off)**
    - Pesan Langsung (DM) Telegram + WhatsApp secara default menggunakan **daftar izinkan (allowlist)** (Anda akan dimintai nomor telepon Anda)
  </Tab>
  <Tab title="Tingkat Lanjut (kontrol penuh)">
    - Menampilkan setiap langkah (mode, workspace, gateway, saluran, daemon, skill).
  </Tab>
</Tabs>

## Apa yang dikonfigurasi oleh wizard

**Mode lokal (default)** memandu Anda melalui langkah-langkah berikut:

1. **Model/Autentikasi** — Kunci API Anthropic (direkomendasikan), OpenAI, atau Penyedia Kustom
   (kompatibel dengan OpenAI, kompatibel dengan Anthropic, atau deteksi otomatis Tidak Diketahui). Pilih model default.
2. **Workspace** — Lokasi untuk file agen (default `~/.openclaw/workspace`). Menyiapkan file bootstrap awal.
3. **Gateway** — Port, alamat bind, mode autentikasi, eksposur Tailscale.
4. **Saluran (Channels)** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles, atau iMessage.
5. **Daemon** — Menginstal LaunchAgent (macOS) atau unit pengguna systemd (Linux/WSL2).
6. **Pemeriksaan kesehatan (Health check)** — Menjalankan Gateway dan memverifikasi bahwa layanan telah berjalan.
7. **Skill** — Menginstal skill yang direkomendasikan dan dependensi opsional.

<Note>
Menjalankan ulang wizard **tidak** akan menghapus apa pun kecuali Anda secara eksplisit memilih **Reset** (atau menambahkan argumen `--reset`).
Jika konfigurasi tidak valid atau berisi kunci lama (legacy), wizard akan meminta Anda menjalankan `openclaw doctor` terlebih dahulu.
</Note>

**Mode jarak jauh (Remote mode)** hanya mengonfigurasi klien lokal untuk terhubung ke Gateway di tempat lain.
Mode ini **tidak** menginstal atau mengubah apa pun di host jarak jauh.

## Menambah agen lain

Gunakan `openclaw agents add <nama>` untuk membuat agen terpisah dengan workspace,
sesi, dan profil autentikasi sendiri. Menjalankan perintah tanpa `--workspace` akan meluncurkan wizard.

Apa yang diatur:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

Catatan:

- Workspace default mengikuti pola `~/.openclaw/workspace-<agentId>`.
- Tambahkan `bindings` untuk merutekan pesan masuk (wizard dapat melakukan ini).
- Bendera (flag) non-interaktif: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Referensi lengkap

Untuk rincian langkah demi langkah yang mendalam, pembuatan skrip non-interaktif, pengaturan Signal,
API RPC, dan daftar lengkap kolom konfigurasi yang ditulis wizard, lihat
[Referensi Wizard](/id-ID/reference/wizard).

## Dokumen terkait

- Referensi perintah CLI: [`openclaw onboard`](/cli/onboard)
- Ringkasan onboarding: [Ringkasan Onboarding](/id-ID/start/onboarding-overview)
- Onboarding aplikasi macOS: [Onboarding](/id-ID/start/onboarding)
- Ritual pertama kali agen: [Bootstrapping Agen](/id-ID/start/bootstrapping)
