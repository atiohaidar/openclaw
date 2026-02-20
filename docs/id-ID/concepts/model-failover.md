---
summary: "Bagaimana OpenClaw melakukan rotasi profil autentikasi dan fallback antar model"
read_when:
  - Mendiagnosis rotasi profil autentikasi, masa tenang (cooldown), atau perilaku fallback model
  - Memperbarui aturan failover untuk profil autentikasi atau model
title: "Failover Model"
---

# Failover model

OpenClaw menangani kegagalan dalam dua tahap:

1.  **Rotasi profil autentikasi** di dalam penyedia saat ini.
2.  **Fallback model** ke model berikutnya dalam daftar `agents.defaults.model.fallbacks`.

Dokumen ini menjelaskan aturan saat aplikasi berjalan (runtime) dan data pendukungnya.

## Penyimpanan Autentikasi (Kunci + OAuth)

OpenClaw menggunakan **profil autentikasi** baik untuk kunci API maupun token OAuth.

- Rahasia (secrets) disimpan di `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (lama: `~/.openclaw/agent/auth-profiles.json`).
- Konfigurasi `auth.profiles` / `auth.order` hanya berisi **metadata + perutean** (tidak ada rahasia).
- File OAuth lama (hanya untuk impor): `~/.openclaw/credentials/oauth.json` (diimpor ke `auth-profiles.json` saat penggunaan pertama).

Detail selengkapnya: [/id-ID/concepts/oauth](/id-ID/concepts/oauth)

Tipe kredensial:

- `type: "api_key"` → `{ provider, key }`
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` untuk beberapa penyedia).

## ID Profil

Login OAuth membuat profil yang berbeda sehingga beberapa akun dapat berdampingan.

- Default: `provider:default` jika email tidak tersedia.
- OAuth dengan email: `provider:<email>` (contoh: `google-antigravity:user@gmail.com`).

Profil berada di dalam `auth-profiles.json` di bawah bagian `profiles`.

## Urutan Rotasi

Bila penyedia memiliki beberapa profil, OpenClaw memilih urutan sebagai berikut:

1.  **Konfigurasi eksplisit**: `auth.order[provider]` (jika diatur).
2.  **Profil yang dikonfigurasi**: `auth.profiles` yang difilter berdasarkan penyedia.
3.  **Profil yang disimpan**: entri dalam `auth-profiles.json` untuk penyedia tersebut.

Jika tidak ada urutan eksplisit yang dikonfigurasi, OpenClaw menggunakan urutan round-robin:

- **Kunci utama**: tipe profil (**OAuth sebelum kunci API**).
- **Kunci sekunder**: `usageStats.lastUsed` (paling lama digunakan terlebih dahulu, di dalam setiap tipe).
- **Profil dalam masa tenang/dinonaktifkan** akan dipindahkan ke akhir, diurutkan berdasarkan waktu kedaluwarsa terdekat.

### Stickiness Sesi (Ramah Cache)

OpenClaw **memasangkan profil autentikasi yang dipilih per sesi** untuk menjaga cache penyedia tetap hangat. Ia **tidak** melakukan rotasi pada setiap permintaan. Profil yang dipasangkan akan digunakan kembali sampai:

- Sesi direset (`/new` / `/reset`).
- Pemadatan (compaction) selesai (jumlah pemadatan bertambah).
- Profil berada dalam masa tenang (cooldown) atau dinonaktifkan.

Pemilihan manual melalui `/model …@<profileId>` akan menetapkan **penimpa pengguna (user override)** untuk sesi tersebut dan tidak akan dirotasi secara otomatis sampai sesi baru dimulai.

Profil yang dipasangkan otomatis (dipilih oleh router sesi) diperlakukan sebagai **preferensi**: profil tersebut dicoba terlebih dahulu, tetapi OpenClaw dapat berotasi ke profil lain jika terjadi batas tingkat (rate limit) atau waktu habis (timeout). Profil yang dipasangkan oleh pengguna tetap terkunci pada profil tersebut; jika gagal dan fallback model sudah dikonfigurasi, OpenClaw akan beralih ke model berikutnya alih-alih beralih profil.

### Mengapa OAuth Bisa "Tampak Hilang"

Jika Anda memiliki profil OAuth dan profil kunci API untuk penyedia yang sama, round-robin dapat berpindah di antara keduanya antar pesan kecuali jika dipasangkan (pinned). Untuk memaksa penggunaan satu profil:

- Pasangkan dengan `auth.order[provider] = ["provider:profileId"]`, atau
- Gunakan penimpa per sesi melalui `/model …` dengan penimpa profil (jika didukung oleh UI/permukaan obrolan Anda).

## Masa Tenang (Cooldowns)

Bila sebuah profil gagal karena kesalahan autentikasi/batas tingkat (atau timeout yang tampak seperti pembatasan tingkat), OpenClaw akan menandainya dalam masa tenang dan beralih ke profil berikutnya. Kesalahan format/permintaan tidak valid (misalnya kegagalan validasi ID pemanggilan alat Cloud Code Assist) juga dianggap layak untuk failover dan menggunakan masa tenang yang sama.

Masa tenang menggunakan backoff eksponensial:

- 1 menit
- 5 menit
- 25 menit
- 1 jam (batas maksimal)

Status disimpan di `auth-profiles.json` di bawah `usageStats`:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## Penonaktifan Tagihan (Billing)

Kegagalan penagihan/kredit (misal: "kredit tidak mencukupi" / "saldo kredit terlalu rendah") dianggap layak untuk failover, tetapi biasanya tidak bersifat sementara. Alih-alih masa tenang singkat, OpenClaw menandai profil tersebut sebagai **dinonaktifkan** (dengan masa tunggu lebih lama) dan berotasi ke profil/penyedia berikutnya.

Status disimpan di `auth-profiles.json`:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Default:

- Backoff tagihan dimulai dari **5 jam**, berlipat ganda per kegagalan tagihan, dan dibatasi pada **24 jam**.
- Penghitung backoff direset jika profil tidak gagal selama **24 jam** (dapat dikonfigurasi).

## Fallback Model

Jika semua profil untuk suatu penyedia gagal, OpenClaw beralih ke model berikutnya dalam `agents.defaults.model.fallbacks`. Aturan ini berlaku untuk kegagalan autentikasi, pembatasan tingkat, dan timeout yang telah menghabiskan rotasi profil (kesalahan lain tidak memicu fallback).

Bila sebuah proses berjalan dengan penimpa model (lewat hook atau CLI), fallback tetap akan berakhir di `agents.defaults.model.primary` setelah mencoba semua fallback yang dikonfigurasi.

## Konfigurasi Terkait

Lihat [Konfigurasi Gateway](/id-ID/gateway/configuration) untuk:

- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- Perutean `agents.defaults.imageModel`

Lihat [Model](/id-ID/concepts/models) untuk ringkasan pemilihan model dan fallback yang lebih luas.
