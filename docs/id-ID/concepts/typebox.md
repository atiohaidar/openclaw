---
summary: "Skema TypeBox sebagai sumber kebenaran tunggal untuk protokol gateway"
read_when:
  - Memperbarui skema protokol atau pembuatan kode (codegen)
title: "TypeBox"
---

# TypeBox sebagai Sumber Kebenaran Protokol

Terakhir diperbarui: 2026-01-10

TypeBox adalah pustaka skema yang mengutamakan TypeScript. Kami menggunakannya untuk mendefinisikan **protokol WebSocket Gateway** (handshake, permintaan/respons, event server). Skema tersebut menggerakkan **validasi runtime**, **ekspor JSON Schema**, dan **Swift codegen** untuk aplikasi macOS. Satu sumber kebenaran; yang lainnya dihasilkan dari sini.

Jika Anda menginginkan konteks protokol tingkat tinggi, mulailah dengan [Arsitektur Gateway](/id-ID/concepts/architecture).

## Model Mental (30 detik)

Setiap pesan WS Gateway adalah salah satu dari tiga bingkai (frame):

-   **Request** (Permintaan): `{ type: "req", id, method, params }`
-   **Response** (Respons): `{ type: "res", id, ok, payload | error }`
-   **Event** (Kejadian): `{ type: "event", event, payload, seq?, stateVersion? }`

Bingkai pertama **harus** berupa permintaan `connect`. Setelah itu, klien dapat memanggil metode (misalnya `health`, `send`, `chat.send`) dan berlangganan event (misalnya `presence`, `tick`, `agent`).

Alur koneksi (minimal):

```
Klien                     Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Metode + Event Umum:

| Kategori | Contoh                                                    | Catatan                             |
| -------- | --------------------------------------------------------- | ----------------------------------- |
| Inti     | `connect`, `health`, `status`                             | `connect` harus yang pertama        |
| Pesan    | `send`, `poll`, `agent`, `agent.wait`                     | efek samping perlu `idempotencyKey` |
| Obrolan  | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat menggunakan ini             |
| Sesi     | `sessions.list`, `sessions.patch`, `sessions.delete`      | administrasi sesi                   |
| Node     | `node.list`, `node.invoke`, `node.pair.*`                 | WS Gateway + aksi node              |
| Event    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | dorongan server (server push)       |

Daftar otoritatif berada di `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## Lokasi Skema

-   Sumber: `src/gateway/protocol/schema.ts`
-   Validator Runtime (AJV): `src/gateway/protocol/index.ts`
-   Handshake Server + Pengiriman Metode: `src/gateway/server.ts`
-   Klien Node: `src/gateway/client.ts`
-   JSON Schema yang dihasilkan: `dist/protocol.schema.json`
-   Model Swift yang dihasilkan: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## Pipeline Saat Ini

-   `pnpm protocol:gen`
    - menulis JSON Schema (draft‑07) ke `dist/protocol.schema.json`
-   `pnpm protocol:gen:swift`
    - menghasilkan model gateway Swift
-   `pnpm protocol:check`
    - menjalankan kedua generator dan memverifikasi bahwa output telah di-commit

## Bagaimana Skema Digunakan Saat Runtime

-   **Sisi Server**: setiap bingkai masuk divalidasi dengan AJV. Handshake hanya menerima permintaan `connect` yang parameternya cocok dengan `ConnectParams`.
-   **Sisi Klien**: klien JS memvalidasi bingkai event dan respons sebelum menggunakannya.
-   **Permukaan Metode**: Gateway mengiklankan `methods` dan `events` yang didukung dalam `hello-ok`.

## Contoh Bingkai (Frame)

Connect (pesan pertama):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Respons hello-ok:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Permintaan + Respons:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Event:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## Klien Minimal (Node.js)

Alur paling sederhana: connect + health.

```ts
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "contoh",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("kesehatan:", msg.payload);
    ws.close();
  }
});
```

## Contoh Pengerjaan: Menambah Metode dari Awal Sampai Akhir

Contoh: menambah permintaan baru `system.echo` yang mengembalikan `{ ok: true, text }`.

1.  **Skema (Sumber Kebenaran)**

Tambahkan ke `src/gateway/protocol/schema.ts`:

```ts
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Tambahkan keduanya ke `ProtocolSchemas` dan ekspor tipenya:

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **Validasi**

Di `src/gateway/protocol/index.ts`, ekspor sebuah validator AJV:

```ts
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **Perilaku Server**

Tambahkan handler di `src/gateway/server-methods/system.ts`:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Daftarkan di `src/gateway/server-methods.ts` (yang sudah menggabungkan `systemHandlers`), lalu tambahkan `"system.echo"` ke `METHODS` di `src/gateway/server.ts`.

4.  **Hasilkan Ulang (Regenerate)**

```bash
pnpm protocol:check
```

5.  **Tes + Dokumentasi**

Tambahkan tes server di `src/gateway/server.*.test.ts` dan beri catatan tentang metodenya di dokumentasi.

## Perilaku Swift Codegen

Generator Swift mengeluarkan:

-   Enum `GatewayFrame` dengan kasus `req`, `res`, `event`, dan `unknown`.
-   Struct/enum payload dengan tipe data yang kuat (strongly typed).
-   Nilai `ErrorCode` dan `GATEWAY_PROTOCOL_VERSION`.

Tipe bingkai yang tidak dikenal dipertahankan sebagai payload mentah untuk kompatibilitas masa depan.

## Versi + Kompatibilitas

-   `PROTOCOL_VERSION` berada di `src/gateway/protocol/schema.ts`.
-   Klien mengirimkan `minProtocol` + `maxProtocol`; server menolak jika tidak cocok.
-   Model Swift mempertahankan tipe bingkai yang tidak dikenal untuk menghindari kerusakan pada klien versi lama.

## Pola dan Konvensi Skema

-   Sebagian besar objek menggunakan `additionalProperties: false` untuk payload yang ketat.
-   `NonEmptyString` adalah default untuk ID dan nama metode/event.
-   `GatewayFrame` tingkat atas menggunakan **diskriminator** pada `type`.
-   Metode dengan efek samping biasanya memerlukan `idempotencyKey` di dalam parameter (contoh: `send`, `poll`, `agent`, `chat.send`).

## JSON Skema Langsung

JSON Schema yang dihasilkan ada di repo pada `dist/protocol.schema.json`. File mentah yang dipublikasikan biasanya tersedia di:

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## Saat Anda Mengubah Skema

1.  Perbarui skema TypeBox.
2.  Jalankan `pnpm protocol:check`.
3.  Commit skema yang dihasilkan ulang beserta model Swift-nya.
