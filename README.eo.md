[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Konstruu 3x-ui nodon (VLESS + WebSocket + TLS) per senpagaj GitHub Actions serviloj, kun fiksa domajno kaj aŭtomata restartigo.

## Kiel ĝi funkcias

Ĉiu rulado aŭtomate instalas la 3x-ui panelon sur la Ubuntu-servilo de GitHub, kreas VLESS+WS inbound per la panela API, kaj eksponas la panelon kaj la nodon per **nomita tunelo** de Cloudflare (fiksa domajno). Ĉiu rulado daŭras ĉ. 5 horojn kaj aŭtomate ekigas la sekvan ruladon antaŭ finiĝo, do la noda adreso neniam ŝanĝiĝas.

## Fiksaj adresoj

- Nodo: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panelo: `https://panel.my-github.ccwu.cc/panel` (uzantnomo `admin`)

## Unua agordo

### 1. Nomita tunelo de Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Nomu la tunelon kaj kopiu la token (la ĉeno komenciĝanta per `eyJ`)
3. Aldonu du public hostnames en la tunelo:
   - `vless.via-domajno.com` → HTTP → `127.0.0.1:8080`
   - `panel.via-domajno.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, aldonu tri:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Tunela token de Cloudflare |
| `PANEL_PASSWORD` | Pasvorto de panela ensaluto |
| `VLESS_UUID` | UUID de nodo (fiksa, daŭre valida en kliento, generu mem ĉe https://1024tools.com/uuid/, **Nepre memoru ĉi tiun UUID — ĝi ne aperos malkaŝe en la protokoloj, kaj vi bezonos ĝin ĉiufoje kiam vi kunmetos la nodan ligilon**) |

### 3. Rulu

Actions → 3x-ui VLESS Node → Run workflow (parametroj povas resti malplenaj, secrets uziĝas aŭtomate).

La protokolo eligas la kompletan `vless://` ligilon, importeblan en v2rayN / v2rayNG / Streisand / NekoBox.

## Aŭtomata restartigo

- Ĉiu rulado daŭras `run_minutes` minutojn (defaŭlte 290) kaj aŭtomate ekigas la sekvan ĉe fino
- Domajno kaj UUID estas fiksaj, la klienta agordo validas daŭre
- Por haltigi: alklaku Cancel en Actions, nuligitaj ruladoj ne restartiĝas aŭtomate

## Atentigoj

- **Rapido**: Trafiko iras per Cloudflare → GitHub datencentroj (Usono/EU), taŭga nur por provizora/malpeza uzo
- **Mallonga interrompo**: Ĉ. 3-5 minutoj da malfunkcio ĉe ĉiu aŭtomata restartigo (nova maŝino startas, 3x-ui instaliĝas, tunelo rekonektiĝas), la kliento rekonektiĝas aŭtomate
- **Konformeco**: Ruli prokurilon sur GitHub Actions povas malobservi la ToS kun risko de konta suspendo, taksu mem
- **Ne** dividu `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` publike
