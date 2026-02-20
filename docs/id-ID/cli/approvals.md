---
summary: "Referensi CLI untuk `openclaw approvals` (persetujuan eksekusi untuk gateway atau host node)"
read_when:
  - Anda ingin mengubah persetujuan eksekusi (exec) dari CLI
  - Anda perlu mengelola daftar izinkan (allowlist) pada gateway atau host node
title: "approvals"
---

# `openclaw approvals`

Mengelola persetujuan eksekusi (exec approvals) untuk **host lokal**, **host gateway**, atau **host node**. Secara default, perintah menargetkan file persetujuan lokal. Gunakan `--gateway` untuk target gateway, atau `--node` untuk menargetkan node tertentu.

## Terkait:
- Persetujuan Eksekusi: [Exec approvals](/id-ID/tools/exec-approvals)
- Node: [Nodes](/id-ID/cli/nodes)

## Perintah Umum

```bash
# Mengambil data persetujuan lokal
openclaw approvals get

# Mengambil data persetujuan dari node tertentu
openclaw approvals get --node <id|nama|ip>

# Mengambil data persetujuan dari gateway
openclaw approvals get --gateway
```

## Pembantu Daftar Izinkan (Allowlist Helpers)

```bash
# Menambah jalur file ke daftar izinkan lokal
openclaw approvals allowlist add "~/Proyek/**/bin/rg"

# Menambah perintah ke daftar izinkan pada node tertentu untuk agen 'main'
openclaw approvals allowlist add --agent main --node <id|nama|ip> "/usr/bin/uptime"

# Menghapus dari daftar izinkan
openclaw approvals allowlist remove "~/Proyek/**/bin/rg"
```

**Catatan:** File persetujuan disimpan di masing-masing host pada lokasi `~/.openclaw/exec-approvals.json`. Host node harus mendukung fitur get/set persetujuan eksekusi agar perintah ini dapat bekerja.



