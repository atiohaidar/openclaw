---
summary: "Terapkan patch multi-file dengan alat apply_patch"
read_when:
  - Anda perlu melakukan pengeditan file terstruktur pada beberapa file sekaligus
  - Anda ingin mendokumentasikan atau menelusuri pengeditan berbasis patch
title: "Alat apply_patch"
---

# Alat apply_patch

Terapkan perubahan file menggunakan format patch terstruktur. Alat ini sangat ideal untuk pengeditan banyak file atau banyak bagian (multi-hunk) di mana pemanggilan satu `edit` saja akan terasa rapuh.

Alat ini menerima satu string `input` yang membungkus satu atau lebih operasi file:

```
*** Begin Patch
*** Add File: jalur/ke/file.txt
+baris 1
+baris 2
*** Update File: src/app.ts
@@
-baris lama
+baris baru
*** Delete File: file_usang.txt
*** End Patch
```

## Parameter

- `input` (wajib): Konten patch lengkap termasuk `*** Begin Patch` dan `*** End Patch`.

## Catatan

- Jalur patch mendukung jalur relatif (dari direktori workspace) dan jalur absolut.
- Secara default, tindakan ini dibatasi di dalam direktori workspace (`workspaceOnly: true`).
- Gunakan `*** Move to:` di dalam bagian `*** Update File:` untuk mengubah nama file.
- Fitur ini bersifat eksperimental dan dinonaktifkan secara default. Aktifkan dengan `tools.exec.applyPatch.enabled`.
- Hanya tersedia untuk model OpenAI (termasuk OpenAI Codex).

## Contoh

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```



