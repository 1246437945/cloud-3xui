[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Rakenna 3x-ui node (VLESS + WebSocket + TLS) ilmaisilla GitHub Actions -palvelimilla, kiinteällä verkkotunnuksella ja automaattisella uudelleenkäynnistyksellä.

## Toimintaperiaate

Jokainen ajo asentaa automaattisesti 3x-ui paneelin GitHubin Ubuntu-palvelimelle, luo VLESS+WS inboundin paneelin API:n kautta ja julkaisee paneelin ja noden Cloudflaren **nimetyn tunnelin** (kiinteä verkkotunnus) kautta. Jokainen ajo kestää noin 5 tuntia ja käynnistää automaattisesti seuraavan ajon ennen päättymistä, joten noden osoite ei koskaan muutu.

## Kiinteät osoitteet

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Paneeli: `https://panel.my-github.ccwu.cc/panel` (käyttäjänimi `admin`)

## Ensiasennus

### 1. Cloudflaren nimetty tunneli

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Nimeä tunneli ja kopioi token (merkkijono joka alkaa `eyJ`)
3. Lisää tunneliin kaksi public hostnamea:
   - `vless.oma-domain.com` → HTTP → `127.0.0.1:8080`
   - `panel.oma-domain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, lisää kolme:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflaren tunnelin token |
| `PANEL_PASSWORD` | Paneelin kirjautumissalasana |
| `VLESS_UUID` | Noden UUID (kiinteä, pysyvästi voimassa asiakasohjelmassa, generoi itse osoitteessa https://1024tools.com/uuid/) |

### 3. Ajo

Actions → 3x-ui VLESS Node → Run workflow (parametrit voivat olla tyhjiä, secretit käytetään automaattisesti).

Loki tulostaa täydellisen `vless://` linkin, joka voidaan tuoda v2rayN / v2rayNG / Streisand / NekoBox -ohjelmiin.

## Automaattinen uudelleenkäynnistys

- Jokainen ajo kestää `run_minutes` minuuttia (oletus 290) ja käynnistää automaattisesti seuraavan päättyessään
- Verkkotunnus ja UUID ovat kiinteitä, asiakasohjelman asetus on pysyvästi voimassa
- Pysäyttäminen: paina Cancel Actionsissa, peruutetut ajot eivät käynnisty automaattisesti uudelleen

## Huomautukset

- **Nopeus**: Liikenne kulkee Cloudflaren → GitHubin datakeskusten (USA/EU) kautta, sopii vain tilapäiseen/kevyeen käyttöön
- **Lyhyt katkos**: Noin 3-5 minuutin katkos jokaisen automaattisen uudelleenkäynnistyksen yhteydessä (uusi kone käynnistyy, 3x-ui asennetaan, tunneli yhdistää uudelleen), asiakasohjelma yhdistää automaattisesti uudelleen
- **Sääntöjenmukaisuus**: Välityspalvelimen ajaminen GitHub Actionsissa voi rikkoa käyttöehtoja ja johtaa tilin jäädytykseen, arvioi itse
- **Älä** jaa `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` julkisesti
