---
summary: "CLI Model: daftar, setel, alias, fallback, pemindaian, status"
read_when:
  - Menambah atau memodifikasi CLI model (models list/set/scan/aliases/fallbacks)
  - Mengubah perilaku fallback model atau UX pemilihan
  - Memperbarui probe pemindaian model (alat/gambar)
title: "CLI Model"
---

# CLI Model

Lihat [/id-ID/concepts/model-failover](/id-ID/concepts/model-failover) untuk rotasi profil autentikasi, masa tenang (cooldown), dan bagaimana hal tersebut berinteraksi dengan fallback.
Ringkasan penyedia + contoh: [/id-ID/concepts/model-providers](/id-ID/concepts/model-providers).

## Cara Kerja Pemilihan Model

OpenClaw memilih model dengan urutan sebagai berikut:

1.  **Utama (Primary)**: model di `agents.defaults.model.primary` atau `agents.defaults.model`.
2.  **Fallback**: daftar di `agents.defaults.model.fallbacks` (sesuai urutan).
3.  **Failover Autentikasi Penyedia**: terjadi di dalam penyedia sebelum beralih ke model berikutnya.

Terkait:

- `agents.defaults.models` adalah daftar izinkan (allowlist)/katalog model yang dapat digunakan OpenClaw (ditambah alias).
- `agents.defaults.imageModel` digunakan **hanya ketika** model utama tidak dapat menerima gambar.
- Default per agen dapat menimpa `agents.defaults.model` melalui `agents.list[].model` ditambah binding (lihat [/id-ID/concepts/multi-agent](/id-ID/concepts/multi-agent)).

## Pilihan Model Cepat (Anekdot)

- **GLM**: sedikit lebih baik untuk coding/pemanggilan alat (tool calling).
- **MiniMax**: lebih baik untuk menulis dan gaya bahasa (vibes).

## Wizard Penyiapan (Direkomendasikan)

Jika Anda tidak ingin mengedit konfigurasi secara manual, jalankan wizard onboarding:

```bash
openclaw onboard
```

Wizard ini dapat menyiapkan model + autentikasi untuk penyedia umum, termasuk langganan **OpenAI Code (Codex)** (OAuth) dan **Anthropic** (direkomendasikan kunci API; `claude setup-token` juga didukung).

## Kunci Konfigurasi (Ringkasan)

- `agents.defaults.model.primary` dan `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel.primary` dan `agents.defaults.imageModel.fallbacks`
- `agents.defaults.models` (daftar izinkan + alias + parameter penyedia)
- `models.providers` (penyedia kustom yang ditulis ke dalam `models.json`)

Referensi model dinormalisasi ke huruf kecil. Alias penyedia seperti `z.ai/*` dinormalisasi menjadi `zai/*`.

