---
summary: "Sandbox per agen + pembatasan alat, prioritas, dan contoh"
title: Sandbox & Alat Multi-Agen
read_when: "Anda ingin sandboxing per agen atau kebijakan izinkan/tolak alat per agen di gateway multi-agen."
---

# Konfigurasi Sandbox & Alat Multi-Agen

## Ringkasan

Setiap agen dalam pengaturan multi-agen kini dapat memiliki:
- **Konfigurasi Sandbox** sendiri (`agents.list[].sandbox` menimpa `agents.defaults.sandbox`).
- **Pembatasan Alat** sendiri (`tools.allow` / `tools.deny`, ditambah `agents.list[].tools`).

Ini memungkinkan Anda menjalankan beberapa agen dengan profil keamanan yang berbeda:
- Asisten pribadi dengan akses penuh.
- Agen keluarga/kerja dengan alat yang dibatasi.
- Agen publik yang berjalan di dalam sandbox.

Autentikasi bersifat per agen: setiap agen membaca dari penyimpanan autentikasinya sendiri di `agentDir`. Kredensial **tidak** dibagikan antar agen.

Untuk penjelasan tentang perilaku sandbox saat runtime, lihat [Sandboxing](/id-ID/gateway/sandboxing).

---

## Contoh Konfigurasi

### Contoh 1: Agen Pribadi + Agen Keluarga yang Dibatasi

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "name": "Asisten Pribadi",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Bot Keluarga",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  }
}
```

---

## Urutan Prioritas Konfigurasi

Ketika konfigurasi global (`agents.defaults.*`) dan spesifik agen (`agents.list[].*`) keduanya ada:

### Konfigurasi Sandbox
Pengaturan spesifik agen akan menimpa pengaturan global. Misalnya, `agents.list[].sandbox.mode` memiliki prioritas lebih tinggi daripada `agents.defaults.sandbox.mode`.

### Pembatasan Alat
Urutan penyaringan adalah:
1. Profil Alat (`tools.profile`)
2. Kebijakan Alat Global (`tools.allow` / `tools.deny`)
3. Kebijakan Alat Per Agen (`agents.list[].tools.allow/deny`)
4. Kebijakan Alat Sandbox

Setiap level dapat membatasi lebih lanjut, tetapi tidak dapat mengizinkan kembali alat yang sudah ditolak di level sebelumnya.

### Grup Alat (Singkatan)
Kebijakan alat mendukung penggunaan `group:*` untuk mempermudah konfigurasi:
- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:messaging`: `message`
- `group:ui`: `browser`, `canvas`

---

## Perangkap Umum: "non-main"

Opsi `agents.defaults.sandbox.mode: "non-main"` didasarkan pada kunci sesi (`session.mainKey`), bukan ID agen. Sesi grup/saluran selalu mendapatkan kunci uniknya sendiri, sehingga akan diperlakukan sebagai "non-main" dan akan dimasukkan ke dalam sandbox. Jika Anda ingin agen tertentu tidak pernah menggunakan sandbox, setel `agents.list[].sandbox.mode: "off"`.

---

## Pemecahan Masalah

### Agen tidak masuk sandbox padahal `mode: "all"`
Periksa apakah ada pengaturan `agents.defaults.sandbox.mode` global yang menimpanya atau pastikan pengaturan spesifik agen sudah benar.

### Alat masih tersedia padahal sudah masuk daftar cekal (deny)
Ingat urutan penyaringan: Global → Agen → Sandbox. Setiap level hanya bisa memperketat pembatasan.



