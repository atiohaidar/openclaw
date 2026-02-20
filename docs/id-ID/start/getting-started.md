---
summary: "Instal OpenClaw dan jalankan chat pertama Anda dalam hitungan menit."
read_when:
  - Pengaturan pertama kali dari nol
  - Anda ingin jalur tercepat menuju chat yang berfungsi
title: "Panduan Memulai"
---

# Panduan Memulai

Tujuan: beralih dari nol ke chat pertama yang berfungsi dengan pengaturan minimal.

<Info>
Chat tercepat: buka Control UI (tidak perlu pengaturan saluran). Jalankan `openclaw dashboard`
dan chat di browser, atau buka `http://127.0.0.1:18789/` pada
<Tooltip headline="Host Gateway" tip="Mesin yang menjalankan layanan gateway OpenClaw.">host gateway</Tooltip>.
Dokumentasi: [Dashboard](/web/dashboard) dan [Control UI](/web/control-ui).
</Info>

## Prasyarat

- Node 22 atau yang lebih baru

<Tip>
Periksa versi Node Anda dengan `node --version` jika Anda tidak yakin.
</Tip>

## Pengaturan Cepat (CLI)

<Steps>
  <Step title="Instal OpenClaw (direkomendasikan)">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
        <img
  src="/assets/install-script.svg"
  alt="Proses Skrip Instalasi"
  className="rounded-lg"
/>
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    <Note>
    Metode instalasi lain dan persyaratannya: [Instal](/id-ID/install/index).
    </Note>

  </Step>
  <Step title="Jalankan wizard onboarding">
    ```bash
    openclaw onboard --install-daemon
    ```

    Wizard mengonfigurasi autentikasi, pengaturan gateway, dan saluran opsional.
    Lihat [Wizard Onboarding](/id-ID/start/wizard) untuk detailnya.

  </Step>
  <Step title="Periksa Gateway">
    Jika Anda menginstal layanan tersebut, seharusnya layanan itu sudah berjalan:

    ```bash
    openclaw gateway status
    ```

  </Step>
  <Step title="Buka Control UI">
    ```bash
    openclaw dashboard
    ```
  </Step>
</Steps>

<Check>
Jika Control UI terbuka, Gateway Anda siap digunakan.
</Check>

## Pemeriksaan opsional dan tambahan

<AccordionGroup>
  <Accordion title="Jalankan Gateway di latar depan (foreground)">
    Berguna untuk pengujian cepat atau pemecahan masalah.

    ```bash
    openclaw gateway --port 18789
    ```

  </Accordion>
  <Accordion title="Kirim pesan uji coba">
    Memerlukan saluran yang sudah terkonfigurasi.

    ```bash
    openclaw message send --target +15555550123 --message "Halo dari OpenClaw"
    ```

  </Accordion>
</AccordionGroup>

## Variabel lingkungan yang berguna

Jika Anda menjalankan OpenClaw sebagai akun layanan atau menginginkan lokasi konfigurasi/status khusus:

- `OPENCLAW_HOME` menetapkan direktori beranda yang digunakan untuk resolusi jalur internal.
- `OPENCLAW_STATE_DIR` mengganti direktori state.
- `OPENCLAW_CONFIG_PATH` mengganti jalur file konfigurasi.

Referensi variabel lingkungan lengkap: [Variabel Lingkungan](/help/environment).

## Pelajari lebih dalam

<Columns>
  <Card title="Wizard Onboarding (detail)" href="/id-ID/start/wizard">
    Referensi lengkap CLI wizard dan opsi tingkat lanjut.
  </Card>
  <Card title="Onboarding aplikasi macOS" href="/id-ID/start/onboarding">
    Alur penggunaan pertama kali untuk aplikasi macOS.
  </Card>
</Columns>

## Apa yang akan Anda dapatkan

- Gateway yang berjalan
- Autentikasi terkonfigurasi
- Akses Control UI atau saluran yang terhubung

## Langkah berikutnya

- Keamanan DM dan Pairing: [Pairing](/channels/pairing)
- Hubungkan lebih banyak saluran: [Saluran](/channels/index)
- Alur kerja tingkat lanjut dan dari sumber (source): [Setup](/start/setup)


