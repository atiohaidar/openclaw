---
summary: "Ekstensi Chrome: izinkan OpenClaw mengontrol tab Chrome Anda yang sudah ada"
read_when:
  - Anda ingin agen mengontrol tab Chrome yang sedang terbuka (tombol toolbar)
  - Anda memerlukan otomatisasi browser lokal melalui Gateway jarak jauh via Tailscale
  - Anda ingin memahami implikasi keamanan dari pengambilalihan browser
title: "Ekstensi Chrome"
---

# Ekstensi Chrome (Relay Browser)

Ekstensi Chrome OpenClaw memungkinkan agen mengontrol **tab Chrome Anda yang sudah ada** (jendela Chrome normal Anda) alih-alih meluncurkan profil Chrome terpisah yang dikelola oleh OpenClaw.

Proses menghubungkan/melepaskan (attach/detach) dilakukan melalui **satu tombol toolbar Chrome**.

## Konsep Dasar

Terdapat tiga bagian utama:
- **Layanan kontrol browser** (Gateway atau node): API yang dipanggil oleh agen/alat (melalui Gateway).
- **Server relay lokal** (loopback CDP): jembatan antara server kontrol dan ekstensi (default: `http://127.0.0.1:18792`).
- **Ekstensi Chrome MV3**: menempel pada tab aktif menggunakan `chrome.debugger` dan meneruskan pesan CDP ke relay.

OpenClaw kemudian mengontrol tab yang terhubung lewat permukaan alat `browser` normal dengan memilih profil yang tepat (biasanya profil bernama `chrome`).

## Instalasi / Pemuatan (Unpacked)

1. Instal ekstensi ke jalur lokal yang stabil:
   ```bash
   openclaw browser extension install
   ```

2. Lihat jalur direktori ekstensi yang terinstal:
   ```bash
   openclaw browser extension path
   ```

3. Buka Chrome → `chrome://extensions`
   - Aktifkan **“Developer mode”** (Mode pengembang).
   - Klik **“Load unpacked”** → pilih direktori yang ditampilkan pada langkah ke-2.

4. Pin (sematkan) ekstensi tersebut di toolbar.

## Cara Menggunakan

OpenClaw menyertakan profil browser bawaan bernama `chrome` yang menargetkan relay ekstensi.

Sebelum pertama kali menghubungkan, buka **Options** ekstensi dan atur:
- **Port** (default `18792`)
- **Gateway token** (harus cocok dengan `gateway.auth.token`)

Gunakan:
- CLI: `openclaw browser --browser-profile chrome tabs`
- Alat Agen: `browser` dengan `profile="chrome"`

## Menghubungkan / Melepaskan (Tombol Toolbar)

- Buka tab yang ingin dikontrol oleh OpenClaw.
- Klik ikon ekstensi.
  - Lencana (badge) akan menunjukkan **`ON`** saat terhubung.
- Klik lagi untuk melepaskan.

Ekstensi ini **hanya** mengontrol tab yang secara eksplisit Anda hubungkan dengan mengklik tombol toolbar.

## Lencana (Badge) & Error Umum

- **`ON`**: Terhubung; OpenClaw dapat mengontrol tab tersebut.
- **`…`**: Sedang menghubungkan ke relay lokal.
- **`!`**: Relay tidak dapat dijangkau atau masalah autentikasi (server relay tidak jalan atau token salah).

## Gateway Jarak Jauh (Remote)

Jika Gateway Anda berjalan di mesin lain, jalankan **node host** di mesin yang menjalankan Chrome. Gateway akan meneruskan (proxy) tindakan browser ke node tersebut; ekstensi dan relay tetap berada di mesin lokal browser.

## Implikasi Keamanan

Fitur ini sangat kuat namun berisiko. Anggap ini seperti memberikan "tangan" kepada model pada browser Anda.

- Ekstensi menggunakan API debugger Chrome. Saat terhubung, model dapat:
  - Klik/ketik/navigasi di tab tersebut.
  - Membaca konten halaman.
  - Mengakses apa pun yang dapat diakses oleh sesi login tab tersebut.
- **Ini tidak terisolasi** seperti profil browser khusus `openclaw`. Jangan hubungkan ke tab yang berisi data sangat sensitif jika Anda tidak sepenuhnya mempercayai model atau agen tersebut.

Rekomendasi:
- Gunakan profil Chrome khusus (terpisah dari aktivitas browsing pribadi Anda) untuk penggunaan relay ekstensi.
- Gunakan jaringan pribadi seperti **Tailscale** untuk menghubungkan Gateway dan node.


