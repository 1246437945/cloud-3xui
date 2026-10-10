[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Build a 3x-ui node (VLESS + WebSocket + TLS) on free GitHub Actions servers, with a fixed domain and automatic renewal.

## How it works

Each run automatically installs the 3x-ui panel on a GitHub Ubuntu server, creates a VLESS+WS inbound via the panel API, then exposes the panel and node through a Cloudflare **named tunnel** (fixed domain). Each run lasts about 5 hours and automatically triggers the next run before it ends, so the node address never changes.

## Fixed addresses

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (username `admin`)

## First-time setup

### 1. Cloudflare named tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Name it, then copy the token (the long string starting with `eyJ`)
3. Add two entries under the tunnel's Public hostnames:
   - `vless.yourdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.yourdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repository Settings → Secrets and variables → Actions, add these three:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel token |
| `PANEL_PASSWORD` | Panel login password |
| `VLESS_UUID` | Node UUID (fixed, permanently valid for clients; generate one at https://1024tools.com/uuid/, **Be sure to save this UUID — it will not appear in plain text in the logs, and you need it every time you assemble the node link**) |

### 3. Run

Actions → 3x-ui VLESS Node → Run workflow (inputs are optional, secrets are used automatically).

The log will print the full `vless://` link; import it into v2rayN / v2rayNG / Streisand / NekoBox.

## Automatic renewal

- Each run lasts `run_minutes` minutes (default 290), then automatically triggers the next run
- Domain and UUID are fixed; configure the client once and it works forever
- To stop, click Cancel in Actions; it won't restart automatically

## Notes

- **Speed**: traffic goes through Cloudflare → GitHub datacenters (US/EU); suitable for temporary/light use only
- **Brief interruption**: each auto-restart handover drops for about 3-5 minutes (new machine boot, 3x-ui install, tunnel reconnect); clients will reconnect automatically
- **Compliance**: running a proxy on GitHub Actions may violate the ToS and risk account suspension; evaluate at your own risk
- **Do not** post `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` anywhere public
