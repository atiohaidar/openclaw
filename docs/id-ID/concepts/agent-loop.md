---
summary: "Siklus hidup loop agen, aliran (stream), dan semantik tunggu (wait)"
read_when:
  - Anda memerlukan panduan mendetail tentang loop agen atau event siklus hidup
title: "Loop Agen"
---

# Loop Agen (OpenClaw)

Loop agenik adalah perjalanan "nyata" sepenuhnya dari sebuah agen: asupan (intake) → perakitan konteks → inferensi model →
eksekusi alat → streaming balasan → persistensi. Ini adalah jalur otoritatif yang mengubah pesan
menjadi tindakan dan balasan akhir, sambil menjaga status sesi tetap konsisten.

Di OpenClaw, loop adalah satu proses terserialisasi per sesi yang memancarkan event siklus hidup dan stream
saat model berpikir, memanggil alat, dan mengalirkan output. Dokumen ini menjelaskan bagaimana loop otentik tersebut
terhubung dari ujung ke ujung.

## Titik masuk (Entry points)

- Gateway RPC: `agent` dan `agent.wait`.
- CLI: perintah `agent`.

## Cara kerjanya (tingkat tinggi)

1. `agent` RPC memvalidasi parameter, menyelesaikan sesi (sessionKey/sessionId), mempertahankan metadata sesi, mengembalikan `{ runId, acceptedAt }` segera.
2. `agentCommand` menjalankan agen:
   - menyelesaikan model + default thinking/verbose
   - memuat snapshot skill
   - memanggil `runEmbeddedPiAgent` (runtime pi-agent-core)
   - memancarkan **akhir/error siklus hidup** jika loop tertanam tidak memancarkannya
3. `runEmbeddedPiAgent`:
   - menserialisasi proses melalui antrean per-sesi + global
   - menyelesaikan model + profil autentikasi dan membangun sesi pi
   - berlangganan event pi dan mengalirkan delta asisten/alat
   - menegakkan batas waktu (timeout) -> membatalkan proses jika terlampaui
   - mengembalikan payload + metadata penggunaan
4. `subscribeEmbeddedPiSession` menjembatani event pi-agent-core ke stream `agent` OpenClaw:
   - event alat => `stream: "tool"`
   - delta asisten => `stream: "assistant"`
   - event siklus hidup => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` menggunakan `waitForAgentJob`:
   - menunggu **akhir/error siklus hidup** untuk `runId`
   - mengembalikan `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## Antrean + konkurensi

- Proses diserialisasi per kunci sesi (jalur sesi) dan opsional melalui jalur global.
- Hal ini mencegah balapan (race condition) alat/sesi dan menjaga riwayat sesi tetap konsisten.
- Saluran perpesanan dapat memilih mode antrean (collect/steer/followup) yang memberi makan sistem jalur ini.
  Lihat [Antrean Perintah](/id-ID/concepts/queue).

## Persiapan sesi + workspace

- Workspace diselesaikan dan dibuat; proses sandboxed dapat dialihkan ke root workspace sandbox.
- Skill dimuat (atau digunakan kembali dari snapshot) dan disuntikkan ke dalam env dan prompt.
- File bootstrap/konteks diselesaikan dan disuntikkan ke dalam laporan prompt sistem.
- Kunci tulis sesi diperoleh; `SessionManager` dibuka dan disiapkan sebelum streaming.

## Perakitan prompt + prompt sistem

- Prompt sistem dibangun dari prompt dasar OpenClaw, prompt skill, konteks bootstrap, dan override per-proses.
- Batas spesifik model dan token cadangan pemadatan (compaction reserve) ditegakkan.
- Lihat [Prompt Sistem](/id-ID/concepts/system-prompt) untuk apa yang dilihat model.

## Titik Hook (tempat Anda dapat menginterupsi)

OpenClaw memiliki dua sistem hook:

- **Hook Internal** (Hook Gateway): skrip berbasis event untuk perintah dan event siklus hidup.
- **Hook Plugin**: titik ekstensi di dalam siklus hidup agen/alat dan pipeline gateway.

### Hook Internal (Hook Gateway)

- **`agent:bootstrap`**: berjalan saat membangun file bootstrap sebelum prompt sistem difinalisasi.
  Gunakan ini untuk menambah/menghapus file konteks bootstrap.
- **Hook Perintah**: `/new`, `/reset`, `/stop`, dan event perintah lainnya (lihat dokumen Hook).

