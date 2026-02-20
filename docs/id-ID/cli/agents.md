---
summary: "Referensi CLI untuk `openclaw agents` (daftar/tambah/hapus/atur identitas)"
read_when:
  - Anda menginginkan banyak agen yang terisolasi (ruang kerja + perutean + autentikasi)
title: "agents"
---

# `openclaw agents`

Mengelola agen yang terisolasi (ruang kerja, autentikasi, dan perutean).

## Terkait:
- Perutean Multi-Agen: [Multi-Agent Routing](/id-ID/concepts/multi-agent)
- Ruang Kerja Agen: [Agent workspace](/id-ID/concepts/agent-workspace)

## Contoh

```bash
# Menampilkan daftar agen
openclaw agents list

# Menambah agen baru (work) dengan ruang kerja spesifik
openclaw agents add work --workspace ~/.openclaw/workspace-work

# Mengatur identitas agen dari file IDENTITY.md
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity

# Menghapus agen
openclaw agents delete work
```

## Identitas Agen

Setiap ruang kerja agen dapat memiliki file `IDENTITY.md` di root folder ruang kerja tersebut.

### Mengatur Identitas (Set Identity)
Perintah `set-identity` menulis data ke bagian identitas agen (nama, tema, emoji, avatar).

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Avatar dapat berupa jalur file relatif terhadap ruang kerja, URL http(s), atau data URI.



