---
summary: "Alur onboarding penggunaan pertama kali untuk OpenClaw (Aplikasi macOS)"
read_when:
  - Mendesain asisten onboarding macOS
  - Mengimplementasikan pengaturan autentikasi atau identitas
title: "Onboarding (Aplikasi macOS)"
sidebarTitle: "Onboarding: Aplikasi macOS"
---

# Onboarding (Aplikasi macOS)

Dokumen ini mendeskripsikan alur onboarding penggunaan pertama kali yang **saat ini** berlaku. Tujuannya adalah memberikan pengalaman "hari ke-0" yang lancar: memilih tempat Gateway dijalankan, menghubungkan autentikasi, menjalankan wizard, dan membiarkan agen melakukan bootstrap pada dirinya sendiri.
Untuk ringkasan umum mengenai jalur onboarding lainnya, lihat [Ringkasan Onboarding](/id-ID/start/onboarding-overview).

<Steps>
<Step title="Setujui peringatan macOS">
<Frame>
<img src="/assets/macos-onboarding/01-macos-warning.jpeg" alt="" />
</Frame>
</Step>
<Step title="Setujui izin untuk menemukan jaringan lokal">
<Frame>
<img src="/assets/macos-onboarding/02-local-networks.jpeg" alt="" />
</Frame>
</Step>
<Step title="Selamat datang dan pemberitahuan keamanan">
<Frame caption="Baca pemberitahuan keamanan yang ditampilkan dan tentukan pilihan Anda">
<img src="/assets/macos-onboarding/03-security-notice.png" alt="" />
</Frame>
</Step>
<Step title="Lokal vs Jarak Jauh (Remote)">
<Frame>
<img src="/assets/macos-onboarding/04-choose-gateway.png" alt="" />
</Frame>

Di mana **Gateway** dijalankan?

- **Mac ini (Lokal saja):** onboarding dapat menjalankan alur OAuth dan menulis kredensial
  secara lokal.
- **Jarak Jauh/Remote (melalui SSH/Tailnet):** onboarding **tidak** menjalankan OAuth secara lokal;
  kredensial harus sudah ada di host gateway.
- **Konfigurasi nanti:** lewati pengaturan dan biarkan aplikasi tidak terkonfigurasi.

<Tip>
**Tip autentikasi Gateway:**
- Wizard kini membuat **token** bahkan untuk loopback, sehingga klien WS lokal harus melakukan autentikasi.
- Jika Anda menonaktifkan autentikasi, proses lokal apa pun dapat terhubung; gunakan ini hanya pada mesin yang sepenuhnya Anda percayai.
- Gunakan **token** untuk akses multi-mesin atau bind non-loopback.
</Tip>
</Step>
<Step title="Izin (Permissions)">
<Frame caption="Pilih izin apa saja yang ingin Anda berikan kepada OpenClaw">
<img src="/assets/macos-onboarding/05-permissions.png" alt="" />
</Frame>

Onboarding meminta izin TCC yang diperlukan untuk:

- Otomatisasi (AppleScript)
- Notifikasi
- Aksesibilitas
- Perekaman Layar
- Mikrofon
- Pengenalan Ucapan
- Kamera
- Lokasi

</Step>
<Step title="CLI">
  <Info>Langkah ini bersifat opsional</Info>
  Aplikasi dapat menginstal CLI `openclaw` secara global melalui npm/pnpm sehingga
  alur kerja terminal dan tugas launchd dapat langsung berfungsi.
</Step>
<Step title="Chat Onboarding (sesi khusus)">
  Setelah pengaturan, aplikasi akan membuka sesi chat onboarding khusus sehingga agen dapat
  memperkenalkan dirinya dan memandu langkah-langkah selanjutnya. Hal ini menjaga agar panduan penggunaan pertama kali terpisah
  dari percakapan normal Anda. Lihat [Bootstrapping](/id-ID/start/bootstrapping) untuk
  mengetahui apa yang terjadi pada host gateway selama pemuatan agen pertama kali.
</Step>
</Steps>
