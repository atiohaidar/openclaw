---
summary: "Referensi CLI untuk `openclaw update` (pembaruan sumber yang aman + mulai ulang otomatis gateway)"
read_when:
  - Anda ingin memperbarui instalasi OpenClaw dengan aman
  - Anda ingin beralih antar saluran pembaruan (stable/beta/dev)
title: "update"
---

# `openclaw update`

Memperbarui OpenClaw dengan aman dan beralih di antara saluran pembaruan `stable`, `beta`, atau `dev`.

Jika Anda menginstal melalui **npm/pnpm** (instalasi global), pembaruan dilakukan melalui alur manajer paket di halaman [Updating](/id-ID/install/updating).

## Penggunaan

```bash
# Menjalankan pembaruan standar
openclaw update

# Memeriksa status pembaruan
openclaw update status

# Beralih ke saluran beta
openclaw update --channel beta

# Memperbarui tanpa memulai ulang Gateway
openclaw update --no-restart
```

## Opsi
- `--channel <stable|beta|dev>`: Mengatur saluran pembaruan (disimpan dalam konfigurasi).
- `--no-restart`: Lewati proses mulai ulang layanan Gateway setelah pembaruan berhasil.
- `--tag <tag|version>`: Menentukan tag npm atau versi tertentu untuk pembaruan ini saja.

## `update status`
Menampilkan saluran pembaruan yang aktif, tag/cabang/SHA git (untuk instalasi sumber), serta ketersediaan pembaruan.

## `update wizard`
Alur interaktif untuk memilih saluran pembaruan dan mengonfirmasi apakah akan memulai ulang Gateway setelah pembaruan (default adalah ya).

## Jenis Saluran
- `stable`: Menggunakan versi stabil terbaru dari npm.
- `beta`: Menggunakan versi beta terbaru untuk mencoba fitur baru lebih awal.
- `dev`: Menggunakan cabang `main` langsung dari repositori git (memerlukan instalasi berbasis git).

## Lihat juga
- `openclaw doctor` (biasanya menawarkan pembaruan terlebih dahulu)
- [Updating](/id-ID/install/updating)



