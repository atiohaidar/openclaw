---
summary: "Referensi CLI untuk `openclaw dns` (pembantu penemuan wide-area)"
read_when:
  - Anda menginginkan penemuan wide-area (DNS-SD) melalui Tailscale + CoreDNS
  - Anda sedang mengatur split DNS untuk domain penemuan kustom (seperti openclaw.internal)
title: "dns"
---

# `openclaw dns`

Pembantu DNS untuk penemuan *wide-area* (Tailscale + CoreDNS). Saat ini difokuskan pada macOS dengan Homebrew CoreDNS.

## Terkait:
- Penemuan Gateway: [Discovery](/id-ID/gateway/discovery)
- Konfigurasi wide-area: [Configuration](/id-ID/gateway/configuration)

## Penyiapan (Setup)

```bash
# Menampilkan status penyiapan DNS
openclaw dns setup

# Menerapkan konfigurasi DNS (setup & apply)
openclaw dns setup --apply
```


