[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Bouw een 3x-ui node (VLESS + WebSocket + TLS) met gratis GitHub Actions-servers, vast domein, automatische herstart.

## Werking

Elke run installeert automatisch het 3x-ui panel op een GitHub Ubuntu-server, maakt via de panel API een VLESS+WS inbound aan, en stelt panel en node beschikbaar via een Cloudflare **named tunnel** (vast domein). Elke run duurt ongeveer 5 uur, voor het einde wordt automatisch de volgende run gestart, het node-adres verandert nooit.

## Vaste adressen

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Paneel: `https://panel.my-github.ccwu.cc/panel` (gebruikersnaam `admin`)

## Eerste configuratie

### 1. Cloudflare named tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Geef een naam en kopieer de token (de string die begint met `eyJ`)
3. Voeg in Public hostnames van de tunnel twee regels toe:
   - `vless.jouwdomein.com` → HTTP → `127.0.0.1:8080`
   - `panel.jouwdomein.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, voeg deze drie toe:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel-token |
| `PANEL_PASSWORD` | Inlogwachtwoord paneel |
| `VLESS_UUID` | Node-UUID (vast, permanent geldig in de client, genereer op https://1024tools.com/uuid/) |

### 3. Uitvoeren

Actions → 3x-ui VLESS Node → Run workflow (parameters mogen leeg blijven, gebruikt automatisch de secrets).

Het log toont de volledige `vless://`-link, importeer deze in v2rayN / v2rayNG / Streisand / NekoBox.

## Automatische herstart

- Elke run duurt `run_minutes` minuten (standaard 290), aan het einde wordt automatisch de volgende gestart
- Domein en UUID zijn vast, de client hoeft maar één keer te worden geconfigureerd
- Om te stoppen ga je naar Actions en klik je op Cancel, na annuleren start hij niet meer opnieuw

## Opmerkingen

- **Snelheid**: verkeer gaat via Cloudflare → GitHub-servers (VS/EU), alleen geschikt voor tijdelijk/licht gebruik
- **Korte onderbreking**: bij elke automatische herstart (elke 5 uur) is er ongeveer 3-5 minuten onderbreking (nieuwe machine start op, 3x-ui wordt geïnstalleerd, tunnel verbindt opnieuw), de client verbindt automatisch opnieuw
- **Beleid**: een proxy draaien op GitHub Actions kan de ToS schenden met risico op een accountban, gebruik op eigen risico
- Publiceer `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` **niet** op openbare plekken
