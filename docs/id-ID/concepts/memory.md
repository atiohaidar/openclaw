---
title: "Memori"
summary: "Cara kerja memori OpenClaw (file workspace + flush memori otomatis)"
read_when:
  - Anda ingin mengetahui tata letak file memori dan alur kerjanya
  - Anda ingin mengatur flush memori otomatis sebelum pemadatan (pre-compaction)
---

# Memori

Memori OpenClaw adalah **file Markdown biasa di workspace agen**. File-file ini adalah sumber kebenaran (source of truth); model hanya "mengingat" apa yang tertulis di disk.

Alat pencarian memori disediakan oleh plugin memori yang aktif (default: `memory-core`). Nonaktifkan plugin memori dengan `plugins.slots.memory = "none"`.

## File Memori (Markdown)

Tata letak workspace default menggunakan dua lapisan memori:

- `memory/YYYY-MM-DD.md`
  - Log harian (hanya untuk penambahan/append-only).
  - Dibaca untuk hari ini + kemarin saat sesi dimulai.
- `MEMORY.md` (opsional)
  - Memori jangka panjang yang dikurasi.
  - **Hanya dimuat dalam sesi utama dan pribadi** (tidak pernah dalam konteks grup).

File-file ini berada di bawah workspace (`agents.defaults.workspace`, default `~/.openclaw/workspace`). Lihat [Workspace Agen](/id-ID/concepts/agent-workspace) untuk tata letak lengkapnya.

## Kapan Harus Menulis Memori

- Keputusan, preferensi, dan fakta yang tahan lama masuk ke `MEMORY.md`.
- Catatan harian dan konteks yang sedang berjalan masuk ke `memory/YYYY-MM-DD.md`.
- Jika seseorang berkata "ingat ini," tuliskan (jangan hanya menyimpannya di RAM).
- Area ini masih terus berkembang. Membantu untuk mengingatkan model agar menyimpan memori; model akan tahu apa yang harus dilakukan.
- Jika Anda ingin sesuatu tetap diingat, **minta bot untuk menulisnya** ke dalam memori.

## Flush Memori Otomatis (Ping Pra-pemadatan)

Ketika sebuah sesi **mendekati pemadatan otomatis (auto-compaction)**, OpenClaw memicu **giliran agen yang senyap (silent turn)** yang mengingatkan model untuk menulis memori yang tahan lama **sebelum** konteks dipadatkan. Prompt default secara eksplisit mengatakan model *boleh menjawab*, tetapi biasanya `NO_REPLY` adalah respons yang benar agar pengguna tidak pernah melihat giliran ini.

Hal ini dikendalikan oleh `agents.defaults.compaction.memoryFlush`:

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Detail:

- **Ambang batas lunak (Soft threshold)**: flush dipicu saat estimasi token sesi melewati `contextWindow - reserveTokensFloor - softThresholdTokens`.
- **Senyap** secara default: prompt menyertakan `NO_REPLY` sehingga tidak ada balasan yang dikirim.
- **Dua prompt**: prompt pengguna ditambah prompt sistem yang ditambahkan sebagai pengingat.
- **Satu kali flush per siklus pemadatan** (dilacak di `sessions.json`).
- **Workspace harus dapat ditulis**: jika sesi berjalan di sandbox dengan `workspaceAccess: "ro"` atau `"none"`, flush akan dilewati.

Untuk siklus hidup pemadatan lengkap, lihat [Manajemen Sesi + Pemadatan](/id-ID/reference/session-management-compaction).

## Pencarian Memori Vektor (Vector Memory Search)

OpenClaw dapat membuat indeks vektor kecil atas `MEMORY.md` dan `memory/*.md` sehingga kueri semantik dapat menemukan catatan terkait bahkan jika kata-katanya berbeda.

Default:

