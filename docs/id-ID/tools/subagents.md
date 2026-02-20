---
summary: "Sub-agen: menjalankan proses agen terisolasi yang mengumumkan hasilnya kembali ke chat peminta"
read_when:
  - Anda menginginkan pekerjaan latar belakang/paralel melalui agen
  - Anda sedang mengubah kebijakan alat sessions_spawn atau sub-agen
title: "Sub-Agen"
---

# Sub-Agen

Sub-agen adalah proses agen di latar belakang yang dijalankan dari proses agen yang sudah ada. Mereka berjalan dalam sesi sendiri (`agent:<agentId>:subagent:<uuid>`) dan, setelah selesai, akan **mengumumkan** hasilnya kembali ke saluran chat peminta.

## Perintah Slash

Gunakan `/subagents` untuk memeriksa atau mengontrol sub-agen untuk **sesi saat ini**:

- `/subagents list`: daftar sub-agen.
- `/subagents kill <id|#|all>`: matikan sub-agen.
- `/subagents log <id|#> [limit] [tools]`: lihat log sub-agen.
- `/subagents info <id|#>`: info detail sub-agen.
- `/subagents send <id|#> <pesan>`: kirim pesan ke sub-agen.
- `/subagents steer <id|#> <pesan>`: arahkan/setir sub-agen yang sedang berjalan.
- `/subagents spawn <agentId> <tugas> [--model <model>] [--thinking <level>]`: jalankan sub-agen baru.

### Perilaku Spawn

Perintah `/subagents spawn` memulai sub-agen di latar belakang dan mengirimkan satu pembaruan penyelesaian akhir kembali ke chat peminta setelah proses selesai.

- Perintah spawn bersifat tidak memblokir (non-blocking); ia segera mengembalikan ID proses.
- Pesan penyelesaian mencakup:
  - `Result`: teks balasan asisten atau hasil alat terakhir.
  - `Status`: `completed successfully` / `failed` / `timed out`.
  - Statistik waktu jalan dan token yang ringkas.

Tujuan utama:
- Paralelisme tugas panjang (seperti riset) tanpa memblokir chat utama.
- Menjaga isolasi sub-agen (pemisahan sesi + sandbox opsional).
- Menghindari penyalahgunaan: sub-agen tidak mendapatkan akses ke alat sesi secara default.

## Alat (Tool)

Gunakan `sessions_spawn`:

- Memulai sub-agen.
- Menjalankan langkah pengumuman dan memposting balasan ke saluran chat peminta.
- Model default: mewarisi dari pemanggil kecuali dikonfigurasi lain.

## Sub-Agen Bertingkat (Nested Sub-Agents)

Secara default, sub-agen tidak dapat menjalankan sub-agen mereka sendiri (`maxSpawnDepth: 1`). Anda dapat mengaktifkan satu tingkat tingkatan dengan mengatur `maxSpawnDepth: 2`, yang memungkinkan **pola orkestrator**: agen utama → sub-agen orkestrator → sub-sub-agen pekerja.

### Tingkat Kedalaman

| Kedalaman | Peran                                   | Bisa Spawn?                     |
| --------- | --------------------------------------- | ------------------------------- |
| 0         | Agen Utama                              | Selalu                          |
| 1         | Sub-agen (Orkestrator jika kedalaman 2) | Hanya jika `maxSpawnDepth >= 2` |
| 2         | Sub-sub-agen (Pekerja terakhir)         | Tidak pernah                    |

### Penghentian Berantai (Cascade Stop)

Menghentikan orkestrator tingkat-1 secara otomatis akan menghentikan semua anak tingkat-2-nya.

## Pengumuman (Announce)

Sub-agen melaporkan kembali melalui langkah pengumuman:

- Langkah pengumuman berjalan di dalam sesi sub-agen.
- Balasan pengumuman diposting ke saluran chat peminta.
- Laporan mencakup Status, Hasil, dan Catatan, serta statistik penggunaan token/biaya.

## Kebijakan Alat (Alat Sub-agen)

Secara default, sub-agen mendapatkan **semua alat kecuali alat sesi** (seperti `sessions_list`, `sessions_history`, dll) dan alat sistem. Orkestrator (pada tingkat-1 jika kedalaman >= 2) mendapatkan akses tambahan ke alat sesi tertentu untuk mengelola anak-anaknya.

## Batasan

- Pengumuman sub-agen bersifat *best-effort*. Jika gateway dimulai ulang, pekerjaan pengumuman yang tertunda mungkin hilang.
- Sub-agen berbagi sumber daya proses gateway yang sama.
- Konteks sub-agen hanya memasukkan `AGENTS.md` + `TOOLS.md`.
- Kedalaman tingkatan maksimal adalah 5.


