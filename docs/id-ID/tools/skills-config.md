---
summary: "Skema konfigurasi skill dan contohnya"
read_when:
  - Menambah atau memodifikasi konfigurasi skill
  - Menyesuaikan daftar izinkan bawaan atau perilaku instalasi
title: "Konfigurasi Skill"
---

# Konfigurasi Skill

Semua konfigurasi yang terkait dengan skill berada di bawah objek `skills` di dalam file `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "KUNCI_GEMINI_DI_SINI",
        env: {
          GEMINI_API_KEY: "KUNCI_GEMINI_DI_SINI",
        },
      },
      peekaboo: { enabled: true },
    },
  },
}
```

## Bidang (Fields)

- `allowBundled`: daftar izinkan opsional khusus untuk skill **bawaan (bundled)**. Jika diatur, hanya skill bawaan dalam daftar ini yang akan aktif.
- `load.extraDirs`: direktori skill tambahan untuk dipindai (prioritas terendah).
- `load.watch`: pantau folder skill dan perbarui daftar skill jika ada perubahan (default: true).
- `load.watchDebounceMs`: waktu tunggu (debounce) untuk kejadian pemantauan skill dalam milidetik (default: 250).
- `install.preferBrew`: prioritaskan penginstal brew jika tersedia (default: true).
- `install.nodeManager`: preferensi penginstal node (`npm` | `pnpm` | `yarn` | `bun`).
- `entries.<skillKey>`: override per skill.

## Bidang Per Skill

- `enabled`: setel ke `false` untuk menonaktifkan skill meskipun sudah bawaan atau terinstal.
- `env`: variabel lingkungan yang dimasukkan saat agen berjalan (hanya jika belum disetel).
- `apiKey`: kemudahan untuk skill yang memerlukan satu variabel lingkungan utama.

## Catatan

- Kunci di bawah `entries` secara default dipetakan ke nama skill. Jika skill mendefinisikan `metadata.openclaw.skillKey`, gunakan kunci tersebut sebagai gantinya.
- Perubahan pada skill akan dideteksi pada putaran agen berikutnya jika pemantau (watcher) diaktifkan.

### Skill dalam Sandbox + Variabel Lingkungan

Ketika sesi berada dalam **sandbox**, proses skill berjalan di dalam Docker. Sandbox **tidak** mewarisi `process.env` dari host. Gunakan `agents.defaults.sandbox.docker.env` untuk menyertakan variabel lingkungan ke dalam sandbox.
 Global `env` dan `skills.entries.<skill>.env/apiKey` hanya berlaku untuk eksekusi di **host**.


