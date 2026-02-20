---
summary: "Pemecahan masalah saluran (channel) dengan gejala kegagalan per saluran dan perbaikannya"
read_when:
  - Saluran terlihat terhubung tetapi balasan gagal terkirim
title: "Troubleshooting Saluran"
---

# Pemecahan Masalah Saluran

Gunakan halaman ini jika saluran terlihat terhubung tetapi tidak berfungsi sebagaimana mestinya.

## Urutan Perintah Diagnosa

Jalankan perintah ini secara berurutan:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Kondisi sehat adalah:
- `Runtime: running`
- `RPC probe: ok`
- Status saluran menunjukkan `connected/ready`

## Gejala Kegagalan dan Solusi

### 🟢 WhatsApp
| Gejala                          | Cek Tercepat                        | Solusi                                               |
| :------------------------------ | :---------------------------------- | :--------------------------------------------------- |
| Terhubung tapi DM tidak dibalas | `openclaw pairing list whatsapp`    | Setujui pengirim atau ubah kebijakan DM.             |
| Pesan grup diabaikan            | Cek `requireMention` di konfigurasi | Sebut bot (@mention) atau longgarkan kebijakan grup. |
| Logout/Disconnect berulang      | `openclaw channels status --probe`  | Login ulang dan cek folder `credentials`.            |

### 🔵 Telegram
| Gejala                          | Cek Tercepat                     | Solusi                                                    |
| :------------------------------ | :------------------------------- | :-------------------------------------------------------- |
| `/start` tapi tidak ada balasan | `openclaw pairing list telegram` | Setujui pairing atau ubah kebijakan DM.                   |
| Bot online tapi grup diam       | Cek Mode Privasi di BotFather    | Matikan Mode Privasi atau sebut bot (@mention).           |
| Gagal kirim pesan (jaringan)    | Cek log dan periksa koneksi      | Perbaiki DNS atau pengaturan proxy ke `api.telegram.org`. |

### 🟣 Discord
| Gejala                                  | Cek Tercepat                       | Solusi                                                 |
| :-------------------------------------- | :--------------------------------- | :----------------------------------------------------- |
| Bot online tapi tidak balas chat server | `openclaw channels status --probe` | Izinkan ID Server/Saluran dan aktifkan Message Intent. |
| Pesan grup diabaikan                    | Cek log untuk filter sebutan       | Sebut bot atau setel `requireMention: false`.          |
| DM tidak masuk                          | `openclaw pairing list discord`    | Setujui pairing DM atau sesuaikan kebijakan DM.        |

### 🟡 Slack
| Gejala                             | Cek Tercepat                       | Solusi                                           |
| :--------------------------------- | :--------------------------------- | :----------------------------------------------- |
| Socket Mode aktif tapi tidak balas | `openclaw channels status --probe` | Verifikasi token bot/aplikasi dan izin (scopes). |
| DM diblokir                        | `openclaw pairing list slack`      | Setujui pairing atau longgarkan kebijakan DM.    |
| Pesan saluran diabaikan            | Cek `groupPolicy` dan allowlist    | Masukkan ID saluran ke daftar izinkan.           |

## Terkait
- [Troubleshooting Gateway](/id-ID/gateway/troubleshooting)
- [Doctor](/id-ID/gateway/doctor)
- [Status](/id-ID/cli/status)


