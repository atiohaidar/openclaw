---
summary: "Referensi CLI untuk `openclaw skills` (daftar/info/cek) dan kelayakan skill"
read_when:
  - Anda ingin melihat skill mana yang tersedia dan siap dijalankan
  - Anda ingin mendebug binari/variabel lingkungan/konfigurasi yang kurang untuk suatu skill
title: "skills"
---

# `openclaw skills`

Memeriksa skill (bawaan + ruang kerja + kelolaan) dan melihat mana yang layak (eligible) vs yang persyaratannya kurang.

## Terkait:
- Sistem Skill: [Skills](/id-ID/tools/skills)
- Konfigurasi Skill: [Skills config](/id-ID/tools/skills-config)
- Instalasi ClawHub: [ClawHub](/id-ID/tools/clawhub)

## Perintah-perintah

```bash
# Menampilkan semua skill
openclaw skills list

# Menampilkan hanya skill yang sudah siap dijalankan (layak)
openclaw skills list --eligible

# Menampilkan detail satu skill spesifik
openclaw skills info <nama>

# Menampilkan ringkasan kesiapan vs persyaratan yang kurang
openclaw skills check
```
