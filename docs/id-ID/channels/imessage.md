---
summary: "Dukungan iMessage lama melalui imsg (JSON-RPC melalui stdio). Penyiapan baru sebaiknya menggunakan BlueBubbles."
read_when:
  - Mengatur dukungan iMessage versi lama
  - Mendebug pengiriman/penerimaan iMessage via imsg
title: "iMessage (Lama)"
---

# iMessage (Legacy: imsg)

<Warning>
Untuk penyiapan iMessage yang baru, gunakan <a href="/id-ID/channels/bluebubbles">BlueBubbles</a>. Integrasi `imsg` ini adalah versi lama (legacy) dan mungkin akan dihapus di masa mendatang.
</Warning>

Status: Integrasi CLI eksternal versi lama. Gateway menjalankan perintah `imsg rpc` dan berkomunikasi melalui JSON-RPC pada stdio (tidak memerlukan daemon/port terpisah).

## Penyiapan Cepat (Quick Setup)

### Pada Mac Lokal (Jalur Cepat)

1. **Instal imsg**:
   ```bash
   brew install steipete/tap/imsg
   ```
2. **Konfigurasi OpenClaw**:
   Tambahkan detail berikut ke `openclaw.json`:
   ```json5
   {
     channels: {
       imessage: {
         enabled: true,
         cliPath: "/usr/local/bin/imsg",
         dbPath: "/Users/<user-anda>/Library/Messages/chat.db",
       },
     },
   }
   ```
3. **Mulai Gateway**: Jalankan `openclaw gateway`.
4. **Setujui Pairing**: Gunakan `openclaw pairing approve imessage <KODE>` untuk menyetujui pesan masuk pertama.

---

## Persyaratan dan Izin (macOS)

- **Login Pesan**: Aplikasi iMessage/Messages harus sudah login di Mac yang menjalankan `imsg`.
- **Akses Disk Penuh (Full Disk Access)**: Diperlukan agar OpenClaw bisa membaca database iMessage (`chat.db`).
- **Izin Otomatisasi (Automation)**: Diperlukan agar skrip bisa mengirim pesan melalui aplikasi Pesan.

<Tip>
Jika Anda menjalankan gateway secara headless (via SSH), jalankan perintah interaktif sekali saja di terminal GUI untuk memicu munculnya jendela permintaan izin:
`imsg chats --limit 1`
</Tip>

## Kontrol Akses

- **Kebijakan DM**: Defaultnya adalah `pairing`. Pengirim harus memiliki kode pairing sebelum bot merespons.
- **Kebijakan Grup**: Defaultnya adalah `allowlist`. Gunakan `groupAllowFrom` untuk membatasi siapa saja yang bisa memicu bot di dalam grup.
- **Penahan Sebutan (Mention Gating)**: iMessage tidak memiliki metadata mention asli, jadi OpenClaw menggunakan pola regex untuk mendeteksi mention.

## Pola Penyebaran (Deployment Patterns)

- **User Mac Khusus**: Disarankan menggunakan akun Mac dan Apple ID khusus untuk bot agar tidak tercampur dengan pesan pribadi Anda.
- **Mac Jarak Jauh via SSH**: Anda bisa menjalankan gateway di Linux/Cloud dan menghubungkannya ke Mac di rumah melalui SSH wrapper untuk mengeksekusi perintah `imsg`.

## Pemecahan Masalah

- **imsg tidak ditemukan**: Pastikan `cliPath` di konfigurasi sudah benar.
- **Pesan DM diabaikan**: Cek daftar pairing dengan `openclaw pairing list imessage`.
- **Lampiran gagal (Mode Remote)**: Cek konfigurasi `remoteHost` dan pastikan kunci SSH sudah terdaftar di `known_hosts`.

Untuk referensi lengkap, lihat [Konfigurasi iMessage](/id-ID/gateway/configuration-reference#imessage).
