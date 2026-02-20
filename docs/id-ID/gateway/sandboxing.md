---
summary: "Cara kerja sandboxing OpenClaw: mode, cakupan, akses workspace, dan citra (image)"
read_when:
  - Anda ingin penjelasan khusus tentang sandboxing
  - Anda perlu menyetel agents.defaults.sandbox
title: "Sandboxing"
---

# Sandboxing

OpenClaw dapat menjalankan **alat di dalam kontainer Docker** untuk mengurangi radius dampak (blast radius) jika terjadi kesalahan. Fitur ini bersifat **opsional** dan dikontrol melalui konfigurasi (`agents.defaults.sandbox` atau `agents.list[].sandbox`). Jika sandboxing dimatikan, alat akan berjalan di host.

Gateway tetap berada di host, sedangkan eksekusi alat berjalan di dalam sandbox yang terisolasi jika diaktifkan.

Ini bukanlah batas keamanan yang sempurna, tetapi secara signifikan membatasi akses sistem file dan proses ketika model melakukan sesuatu yang tidak terduga.

## Apa yang di-sandbox

- Eksekusi alat (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, dll.).
- Browser sandbox opsional (`agents.defaults.sandbox.browser`).
  - Secara default, browser sandbox otomatis dimulai (memastikan CDP dapat dijangkau) saat alat browser membutuhkannya. Konfigurasikan via `agents.defaults.sandbox.browser.autoStart` dan `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  - `agents.defaults.sandbox.browser.allowHostControl` memungkinkan sesi sandbox menargetkan browser host secara eksplisit.
  - Daftar izinkan (allowlist) opsional untuk membatasi `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Yang tidak di-sandbox:

- Proses Gateway itu sendiri.
- Alat apa pun yang secara eksplisit diizinkan berjalan di host (misal: `tools.elevated`).
  - **Eksekusi 'elevated' berjalan di host dan melewati sandboxing.**
  - Jika sandboxing dalam kondisi mati, `tools.elevated` tidak mengubah cara eksekusi (memang sudah di host). Lihat [Mode Elevated](/id-ID/tools/elevated).

## Mode

`agents.defaults.sandbox.mode` mengontrol **kapan** sandboxing digunakan:

- `"off"`: tidak ada sandboxing.
- `"non-main"`: sandbox hanya untuk sesi **non-utama** (default jika Anda ingin chat normal berjalan di host).
- `"all"`: setiap sesi berjalan di dalam sandbox.
  Catatan: `"non-main"` didasarkan pada `session.mainKey` (defaultnya `"main"`), bukan ID agen. Sesi grup/saluran menggunakan kunci mereka sendiri, sehingga dianggap sebagai non-utama dan akan di-sandbox.

## Cakupan (Scope)

`agents.defaults.sandbox.scope` mengontrol **berapa banyak kontainer** yang dibuat:

- `"session"` (default): satu kontainer per sesi.
- `"agent"`: satu kontainer per agen.
- `"shared"`: satu kontainer yang digunakan bersama oleh semua sesi yang di-sandbox.

## Akses Workspace

`agents.defaults.sandbox.workspaceAccess` mengontrol **apa yang dapat dilihat oleh sandbox**:

- `"none"` (default): alat melihat workspace sandbox di bawah `~/.openclaw/sandboxes`.
- `"ro"`: melakukan mount workspace agen secara baca-saja (read-only) di `/agent` (menonaktifkan `write`/`edit`/`apply_patch`).
- `"rw"`: melakukan mount workspace agen secara baca/tulis di `/workspace`.

Media masuk disalin ke dalam workspace sandbox yang aktif (`media/inbound/*`).
Catatan Skills: alat `read` berakar pada sandbox (sandbox-rooted). Dengan `workspaceAccess: "none"`, OpenClaw menduplikasi skill yang memenuhi syarat ke dalam workspace sandbox (`.../skills`) agar dapat dibaca. Dengan `"rw"`, skill dalam workspace dapat dibaca dari `/workspace/skills`.

## Bind mount kustom

`agents.defaults.sandbox.docker.binds` melakukan mount direktori host tambahan ke dalam kontainer. Format: `host:kontainer:mode` (contoh: `"/home/user/source:/source:rw"`).

Bind global dan per-agen akan **digabungkan** (tidak diganti). Pada `scope: "shared"`, bind per-agen akan diabaikan.

`agents.defaults.sandbox.browser.binds` melakukan mount direktori host tambahan hanya ke dalam kontainer **browser sandbox**.

- Jika diatur (termasuk `[]`), ini menggantikan `agents.defaults.sandbox.docker.binds` untuk kontainer browser.
- Jika dikosongkan, kontainer browser akan menggunakan `agents.defaults.sandbox.docker.binds` (kompatibel dengan versi lama).

