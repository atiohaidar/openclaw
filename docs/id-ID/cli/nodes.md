---
summary: "Referensi CLI untuk `openclaw nodes` (daftar/status/setujui/jalankan, kamera/kanvas/layar)"
read_when:
  - Anda mengelola node yang dipasangkan (kamera, layar, kanvas)
  - Anda perlu menyetujui permintaan atau menjalankan perintah pada node
title: "nodes"
---

# `openclaw nodes`

Mengelola node (perangkat) yang dipasangkan dan menjalankan kemampuan (capabilities) dari node tersebut.

## Terkait:
- Ringkasan Node: [Nodes](/id-ID/nodes)
- Kamera: [Camera nodes](/id-ID/nodes/camera)

## Perintah Umum

```bash
# Menampilkan daftar node (termasuk yang menunggu persetujuan)
openclaw nodes list

# Menampilkan node yang sedang terhubung saja
openclaw nodes list --connected

# Melihat permintaan yang sedang menunggu persetujuan
openclaw nodes pending

# Menyetujui permintaan pemasangan node
openclaw nodes approve <requestId>

# Menampilkan status node
openclaw nodes status
```

## Menjalankan Perintah (Invoke / Run)

```bash
# Menjalankan perintah spesifik pada node
openclaw nodes invoke --node <id|nama|ip> --command <perintah> --params <json>

# Menjalankan perintah eksekusi shell pada node
openclaw nodes run --node <id|nama|ip> <perintah...>

# Menjalankan string shell mentah
openclaw nodes run --raw "git status"
```

### Opsi Eksekusi
- `--cwd <jalur>`: Menentukan direktori kerja (working directory).
- `--env <kunci=nilai>`: Memberikan variabel lingkungan (env).
- `--command-timeout <ms>`: Batas waktu perintah.
- `--needs-screen-recording`: Memerlukan izin rekaman layar.
- `--ask <off|on-miss|always>`: Mengatur kebijakan persetujuan eksekusi.