Contoh konfigurasi penyedia (termasuk OpenCode Zen) ada di [/id-ID/gateway/configuration](/id-ID/gateway/configuration#opencode-zen-multi-model-proxy).

## "Model is not allowed" (Dan Mengapa Balasan Berhenti)

Jika `agents.defaults.models` disetel, hal tersebut menjadi **daftar izinkan (allowlist)** untuk perintah `/model` dan untuk override sesi. Ketika pengguna memilih model yang tidak ada dalam daftar tersebut, OpenClaw mengembalikan pesan:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Hal ini terjadi **sebelum** balasan normal dihasilkan, sehingga pesan masuk bisa terasa seperti "tidak ditanggapi". Solusinya adalah dengan:

- Menambahkan model ke `agents.defaults.models`, atau
- Menghapus daftar izinkan (hapus `agents.defaults.models`), atau
- Pilih model dari daftar `/model list`.

Contoh konfigurasi daftar izinkan:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## Mengganti Model Dalam Obrolan (`/model`)

Anda dapat mengganti model untuk sesi saat ini tanpa memulai ulang:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Catatan:

- `/model` (dan `/model list`) adalah pemilih ringkas bernomor (famili model + penyedia yang tersedia).
- `/model <#>` memilih dari pemilih tersebut.
- `/model status` adalah tampilan detail (kandidat autentikasi dan, jika dikonfigurasi, `baseUrl` endpoint penyedia + mode `api`).
- Referensi model diurai dengan memisahkan pada tanda `/` **pertama**. Gunakan `provider/model` saat mengetik `/model <ref>`.
- Jika ID model itu sendiri mengandung `/` (gaya OpenRouter), Anda harus menyertakan awalan penyedia (contoh: `/model openrouter/moonshotai/kimi-k2`).
- Jika Anda tidak menyertakan penyedia, OpenClaw memperlakukan input sebagai alias atau model untuk **penyedia default** (hanya berfungsi jika tidak ada `/` dalam ID model).

Perilaku/konfigurasi perintah lengkap: [Perintah Slash](/id-ID/tools/slash-commands).

## Perintah CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (tanpa sub-perintah) adalah jalan pintas untuk `models status`.

### `models list`

Menampilkan model yang dikonfigurasi secara default. Bendera (flags) yang berguna:

- `--all`: katalog lengkap.
- `--local`: hanya penyedia lokal.
- `--provider <name>`: filter berdasarkan penyedia.
- `--plain`: satu model per baris.
- `--json`: output yang dapat dibaca mesin.

### `models status`

Menampilkan model utama yang diselesaikan, fallback, model gambar, dan ringkasan autentikasi dari penyedia yang dikonfigurasi. Ini juga menampilkan status kedaluwarsa OAuth untuk profil yang ditemukan di penyimpanan autentikasi (memberi peringatan dalam 24 jam secara default). `--plain` hanya mencetak model utama yang diselesaikan.

Status OAuth selalu ditampilkan (dan disertakan dalam output `--json`). Jika penyedia yang dikonfigurasi tidak memiliki kredensial, `models status` akan mencetak bagian **Missing auth**. JSON menyertakan `auth.oauth` (jendela peringatan + profil) dan `auth.providers` (autentikasi efektif per penyedia).

Gunakan `--check` untuk otomatisasi (keluar dengan kode `1` jika hilang/kedaluwarsa, `2` jika akan kedaluwarsa).

Autentikasi Anthropic yang disukai adalah lewat Claude Code CLI setup-token (jalankan di mana saja; tempel di host gateway jika perlu):

```bash
claude setup-token
openclaw models status
```

## Pemindaian (Model Gratis OpenRouter)

`openclaw models scan` memeriksa **katalog model gratis** OpenRouter dan secara opsional dapat menguji (probe) model untuk dukungan alat dan gambar.

Bendera utama:

- `--no-probe`: lewati pengujian langsung (hanya metadata).
- `--min-params <b>`: ukuran parameter minimum (miliar).
- `--max-age-days <days>`: lewati model lama.
- `--provider <name>`: filter awalan penyedia.
- `--max-candidates <n>`: ukuran daftar fallback.
- `--set-default`: setel `agents.defaults.model.primary` ke pilihan pertama.
- `--set-image`: setel `agents.defaults.imageModel.primary` ke pilihan gambar pertama.

Pengujian (probing) memerlukan kunci API OpenRouter (dari profil autentikasi atau `OPENROUTER_API_KEY`). Tanpa kunci, gunakan `--no-probe` untuk mencantumkan kandidat saja.

Hasil pemindaian diurutkan berdasarkan:

1.  Dukungan gambar.
2.  Latensi alat.
3.  Ukuran konteks.
4.  Jumlah parameter.

Input:

- Daftar `/models` OpenRouter (filter `:free`).
- Memerlukan kunci API OpenRouter dari profil autentikasi atau `OPENROUTER_API_KEY` (lihat [/id-ID/help/environment](/id-ID/help/environment)).
- Filter opsional: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`.
- Kontrol pengujian: `--timeout`, `--concurrency`.

Saat dijalankan di TTY, Anda dapat memilih fallback secara interaktif. Dalam mode non-interaktif, gunakan `--yes` untuk menerima default.

## Registri Model (`models.json`)

Penyedia kustom di `models.providers` ditulis ke dalam `models.json` di bawah direktori agen (default `~/.openclaw/agents/<agentId>/models.json`). File ini digabungkan secara default kecuali jika `models.mode` disetel ke `replace`.
