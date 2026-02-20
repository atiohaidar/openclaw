---
title: "Node.js"
summary: "Instal dan konfigurasi Node.js untuk OpenClaw — persyaratan versi, opsi instalasi, dan pemecahan masalah PATH"
read_when:
  - "Anda perlu menginstal Node.js sebelum menginstal OpenClaw"
  - "Anda telah menginstal OpenClaw tetapi muncul error `openclaw: command not found`"
  - "Instalasi via `npm install -g` gagal karena masalah izin atau PATH"
---

# Node.js

OpenClaw membutuhkan **Node 22 atau yang lebih baru**. [Skrip instalasi](/id-ID/install#metode-instalasi) akan mendeteksi dan menginstal Node secara otomatis — halaman ini ditujukan bagi Anda yang ingin menyiapkan Node sendiri dan memastikan semuanya terhubung dengan benar (versi, PATH, instalasi global).

## Periksa versi Anda

```bash
node -v
```

Jika perintah ini mencetak `v22.x.x` atau lebih tinggi, Anda sudah siap. Jika Node belum terinstal atau versinya terlalu lama, pilih salah satu metode instalasi di bawah ini.

## Instal Node

<Tabs>
  <Tab title="macOS">
    **Homebrew** (direkomendasikan):

    ```bash
    brew install node
    ```

    Atau unduh installer macOS dari [nodejs.org](https://nodejs.org/).

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian:**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL:**

    ```bash
    sudo dnf install nodejs
    ```

    Atau gunakan pengelola versi (lihat di bawah).

  </Tab>
  <Tab title="Windows">
    **winget** (direkomendasikan):

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey:**

    ```powershell
    choco install nodejs-lts
    ```

    Atau unduh installer Windows dari [nodejs.org](https://nodejs.org/).

  </Tab>
</Tabs>

<Accordion title="Menggunakan pengelola versi (nvm, fnm, mise, asdf)">
  Pengelola versi memungkinkan Anda berpindah antar versi Node dengan mudah. Opsi populer:

- [**fnm**](https://github.com/Schniz/fnm) — cepat, lintas platform
- [**nvm**](https://github.com/nvm-sh/nvm) — digunakan secara luas di macOS/Linux
- [**mise**](https://mise.jdx.dev/) — poliglot (Node, Python, Ruby, dll.)

Contoh menggunakan fnm:

```bash
fnm install 22
fnm use 22
```

  <Warning>
  Pastikan pengelola versi Anda telah diinisialisasi di file startup shell Anda (`~/.zshrc` atau `~/.bashrc`). Jika tidak, perintah `openclaw` mungkin tidak ditemukan di sesi terminal baru karena PATH tidak menyertakan direktori biner Node.
  </Warning>
</Accordion>

## Pemecahan Masalah

### `openclaw: command not found`

Ini hampir selalu berarti direktori biner (bin) global npm tidak ada di PATH Anda.

<Steps>
  <Step title="Cari prefix npm global Anda">
    ```bash
    npm prefix -g
    ```
  </Step>
  <Step title="Periksa apakah jalur tersebut ada di PATH">
    ```bash
    echo "$PATH"
    ```

    Cari `<npm-prefix>/bin` (macOS/Linux) atau `<npm-prefix>` (Windows) dalam output tersebut.

  </Step>
  <Step title="Tambahkan ke file startup shell Anda">
    <Tabs>
      <Tab title="macOS / Linux">
        Tambahkan ke `~/.zshrc` atau `~/.bashrc`:

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        Kemudian buka terminal baru (atau jalankan `rehash` di zsh / `hash -r` di bash).
      </Tab>
      <Tab title="Windows">
        Tambahkan output dari `npm prefix -g` ke PATH sistem Anda melalui Pengaturan → Sistem → Variabel Lingkungan.
      </Tab>
    </Tabs>

  </Step>
</Steps>

### Error izin (permissions) pada `npm install -g` (Linux)

Jika Anda melihat error `EACCES`, ubah prefix global npm ke direktori yang dapat ditulis oleh pengguna:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Tambahkan baris `export PATH=...` ke `~/.bashrc` atau `~/.zshrc` Anda agar perubahan bersifat permanen.