- Diaktifkan secara default.
- Memantau perubahan file memori (dengan debouncing).
- Konfigurasi pencarian memori di bawah `agents.defaults.memorySearch` (bukan `memorySearch` tingkat atas).
- Menggunakan embedding jarak jauh (remote) secara default. Jika `memorySearch.provider` tidak setel, OpenClaw akan memilih otomatis:
  1. `local` jika `memorySearch.local.modelPath` dikonfigurasi dan filenya ada.
  2. `openai` jika kunci OpenAI tersedia.
  3. `gemini` jika kunci Gemini tersedia.
  4. `voyage` jika kunci Voyage tersedia.
  5. Jika tidak, pencarian memori tetap dinonaktifkan sampai dikonfigurasi.
- Mode lokal menggunakan node-llama-cpp dan mungkin memerlukan `pnpm approve-builds`.
- Menggunakan sqlite-vec (jika tersedia) untuk mempercepat pencarian vektor di dalam SQLite.

Embedding jarak jauh **memerlukan** kunci API untuk penyedia embedding. OpenClaw mengambil kunci dari profil autentikasi, `models.providers.*.apiKey`, atau variabel lingkungan. Codex OAuth hanya mencakup obrolan/penyelesaian (chat/completions) dan **tidak** mencukupi untuk embedding untuk pencarian memori. Untuk Gemini, gunakan `GEMINI_API_KEY` atau `models.providers.google.apiKey`. Untuk Voyage, gunakan `VOYAGE_API_KEY` atau `models.providers.voyage.apiKey`. Saat menggunakan endpoint kustom yang kompatibel dengan OpenAI, setel `memorySearch.remote.apiKey` (dan opsional `memorySearch.remote.headers`).

### Backend QMD (Eksperimental)

