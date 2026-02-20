---
summary: "Permukaan pelacakan penggunaan dan persyaratan kredensial"
read_when:
  - Anda sedang menghubungkan permukaan penggunaan/kuota penyedia
  - Anda perlu menjelaskan perilaku pelacakan penggunaan atau persyaratan autentikasi
title: "Pelacakan Penggunaan"
---

# Pelacakan penggunaan

## Apa itu

- Menarik data penggunaan/kuota penyedia langsung dari endpoint penggunaan mereka.
- Tidak menggunakan estimasi biaya; hanya jendela waktu yang dilaporkan oleh penyedia.

## Di Mana Ini Muncul

- `/status` dalam obrolan: kartu status kaya emoji dengan token sesi + perkiraan biaya (hanya untuk kunci API). Penggunaan penyedia ditampilkan untuk **penyedia model saat ini** jika tersedia.
- `/usage off|tokens|full` dalam obrolan: footer penggunaan per respons (OAuth hanya menampilkan token).
- `/usage cost` dalam obrolan: ringkasan biaya lokal yang dikumpulkan dari log sesi OpenClaw.
- CLI: `openclaw status --usage` mencetak rincian lengkap per penyedia.
- CLI: `openclaw channels list` mencetak snapshot penggunaan yang sama di samping konfigurasi penyedia (gunakan `--no-usage` untuk melewatkan).
- Menu bar macOS: Bagian "Usage" di bawah konteks (hanya jika tersedia).

## Penyedia + Kredensial

-   **Anthropic (Claude)**: Token OAuth dalam profil autentikasi.
-   **GitHub Copilot**: Token OAuth dalam profil autentikasi.
-   **Gemini CLI**: Token OAuth dalam profil autentikasi.
-   **Antigravity**: Token OAuth dalam profil autentikasi.
-   **OpenAI Codex**: Token OAuth dalam profil autentikasi (accountId digunakan jika ada).
-   **MiniMax**: Kunci API (kunci rencana coding; `MINIMAX_CODE_PLAN_KEY` atau `MINIMAX_API_KEY`); menggunakan jendela waktu rencana coding 5 jam.
-   **z.ai**: Kunci API via env/konfigurasi/penyimpanan autentikasi.

Data penggunaan akan disembunyikan jika tidak ada kredensial OAuth/API yang cocok.
