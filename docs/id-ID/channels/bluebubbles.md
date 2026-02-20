---
summary: "iMessage melalui server macOS BlueBubbles (pengiriman/penerimaan REST, pengetikan, reaksi, pairing, aksi lanjutan)."
read_when:
  - Menyiapkan saluran BlueBubbles
  - Memecahkan masalah pairing webhook
  - Mengonfigurasi iMessage di macOS
title: "BlueBubbles"
---

# BlueBubbles (macOS REST)

Status: Plugin bawaan yang berkomunikasi dengan server macOS BlueBubbles melalui HTTP. **Sangat direkomendasikan untuk integrasi iMessage** karena API yang lebih kaya dan penyiapan yang lebih mudah dibandingkan saluran iMessage tradisional.

## Ringkasan

- Berjalan di macOS melalui aplikasi pembantu BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
- OpenClaw berkomunikasi melalui REST API (`GET /api/v1/ping`, `POST /message/text`, dsb).
- Pesan masuk tiba melalui webhook; balasan keluar, indikator mengetik, dan laporan dibaca dilakukan melalui panggilan REST.
- Lampiran dan stiker diterima sebagai media masuk.
- Pairing dan Daftar Izinkan (Allowlist) bekerja sama seperti saluran lainnya.
- Fitur lanjutan: edit pesan, batal kirim (unsend), utasan balasan, efek pesan, dan manajemen grup.

## Penyiapan Cepat (Quick Start)

1. Instal server BlueBubbles di Mac Anda (ikuti instruksi di [bluebubbles.app/install](https://bluebubbles.app/install)).
2. Di konfigurasi BlueBubbles, aktifkan **Web API** dan setel kata sandi.
3. Jalankan `openclaw onboard` dan pilih BlueBubbles, atau konfigurasi manual di `openclaw.json`:

   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "password-anda",
         webhookPath: "/bluebubbles-webhook",
       },
     },
   }
   ```

4. Arahkan webhook BlueBubbles ke gateway Anda (contoh: `https://host-gateway-anda:3000/bluebubbles-webhook?password=<password>`).
5. Mulai gateway; ia akan mendaftarkan handler webhook dan memulai proses pairing.

## Menjaga Messages.app Tetap Aktif

Pada beberapa setup macOS (terutama VM atau server headless), aplikasi Pesan (Messages.app) terkadang masuk ke mode "tidur" sehingga pesan tidak masuk. Solusinya adalah menjalankan AppleScript setiap 5 menit untuk "membangunkan" aplikasi.

### 1) Simpan AppleScript
Simpan kode ini sebagai `~/Scripts/poke-messages.scpt`:
```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if
    set _chatCount to (count of chats)
  end tell
on error
end try
```

### 2) Instal LaunchAgent
Simpan sebagai `~/Library/LaunchAgents/com.user.poke-messages.plist`:
```xml
<!-- XML konfigurasi LaunchAgent untuk menjalankan script di atas setiap 300 detik -->
```
*(Catatan: Gunakan `launchctl load` untuk mengaktifkannya)*.

## Kontrol Akses (DM + Grup)

### Pesan Langsung (DM)
- Default: `dmPolicy = "pairing"`.
- Pengirim yang tidak dikenal akan menerima kode pairing; pesan mereka diabaikan sampai disetujui.
- Setujui melalui: `openclaw pairing approve bluebubbles <KODE>`.

### Grup
- Mendukung fitur penahan sebutan (**Mention Gating**). Agen hanya akan membalas jika di-mention (@mention).
- Anda dapat mengecualikan grup tertentu agar agen selalu membalas melalui properti `groups` di konfigurasi.

## Aksi Lanjutan (Advanced Actions)

BlueBubbles mendukung berbagai aksi pesan jika diaktifkan di konfigurasi:
- **Reaksi (Tapbacks)**: Menambahkan atau menghapus reaksi jempol, hati, dll.
- **Edit**: Mengubah pesan yang sudah terkirim (hanya di macOS 13+).
- **Unsend**: Menarik kembali pesan (hanya di macOS 13+).
- **Balas (Reply)**: Membalas pesan spesifik dalam sebuah utasan.
- **Kirim Lampiran**: Mengirim gambar, file, atau pesan suara (voice memo).

## Pemecahan Masalah

- **Pesan tidak masuk**: Cek log webhook BlueBubbles dan pastikan URL gateway sudah benar.
- **Kode pairing kadaluwarsa**: Kode hanya berlaku selama satu jam. Gunakan `openclaw pairing list bluebubbles` untuk melihat daftar terbaru.
- **Reaksi gagal**: Pastikan **Private API** di server BlueBubbles sudah diaktifkan.
- **Edit tidak bekerja**: Fitur ini diketahui bermasalah di macOS versi sangat baru (Tahoe/macOS 26) karena perubahan API internal Apple.

Untuk bantuan lebih lanjut, jalankan `openclaw status --all` atau hubungi komunitas OpenClaw.


