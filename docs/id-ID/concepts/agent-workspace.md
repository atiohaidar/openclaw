---
summary: "Workspace agen: lokasi, tata letak, dan strategi pencadangan"
read_when:
  - Anda perlu menjelaskan workspace agen atau tata letak file-nya
  - Anda ingin mencadangkan atau memigrasi workspace agen
title: "Workspace Agen"
---

# Workspace agen

Workspace adalah rumah bagi agen. Ini adalah satu-satunya direktori kerja yang digunakan untuk
alat file dan untuk konteks workspace. Jaga agar tetap pribadi dan perlakukan sebagai memori.

Ini terpisah dari `~/.openclaw/`, yang menyimpan konfigurasi, kredensial, dan
sesi.

**Penting:** workspace adalah **cwd default**, bukan sandbox keras. Alat
menyelesaikan jalur relatif terhadap workspace, tetapi jalur absolut masih dapat menjangkau
tempat lain di host kecuali sandboxing diaktifkan. Jika Anda memerlukan isolasi, gunakan
[`agents.defaults.sandbox`](/id-ID/gateway/sandboxing) (dan/atau konfigurasi sandbox per-agen).
Saat sandboxing diaktifkan dan `workspaceAccess` tidak `"rw"`, alat beroperasi
di dalam workspace sandbox di bawah `~/.openclaw/sandboxes`, bukan workspace host Anda.

## Lokasi default

- Default: `~/.openclaw/workspace`
- Jika `OPENCLAW_PROFILE` diatur dan bukan `"default"`, default menjadi
  `~/.openclaw/workspace-<profile>`.
