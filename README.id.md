[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Membangun node 3x-ui (VLESS + WebSocket + TLS) dengan server gratis GitHub Actions, domain tetap, restart otomatis.

## Cara kerja

Setiap run otomatis menginstal panel 3x-ui di server Ubuntu GitHub, membuat inbound VLESS+WS melalui panel API, lalu mengekspos panel dan node lewat Cloudflare **named tunnel** (domain tetap). Setiap run berlangsung sekitar 5 jam, sebelum berakhir otomatis memicu run berikutnya, alamat node tidak pernah berubah.

## Alamat tetap

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (username `admin`)

## Konfigurasi awal

### 1. Cloudflare named tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Beri nama lalu salin token (string yang diawali `eyJ`)
3. Di Public hostnames tunnel tambahkan dua baris:
   - `vless.domainkamu.com` → HTTP → `127.0.0.1:8080`
   - `panel.domainkamu.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, tambahkan tiga ini:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token tunnel Cloudflare |
| `PANEL_PASSWORD` | Kata sandi login panel |
| `VLESS_UUID` | UUID node (tetap, berlaku permanen di klien, buat di https://1024tools.com/uuid/, **Pastikan untuk mencatat UUID ini, tidak akan ditampilkan secara jelas di log, Anda memerlukannya setiap kali menyusun tautan node**) |

### 3. Menjalankan

Actions → 3x-ui VLESS Node → Run workflow (parameter boleh kosong, otomatis memakai secret).

Log akan menampilkan link `vless://` lengkap, impor ke klien v2rayN / v2rayNG / Streisand / NekoBox.

## Restart otomatis

- Setiap run berlangsung `run_minutes` menit (default 290), di akhir otomatis memicu run berikutnya
- Domain dan UUID tetap, konfigurasi klien cukup sekali dan berlaku selamanya
- Untuk berhenti, buka Actions lalu klik Cancel, setelah dibatalkan tidak akan restart otomatis

## Catatan

- **Kecepatan**: traffic lewat Cloudflare → server GitHub (AS/EU), hanya cocok untuk pemakaian sementara/ringan
- **Gangguan singkat**: tiap restart otomatis 5 jam sekali terputus sekitar 3-5 menit (mesin baru booting, instalasi 3x-ui, tunnel reconnect), klien akan reconnect otomatis
- **Kepatuhan**: menjalankan proxy di GitHub Actions berisiko melanggar ToS dan banned akun, gunakan atas risiko sendiri
- **Jangan** mempublikasikan `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` di tempat umum
