---
summary: "Referensi CLI untuk `openclaw onboard` (panduan onboarding interaktif)"
read_when:
  - Anda menginginkan penyiapan terpandu untuk gateway, ruang kerja, autentikasi, saluran, dan skill
title: "onboard"
---

# `openclaw onboard`

Panduan onboarding interaktif untuk penyiapan Gateway lokal atau jarak jauh.

## Panduan Terkait:
- Pusat onboarding CLI: [Wizard Onboarding (CLI)](/id-ID/start/wizard)
- Ringkasan onboarding: [Ringkasan Onboarding](/id-ID/start/onboarding-overview)
- Referensi onboarding CLI: [Referensi Onboarding CLI](/id-ID/start/wizard-cli-reference)

## Contoh

```bash
# Menjalankan panduan interaktif
openclaw onboard

# Mode Quickstart (prompt minimal)
openclaw onboard --flow quickstart

# Menghubungkan ke Gateway jarak jauh (remote)
openclaw onboard --mode remote --remote-url ws://alamat-host:18789
```

### Penyiapan Non-interaktif (Skrip)
```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "nama-model" \
  --custom-api-key "$KUNCI_API" \
  --custom-compatibility openai
```

## Catatan Alur (Flow):
- `quickstart`: Prompt minimal, otomatis menghasilkan token gateway.
- `manual`: Prompt lengkap untuk port/bind/autentikasi (sama dengan `advanced`).
- Obrolan pertama tercepat: `openclaw dashboard` (UI Kontrol, tanpa perlu setup saluran).

## Perintah Lanjutan Umum:
```bash
openclaw configure
openclaw agents add <nama>
```



