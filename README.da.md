[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Byg en 3x-ui node (VLESS + WebSocket + TLS) på GitHub Actions gratis servere, med fast domæne og automatisk genstart.

## Sådan virker det

Hver kørsel installerer automatisk 3x-ui panelet på GitHubs Ubuntu-server, opretter en VLESS+WS inbound via panel API og eksponerer panelet og noden via en Cloudflare **navngivet tunnel** (fast domæne). Hver kørsel varer ca. 5 timer og udløser automatisk den næste kørsel inden afslutning, så nodeadressen aldrig ændres.

## Faste adresser

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (brugernavn `admin`)

## Første opsætning

### 1. Cloudflare navngivet tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Navngiv tunnelen og kopier token (strengen der starter med `eyJ`)
3. Tilføj to public hostnames i tunnelen:
   - `vless.dit-domæne.com` → HTTP → `127.0.0.1:8080`
   - `panel.dit-domæne.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, tilføj tre:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel token |
| `PANEL_PASSWORD` | Panel login adgangskode |
| `VLESS_UUID` | Node UUID (fast, gyldig permanent i klienten, generer en selv på https://1024tools.com/uuid/, **Husk at gemme dette UUID — det vises ikke i klartekst i loggene, og du skal bruge det hver gang du samler node-linket**) |

### 3. Kør

Actions → 3x-ui VLESS Node → Run workflow (parametre kan være tomme, secrets bruges automatisk).

Loggen udskriver det komplette `vless://` link, som kan importeres i v2rayN / v2rayNG / Streisand / NekoBox.

## Automatisk genstart

- Hver kørsel varer `run_minutes` minutter (standard 290) og udløser automatisk den næste ved afslutning
- Domæne og UUID er faste, klientkonfigurationen er permanent gyldig
- For at stoppe: Cancel i Actions, annullerede kørsler genstarter ikke automatisk

## Bemærkninger

- **Hastighed**: Trafik går via Cloudflare → GitHub datacentre (USA/EU), kun egnet til midlertidig/let brug
- **Kort afbrydelse**: Ca. 3-5 minutters nedetid ved hver automatisk genstart (ny maskine starter, 3x-ui installeres, tunnel genopretter), klienten genopretter automatisk
- **Overholdelse**: Kørsel af proxy på GitHub Actions kan overtræde ToS med risiko for kontosuspension, vurder selv
- **Del ikke** `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` offentligt
