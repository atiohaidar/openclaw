---
summary: "Ringkasan opsi dan alur onboarding OpenClaw"
read_when:
  - Memilih jalur onboarding
  - Menyiapkan lingkungan baru
title: "Ringkasan Onboarding"
sidebarTitle: "Ringkasan Onboarding"
---

# Ringkasan Onboarding

OpenClaw mendukung beberapa jalur onboarding tergantung di mana Gateway dijalankan
dan bagaimana Anda ingin mengonfigurasi penyedia (provider).

## Pilih jalur onboarding Anda

- **Wizard CLI** untuk macOS, Linux, dan Windows (melalui WSL2).
- **Aplikasi macOS** untuk panduan penggunaan pertama kali pada Mac berbasis Apple silicon atau Intel.

## Wizard onboarding CLI

Jalankan wizard di terminal:

```bash
openclaw onboard
```

Gunakan wizard CLI jika Anda menginginkan kontrol penuh terhadap Gateway, workspace,
saluran, dan skill. Dokumentasi:

- [Wizard Onboarding (CLI)](/id-ID/start/wizard)
- [Perintah `openclaw onboard`](/cli/onboard)

## Onboarding aplikasi macOS

Gunakan aplikasi OpenClaw jika Anda menginginkan pengaturan terpandu di macOS. Dokumentasi:

- [Onboarding (Aplikasi macOS)](/id-ID/start/onboarding)

## Penyedia Kustom (Custom Provider)

Jika Anda memerlukan endpoint yang tidak terdaftar, termasuk penyedia hosted yang
mengekspos API standar OpenAI atau Anthropic, pilihlah **Penyedia Kustom (Custom Provider)** di
wizard CLI. Anda akan diminta untuk:

- Memilih kompatibel dengan OpenAI, kompatibel dengan Anthropic, atau **Tidak Diketahui (Unknown)** (deteksi otomatis).
- Masukkan URL dasar dan kunci API (jika diperlukan oleh penyedia).
- Berikan ID model dan alias opsional.
- Pilih ID Endpoint sehingga beberapa endpoint kustom dapat digunakan secara bersamaan.

Untuk langkah-langkah mendalam, ikuti dokumentasi onboarding CLI di atas.


