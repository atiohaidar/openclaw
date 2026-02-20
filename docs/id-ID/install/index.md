---
summary: "Cara Menginstal OpenClaw — skrip instalasi, npm/pnpm, dari sumber, Docker, dan lainnya"
read_when:
  - Anda memerlukan metode instalasi selain jalur cepat Panduan Memulai
  - Anda ingin mendeploy ke platform cloud
  - Anda perlu memperbarui, memigrasi, atau menghapus instalasi
title: "Instal"
---

# Instal

Sudah mengikuti [Panduan Memulai](/id-ID/start/getting-started)? Anda sudah siap — halaman ini ditujukan untuk metode instalasi alternatif, instruksi spesifik platform, dan pemeliharaan.

## Persyaratan sistem

- **[Node 22+](/id-ID/install/node)** ([skrip instalasi](#metode-instalasi) akan menginstalnya jika belum ada)
- macOS, Linux, atau Windows
- `pnpm` hanya diperlukan jika Anda membangun dari sumber (source)

<Note>
Di Windows, kami sangat merekomendasikan menjalankan OpenClaw di bawah [WSL2](https://learn.microsoft.com/id-id/windows/wsl/install).
</Note>

## Metode instalasi

<Tip>
**Skrip instalasi** adalah cara yang direkomendasikan untuk menginstal OpenClaw. Skrip ini menangani deteksi Node, instalasi, dan onboarding dalam satu langkah.
</Tip>

<Warning>
Untuk host VPS/cloud, hindari penggunaan citra (image) marketplace "1-click" pihak ketiga jika memungkinkan. Lebih baik gunakan citra OS dasar yang bersih (misalnya Ubuntu LTS), lalu instal OpenClaw sendiri menggunakan skrip instalasi.
</Warning>

<AccordionGroup>
  <Accordion title="Skrip instalasi" icon="rocket" defaultOpen>
    Mengunduh CLI, menginstalnya secara global via npm, dan meluncurkan wizard onboarding.

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    Selesai — skrip menangani deteksi Node, instalasi, dan onboarding.

    Untuk melewati onboarding dan hanya menginstal binernya saja:

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
        ```
      </Tab>
    </Tabs>

    Untuk semua bendera (flag), variabel lingkungan (env var), dan opsi CI/otomatisasi, lihat [Detail teknis installer](/id-ID/install/installer).

  </Accordion>

  <Accordion title="npm / pnpm" icon="package">
    Jika Anda sudah memiliki Node 22+ dan lebih suka mengelola instalasi sendiri:

    <Tabs>
      <Tab title="npm">
        ```bash
        npm install -g openclaw@latest
        openclaw onboard --install-daemon
        ```

        <Accordion title="error saat build sharp?">
          Jika Anda memiliki libvips yang terinstal secara global (umum di macOS via Homebrew) dan `sharp` gagal, paksa penggunaan biner prebuilt:

          ```bash
          SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
          ```

          Jika Anda melihat pesan `sharp: Please add node-gyp to your dependencies`, instal build tooling (macOS: Xcode CLT + `npm install -g node-gyp`) atau gunakan variabel lingkungan di atas.
        </Accordion>
      </Tab>
      <Tab title="pnpm">
        ```bash
        pnpm add -g openclaw@latest
        pnpm approve-builds -g        # setujui openclaw, node-llama-cpp, sharp, dll.
        openclaw onboard --install-daemon
        ```

        <Note>
        pnpm memerlukan persetujuan eksplisit untuk paket dengan skrip build. Setelah instalasi pertama menunjukkan peringatan "Ignored build scripts", jalankan `pnpm approve-builds -g` dan pilih paket yang terdaftar.
        </Note>
      </Tab>
    </Tabs>

  </Accordion>

  <Accordion title="Dari sumber (source)" icon="github">
    Untuk kontributor atau siapa pun yang ingin menjalankan dari checkout lokal.

    <Steps>
      <Step title="Clone dan bangun (build)">
        Clone [repositori OpenClaw](https://github.com/openclaw/openclaw) dan bangun:

        ```bash
        git clone https://github.com/openclaw/openclaw.git
        cd openclaw
        pnpm install
        pnpm ui:build
        pnpm build
        ```
      </Step>
      <Step title="Hubungkan (link) CLI">
        Buat perintah `openclaw` tersedia secara global:

        ```bash
        pnpm link --global
        ```

        Atau, lewati langkah link dan jalankan perintah via `pnpm openclaw ...` dari dalam repositori.
      </Step>
      <Step title="Jalankan onboarding">
        ```bash
        openclaw onboard --install-daemon
        ```
      </Step>
    </Steps>

    Untuk alur kerja pengembangan yang lebih mendalam, lihat [Setup](/id-ID/start/setup).

  </Accordion>
</AccordionGroup>

## Metode instalasi lainnya

<CardGroup cols={2}>
  <Card title="Docker" href="/id-ID/install/docker" icon="container">
    Deployment dalam kontainer atau headless.
  </Card>
  <Card title="Podman" href="/id-ID/install/podman" icon="container">
    Kontainer rootless: jalankan `setup-podman.sh` sekali, lalu skrip peluncuran.
  </Card>
  <Card title="Nix" href="/id-ID/install/nix" icon="snowflake">
    Instalasi deklaratif melalui Nix.
  </Card>
  <Card title="Ansible" href="/id-ID/install/ansible" icon="server">
    Provisioning armada otomatis.
  </Card>
  <Card title="Bun" href="/id-ID/install/bun" icon="zap">
    Penggunaan khusus CLI melalui runtime Bun.
  </Card>
</CardGroup>

## Setelah instalasi

Pastikan semuanya berfungsi:

```bash
openclaw doctor         # periksa masalah konfigurasi
openclaw status         # status gateway
openclaw dashboard      # buka UI browser
```

Jika Anda memerlukan jalur runtime kustom, gunakan:

- `OPENCLAW_HOME` untuk jalur internal berbasis direktori home
- `OPENCLAW_STATE_DIR` untuk lokasi state yang dapat berubah
- `OPENCLAW_CONFIG_PATH` untuk lokasi file konfigurasi

Lihat [Variabel Lingkungan](/id-ID/help/environment) untuk aturan prioritas dan detail lengkap.

## Pemecahan Masalah: `openclaw` tidak ditemukan

<Accordion title="Diagnosis PATH dan perbaikan">
  Diagnosis cepat:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Jika `$(npm prefix -g)/bin` (macOS/Linux) atau `$(npm prefix -g)` (Windows) **tidak** ada dalam `$PATH` Anda, shell Anda tidak dapat menemukan biner npm global (termasuk `openclaw`).

Perbaikan — tambahkan jalur tersebut ke file startup shell Anda (`~/.zshrc` atau `~/.bashrc`):

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Di Windows, tambahkan output dari `npm prefix -g` ke PATH Anda.

Kemudian buka terminal baru (atau jalankan `rehash` di zsh / `hash -r` di bash).
</Accordion>

## Perbarui / Hapus Instalasi

<CardGroup cols={3}>
  <Card title="Memperbarui" href="/id-ID/install/updating" icon="refresh-cw">
    Jaga OpenClaw tetap mutakhir.
  </Card>
  <Card title="Migrasi" href="/id-ID/install/migrating" icon="arrow-right">
    Pindah ke mesin baru.
  </Card>
  <Card title="Hapus Instalasi" href="/id-ID/install/uninstall" icon="trash-2">
    Hapus OpenClaw sepenuhnya.
  </Card>
</CardGroup>


