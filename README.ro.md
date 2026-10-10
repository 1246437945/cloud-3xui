[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Construiește un nod 3x-ui (VLESS + WebSocket + TLS) pe serverele gratuite GitHub Actions, cu domeniu fix și reînnoire automată.

## Cum funcționează

Fiecare rulare instalează automat panoul 3x-ui pe un server Ubuntu GitHub, creează un inbound VLESS+WS prin API-ul panoului, apoi expune panoul și nodul printr-un tunel denumit Cloudflare (domeniu fix). Fiecare rulare durează aproximativ 5 ore, iar înainte de final declanșează automat următoarea — adresa nodului nu se schimbă niciodată.

## Adrese fixe

- Nod: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panou: `https://panel.my-github.ccwu.cc/panel` (utilizator `admin`)

## Configurare inițială

### 1. Tunel denumit Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. După denumire, copiază token-ul (șirul care începe cu `eyJ`)
3. În fila Public hostnames a tunelului adaugă două intrări:
   - `vless.domeniul-tau.com` → HTTP → `127.0.0.1:8080`
   - `panel.domeniul-tau.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

În repository: Settings → Secrets and variables → Actions, adaugă trei:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token-ul tunelului Cloudflare |
| `PANEL_PASSWORD` | Parola de autentificare în panou |
| `VLESS_UUID` | UUID-ul nodului (fix, valabil permanent în client, generează la https://1024tools.com/uuid/, **Asigurați-vă că salvați acest UUID, nu va fi afișat în clar în jurnale, veți avea nevoie de el de fiecare dată când compuneți linkul nodului**) |

### 3. Rulare

Actions → 3x-ui VLESS Node → Run workflow (parametrii opționali, folosește automat secretele).

În loguri găsești linkul complet `vless://`, importă-l în v2rayN / v2rayNG / Streisand / NekoBox.

## Reînnoire automată

- Fiecare rulare durează `run_minutes` minute (implicit 290), la final declanșează automat următoarea
- Domeniul și UUID-ul sunt fixe, configurezi clientul o singură dată
- Pentru oprire, apasă Cancel în Actions — după anulare nu mai repornește singur

## Note

- **Viteză**: traficul trece prin Cloudflare → centrele de date GitHub (SUA/Europa), potrivit doar pentru utilizare temporară / ușoară
- **Întreruperi scurte**: la fiecare repornire automată de 5 ore apare o întrerupere de 3-5 minute (pornirea mașinii noi, instalarea 3x-ui, reconectarea tunelului), clientul se reconectează automat
- **Conformitate**: rularea unui proxy pe GitHub Actions poate încălca ToS și poate duce la blocarea contului, evaluează singur
- **Nu** publica `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` în locuri publice
