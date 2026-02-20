---
summary: "Layanan kontrol browser terintegrasi + perintah tindakan"
read_when:
  - Menambahkan otomasi browser yang dikontrol agen
  - Menelusuri masalah mengapa OpenClaw mengganggu Chrome pribadi Anda
  - Menerapkan pengaturan browser + siklus hidup di aplikasi macOS
title: "Browser (Dikelola OpenClaw)"
---

# Browser (Dikelola OpenClaw)

OpenClaw dapat menjalankan **profil Chrome/Brave/Edge/Chromium khusus** yang dikontrol oleh agen.
Profil ini terisolasi dari browser pribadi Anda dan dikelola melalui layanan kontrol lokal kecil di dalam Gateway (hanya loopback).

Panduan untuk pemula:

- Anggap ini sebagai **browser terpisah khusus untuk agen**.
- Profil `openclaw` **tidak** akan menyentuh profil browser pribadi Anda.
- Agen dapat **membuka tab, membaca halaman, mengklik, dan mengetik** di jalur yang aman.
- Profil default `chrome` menggunakan **browser Chromium default sistem** melalui relay ekstensi; beralihlah ke `openclaw` untuk browser terisolasi yang dikelola sepenuhnya.

## Apa yang Anda dapatkan

- Profil browser terpisah bernama **openclaw** (dengan aksen oranye secara default).
- Kontrol tab yang deterministik (daftar/buka/fokus/tutup).
- Tindakan agen (klik/ketik/seret/pilih), snapshot, screenshot, PDF.
- Dukungan multi-profil opsional (`openclaw`, `work`, `remote`, ...).

Browser ini **bukan** untuk penggunaan harian Anda. Ini adalah permukaan yang aman dan terisolasi untuk otomasi dan verifikasi agen.

## Memulai dengan cepat

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Jika Anda mendapatkan pesan “Browser disabled”, aktifkan di konfigurasi (lihat di bawah) dan mulai ulang Gateway.

## Profil: `openclaw` vs `chrome`

- `openclaw`: browser terisolasi yang dikelola (tidak memerlukan ekstensi).
- `chrome`: relay ekstensi ke **browser sistem** Anda (memerlukan ekstensi OpenClaw yang terpasang pada sebuah tab).

Atur `browser.defaultProfile: "openclaw"` jika Anda ingin mode terkelola sebagai default.

## Konfigurasi

Pengaturan browser berada di `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true, // default: true
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Catatan:

- Profil `chrome` adalah relay ekstensi (default `defaultProfile`).
- Urutan deteksi otomatis: browser default sistem jika berbasis Chromium; jika tidak, akan mencari Chrome → Brave → Edge → Chromium.
- Profil `openclaw` lokal akan mengalokasikan `cdpPort` secara otomatis.

## Gunakan Brave (atau browser berbasis Chromium lainnya)

Jika browser **default sistem** Anda berbasis Chromium (Chrome/Brave/Edge/dll), OpenClaw akan menggunakannya secara otomatis. Atur `browser.executablePath` untuk menimpa deteksi otomatis.

## Kontrol Lokal vs Jarak Jauh (Remote)

- **Kontrol Lokal (default):** Gateway memulai layanan kontrol loopback dan dapat meluncurkan browser lokal.
- **Remote CDP:** atur `cdpUrl` untuk menghubungkan ke browser berbasis Chromium yang berjalan di tempat lain. Dalam hal ini, OpenClaw tidak akan meluncurkan browser lokal.

## Relay Ekstensi Chrome (Gunakan Chrome Anda yang sudah ada)

OpenClaw juga dapat mengontrol **tab Chrome Anda yang sudah ada** (tanpa instansi Chrome "openclaw" terpisah) melalui relay CDP lokal + ekstensi Chrome.

Panduan lengkap: [Ekstensi Chrome](/id-ID/tools/chrome-extension)

Alur:

- Anda mengklik ikon ekstensi **OpenClaw Browser Relay** pada tab untuk menghubungkannya (tidak terhubung otomatis).
- Agen mengontrol tab tersebut melalui alat `browser` normal dengan memilih profil yang tepat (biasanya profil `chrome`).

## Keamanan

- Kontrol browser hanya melalui loopback; akses mengalir melalui autentikasi Gateway atau pairing node.
- Jaga agar Gateway dan host node tetap berada di jaringan pribadi (Tailscale); hindari paparan publik.
- Perlakukan URL/token remote CDP sebagai data rahasia.

## Profil (Multi-browser)

OpenClaw mendukung beberapa profil bernama. Profil dapat berupa:

- **dikelola-openclaw (openclaw-managed)**: instansi browser khusus.
- **jarak jauh (remote)**: URL CDP eksplisit.
- **relay ekstensi**: tab Chrome Anda yang sudah ada melalui ekstensi.

## API Kontrol (Opsional)

Gateway mengekspos API HTTP loopback kecil untuk integrasi lokal:

- Status/start/stop: `GET /`, `POST /start`, `POST /stop`
- Tab: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
- Snapshot/screenshot: `GET /snapshot`, `POST /screenshot`

Semua endpoint menerima parameter `?profile=<nama>`.

### Persyaratan Playwright

Beberapa fitur (navigasi/tindakan/snapshot AI, screenshot elemen, PDF) memerlukan **Playwright**. Jika Playwright tidak terinstal, fitur tersebut akan mengembalikan error 501. Snapshot ARIA dan screenshot dasar tetap berfungsi untuk Chrome yang dikelola openclaw.

## Referensi Cepat CLI

Semua perintah menerima `--browser-profile <nama>` dan `--json`.

Dasar:

- `openclaw browser status`
- `openclaw browser start`
- `openclaw browser open https://example.com`