Lihat [Hook](/id-ID/automation/hooks) untuk pengaturan dan contoh.

### Hook Plugin (siklus hidup agen + gateway)

Ini berjalan di dalam loop agen atau pipeline gateway:

- **`before_model_resolve`**: berjalan pra-sesi (tanpa `messages`) untuk secara deterministik menimpa provider/model sebelum resolusi model.
- **`before_prompt_build`**: berjalan setelah pemuatan sesi (dengan `messages`) untuk menyuntikkan `prependContext`/`systemPrompt` sebelum pengiriman prompt.
- **`before_agent_start`**: hook kompatibilitas lama yang mungkin berjalan di salah satu fase; lebih disukai hook eksplisit di atas.
- **`agent_end`**: memeriksa daftar pesan akhir dan menjalankan metadata setelah selesai.
- **`before_compaction` / `after_compaction`**: mengamati atau memberi anotasi pada siklus pemadatan.
- **`before_tool_call` / `after_tool_call`**: menginterupsi parameter/hasil alat.
- **`tool_result_persist`**: secara sinkron mengubah hasil alat sebelum ditulis ke transkrip sesi.
- **`message_received` / `message_sending` / `message_sent`**: hook pesan masuk + keluar.
- **`session_start` / `session_end`**: batas siklus hidup sesi.
- **`gateway_start` / `gateway_stop`**: event siklus hidup gateway.

Lihat [Plugin](/id-ID/tools/plugin#plugin-hooks) untuk API hook dan detail pendaftaran.

## Streaming + balasan parsial

- Delta asisten didirkan dari pi-agent-core dan dipancarkan sebagai event `assistant`.
- Block streaming dapat memancarkan balasan parsial baik pada `text_end` atau `message_end`.
- Streaming penalaran (reasoning) dapat dipancarkan sebagai stream terpisah atau sebagai balasan blok.
- Lihat [Streaming](/id-ID/concepts/streaming) untuk perilaku chunking dan balasan blok.

## Eksekusi alat + alat perpesanan

- Event mulai/perbarui/akhir alat dipancarkan pada stream `tool`.
- Hasil alat dibersihkan untuk ukuran dan payload gambar sebelum dicatat/dipancarkan.
- Pengiriman alat perpesanan dilacak untuk menekan konfirmasi asisten duplikat.

## Pembentukan balasan + penekanan (suppression)

- Payload akhir dirakit dari:
  - teks asisten (dan penalaran opsional)
  - ringkasan alat inline (jika verbose + diizinkan)
  - teks error asisten saat model error
- `NO_REPLY` diperlakukan sebagai token diam dan difilter dari payload keluar.
- Duplikat alat perpesanan dihapus dari daftar payload akhir.
- Jika tidak ada payload yang dapat dirender tersisa dan alat error, balasan error alat fallback dipancarkan
  (kecuali alat perpesanan sudah mengirim balasan yang terlihat oleh pengguna).

## Pemadatan + coba ulang (retries)

- Pemadatan otomatis memancarkan event stream `compaction` dan dapat memicu coba ulang.
- Pada coba ulang, buffer dalam-memori dan ringkasan alat diatur ulang untuk menghindari output duplikat.
- Lihat [Pemadatan](/id-ID/concepts/compaction) untuk pipeline pemadatan.

## Stream event (saat ini)

- `lifecycle`: dipancarkan oleh `subscribeEmbeddedPiSession` (dan sebagai fallback oleh `agentCommand`)
- `assistant`: delta yang dialirkan dari pi-agent-core
- `tool`: event alat yang dialirkan dari pi-agent-core

## Penanganan saluran chat

- Delta asisten di-buffer menjadi pesan `delta` chat.
- Chat `final` dipancarkan pada **akhir/error siklus hidup**.

## Batas waktu (Timeouts)

- Default `agent.wait`: 30 detik (hanya menunggu). Parameter `timeoutMs` menimpa.
- Runtime agen: `agents.defaults.timeoutSeconds` default 600 detik; ditegakkan dalam timer abort `runEmbeddedPiAgent`.

## Di mana hal-hal dapat berakhir lebih awal

- Batas waktu agen (abort)
- AbortSignal (batal)
- Putus koneksi Gateway atau batas waktu RPC
- Batas waktu `agent.wait` (hanya menunggu, tidak menghentikan agen)


