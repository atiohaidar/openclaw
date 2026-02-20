---
summary: "Referensi CLI untuk `openclaw devices` (pemasangan perangkat + rotasi/pencabutan token)"
read_when:
  - Anda sedang menyetujui permintaan pemasangan perangkat
  - Anda perlu melakukan rotasi atau pencabutan token perangkat
title: "devices"
---

# `openclaw devices`

Mengelola permintaan pemasangan (pairing) perangkat dan token dengan cakupan perangkat.

## Perintah-perintah Utama

### `list`
Menampilkan daftar permintaan pemasangan yang tertunda dan perangkat yang sudah dipasangkan.
```bash
openclaw devices list
```

### `approve` (Setujui)
Menyetujui permintaan pemasangan perangkat. Jika ID permintaan dilewati, OpenClaw akan otomatis menyetujui permintaan yang terbaru.
```bash
openclaw devices approve
openclaw devices approve <requestId>
```

### `reject` (Tolak)
Menolak permintaan pemasangan perangkat yang sedang tertunda.
```bash
openclaw devices reject <requestId>
```

### `rotate` (Rotasi)
Memutar (mengganti) token perangkat untuk peran (role) tertentu.
```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read
```

### `revoke` (Cabut)
Mencabut token perangkat untuk peran tertentu.
```bash
openclaw devices revoke --device <deviceId> --role node
```

## Opsi Utama
- `--url <url>`: URL WebSocket Gateway.
- `--token <token>`: Token Gateway.
- `--json`: Output dalam format JSON.

**Catatan:** Rotasi token akan menghasilkan token baru; perlakukan sebagai rahasia.


