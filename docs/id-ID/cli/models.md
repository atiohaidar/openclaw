---
summary: "Referensi CLI untuk `openclaw models` (status/daftar/atur/pindai, alias, cadangan, autentikasi)"
read_when:
  - Anda ingin mengubah model default atau melihat status autentikasi penyedia
  - Anda ingin memindai model/penyedia yang tersedia dan mendebug profil autentikasi
title: "models"
---

# `openclaw models`

Penemuan model, pemindaian, dan konfigurasi (model default, model cadangan / fallbacks, dan profil autentikasi).

## Perintah Umum

```bash
# Menampilkan status model dan autentikasi
openclaw models status

# Menampilkan daftar model yang tersedia
openclaw models list

# Mengatur model default (mendukung alias)
openclaw models set <model-atau-alias>

# Memindai model/penyedia yang tersedia
openclaw models scan
```

### `models status`
Menampilkan model default, daftar model cadangan, dan ringkasan autentikasi. Tambahkan `--probe` untuk melakukan tes langsung terhadap profil penyedia yang dikonfigurasi.

### Alias & Fallbacks
```bash
# Menampilkan daftar alias model
openclaw models aliases list

# Menampilkan daftar model cadangan (fallbacks)
openclaw models fallbacks list
```

## Profil Autentikasi
Gunakan sub-perintah ini untuk mengelola kredensial penyedia model:

```bash
openclaw models auth add         # Menambah autentikasi secara interaktif
openclaw models auth login       # Menjalankan alur login penyedia (OAuth/API Key)
openclaw models auth setup-token # Mengatur token penyiapan (setup-token)
```

**Catatan:** `models auth login` akan menjalankan alur autentikasi sesuai dengan plugin penyedia yang bersangkutan.


