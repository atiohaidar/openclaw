---
summary: "Referensi CLI untuk `openclaw webhooks` (pembantu webhook + Gmail Pub/Sub)"
read_when:
  - Anda ingin menghubungkan kejadian Gmail Pub/Sub ke OpenClaw
  - Anda menginginkan perintah pembantu untuk webhook
title: "webhooks"
---

# `openclaw webhooks`

Pembantu dan integrasi webhook (khususnya untuk Gmail Pub/Sub).

## Terkait:
- Webhooks: [Webhook](/id-ID/automation/webhook)
- Gmail Pub/Sub: [Gmail Pub/Sub](/id-ID/automation/gmail-pubsub)

## Gmail

Perintah untuk menyiapkan dan menjalankan integrasi Gmail:

```bash
# Menyiapkan integrasi Gmail
openclaw webhooks gmail setup --account email@anda.com

# Menjalankan listener Gmail
openclaw webhooks gmail run
```

Lihat [dokumentasi Gmail Pub/Sub](/id-ID/automation/gmail-pubsub) untuk detail lebih lanjut mengenai persyaratan dan cara kerjanya.



