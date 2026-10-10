[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Postavte uzel 3x-ui (VLESS + WebSocket + TLS) na bezplatných serverech GitHub Actions, s pevnou doménou a automatickým obnovováním.

## Jak to funguje

Každé spuštění automaticky nainstaluje panel 3x-ui na server GitHub Ubuntu, vytvoří VLESS+WS inbound přes API panelu a zpřístupní panel i uzel přes pojmenovaný tunel Cloudflare (pevná doména). Každé spuštění trvá asi 5 hodin a před koncem automaticky spustí další, takže se adresa uzlu nikdy nemění.

## Pevné adresy

- Uzel: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (uživatel `admin`)

## První nastavení

### 1. Pojmenovaný tunel Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Pojmenujte ho a zkopírujte token (dlouhý řetězec začínající `eyJ`)
3. Přidejte dva záznamy v Public hostnames tunelu:
   - `vless.vasedomena.com` → HTTP → `127.0.0.1:8080`
   - `panel.vasedomena.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Nastavení repozitáře Settings → Secrets and variables → Actions, přidejte tyto tři:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token tunelu Cloudflare |
| `PANEL_PASSWORD` | Heslo do panelu |
| `VLESS_UUID` | UUID uzlu (pevné, pro klienty trvale platné; vygenerujte na https://1024tools.com/uuid/, **Tento UUID si nezapomeňte poznamenat — v protokolech se nezobrazí v otevřené podobě a budete ho potřebovat při každém sestavování odkazu uzlu**) |

### 3. Spuštění

Actions → 3x-ui VLESS Node → Run workflow (parametry jsou volitelné, secrets se použijí automaticky).

Log vypíše úplný odkaz `vless://`; importujte ho do v2rayN / v2rayNG / Streisand / NekoBox.

## Automatické obnovování

- Každé spuštění trvá `run_minutes` minut (výchozí 290) a na konci automaticky spustí další
- Doména i UUID jsou pevné; klienta nastavte jednou a funguje napořád
- Pro zastavení klikněte v Actions na Cancel; samo se nerestartuje

## Poznámky

- **Rychlost**: provoz jde přes Cloudflare → datacentra GitHub (USA/EU); vhodné jen pro dočasné/lehké použití
- **Krátký výpadek**: při každém předání automatického restartu výpadek asi 3–5 minut (start nového stroje, instalace 3x-ui, znovupřipojení tunelu); klienti se připojí znovu automaticky
- **Soulad s pravidly**: provoz proxy na GitHub Actions může porušovat ToS a hrozí zablokování účtu; posuďte sami
- **Nezveřejňujte** `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` na veřejných místech
