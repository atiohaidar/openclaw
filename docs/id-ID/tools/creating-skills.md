---
title: "Membuat Skill"
---

# Membuat Skill Kustom 🛠

OpenClaw dirancang agar mudah diperluas. "Skill" adalah cara utama untuk menambahkan kemampuan baru ke asisten Anda.

## Apa itu Skill?

Skill adalah sebuah direktori yang berisi file `SKILL.md` (yang memberikan instruksi dan definisi alat kepada LLM) dan secara opsional menyertakan beberapa skrip atau sumber daya (resources).

## Langkah-demi-Langkah: Skill Pertama Anda

### 1. Buat Direktori

Skill berada di workspace Anda, biasanya di `~/.openclaw/workspace/skills/`. Buat folder baru untuk skill Anda:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Definisikan `SKILL.md`

Buat file `SKILL.md` di direktori tersebut. File ini menggunakan YAML frontmatter untuk metadata dan Markdown untuk instruksi.

```markdown
---
name: hello_world
description: Skill sederhana yang mengucapkan halo.
---

# Hello World Skill

Ketika pengguna meminta sapaan, gunakan alat `echo` untuk mengucapkan "Halo dari skill kustom Anda!".
```

### 3. Tambahkan Alat (Opsional)

Anda dapat mendefinisikan alat (tool) kustom di frontmatter atau menginstruksikan agen untuk menggunakan alat sistem yang sudah ada (seperti `bash` atau `browser`).

### 4. Segarkan (Refresh) OpenClaw

Mintalah agen Anda untuk "refresh skills" atau mulai ulang gateway. OpenClaw akan menemukan direktori baru tersebut dan mengindeks `SKILL.md`.

## Praktik Terbaik

- **Singkat dan Jelas**: Instruksikan model tentang *apa* yang harus dilakukan, bukan cara menjadi AI.
- **Utamakan Keamanan**: Jika skill Anda menggunakan `bash`, pastikan prompt tidak memungkinkan injeksi perintah sembarangan dari input pengguna yang tidak terpercaya.
- **Uji Secara Lokal**: Gunakan `openclaw agent --message "gunakan skill baru saya"` untuk menguji.

## Skill Bersama

Anda juga dapat menjelajahi dan berkontribusi skill ke [ClawHub](https://clawhub.com).



