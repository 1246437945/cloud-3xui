[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Crea un nodo 3x-ui (VLESS + WebSocket + TLS) con i server gratuiti di GitHub Actions, dominio fisso, riavvio automatico.

## Come funziona

Ogni esecuzione installa automaticamente il pannello 3x-ui su un server Ubuntu di GitHub, crea un inbound VLESS+WS tramite le panel API, poi espone pannello e nodo tramite un **named tunnel** Cloudflare (dominio fisso). Ogni esecuzione dura circa 5 ore, prima della scadenza avvia automaticamente la successiva, l'indirizzo del nodo non cambia mai.

## Indirizzi fissi

- Nodo: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Pannello: `https://panel.my-github.ccwu.cc/panel` (nome utente `admin`)

## Configurazione iniziale

### 1. Named tunnel Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Dagli un nome e copia il token (la stringa che inizia con `eyJ`)
3. Nei Public hostnames del tunnel aggiungi due righe:
   - `vless.tuodominio.com` → HTTP → `127.0.0.1:8080`
   - `panel.tuodominio.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, aggiungi questi tre:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token del tunnel Cloudflare |
| `PANEL_PASSWORD` | Password di accesso al pannello |
| `VLESS_UUID` | UUID del nodo (fisso, valido per sempre nel client, generalo su https://1024tools.com/uuid/, **Assicurati di salvare questo UUID, non verrà mostrato in chiaro nei log, ti servirà ogni volta che componi il link del nodo**) |

### 3. Esecuzione

Actions → 3x-ui VLESS Node → Run workflow (i parametri possono restare vuoti, usa automaticamente i secret).

Il log mostrerà il link `vless://` completo, importalo in v2rayN / v2rayNG / Streisand / NekoBox.

## Riavvio automatico

- Ogni esecuzione dura `run_minutes` minuti (default 290), alla fine avvia automaticamente la successiva
- Dominio e UUID sono fissi, il client si configura una volta sola
- Per fermare, vai su Actions e premi Cancel, dopo l'annullamento non si riavvia più

## Note

- **Velocità**: il traffico passa per Cloudflare → server GitHub (USA/UE), adatto solo a un uso temporaneo/leggero
- **Breve interruzione**: a ogni riavvio automatico (ogni 5 ore) c'è un'interruzione di circa 3-5 minuti (avvio nuova macchina, installazione 3x-ui, riconnessione tunnel), il client si riconnette da solo
- **Conformità**: eseguire un proxy su GitHub Actions rischia di violare i ToS con possibile ban dell'account, usalo a tuo rischio
- **Non** pubblicare `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` in luoghi pubblici