Setel `memory.backend = "qmd"` untuk mengganti pengindeks SQLite bawaan dengan [QMD](https://github.com/tobi/qmd): sebuah sidecar pencarian lokal-terlebih-dahulu yang menggabungkan BM25 + vektor + pemeringkatan ulang (reranking). Markdown tetap menjadi sumber kebenaran; OpenClaw memanggil QMD untuk pengambilan data. Poin utama:

**Prasyarat**

- Dinonaktifkan secara default. Pilih per konfigurasi (`memory.backend = "qmd"`).
- Instal CLI QMD secara terpisah (`bun install -g https://github.com/tobi/qmd` atau ambil dari rilis resmi) dan pastikan biner `qmd` ada di `PATH` gateway.
- QMD memerlukan build SQLite yang memungkinkan ekstensi (`brew install sqlite` di macOS).
- QMD berjalan sepenuhnya secara lokal melalui Bun + `node-llama-cpp` dan mengunduh model GGUF secara otomatis dari HuggingFace pada penggunaan pertama (tidak memerlukan daemon Ollama terpisah).
- Gateway menjalankan QMD dalam XDG home yang mandiri di bawah `~/.openclaw/agents/<agentId>/qmd/` dengan menyetel `XDG_CONFIG_HOME` dan `XDG_CACHE_HOME`.
- Dukungan OS: macOS dan Linux bekerja langsung setelah Bun + SQLite terinstal. Windows paling baik didukung melalui WSL2.

**Cara Sidecar Berjalan**

- Gateway menulis XDG home QMD yang mandiri di bawah `~/.openclaw/agents/<agentId>/qmd/` (konfigurasi + cache + DB sqlite).
- Koleksi dibuat melalui `qmd collection add` dari `memory.qmd.paths` (ditambah file memori workspace default), kemudian `qmd update` + `qmd embed` dijalankan saat boot dan pada interval yang dapat dikonfigurasi (`memory.qmd.update.interval`, default 5 menit).
- Gateway sekarang menginisialisasi manajer QMD saat startup, sehingga timer pembaruan periodik sudah disiapkan bahkan sebelum panggilan `memory_search` pertama.
- Penyegaran saat boot sekarang berjalan di latar belakang secara default agar startup obrolan tidak terhambat; setel `memory.qmd.update.waitForBootSync = true` jika ingin mempertahankan perilaku pemblokiran sebelumnya.
- Pencarian dijalankan melalui `memory.qmd.searchMode` (default `qmd search --json`; juga mendukung `vsearch` dan `query`). Jika mode yang dipilih menolak flag pada build QMD Anda, OpenClaw akan mencoba ulang dengan `qmd query`. Jika QMD gagal atau binernya hilang, OpenClaw secara otomatis beralih kembali ke manajer SQLite bawaan agar alat memori tetap berfungsi.
- OpenClaw saat ini tidak mengekspos pengaturan ukuran batch embedding QMD; perilaku batch dikendalikan oleh QMD itu sendiri.
- **Pencarian pertama mungkin lambat**: QMD mungkin mengunduh model GGUF lokal (reranker/ekspansi kueri) pada saat `qmd query` pertama kali dijalankan.
  - OpenClaw menyetel `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` secara otomatis saat menjalankan QMD.
  - Jika Anda ingin mengunduh model secara manual terlebih dahulu (dan memanaskan indeks yang sama dengan yang digunakan OpenClaw), jalankan kueri sekali jalan dengan direktori XDG agen.

    Status QMD OpenClaw berada di bawah direktori status Anda (default `~/.openclaw`). Anda dapat mengarahkan `qmd` ke indeks yang sama persis dengan mengekspor variabel XDG yang sama:

    ```bash
    # Pilih direktori status yang sama dengan yang digunakan OpenClaw
    STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"

    export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
    export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"

    # (Opsional) paksa penyegaran indeks + embedding
    qmd update
    qmd embed

    # Panaskan / pemicu unduhan model pertama kali
    qmd query "test" -c memory-root --json >/dev/null 2>&1
    ```

**Konfigurasi (`memory.qmd.*`)**

- `command` (default `qmd`): menimpa jalur eksekutabel.
- `searchMode` (default `search`): memilih perintah QMD yang mendukung `memory_search` (`search`, `vsearch`, `query`).
- `includeDefaultMemory` (default `true`): mengindeks otomatis `MEMORY.md` + `memory/**/*.md`.
- `paths[]`: menambahkan direktori/file tambahan (`path`, opsional `pattern`, opsional `name` yang stabil).
- `sessions`: memilih pengindeksan sesi JSONL (`enabled`, `retentionDays`, `exportDir`).
- `update`: mengontrol ritme penyegaran dan eksekusi pemeliharaan: (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`, `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`).
- `limits`: membatasi payload pemanggilan (`maxResults`, `maxSnippetChars`, `maxInjectedChars`, `timeoutMs`).
- `scope`: skema yang sama dengan [`session.sendPolicy`](/id-ID/gateway/configuration#sessions). Defaultnya adalah hanya DM (`deny` semua, `allow` obrolan langsung); longgarkan untuk menampilkan hasil QMD di grup/saluran.
  - `match.keyPrefix` mencocokkan kunci sesi yang **dinormalisasi** (huruf kecil, dengan awalan `agent:<id>:` yang dihapus). Contoh: `discord:channel:`.
  - `match.rawKeyPrefix` mencocokkan kunci sesi **asli** (huruf kecil), termasuk `agent:<id>:`. Contoh: `agent:main:discord:`.
  - Legacy: `match.keyPrefix: "agent:..."` masih diperlakukan sebagai awalan kunci asli, tetapi lebih disarankan menggunakan `rawKeyPrefix` untuk kejelasan.
- Ketika `scope` menolak pencarian, OpenClaw mencatat peringatan dengan `channel`/`chatType` terkait agar hasil kosong lebih mudah didebug.
- Potongan teks (snippets) yang bersumber dari luar workspace akan muncul sebagai `qmd/<collection>/<relative-path>` dalam hasil `memory_search`; `memory_get` memahami awalan tersebut dan membaca dari akar koleksi QMD yang dikonfigurasi.
- Saat `memory.qmd.sessions.enabled = true`, OpenClaw mengekspor transkrip sesi yang telah dibersihkan (User/Assistant turns) ke dalam koleksi QMD khusus di bawah `~/.openclaw/agents/<id>/qmd/sessions/`, sehingga `memory_search` dapat memanggil kembali percakapan terbaru tanpa menyentuh indeks SQLite bawaan.
- Cuplikan `memory_search` sekarang menyertakan catatan kaki `Source: <path#line>` jika `memory.citations` diatur ke `auto`/`on`; setel `memory.citations = "off"` untuk menjaga metadata jalur tetap internal (agen tetap menerima jalur untuk `memory_get`, tetapi teks cuplikan tidak menyertakan catatan kaki dan prompt sistem memperingatkan agen untuk tidak mensitasinya).

**Contoh**

```json5
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // Awalan kunci sesi dinormalisasi (menghapus `agent:<id>:`).
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // Awalan kunci sesi asli (termasuk `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**Sitasi & Fallback**

- `memory.citations` berlaku apa pun backend-nya (`auto`/`on`/`off`).
- Saat `qmd` berjalan, kami menandai `status().backend = "qmd"` sehingga diagnostik menunjukkan mesin mana yang melayani hasil tersebut. Jika subproses QMD keluar atau output JSON tidak dapat diurai, manajer pencarian mencatat peringatan dan kembali ke penyedia bawaan (embedding Markdown yang ada) sampai QMD pulih.

### Jalur Memori Tambahan

Jika Anda ingin mengindeks file Markdown di luar tata letak workspace default, tambahkan jalur eksplisit:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Catatan:

- Jalur dapat berupa absolut atau relatif terhadap workspace.
- Direktori dipindai secara rekursif untuk file `.md`.
- Hanya file Markdown yang diindeks.
- Symlink diabaikan (baik untuk file maupun direktori).

### Embedding Gemini (Asli)

Setel penyedia ke `gemini` untuk menggunakan API embedding Gemini secara langsung:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Catatan:

- `remote.baseUrl` opsional (default ke base URL API Gemini).
- `remote.headers` memungkinkan Anda menambahkan header tambahan jika diperlukan.
- Model default: `gemini-embedding-001`.

Jika Anda ingin menggunakan **endpoint kustom yang kompatibel dengan OpenAI** (OpenRouter, vLLM, atau proxy), Anda dapat menggunakan konfigurasi `remote` dengan penyedia OpenAI:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Jika Anda tidak ingin menyetel kunci API, gunakan `memorySearch.provider = "local"` atau setel `memorySearch.fallback = "none"`.

Fallback:

- `memorySearch.fallback` dapat berupa `openai`, `gemini`, `local`, atau `none`.
- Penyedia fallback hanya digunakan ketika penyedia embedding utama gagal.

Pengindeksan Batch (OpenAI + Gemini + Voyage):

- Dinonaktifkan secara default. Setel `agents.defaults.memorySearch.remote.batch.enabled = true` untuk mengaktifkan pengindeksan korpus besar (OpenAI, Gemini, dan Voyage).
- Perilaku default akan menunggu penyelesaian batch; atur `remote.batch.wait`, `remote.batch.pollIntervalMs`, dan `remote.batch.timeoutMinutes` jika diperlukan.
- Setel `remote.batch.concurrency` untuk mengontrol berapa banyak pekerjaan batch yang dikirim secara paralel (default: 2).
- Mode batch berlaku saat `memorySearch.provider = "openai"` atau `"gemini"` dan menggunakan kunci API yang sesuai.
- Pekerjaan batch Gemini menggunakan endpoint batch embedding asinkron dan memerlukan ketersediaan Gemini Batch API.

Mengapa batch OpenAI cepat + murah:

- Untuk pengisian data besar (backfills), OpenAI biasanya merupakan pilihan tercepat yang kami dukung karena kami dapat mengirimkan banyak permintaan embedding dalam satu pekerjaan batch dan membiarkan OpenAI memprosesnya secara asinkron.
- OpenAI menawarkan harga diskon untuk beban kerja Batch API, sehingga proses pengindeksan besar biasanya lebih murah daripada mengirim permintaan yang sama secara sinkron.
- Lihat dokumentasi dan harga OpenAI Batch API untuk lebih jelasnya:
  - [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
  - [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

Contoh konfigurasi:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Alat:

- `memory_search` — mengembalikan cuplikan dengan file + rentang baris.
- `memory_get` — membaca konten file memori berdasarkan jalur.

Mode Lokal:

- Setel `agents.defaults.memorySearch.provider = "local"`.
- Berikan `agents.defaults.memorySearch.local.modelPath` (GGUF atau URI `hf:`).
- Opsional: setel `agents.defaults.memorySearch.fallback = "none"` untuk menghindari fallback ke jarak jauh.

### Bagaimana Alat Memori Bekerja

- `memory_search` mencari secara semantik dalam potongan-potongan Markdown (target ~400 token, tumpang tindih 80 token) dari `MEMORY.md` + `memory/**/*.md`. Ia mengembalikan teks cuplikan (dibatasi ~700 karakter), jalur file, rentang baris, skor, penyedia/model, dan apakah beralih ke embedding jarak jauh (local → remote). Seluruh payload file tidak dikembalikan.
- `memory_get` membaca file Markdown memori tertentu (relatif terhadap workspace), secara opsional dari baris awal dan untuk sejumlah N baris. Jalur di luar `MEMORY.md` / `memory/` akan ditolak.
- Kedua alat tersebut hanya diaktifkan ketika `memorySearch.enabled` bernilai true untuk agen tersebut.

### Apa yang Diindeks (dan Kapan)

- Tipe file: hanya Markdown (`MEMORY.md`, `memory/**/*.md`).
- Penyimpanan indeks: SQLite per agen di `~/.openclaw/memory/<agentId>.sqlite` (dapat dikonfigurasi melalui `agents.defaults.memorySearch.store.path`, mendukung token `{agentId}`).
- Kesegaran data: pemantau (watcher) pada `MEMORY.md` + `memory/` menandai indeks sebagai kotor (debounce 1.5 detik). Sinkronisasi dijadwalkan saat sesi dimulai, saat pencarian, atau pada interval tertentu dan berjalan secara asinkron. Transkrip sesi menggunakan ambang batas delta untuk memicu sinkronisasi latar belakang.
- Pemicu pengindeksan ulang: indeks menyimpan **penyedia/model embedding + sidik jari endpoint + parameter pemotongan (chunking)**. Jika salah satu di antaranya berubah, OpenClaw akan mereset dan mengindeks ulang seluruh penyimpanan secara otomatis.

### Pencarian Hibrida (BM25 + Vektor)

Saat diaktifkan, OpenClaw menggabungkan:

- **Kemiripan vektor** (kecocokan semantik, kata-kata boleh berbeda)
- **Relevansi kata kunci BM25** (token eksak seperti ID, variabel lingkungan, simbol kode)

Jika pencarian teks lengkap (full-text search) tidak tersedia di platform Anda, OpenClaw akan beralih ke pencarian vektor saja.

#### Mengapa Hibrida?

Pencarian vektor sangat bagus untuk "ini artinya sama":

- "Mac Studio gateway host" vs "mesin yang menjalankan gateway"
- "debounce pembaruan file" vs "menghindari pengindeksan di setiap penulisan"

Tetapi pencarian vektor bisa lemah pada token eksak dengan sinyal tinggi:

- ID (`a828e60`, `b3b9895a…`)
- Simbol kode (`memorySearch.query.hybrid`)
- String kesalahan ("sqlite-vec unavailable")

BM25 (teks lengkap) adalah kebalikannya: kuat pada token eksak, lebih lemah pada parafrasa.
Pencarian hibrida adalah jalan tengah yang pragmatis: **gunakan kedua sinyal pengambilan** sehingga Anda mendapatkan hasil yang baik untuk kueri "bahasa alami" dan kueri "jarum dalam tumpukan jerami".

#### Bagaimana Kami Menggabungkan Hasil (Desain Saat Ini)

Sketsa implementasi:

1. Ambil sekumpulan kandidat dari kedua sisi:

- **Vektor**: top `maxResults * candidateMultiplier` berdasarkan kemiripan kosinus.
- **BM25**: top `maxResults * candidateMultiplier` berdasarkan peringkat BM25 FTS5 (semakin rendah semakin baik).

2. Ubah peringkat BM25 menjadi skor berkisar 0..1-ish:

- `textScore = 1 / (1 + max(0, bm25Rank))`

3. Gabungkan kandidat berdasarkan ID potongan (chunk ID) dan hitung skor tertimbang:

- `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Catatan:

- `vectorWeight` + `textWeight` dinormalisasi menjadi 1.0 dalam resolusi konfigurasi, sehingga bobot berperilaku sebagai persentase.
- Jika embedding tidak tersedia (atau penyedia mengembalikan vektor nol), kami tetap menjalankan BM25 dan mengembalikan kecocokan kata kunci.
- Jika FTS5 tidak dapat dibuat, kami tetap menggunakan pencarian vektor saja (tidak ada kesalahan fatal).

Ini bukan "kesempurnaan teori IR", tetapi sederhana, cepat, dan cenderung meningkatkan pemanggilan (recall)/presisi pada catatan nyata. Jika ingin lebih canggih nanti, langkah umum berikutnya adalah Reciprocal Rank Fusion (RRF) atau normalisasi skor (min/max atau z-score) sebelum pencampuran.

#### Alur Pascaproses (Post-processing pipeline)

Setelah menggabungkan skor vektor dan kata kunci, dua tahap pascaproses opsional menyempurnakan daftar hasil sebelum mencapai agen:

```
Vektor + Kata Kunci → Penggabungan Tertimbang → Peluruhan Temporal → Urutkan → MMR → Hasil Top-K
```

Kedua tahap tersebut **nonaktif secara default** dan dapat diaktifkan secara terpisah.

#### Pemeringkatan Ulang MMR (Diversitas)

Ketika pencarian hibrida mengembalikan hasil, beberapa potongan mungkin berisi konten yang serupa atau tumpang tindih. Sebagai contoh, mencari "pengaturan jaringan rumah" mungkin mengembalikan lima potongan yang hampir identik dari catatan harian yang berbeda yang semuanya menyebutkan konfigurasi router yang sama.

**MMR (Maximal Marginal Relevance)** mengatur ulang peringkat hasil untuk menyeimbangkan relevansi dengan keberagaman (diversitas), memastikan hasil teratas mencakup berbagai aspek kueri alih-alih mengulangi informasi yang sama.

Cara kerjanya:

1. Hasil diberi skor berdasarkan relevansi asli mereka (skor tertimbang vektor + BM25).
2. MMR secara iteratif memilih hasil yang memaksimalkan: `λ × relevansi − (1−λ) × kemiripan_maksimum_terhadap_yang_terpilih`.
3. Kemiripan antar hasil diukur menggunakan kemiripan teks Jaccard pada konten yang telah ditokenisasi.

Parameter `lambda` mengontrol pertukaran (trade-off) tersebut:

- `lambda = 1.0` → murni relevansi (tanpa penalti keberagaman)
- `lambda = 0.0` → keberagaman maksimum (mengabaikan relevansi)
- Default: `0.7` (seimbang, sedikit miring ke arah relevansi)

**Contoh — kueri: "pengaturan jaringan rumah"**

Diberikan file memori berikut:

```
memory/2026-02-10.md  → "Mengonfigurasi router Omada, set VLAN 10 untuk perangkat IoT"
memory/2026-02-08.md  → "Mengonfigurasi router Omada, memindahkan IoT ke VLAN 10"
memory/2026-02-05.md  → "Menyiapkan DNS AdGuard di 192.168.10.2"
memory/network.md     → "Router: Omada ER605, AdGuard: 192.168.10.2, VLAN 10: IoT"
```

Tanpa MMR — 3 hasil teratas:

```
1. memory/2026-02-10.md  (skor: 0.92)  ← router + VLAN
2. memory/2026-02-08.md  (skor: 0.89)  ← router + VLAN (hampir duplikat!)
3. memory/network.md     (skor: 0.85)  ← dokumen referensi
```

Dengan MMR (λ=0.7) — 3 hasil teratas:

```
1. memory/2026-02-10.md  (skor: 0.92)  ← router + VLAN
2. memory/network.md     (skor: 0.85)  ← dokumen referensi (beragam!)
3. memory/2026-02-05.md  (skor: 0.78)  ← DNS AdGuard (beragam!)
```

Hasil yang hampir duplikat dari 8 Februari dibuang, dan agen mendapatkan tiga informasi yang berbeda.

**Kapan Mengaktifkan:** Jika Anda melihat `memory_search` mengembalikan cuplikan yang redundan atau hampir duplikat, terutama dengan catatan harian yang sering mengulang informasi serupa di berbagai hari.

#### Peluruhan Temporal (Peningkatan Kebaruan)

Agen dengan catatan harian mengumpulkan ratusan file bertanggal seiring berjalannya waktu. Tanpa peluruhan, catatan yang ditulis dengan baik dari enam bulan lalu bisa mengalahkan pembaruan kemarin tentang topik yang sama.

**Peluruhan Temporal (Temporal decay)** menerapkan pengali eksponensial pada skor berdasarkan usia setiap hasil, sehingga memori terbaru secara alami menempati peringkat lebih tinggi sementara yang lama memudar:

```
skor_meluruh = skor × e^(-λ × usiaDalamHari)
```

di mana `λ = ln(2) / waktuParuhHari`.

Dengan waktu paruh (half-life) default 30 hari:

- Catatan hari ini: **100%** dari skor asli
- 7 hari yang lalu: **~84%**
- 30 hari yang lalu: **50%**
- 90 hari yang lalu: **12.5%**
- 180 hari yang lalu: **~1.6%**

**File Evergreen tidak pernah mengalami peluruhan:**

- `MEMORY.md` (file memori akar)
- File yang tidak bertanggal di `memory/` (misalnya, `memory/projects.md`, `memory/network.md`)
- File-file ini berisi informasi referensi tahan lama yang harus selalu menempati peringkat secara normal.

**File harian bertanggal** (`memory/YYYY-MM-DD.md`) menggunakan tanggal yang diekstrak dari nama file. Sumber lain (misalnya, transkrip sesi) menggunakan waktu modifikasi file (`mtime`).

**Contoh — kueri: "apa jadwal kerja Rod?"**

Diberikan file memori berikut (hari ini adalah 10 Februari):

```
memory/2025-09-15.md  → "Rod bekerja Sen-Jum, standup jam 10 pagi, pairing jam 2 siang"  (berusia 148 hari)
memory/2026-02-10.md  → "Rod ada standup jam 14:15, 1:1 dengan Zeb jam 14:45"           (hari ini)
memory/2026-02-03.md  → "Rod memulai tim baru, standup pindah ke 14:15"               (berusia 7 hari)
```

Tanpa peluruhan:

```
1. memory/2025-09-15.md  (skor: 0.91)  ← kecocokan semantik terbaik, tapi basi!
2. memory/2026-02-10.md  (skor: 0.82)
3. memory/2026-02-03.md  (skor: 0.80)
```

Dengan peluruhan (waktuParuh=30):

```
1. memory/2026-02-10.md  (skor: 0.82 × 1.00 = 0.82)  ← hari ini, tanpa peluruhan
2. memory/2026-02-03.md  (skor: 0.80 × 0.85 = 0.68)  ← 7 hari, peluruhan ringan
3. memory/2025-09-15.md  (skor: 0.91 × 0.03 = 0.03)  ← 148 hari, hampir hilang
```

Catatan bulan September yang sudah basi turun ke posisi terbawah meskipun memiliki kecocokan semantik mentah yang terbaik.

**Kapan Mengaktifkan:** Jika agen Anda memiliki catatan harian selama berbulan-bulan dan Anda menemukan bahwa informasi lama yang sudah basi mengalahkan konteks terbaru. Waktu paruh 30 hari bekerja dengan baik untuk alur kerja yang padat catatan harian; tingkatkan (misalnya, 90 hari) jika Anda sering mereferensikan catatan yang lebih lama.

#### Konfigurasi

Kedua fitur tersebut dikonfigurasi di bawah `memorySearch.query.hybrid`:

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4,
          // Keberagaman: mengurangi hasil yang redundan
          mmr: {
            enabled: true,    // default: false
            lambda: 0.7       // 0 = keberagaman maks, 1 = relevansi maks
          },
          // Kebaruan: meningkatkan memori yang lebih baru
          temporalDecay: {
            enabled: true,    // default: false
            halfLifeDays: 30  // skor berkurang separuh setiap 30 hari
          }
        }
      }
    }
  }
}
```

Anda dapat mengaktifkan salah satu fitur secara independen:

- **MMR saja** — berguna saat Anda memiliki banyak catatan serupa tetapi usia tidak menjadi masalah.
- **Peluruhan temporal saja** — berguna saat kebaruan penting tetapi hasil Anda sudah cukup beragam.
- **Keduanya** — direkomendasikan untuk agen dengan riwayat catatan harian yang besar dan berjalan lama.

### Cache Embedding

OpenClaw dapat mencache **embedding potongan (chunk embeddings)** di SQLite sehingga pengindeksan ulang dan pembaruan yang sering (terutama transkrip sesi) tidak perlu mengenkripsi ulang teks yang tidak berubah.

Konfigurasi:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

### Pencarian Memori Sesi (Eksperimental)

Anda secara opsional dapat mengindeks **transkrip sesi** dan menampilkannya melalui `memory_search`. Fitur ini dibatasi di belakang flag eksperimental.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Catatan:

- Pengindeksan sesi bersifat **opt-in** (nonaktif secara default).
- Pembaruan sesi mengalami debouncing dan **diindeks secara asinkron** setelah melewati ambang batas delta (upaya terbaik/best-effort).
- `memory_search` tidak pernah memblokir proses pengindeksan; hasil mungkin sedikit basi sampai sinkronisasi latar belakang selesai.
- Hasil tetap mencakup cuplikan saja; `memory_get` tetap terbatas pada file memori.
- Pengindeksan sesi diisolasi per agen (hanya log sesi agen tersebut yang diindeks).
- Log sesi berada di disk (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Setiap proses/pengguna dengan akses sistem file dapat membacanya, jadi perlakukan akses disk sebagai batas kepercayaan. Untuk isolasi yang lebih ketat, jalankan agen di bawah pengguna OS atau host yang terpisah.

Ambang Batas Delta (nilai default ditampilkan):

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // jumlah baris JSONL
        }
      }
    }
  }
}
```

### Akselerasi Vektor SQLite (sqlite-vec)

Ketika ekstensi sqlite-vec tersedia, OpenClaw menyimpan embedding dalam tabel virtual SQLite (`vec0`) dan melakukan kueri jarak vektor di dalam database. Hal ini menjaga pencarian tetap cepat tanpa memuat setiap embedding ke dalam JS.

Konfigurasi (opsional):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Catatan:

- `enabled` default ke true; saat dinonaktifkan, pencarian beralih ke kemiripan kosinus dalam proses (in-process) atas embedding yang disimpan.
- Jika ekstensi sqlite-vec hilang atau gagal dimuat, OpenClaw mencatat kesalahan dan melanjutkan dengan fallback JS (tanpa tabel vektor).
- `extensionPath` menimpa jalur sqlite-vec bawaan (berguna untuk build kustom atau lokasi instalasi non-standar).

### Unduh Otomatis Embedding Lokal

- Model embedding lokal default: `hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF/embeddinggemma-300m-qat-Q8_0.gguf` (~0.6 GB).
- Ketika `memorySearch.provider = "local"`, `node-llama-cpp` menentukan `modelPath`; jika GGUF-nya tidak ada, ia akan **mengunduh otomatis** ke cache (atau `local.modelCacheDir` jika disetel), lalu memuatnya. Unduhan akan dilanjutkan jika diulang.
- Persyaratan build asli: jalankan `pnpm approve-builds`, pilih `node-llama-cpp`, lalu `pnpm rebuild node-llama-cpp`.
- Fallback: jika pengaturan lokal gagal dan `memorySearch.fallback = "openai"`, kami secara otomatis beralih ke embedding jarak jauh (`openai/text-embedding-3-small` kecuali jika ditimpa) dan mencatat alasannya.

### Contoh Endpoint Kustom yang Kompatibel dengan OpenAI

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Catatan:

- `remote.*` diprioritaskan di atas `models.providers.openai.*`.
- `remote.headers` digabungkan dengan header OpenAI; kunci dari remote menang jika terjadi konflik. Kosongkan `remote.headers` untuk menggunakan default OpenAI.
