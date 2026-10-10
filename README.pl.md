[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Zbuduj węzeł 3x-ui (VLESS + WebSocket + TLS) na darmowych serwerach GitHub Actions, ze stałą nazwą domeny i automatycznym odnawianiem.

## Jak to działa

Każde uruchomienie automatycznie instaluje panel 3x-ui na serwerze Ubuntu w GitHub, tworzy inbound VLESS+WS przez API panelu, a następnie udostępnia panel i węzeł przez nazwany tunel Cloudflare (stała nazwa domeny). Każde uruchomienie trwa około 5 godzin, a przed zakończeniem automatycznie uruchamia następne — adres węzła nigdy się nie zmienia.

## Stałe adresy

- Węzeł: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (nazwa użytkownika `admin`)

## Konfiguracja początkowa

### 1. Nazwany tunel Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Po nadaniu nazwy skopiuj token (ciąg zaczynający się od `eyJ`)
3. W zakładce Public hostnames tunelu dodaj dwa wpisy:
   - `vless.twoja-domena.com` → HTTP → `127.0.0.1:8080`
   - `panel.twoja-domena.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

W repozytorium: Settings → Secrets and variables → Actions, dodaj trzy:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token tunelu Cloudflare |
| `PANEL_PASSWORD` | Hasło logowania do panelu |
| `VLESS_UUID` | UUID węzła (stały, ważny na stałe w kliencie, wygeneruj na https://1024tools.com/uuid/) |

### 3. Uruchomienie

Actions → 3x-ui VLESS Node → Run workflow (parametry opcjonalne, automatycznie użyje secretów).

W logach znajdziesz pełny link `vless://`, zaimportuj go do v2rayN / v2rayNG / Streisand / NekoBox.

## Automatyczne odnawianie

- Każde uruchomienie trwa `run_minutes` minut (domyślnie 290), na końcu automatycznie uruchamia następne
- Domena i UUID są stałe, konfiguracja klienta raz na zawsze
- Aby zatrzymać, kliknij Cancel w Actions — po anulowaniu nie uruchomi się ponownie

## Uwagi

- **Prędkość**: ruch idzie przez Cloudflare → centra danych GitHub (USA/Europa), nadaje się tylko do tymczasowego / lekkiego użytku
- **Krótkie przerwy**: przy automatycznym restarcie co 5 godzin występuje przerwa ok. 3-5 minut (start nowej maszyny, instalacja 3x-ui, ponowne łączenie tunelu), klient automatycznie połączy się ponownie
- **Zgodność**: uruchamianie proxy w GitHub Actions może naruszać ToS i grozić blokadą konta, oceń ryzyko samodzielnie
- **Nie** publikuj `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` w miejscach publicznych
