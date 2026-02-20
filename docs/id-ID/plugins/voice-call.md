---
summary: "Plugin Voice Call: panggilan keluar + masuk melalui Twilio/Telnyx/Plivo (instalasi + konfigurasi + CLI)"
read_when:
  - Anda ingin melakukan panggilan suara keluar dari OpenClaw
  - Anda sedang mengonfigurasi atau mengembangkan plugin voice-call
title: "Plugin Voice Call"
---

# Panggilan Suara (Voice Call Plugin)

Panggilan suara untuk OpenClaw melalui sebuah plugin. Mendukung notifikasi keluar dan percakapan multi-putaran dengan kebijakan panggilan masuk.

Penyedia (provider) saat ini:
- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + GetInput speech)
- `mock` (pengembangan/tanpa jaringan)

## Lokasi Berjalan

Plugin Voice Call berjalan **di dalam proses Gateway**.
Jika Anda menggunakan Gateway jarak jauh, instal dan konfigurasi plugin pada **mesin yang menjalankan Gateway**, lalu mulai ulang Gateway untuk memuatnya.

## Instalasi

### Opsi A: Instal dari npm (Direkomendasikan)
```bash
openclaw plugins install @openclaw/voice-call
```
Mulai ulang Gateway setelahnya.

### Opsi B: Instal dari folder lokal (Pengembangan)
```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```
Mulai ulang Gateway setelahnya.

## Konfigurasi

Atur konfigurasi di bawah `plugins.entries.voice-call.config`:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // atau "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },
          
          // Server webhook
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          outbound: {
            defaultMode: "notify", // notify | conversation
          },
        },
      },
    },
  },
}
```

Catatan:
- Twilio/Telnyx/Plivo memerlukan URL webhook yang **dapat diakses secara publik**.
- `mock` adalah penyedia pengembangan lokal (tidak ada panggilan jaringan).

## Keamanan Webhook

Jika terdapat proxy atau tunnel di depan Gateway, plugin akan menyusun kembali URL publik untuk verifikasi tanda tangan.

`webhookSecurity.allowedHosts` mencantumkan host yang diizinkan dari header penerusan (forwarding headers).

## TTS untuk Panggilan

Voice Call menggunakan konfigurasi `messages.tts` inti (OpenAI atau ElevenLabs) untuk mensintesis suara pada panggilan. Anda dapat menimpanya (override) di bawah konfigurasi plugin dengan bentuk yang sama.

Catatan:
- **Edge TTS diabaikan untuk panggilan suara** (telepon memerlukan format PCM; output Edge tidak dapat diandalkan).
- TTS inti digunakan jika media streaming Twilio diaktifkan; jika tidak, panggilan akan menggunakan suara bawaan penyedia.

## Panggilan Masuk (Inbound)

Kebijakan panggilan masuk defaultnya adalah `disabled`. Untuk mengaktifkannya:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?",
}
```

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Ada pertanyaan?"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
```

## Alat Agen

Nama alat: `voice_call`

Tindakan:
- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)


