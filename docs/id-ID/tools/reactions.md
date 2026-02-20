---
summary: "Semantik reaksi yang dibagikan ke seluruh saluran"
read_when:
  - Bekerja dengan reaksi di saluran mana pun
title: "Reaksi (Reactions)"
---

# Alat Reaksi (Reaction Tooling)

Semantik reaksi yang dibagikan ke seluruh saluran:

- `emoji` wajib diisi saat menambahkan reaksi.
- `emoji=""` menghapus reaksi bot jika didukung oleh saluran tersebut.
- `remove: true` menghapus emoji yang ditentukan (memerlukan `emoji`).

## Catatan Per Saluran

- **Discord/Slack**: `emoji` yang kosong menghapus semua reaksi bot pada pesan; `remove: true` hanya menghapus emoji tersebut.
- **Google Chat**: `emoji` yang kosong menghapus semua reaksi aplikasi pada pesan; `remove: true` hanya menghapus emoji tersebut.
- **Telegram**: `emoji` yang kosong menghapus reaksi bot; `remove: true` juga menghapus reaksi tetapi tetap memerlukan `emoji` yang tidak kosong untuk validasi alat.
- **WhatsApp**: `emoji` yang kosong menghapus reaksi bot; `remove: true` dipetakan ke emoji kosong (tetap memerlukan `emoji`).
- **Signal**: Notifikasi reaksi masuk akan menghasilkan kejadian sistem jika `channels.signal.reactionNotifications` diaktifkan.



