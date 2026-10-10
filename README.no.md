[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Bygg en 3x-ui-node (VLESS + WebSocket + TLS) med gratis GitHub Actions-servere, fast domene, automatisk omstart.

## Slik fungerer det

Hver kjøring installerer automatisk 3x-ui-panelet på en GitHub Ubuntu-server, oppretter en VLESS+WS-inbound via panel-API-et, og eksponerer panelet og noden via en Cloudflare **named tunnel** (fast domene). Hver kjøring varer omtrent 5 timer, før den utløper starter den automatisk neste kjøring, nodeadressen endres aldri.

## Faste adresser

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (brukernavn `admin`)

## Første gangs oppsett

### 1. Cloudflare named tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Gi den et navn og kopier tokenet (strengen som begynner med `eyJ`)
3. I tunnelens Public hostnames legger du til to linjer:
   - `vless.dittdomene.com` → HTTP → `127.0.0.1:8080`
   - `panel.dittdomene.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, legg til disse tre:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel-token |
| `PANEL_PASSWORD` | Passord for panelet |
| `VLESS_UUID` | Node-UUID (fast, permanent gyldig i klienten, generer på https://1024tools.com/uuid/) |

### 3. Kjøring

Actions → 3x-ui VLESS Node → Run workflow (parametere kan stå tomme, bruker automatisk secrets).

Loggen viser den fullstendige `vless://`-lenken, importer den i v2rayN / v2rayNG / Streisand / NekoBox.

## Automatisk omstart

- Hver kjøring varer `run_minutes` minutter (standard 290), på slutten starter den automatisk neste
- Domene og UUID er faste, klienten konfigureres én gang for alle
- For å stoppe, gå til Actions og trykk Cancel, etter avbrudd starter den ikke på nytt

## Merknader

- **Hastighet**: trafikken går via Cloudflare → GitHub-servere (USA/EU), kun egnet for midlertidig/lett bruk
- **Kort avbrudd**: ved hver automatiske omstart (hver 5. time) er det ca. 3-5 minutters avbrudd (ny maskin starter, 3x-ui installeres, tunnel kobler til på nytt), klienten kobler til igjen automatisk
- **Regelverk**: å kjøre proxy på GitHub Actions kan bryte ToS med risiko for utestengelse av kontoen, bruk på eget ansvar
- **Ikke** publiser `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` på offentlige steder