Contoh (sumber baca-saja + direktori data tambahan):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

Catatan keamanan:

- Bind melewati sistem file sandbox: mereka mengekspos jalur host dengan mode apa pun yang Anda atur (`:ro` atau `:rw`).
- OpenClaw memblokir sumber bind yang berbahaya (misalnya: `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev`, dan mount induk yang akan mengekspos mereka).
- Mount sensitif (rahasia, kunci SSH, kredensial layanan) harus diatur ke `:ro` kecuali benar-benar diperlukan.
- Gabungkan dengan `workspaceAccess: "ro"` jika Anda hanya membutuhkan akses baca ke workspace; mode bind tetap berdiri sendiri.
- Lihat [Sandbox vs Tool Policy vs Elevated](/id-ID/gateway/sandbox-vs-tool-policy-vs-elevated) untuk melihat bagaimana bind berinteraksi dengan kebijakan alat dan eksekusi elevated.

## Citra (Image) + Pengaturan

Citra default: `openclaw-sandbox:bookworm-slim`

Bangun citra tersebut sekali:

```bash
scripts/sandbox-setup.sh
```

Catatan: citra default **tidak** menyertakan Node. Jika sebuah skill membutuhkan Node (atau runtime lainnya), Anda bisa memasukannya ke citra kustom (bake) atau instal via `sandbox.docker.setupCommand` (memerlukan akses internet/egress + root yang dapat ditulis + pengguna root).

Citra browser sandbox:

```bash
scripts/sandbox-browser-setup.sh
```

Secara default, kontainer sandbox berjalan **tanpa jaringan**. Ubah dengan `agents.defaults.sandbox.docker.network`.

Instalasi Docker dan gateway dalam kontainer dibahas di sini: [Docker](/id-ID/install/docker)

## setupCommand (pengaturan kontainer sekali jalan)

`setupCommand` berjalan **satu kali** setelah kontainer sandbox dibuat (tidak pada setiap jalankan). Perintah ini dieksekusi di dalam kontainer melalui `sh -lc`.

Jalur:

- Global: `agents.defaults.sandbox.docker.setupCommand`
- Per-agen: `agents.list[].sandbox.docker.setupCommand`

Kesalahan umum:

- Default `docker.network` adalah `"none"` (tidak ada akses internet/egress), sehingga instalasi paket akan gagal.
- `readOnlyRoot: true` mencegah penulisan file; atur ke `readOnlyRoot: false` atau gunakan citra kustom.
- `user` haruslah root untuk instalasi paket (kosongkan `user` atau atur `user: "0:0"`).
- Eksekusi sandbox **tidak** mewarisi `process.env` dari host. Gunakan `agents.defaults.sandbox.docker.env` (atau citra kustom) untuk kunci API skill.

## Kebijakan alat + lubang penyelamatan (escape hatch)

Kebijakan izinkan/tolak alat tetap berlaku sebelum aturan sandbox. Jika sebuah alat ditolak secara global atau per-agen, sandboxing tidak akan mengaktifkannya kembali.

`tools.elevated` adalah lubang penyelamatan (escape hatch) eksplisit yang menjalankan `exec` di host. Direktif `/exec` hanya berlaku untuk pengirim yang berwenang dan menetap per sesi; untuk benar-benar menonaktifkan `exec`, gunakan kebijakan tolak pada alat (lihat [Sandbox vs Tool Policy vs Elevated](/id-ID/gateway/sandbox-vs-tool-policy-vs-elevated)).

Debugging:

- Gunakan `openclaw sandbox explain` untuk memeriksa mode sandbox yang efektif, kebijakan alat, dan kunci konfigurasi perbaikan.
- Lihat [Sandbox vs Tool Policy vs Elevated](/id-ID/gateway/sandbox-vs-tool-policy-vs-elevated) untuk memahami model mental "mengapa ini diblokir?". Jaga agar tetap terkunci rapat.

## Override Multi-agen

Masing-masing agen dapat menimpa (override) pengaturan sandbox + alat: `agents.list[].sandbox` dan `agents.list[].tools` (ditambah `agents.list[].tools.sandbox.tools` untuk kebijakan alat sandbox). Lihat [Sandbox & Alat Multi-Agen](/id-ID/tools/multi-agent-sandbox-tools) untuk urutan prioritas.

## Contoh pengaktifan minimal

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## Dokumen terkait

- [Konfigurasi Sandbox](/id-ID/gateway/configuration#agentsdefaults-sandbox)
- [Sandbox & Alat Multi-Agen](/id-ID/tools/multi-agent-sandbox-tools)
- [Keamanan](/id-ID/gateway/security)


