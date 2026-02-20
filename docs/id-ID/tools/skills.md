---
summary: "Skill: terkelola vs workspace, aturan gating, dan pengkabelan konfigurasi/lingkungan"
read_when:
  - Menambah atau memodifikasi skill
  - Mengubah penyaringan (gating) skill atau aturan pemuatan
title: "Skill"
---

# Skill (OpenClaw)

OpenClaw menggunakan folder skill yang **kompatibel dengan [AgentSkills](https://agentskills.io)** untuk mengajarkan agen cara menggunakan alat. Setiap skill adalah direktori yang berisi file `SKILL.md` dengan instruksi dan metadata YAML. OpenClaw memuat **skill bawaan** serta override lokal opsional, dan memfilternya saat dimuat berdasarkan lingkungan, konfigurasi, dan keberadaan biner.

## Lokasi dan Prioritas

Skill dimuat dari **tiga** tempat:

1. **Skill bawaan (Bundled)**: disertakan dalam instalasi (paket npm atau aplikasi OpenClaw).
2. **Skill terkelola/lokal (Managed)**: `~/.openclaw/skills`.
3. **Skill workspace**: `<workspace>/skills`.

Jika terjadi konflik nama skill, urutan prioritasnya adalah:

`<workspace>/skills` (tertinggi) → `~/.openclaw/skills` → skill bawaan (terendah).

## Skill Per Agen vs Skill Bersama

Dalam pengaturan **multi-agen**, setiap agen memiliki workspace-nya sendiri:

- **Skill per agen** berada di `<workspace>/skills` hanya untuk agen tersebut.
- **Skill bersama** berada di `~/.openclaw/skills` dan dapat dilihat oleh **semua agen** di mesin yang sama.

## ClawHub (Instalasi + Sinkronisasi)

ClawHub adalah registri skill publik untuk OpenClaw. Jelajahi di [https://clawhub.com](https://clawhub.com). Gunakan ini untuk menemukan, menginstal, memperbarui, dan mencadangkan skill.
Panduan lengkap: [ClawHub](/id-ID/tools/clawhub).

## Format (Kompatibel dengan AgentSkills)

`SKILL.md` setidaknya harus menyertakan:

```markdown
---
name: nama-skill
description: Penjelasan singkat tentang fungsi skill ini
---
```

Beberapa opsi di frontmatter:
- `user-invocable`: `true|false` (default: `true`). Jika `true`, skill diekspos sebagai perintah slash.
- `command-dispatch`: `tool`. Jika disetel, perintah slash akan langsung memanggil alat tanpa melibatkan model LLM.

## Penyaringan (Gating - Filter Waktu Muat)

OpenClaw memfilter skill saat dimuat menggunakan `metadata`:

```markdown
---
name: contoh-skill
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["git"], "env": ["GITHUB_TOKEN"] }
      }
  }
---
```

Bidang dalam `metadata.openclaw`:
- `requires.bins`: daftar biner yang harus ada di `PATH`.
- `requires.env`: variabel lingkungan yang harus ada.
- `requires.config`: jalur konfigurasi di `openclaw.json` yang harus bernilai true.

## Override Konfigurasi (`~/.openclaw/openclaw.json`)

Skill bawaan/terkelola dapat diaktifkan/dinonaktifkan dan diberikan nilai lingkungan melalui konfigurasi:

```json5
{
  skills: {
    entries: {
      "nama-skill": {
        enabled: true,
        apiKey: "KUNCI_API_DI_SINI",
        env: {
          VAR_LINGKUNGAN: "NILAI"
        }
      }
    }
  }
}
```

## Dampak Token

Skill yang dikirim ke prompt sistem memakan biaya token. OpenClaw mengirimkan daftar XML singkat untuk meminimalkan biaya ini. Secara kasar, satu skill membutuhkan sekitar 24 token ditambah panjang deskripsi dan nama yang Anda berikan.

## Referensi Konfigurasi

Lihat [Konfigurasi skill](/id-ID/tools/skills-config) untuk skema konfigurasi lengkap.
 JELAJAHI skill lainnya di [https://clawhub.com](https://clawhub.com).


