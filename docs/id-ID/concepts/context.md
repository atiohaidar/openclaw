---
summary: "Konteks: apa yang dilihat model, bagaimana dibangun, dan cara memeriksanya"
read_when:
  - Anda ingin memahami apa arti “konteks” di OpenClaw
  - Anda sedang mendebug mengapa model “tahu” sesuatu (atau melupakannya)
  - Anda ingin mengurangi overhead konteks (/context, /status, /compact)
title: "Konteks"
---

# Konteks

“Konteks” adalah **segala sesuatu yang dikirim OpenClaw ke model untuk sebuah proses**. Ini dibatasi oleh **jendela konteks** model (batas token).

Model mental pemula:

- **Prompt sistem** (dibangun OpenClaw): aturan, alat, daftar skill, waktu/runtime, dan file workspace yang disuntikkan.
- **Riwayat percakapan**: pesan Anda + pesan asisten untuk sesi ini.
- **Pemanggilan alat/hasil + lampiran**: output perintah, pembacaan file, gambar/audio, dll.

Konteks _tidak sama_ dengan “memori”: memori dapat disimpan di disk dan dimuat ulang nanti; konteks adalah apa yang ada di dalam jendela model saat ini.

## Mulai cepat (memeriksa konteks)

- `/status` → tampilan cepat “seberapa penuh jendela saya?” + pengaturan sesi.
- `/context list` → apa yang disuntikkan + ukuran kasar (per file + total).
- `/context detail` → rincian lebih dalam: per-file, ukuran skema per-alat, ukuran entri per-skill, dan ukuran prompt sistem.
- `/usage tokens` → lampirkan footer penggunaan per-balasan ke balasan normal.
- `/compact` → ringkas riwayat lama menjadi entri padat untuk membebaskan ruang jendela.

Lihat juga: [Perintah Slash](/id-ID/tools/slash-commands), [Penggunaan token & biaya](/id-ID/reference/token-use), [Pemadatan](/id-ID/concepts/compaction).

## Contoh output

Nilai bervariasi berdasarkan model, penyedia, kebijakan alat, dan apa yang ada di workspace Anda.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Apa yang dihitung terhadap jendela konteks

Segala sesuatu yang diterima model dihitung, termasuk:

- Prompt sistem (semua bagian).
- Riwayat percakapan.
- Pemanggilan alat + hasil alat.
- Lampiran/transkrip (gambar/audio/file).
- Ringkasan pemadatan dan artefak pemangkasan.
- “Wrapper” penyedia atau header tersembunyi (tidak terlihat, tetap dihitung).

## Bagaimana OpenClaw membangun prompt sistem

Prompt sistem **dimiliki oleh OpenClaw** dan dibangun ulang setiap proses. Ini mencakup:

- Daftar alat + deskripsi singkat.
- Daftar skill (hanya metadata; lihat di bawah).
- Lokasi workspace.
- Waktu (UTC + waktu pengguna yang dikonversi jika dikonfigurasi).
- Metadata runtime (host/OS/model/thinking).
- File bootstrap workspace yang disuntikkan di bawah **Konteks Proyek**.

Rincian lengkap: [Prompt Sistem](/id-ID/concepts/system-prompt).

## File workspace yang disuntikkan (Konteks Proyek)

Secara default, OpenClaw menyuntikkan set file workspace tetap (jika ada):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (hanya run pertama)

File besar dipangkas per-file menggunakan `agents.defaults.bootstrapMaxChars` (default `20000` karakter). OpenClaw juga menegakkan batas total injeksi bootstrap di seluruh file dengan `agents.defaults.bootstrapTotalMaxChars` (default `150000` karakter). `/context` menunjukkan ukuran **mentah vs disuntikkan** dan apakah pemangkasan terjadi.

## Skill: apa yang disuntikkan vs dimuat sesuai permintaan

Prompt sistem menyertakan **daftar skill** ringkas (nama + deskripsi + lokasi). Daftar ini memiliki overhead nyata.

Instruksi skill _tidak_ disertakan secara default. Model diharapkan untuk `read` `SKILL.md` milik skill **hanya bila diperlukan**.

## Alat: ada dua biaya

Alat memengaruhi konteks dalam dua cara:

1. **Teks daftar alat** dalam prompt sistem (apa yang Anda lihat sebagai “Tooling”).
2. **Skema alat** (JSON). Ini dikirim ke model agar dapat memanggil alat. Mereka dihitung terhadap konteks meskipun Anda tidak melihatnya sebagai teks biasa.

`/context detail` merinci skema alat terbesar sehingga Anda dapat melihat apa yang mendominasi.

## Perintah, direktif, dan “pintasan inline”

Perintah slash ditangani oleh Gateway. Ada beberapa perilaku berbeda:

- **Perintah mandiri**: pesan yang hanya `/...` berjalan sebagai perintah.
- **Direktif**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` dihapus sebelum model melihat pesan.
  - Pesan yang hanya berisi direktif mempertahankan pengaturan sesi.
  - Direktif inline dalam pesan normal bertindak sebagai petunjuk per-pesan.
- **Pintasan inline** (hanya pengirim yang diizinkan): token `/...` tertentu di dalam pesan normal dapat berjalan segera (contoh: “hey /status”), dan dihapus sebelum model melihat teks yang tersisa.

Detail: [Perintah Slash](/id-ID/tools/slash-commands).

## Sesi, pemadatan, dan pemangkasan (apa yang bertahan)

Apa yang bertahan melintasi pesan bergantung pada mekanismenya:

- **Riwayat normal** bertahan dalam transkrip sesi hingga dipadatkan/dipangkas oleh kebijakan.
- **Pemadatan** mempertahankan ringkasan ke dalam transkrip dan menjaga pesan terbaru tetap utuh.
- **Pemangkasan** menghapus hasil alat lama dari prompt _dalam-memori_ untuk sebuah proses, tetapi tidak menulis ulang transkrip.

Dokumen: [Sesi](/id-ID/concepts/session), [Pemadatan](/id-ID/concepts/compaction), [Pemangkasan sesi](/id-ID/concepts/session-pruning).

## Apa yang sebenarnya dilaporkan `/context`

`/context` lebih memilih laporan prompt sistem **buatan-run** terbaru bila tersedia:

- `System prompt (run)` = ditangkap dari run (berkemampuan alat) tertanam terakhir dan disimpan di penyimpanan sesi.
- `System prompt (estimate)` = dihitung saat itu juga ketika tidak ada laporan run (atau saat berjalan melalui backend CLI yang tidak menghasilkan laporan).

Bagaimanapun, ia melaporkan ukuran dan kontributor utama; ia **tidak** membuang prompt sistem penuh atau skema alat.


