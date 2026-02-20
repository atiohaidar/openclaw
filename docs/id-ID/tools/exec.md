---
summary: "Penggunaan alat exec, mode stdin, dan dukungan TTY"
read_when:
  - Menggunakan atau memodifikasi alat exec
  - Menelusuri masalah perilaku stdin atau TTY
title: "Alat Exec"
---

# Alat Exec

Jalankan perintah shell di workspace. Mendukung eksekusi di latar depan (foreground) + latar belakang (background) melalui alat `process`.

Jika `process` tidak diizinkan, `exec` akan berjalan secara sinkron dan mengabaikan parameter `yieldMs`/`background`. Sesi latar belakang dibatasi per agen; alat `process` hanya dapat melihat sesi dari agen yang sama.

## Parameter

- `command` (wajib): perintah yang akan dijalankan.
- `workdir` (default ke direktori saat ini): direktori kerja.
- `env` (override kunci/nilai): variabel lingkungan.
- `yieldMs` (default 10000): otomatis pindah ke latar belakang setelah jeda ini.
- `background` (boolean): segera jalankan di latar belakang.
- `timeout` (detik, default 1800): matikan proses jika waktu habis.
- `pty` (boolean): jalankan di pseudo-terminal jika tersedia (berguna untuk CLI yang membutuhkan TTY).
- `host` (`sandbox | gateway | node`): tempat eksekusi dilakukan.
- `security` (`deny | allowlist | full`): mode penegakan keamanan untuk `gateway`/`node`.
- `ask` (`off | on-miss | always`): permintaan persetujuan untuk `gateway`/`node`.
- `node` (string): ID/nama node jika `host=node`.
- `elevated` (boolean): minta mode elevasi (host gateway).

Catatan:

- `host` secara default adalah `sandbox`.
- `elevated` diabaikan jika sandboxing dinonaktifkan (karena exec sudah berjalan di host).
- Persetujuan `gateway`/`node` dikontrol oleh file `~/.openclaw/exec-approvals.json`.
- Penting: sandboxing **nonaktif secara default**. Jika sandboxing nonaktif, `host=sandbox` berjalan langsung di host gateway dan **tidak memerlukan persetujuan**. Untuk mewajibkan persetujuan, jalankan dengan `host=gateway` dan konfigurasi persetujuan exec (atau aktifkan sandboxing).

## Konfigurasi

- `tools.exec.notifyOnExit` (default: true): jika true, sesi exec latar belakang yang selesai akan memicu event sistem.
- `tools.exec.approvalRunningNoticeMs` (default: 10000): kirim pemberitahuan “sedang berjalan” jika exec yang memerlukan persetujuan berjalan lebih lama dari ini.
- `tools.exec.pathPrepend`: daftar direktori untuk ditambahkan ke awal `PATH`.
- `tools.exec.safeBins`: binari aman khusus stdin yang dapat berjalan tanpa entri daftar izinkan eksplisit. Lihat [Bin aman](/id-ID/tools/exec-approvals#safe-bins-stdin-only).

## Penanganan PATH

- `host=gateway`: menggabungkan `PATH` dari login shell Anda ke dalam lingkungan exec. Override `env.PATH` ditolak untuk eksekusi host.
- `host=sandbox`: menjalankan `sh -lc` (login shell) di dalam kontainer. OpenClaw menambahkan `env.PATH` setelah pemuatan profil.
- `host=node`: hanya override env yang tidak diblokir yang dikirim ke node.

## Override Sesi (`/exec`)

Gunakan **/exec** untuk menetapkan default **per sesi** untuk `host`, `security`, `ask`, dan `node`. Kirim `/exec` tanpa argumen untuk melihat nilai saat ini.

Contoh:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Model Otorisasi

Instruksi `/exec` hanya dipatuhi untuk **pengirim terotorisasi**. Ini memperbarui **status sesi saja** dan tidak menulis ke file konfigurasi. Persetujuan host tetap berlaku kecuali Anda menyetel `security=full` dan `ask=off`.

## Persetujuan Exec (Companion App / Host Node)

Agen yang di-sandbox dapat mewajibkan persetujuan per permintaan sebelum `exec` dijalankan di gateway atau host node. Lihat [Persetujuan Exec](/id-ID/tools/exec-approvals) untuk kebijakan, daftar izinkan, dan alur UI.

## Daftar Izinkan + Bin Aman

Penegakan daftar izinkan (allowlist) mencocokkan **jalur binari yang teresolusi (resolved binary paths)** saja. Dalam mode `security=allowlist`, perintah shell diizinkan otomatis hanya jika setiap segmen pipa (pipeline segment) ada dalam daftar izinkan atau merupakan bin aman. Rantai perintah (`;`, `&&`, `||`) dan pengalihan (redirection) ditolak kecuali setiap segmen memenuhi syarat.

## Contoh

Latar depan (Foreground):

```json
{ "tool": "exec", "command": "ls -la" }
```

Latar belakang (Background) + polling:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Kirim tombol (gaya tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
```

## apply_patch (Eksperimental)

`apply_patch` adalah sub-alat dari `exec` untuk pengeditan multi-file terstruktur. Aktifkan secara eksplisit di konfigurasi.

Catatan:

- Hanya tersedia untuk model OpenAI/OpenAI Codex.
- Kebijakan alat tetap berlaku; `allow: ["exec"]` secara implisit mengizinkan `apply_patch`.