Inspeksi:

- `openclaw browser screenshot`
- `openclaw browser snapshot`
- `openclaw browser console`

Tindakan:

- `openclaw browser click 12`
- `openclaw browser type 23 "halo" --submit`
- `openclaw browser press Enter`

Catatan:

- `upload` dan `dialog` adalah panggilan **persiapan (arming)**; jalankan sebelum klik/tekan yang memicu dialog/pemilihan file.
- `snapshot` mendukung format `ai` (default) dan `aria`.
- `act` (klik/ketik/dll) memerlukan referensi (`ref`) dari `snapshot`.

## Snapshot dan Referensi (Refs)

OpenClaw mendukung dua gaya snapshot:

- **Snapshot AI (referensi numerik)**: `openclaw browser snapshot`
  - Output: snapshot teks dengan referensi numerik (misal: `12`).
  - Tindakan: `openclaw browser click 12`.
- **Snapshot Peran (referensi peran seperti `e12`)**: `openclaw browser snapshot --interactive`
  - Output: daftar elemen interaktif dengan referensi peran (misal: `ref=e12`).
  - Tindakan: `openclaw browser click e12`.

## Fitur Tunggu (Wait)

Anda dapat menunggu lebih dari sekadar waktu:

- Tunggu URL: `openclaw browser wait --url "**/dash"`
- Tunggu status muat: `openclaw browser wait --load networkidle`
- Tunggu predikat JS: `openclaw browser wait --fn "window.ready===true"`

## Alur Kerja Debugging

Jika tindakan gagal:

1. `openclaw browser snapshot --interactive`
2. Gunakan `click <ref>`
3. Jika masih gagal, gunakan `openclaw browser highlight <ref>` untuk melihat apa yang ditargetkan Playwright.

## Keamanan & Privasi

- Profil browser openclaw mungkin berisi sesi login; perlakukan sebagai data sensitif.
- `browser.evaluateEnabled=false` dapat digunakan untuk menonaktifkan eksekusi JavaScript sembarang dari prompt.
- Untuk info login dan catatan anti-bot, lihat [Login Browser](/id-ID/tools/browser-login).

## Alat Agen + Cara Kerja Kontrol

Agen mendapatkan **satu alat** untuk otomasi browser:

- `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

Pencocokan:

- `browser snapshot` mengembalikan pohon UI yang stabil.
- `browser act` menggunakan ID referensi snapshot untuk melakukan tindakan.
- `browser` menerima parameter `profile` dan `target` (`sandbox` atau `host`).


