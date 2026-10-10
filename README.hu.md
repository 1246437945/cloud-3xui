[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

3x-ui csomópont építése ingyenes GitHub Actions szerverekkel (VLESS + WebSocket + TLS), fix domainnel, automatikus újraindítással.

## Működési elv

Minden futtatás automatikusan telepíti a 3x-ui panelt egy GitHub Ubuntu szerverre, a panel API-n keresztül létrehoz egy VLESS+WS inboundot, majd Cloudflare **névvel ellátott tunnellel** (fix domain) teszi elérhetővé a panelt és a csomópontot. Minden futtatás kb. 5 órán át tart, lejárat előtt automatikusan elindítja a következőt, a csomópont címe sosem változik.

## Fix címek

- Csomópont: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (felhasználónév: `admin`)

## Első beállítás

### 1. Cloudflare névvel ellátott tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Nevezd el, majd másold ki a tokent (az `eyJ`-vel kezdődő karaktersor)
3. A tunnel Public hostnames részében adj hozzá két sort:
   - `vless.sajatdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.sajatdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, add hozzá ezt a hármat:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel token |
| `PANEL_PASSWORD` | Panel belépési jelszó |
| `VLESS_UUID` | Csomópont UUID (fix, a kliensben örökre érvényes, generálj egyet: https://1024tools.com/uuid/, **Jegyezd meg ezt az UUID-t, nem jelenik meg nyíltan a naplókban, minden alkalommal szükséged lesz rá a csomópont-link összeállításához**) |

### 3. Futtatás

Actions → 3x-ui VLESS Node → Run workflow (a paraméterek üresen hagyhatók, automatikusan a secretet használja).

A logban megjelenik a teljes `vless://` link, importáld v2rayN / v2rayNG / Streisand / NekoBox kliensbe.

## Automatikus újraindítás

- Minden futtatás `run_minutes` percig tart (alapértelmezett 290), a végén automatikusan elindítja a következőt
- A domain és az UUID fix, a klienst elég egyszer beállítani
- Leállításhoz menj az Actions oldalra és nyomj Cancel-t, a megszakítás után nem indul újra

## Megjegyzések

- **Sebesség**: a forgalom Cloudflare → GitHub szervereken (USA/EU) keresztül halad, csak alkalmi/könnyű használatra alkalmas
- **Rövid kiesés**: az 5 óránkénti automatikus újraindításkor kb. 3-5 perc kiesés van (új gép indul, 3x-ui települ, tunnel újracsatlakozik), a kliens automatikusan újracsatlakozik
- **Szabályzat**: proxy futtatása GitHub Actionsön sértheti a ToS-t, fiókletiltás kockázatával jár, saját felelősségre használd
- **Ne** tedd közzé a `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` értékeket nyilvános helyen
