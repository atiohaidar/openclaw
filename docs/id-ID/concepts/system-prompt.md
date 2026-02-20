---
summary: "Apa yang terkandung dalam prompt sistem OpenClaw dan bagaimana ia dirakit"
read_when:
  - Mengedit teks prompt sistem, daftar alat, atau bagian waktu/heartbeat
  - Mengubah bootstrap workspace atau perilaku injeksi skill
title: "Prompt Sistem"
---

# Prompt Sistem

OpenClaw membangun prompt sistem kustom untuk setiap proses agen. Prompt ini **dimiliki oleh OpenClaw** dan tidak menggunakan prompt default pi-coding-agent.

Prompt dirakit oleh OpenClaw dan disuntikkan ke setiap proses agen.

## Struktur

Prompt ini sengaja dibuat ringkas dan menggunakan bagian-bagian tetap:

- **Tooling**: daftar alat saat ini + deskripsi singkat.
- **Keselamatan (Safety)**: pengingat pagar pembatas (guardrail) singkat untuk menghindari perilaku mencari kekuasaan atau menghindari pengawasan.
- **Skill** (bila tersedia): memberi tahu model cara memuat instruksi skill sesuai permintaan.
- **Pembaruan Diri OpenClaw**: cara menjalankan `config.apply` dan `update.run`.
- **Workspace**: direktori kerja (`agents.defaults.workspace`).
- **Dokumentasi**: lokasi lokal ke dokumen OpenClaw (repo atau paket npm) dan kapan harus membacanya.
- **File Workspace (disuntikkan)**: menunjukkan file bootstrap disertakan di bawah.
- **Sandbox** (bila diaktifkan): menunjukkan runtime sandbox, jalur sandbox, dan apakah eksekusi yang ditingkatkan tersedia.
- **Tanggal & Waktu Saat Ini**: waktu lokal pengguna, zona waktu, dan format waktu.
- **Tag Balasan**: sintaks tag balasan opsional untuk penyedia yang didukung.
- **Heartbeats**: prompt heartbeat dan perilaku ack.
- **Runtime**: host, OS, node, model, root repo (bila terdeteksi), tingkat thinking (satu baris).
- **Penalaran (Reasoning)**: tingkat visibilitas saat ini + petunjuk toggle /reasoning.

Pagar pembatas keselamatan dalam prompt sistem bersifat nasihat. Mereka memandu perilaku model tetapi tidak menegakkan kebijakan. Gunakan kebijakan alat, persetujuan eksekusi, sandboxing, dan daftar yang diizinkan saluran untuk penegakan keras; operator dapat menonaktifkan ini sesuai desain.

## Mode prompt

OpenClaw dapat merender prompt sistem yang lebih kecil untuk sub-agen. Runtime mengatur
`promptMode` untuk setiap proses (bukan konfigurasi yang menghadap pengguna):

- `full` (default): mencakup semua bagian di atas.
- `minimal`: digunakan untuk sub-agen; menghilangkan **Skill**, **Recall Memori**, **Pembaruan Diri
  OpenClaw**, **Alias Model**, **Identitas Pengguna**, **Tag Balasan**,
  **Perpesanan**, **Balasan Senyap**, dan **Heartbeats**. Tooling, **Keselamatan**,
  Workspace, Sandbox, Tanggal & Waktu Saat Ini (bila diketahui), Runtime, dan konteks
  yang disuntikkan tetap tersedia.
- `none`: hanya mengembalikan baris identitas dasar.

Saat `promptMode=minimal`, prompt tambahan yang disuntikkan diberi label **Konteks
Subagen** alih-alih **Konteks Chat Grup**.

## Injeksi bootstrap workspace

File bootstrap dipangkas dan ditambahkan di bawah **Konteks Proyek** sehingga model melihat identitas dan konteks profil tanpa perlu membaca eksplisit:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (hanya pada workspace baru)
- `MEMORY.md` dan/atau `memory.md` (bila ada di workspace; salah satu atau keduanya dapat disuntikkan)

Semua file ini **disuntikkan ke dalam jendela konteks** pada setiap giliran, yang
berarti mereka mengonsumsi token. Buatlah ringkas — terutama `MEMORY.md`, yang dapat
tumbuh seiring waktu dan menyebabkan penggunaan konteks yang tinggi secara tak terduga dan pemadatan
yang lebih sering.

> **Catatan:** File harian `memory/*.md` **tidak** disuntikkan secara otomatis. Mereka
> diakses sesuai permintaan melalui alat `memory_search` dan `memory_get`, jadi mereka
> tidak menghitung terhadap jendela konteks kecuali model secara eksplisit membacanya.

File besar dipotong dengan penanda. Ukuran maksimum per-file dikontrol oleh
`agents.defaults.bootstrapMaxChars` (default: 20000). Total konten bootstrap
yang disuntikkan di seluruh file dibatasi oleh `agents.defaults.bootstrapTotalMaxChars`
(default: 150000). File yang hilang menyuntikkan penanda file-hilang singkat.

Sesi sub-agen hanya menyuntikkan `AGENTS.md` dan `TOOLS.md` (file bootstrap lainnya
difilter agar konteks sub-agen tetap kecil).

Hook internal dapat menginterupsi langkah ini melalui `agent:bootstrap` untuk memutasi atau mengganti
file bootstrap yang disuntikkan (misalnya menukar `SOUL.md` untuk persona alternatif).

Untuk memeriksa seberapa besar kontribusi setiap file yang disuntikkan (mentah vs disuntikkan, pemotongan, ditambah overhead skema alat), gunakan `/context list` atau `/context detail`. Lihat [Konteks](/id-ID/concepts/context).

## Penanganan waktu

Prompt sistem menyertakan bagian **Tanggal & Waktu Saat Ini** khusus ketika
zona waktu pengguna diketahui. Untuk menjaga cache prompt tetap stabil, sekarang hanya menyertakan
**zona waktu** (tidak ada jam dinamis atau format waktu).

Gunakan `session_status` saat agen membutuhkan waktu saat ini; kartu status
menyertakan baris stempel waktu.

Konfigurasikan dengan:

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Lihat [Tanggal & Waktu](/id-ID/date-time) untuk detail perilaku lengkap.

## Skill

Ketika skill yang memenuhi syarat ada, OpenClaw menyuntikkan **daftar skill tersedia** yang ringkas
(`formatSkillsForPrompt`) yang menyertakan **jalur file** untuk setiap skill. Prompt
menginstruksikan model untuk menggunakan `read` untuk memuat SKILL.md di lokasi yang terdaftar
(workspace, dikelola, atau dibundel). Jika tidak ada skill yang memenuhi syarat, bagian
Skill dihilangkan.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Ini menjaga prompt dasar tetap kecil sambil tetap memungkinkan penggunaan skill yang ditargetkan.

## Dokumentasi

Bila tersedia, prompt sistem menyertakan bagian **Dokumentasi** yang menunjuk ke
direktori dokumen OpenClaw lokal (baik `docs/` di workspace repo atau docs paket
npm yang dibundel) dan juga mencatat mirror publik, repo sumber, Discord komunitas, dan
ClawHub ([https://clawhub.com](https://clawhub.com)) untuk penemuan skill. Prompt menginstruksikan model untuk berkonsultasi dengan dokumen lokal terlebih dahulu
untuk perilaku OpenClaw, perintah, konfigurasi, atau arsitektur, dan untuk menjalankan
`openclaw status` sendiri bila memungkinkan (meminta pengguna hanya bila kekurangan akses).


