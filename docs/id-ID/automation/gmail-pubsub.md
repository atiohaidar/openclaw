---
summary: "Notifikasi push Gmail Pub/Sub yang dihubungkan ke webhook OpenClaw melalui gogcli"
read_when:
  - Menghubungkan pemicu inbox Gmail ke OpenClaw
  - Menyiapkan push Pub/Sub untuk membangunkan agen
title: "Gmail PubSub"
---

# Gmail Pub/Sub -> OpenClaw

Tujuan: Pemantauan Gmail -> Push Pub/Sub -> `gog gmail watch serve` -> Webhook OpenClaw.

## Prasyarat

- `gcloud` terinstal dan sudah login.
- `gog` (gogcli) terinstal dan diotorisasi untuk akun Gmail ([gogcli.sh](https://gogcli.sh/)).
- Hook OpenClaw diaktifkan (lihat [Webhooks](/id-ID/automation/webhook)).
- `tailscale` sudah login. Pengaturan yang didukung menggunakan Tailscale Funnel.

## Wizard (Direkomendasikan)

Gunakan pembantu OpenClaw untuk menghubungkan semuanya (menginstal dependensi di macOS melalui brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Pengaturan default:
- Menggunakan Tailscale Funnel untuk endpoint push publik.
- Mengaktifkan preset hook Gmail (`hooks.presets: ["gmail"]`).

Gateway akan otomatis menjalankan `gog gmail watch serve` saat booting jika `hooks.gmail.account` sudah diatur.

## Daemon Manual

Untuk menjalankan daemon secara manual yang menangani push dan memperbarui pemantauan otomatis:

```bash
openclaw webhooks gmail run
```

## Langkah Pengaturan Satu Kali (Manual)

1. Pilih proyek GCP yang memiliki klien OAuth yang digunakan oleh `gog`:
```bash
gcloud config set project <project-id>
```

2. Aktifkan API:
```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Buat topik:
```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Izinkan push Gmail API untuk memublikasikan pesan:
```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## Jalankan Push Handler

Setelah pemantauan dimulai dengan `gog gmail watch start`, jalankan handler:

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body
```

## Pemecahan Masalah

- `Invalid topicName`: terjadi ketidakcocokan proyek (topik tidak berada dalam proyek klien OAuth).
- `User not authorized`: izin `roles/pubsub.publisher` pada topik belum diberikan.
- Pesan kosong: Push Gmail hanya memberikan `historyId`; data email harus diambil melalui `gog gmail history`.