- Timpa di `~/.openclaw/openclaw.json`:

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`, `openclaw configure`, atau `openclaw setup` akan membuat
workspace dan menyemai file bootstrap jika file tersebut hilang.

Jika Anda sudah mengelola file workspace sendiri, Anda dapat menonaktifkan pembuatan file bootstrap:

```json5
{ agent: { skipBootstrap: true } }
```

## Folder workspace ekstra

Instalasi lama mungkin telah membuat `~/openclaw`. Menyimpan beberapa direktori
workspace dapat menyebabkan kebingungan autentikasi atau penyimpangan status, karena hanya satu
workspace yang aktif pada satu waktu.

**Rekomendasi:** simpan satu workspace aktif. Jika Anda tidak lagi menggunakan
folder ekstra, arsipkan atau pindahkan ke Tempat Sampah (misalnya `trash ~/openclaw`).
Jika Anda sengaja menyimpan beberapa workspace, pastikan
`agents.defaults.workspace` menunjuk ke yang aktif.

`openclaw doctor` memperingatkan saat mendeteksi direktori workspace ekstra.

## Peta file workspace (arti setiap file)

Ini adalah file standar yang diharapkan OpenClaw di dalam workspace:

- `AGENTS.md`
  - Instruksi operasi untuk agen dan bagaimana ia harus menggunakan memori.
  - Dimuat pada awal setiap sesi.
  - Tempat yang baik untuk aturan, prioritas, dan detail "cara berperilaku".

- `SOUL.md`
  - Persona, nada bicara, dan batasan.
  - Dimuat setiap sesi.

- `USER.md`
  - Siapa pengguna dan bagaimana menyapanya.
  - Dimuat setiap sesi.

- `IDENTITY.md`
  - Nama agen, vibe, dan emoji (🦞).
  - Dibuat/diperbarui selama ritual bootstrap.

- `TOOLS.md`
  - Catatan tentang alat lokal dan konvensi Anda.
  - Tidak mengontrol ketersediaan alat; ini hanya panduan.

- `HEARTBEAT.md`
  - Daftar periksa kecil opsional untuk proses heartbeat.
  - Buatlah singkat untuk menghindari pembakaran token.

- `BOOT.md`
  - Daftar periksa startup opsional yang dijalankan pada restart gateway saat hook internal diaktifkan.
  - Buatlah singkat; gunakan alat pesan untuk pengiriman keluar.

- `BOOTSTRAP.md`
  - Ritual penggunaan pertama kali satu kali.
  - Hanya dibuat untuk workspace yang benar-benar baru.
  - Hapus setelah ritual selesai.

- `memory/YYYY-MM-DD.md`
  - Log memori harian (satu file per hari).
  - Disarankan untuk membaca hari ini + kemarin pada awal sesi.

- `MEMORY.md` (opsional)
  - Memori jangka panjang yang dikurasi.
  - Hanya dimuat di sesi utama pribadi (bukan konteks bersama/grup).

Lihat [Memori](/id-ID/concepts/memory) untuk alur kerja dan flush memori otomatis.

- `skills/` (opsional)
  - Skill spesifik workspace.
  - Menimpa skill yang dikelola/dibundel saat nama bertabrakan.

- `canvas/` (opsional)
  - File UI Canvas untuk tampilan node (misalnya `canvas/index.html`).

Jika file bootstrap hilang, OpenClaw menyuntikkan penanda "file hilang" ke dalam
sesi dan melanjutkan. File bootstrap besar dipotong saat disuntikkan;
sesuaikan batas dengan `agents.defaults.bootstrapMaxChars` (default: 20000) dan
`agents.defaults.bootstrapTotalMaxChars` (default: 150000).
`openclaw setup` dapat membuat ulang default yang hilang tanpa menimpa file
yang ada.

## Apa yang TIDAK ada di workspace

Ini berada di bawah `~/.openclaw/` dan TIDAK boleh dikomit ke repo workspace:

- `~/.openclaw/openclaw.json` (konfigurasi)
- `~/.openclaw/credentials/` (token OAuth, kunci API)
- `~/.openclaw/agents/<agentId>/sessions/` (transkrip sesi + metadata)
- `~/.openclaw/skills/` (skill yang dikelola)

Jika Anda perlu memigrasi sesi atau konfigurasi, salin secara terpisah dan simpan
di luar kontrol versi.

## Pencadangan Git (direkomendasikan, pribadi)

Perlakukan workspace sebagai memori pribadi. Letakkan di repo git **pribadi** (private) sehingga
dicadangkan dan dapat dipulihkan.

Jalankan langkah-langkah ini di mesin tempat Gateway berjalan (di situlah
workspace berada).

### 1) Inisialisasi repo

Jika git terinstal, workspace baru diinisialisasi secara otomatis. Jika workspace ini
belum menjadi repo, jalankan:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) Tambahkan remote pribadi (opsi ramah pemula)

Opsi A: UI Web GitHub

1. Buat repositori **pribadi** baru di GitHub.
2. Jangan inisialisasi dengan README (menghindari konflik merge).
3. Salin URL remote HTTPS.
4. Tambahkan remote dan push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Opsi B: CLI GitHub (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Opsi C: UI Web GitLab

1. Buat repositori **pribadi** baru di GitLab.
2. Jangan inisialisasi dengan README (menghindari konflik merge).
3. Salin URL remote HTTPS.
4. Tambahkan remote dan push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) Pembaruan berkelanjutan

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## Jangan komit rahasia

Bahkan di repo pribadi, hindari menyimpan rahasia di workspace:

- Kunci API, token OAuth, kata sandi, atau kredensial pribadi.
- Apa pun di bawah `~/.openclaw/`.
- Dump mentah dari chat atau lampiran sensitif.

Jika Anda harus menyimpan referensi sensitif, gunakan placeholder dan simpan rahasia
nyata di tempat lain (pengelola kata sandi, variabel lingkungan, atau `~/.openclaw/`).

Starter `.gitignore` yang disarankan:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## Memindahkan workspace ke mesin baru

1. Clone repo ke jalur yang diinginkan (default `~/.openclaw/workspace`).
2. Atur `agents.defaults.workspace` ke jalur tersebut di `~/.openclaw/openclaw.json`.
3. Jalankan `openclaw setup --workspace <path>` untuk menyemai file yang hilang.
4. Jika Anda memerlukan sesi, salin `~/.openclaw/agents/<agentId>/sessions/` dari
   mesin lama secara terpisah.

## Catatan Tingkat Lanjut

- Perutean multi-agen dapat menggunakan workspace yang berbeda per agen. Lihat
  [Perutean Saluran](/id-ID/channels/channel-routing) untuk konfigurasi perutean.
- Jika `agents.defaults.sandbox` diaktifkan, sesi non-utama dapat menggunakan workspace
  sandbox per-sesi di bawah `agents.defaults.sandbox.workspaceRoot`.


