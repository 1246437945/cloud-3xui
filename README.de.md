[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Baue einen 3x-ui Node (VLESS + WebSocket + TLS) auf kostenlosen GitHub Actions Servern, mit fester Domain und automatischem Neustart.

## Funktionsweise

Jeder Lauf installiert automatisch das 3x-ui Panel auf GitHubs Ubuntu-Server, erstellt einen VLESS+WS Inbound über die Panel API und stellt Panel und Node über einen Cloudflare **benannten Tunnel** (feste Domain) bereit. Jeder Lauf dauert ca. 5 Stunden und löst vor dem Ende automatisch den nächsten Lauf aus, sodass sich die Node-Adresse nie ändert.

## Feste Adressen

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (Benutzername `admin`)

## Ersteinrichtung

### 1. Cloudflare benannter Tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Tunnel benennen und Token kopieren (die Zeichenkette beginnend mit `eyJ`)
3. Zwei Public Hostnames im Tunnel hinzufügen:
   - `vless.deine-domain.com` → HTTP → `127.0.0.1:8080`
   - `panel.deine-domain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, drei hinzufügen:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare Tunnel Token |
| `PANEL_PASSWORD` | Panel Login-Passwort |
| `VLESS_UUID` | Node UUID (fest, dauerhaft gültig im Client, selbst generieren auf https://1024tools.com/uuid/) |

### 3. Ausführen

Actions → 3x-ui VLESS Node → Run workflow (Parameter können leer bleiben, Secrets werden automatisch verwendet).

Das Log gibt den vollständigen `vless://` Link aus, der in v2rayN / v2rayNG / Streisand / NekoBox importiert werden kann.

## Automatischer Neustart

- Jeder Lauf dauert `run_minutes` Minuten (Standard 290) und löst am Ende automatisch den nächsten aus
- Domain und UUID sind fest, die Client-Konfiguration bleibt dauerhaft gültig
- Zum Stoppen: In Actions auf Cancel klicken, abgebrochene Läufe starten nicht automatisch neu

## Hinweise

- **Geschwindigkeit**: Traffic läuft über Cloudflare → GitHub Rechenzentren (USA/EU), nur für temporäre/leichte Nutzung geeignet
- **Kurze Unterbrechung**: Ca. 3-5 Minuten Ausfallzeit bei jedem automatischen Neustart (neue Maschine startet, 3x-ui wird installiert, Tunnel verbindet neu), der Client verbindet automatisch neu
- **Compliance**: Der Betrieb eines Proxys auf GitHub Actions kann gegen die ToS verstoßen mit Risiko einer Kontosperrung, bitte selbst bewerten
- **Nicht** `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` öffentlich teilen
