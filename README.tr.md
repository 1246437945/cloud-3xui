[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

GitHub Actions ücretsiz sunucularında 3x-ui düğümü (VLESS + WebSocket + TLS) kurun; sabit alan adı ve otomatik yenileme ile.

## Nasıl çalışır

Her çalıştırma, GitHub Ubuntu sunucusuna 3x-ui panelini otomatik kurar, panel API'si ile VLESS+WS inbound oluşturur ve ardından paneli ve düğümü Cloudflare adlandırılmış tünel (sabit alan adı) ile dışa açar. Her çalıştırma yaklaşık 5 saat sürer ve bitmeden otomatik olarak bir sonrakini tetikler — düğüm adresi asla değişmez.

## Sabit adresler

- Düğüm: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (kullanıcı adı `admin`)

## İlk kurulum

### 1. Cloudflare adlandırılmış tünel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Adlandırdıktan sonra token'ı kopyalayın (`eyJ` ile başlayan dize)
3. Tünelin Public hostnames sekmesinde iki kayıt ekleyin:
   - `vless.alanadiniz.com` → HTTP → `127.0.0.1:8080`
   - `panel.alanadiniz.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Depoda: Settings → Secrets and variables → Actions, üç tane ekleyin:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tünel token'ı |
| `PANEL_PASSWORD` | Panel giriş şifresi |
| `VLESS_UUID` | Düğüm UUID'si (sabit, istemcide kalıcı geçerli, https://1024tools.com/uuid/ adresinden üretin, **Bu UUID'yi mutlaka kaydedin, loglarda açık metin olarak görünmez, düğüm bağlantısını her birleştirdiğinizde buna ihtiyacınız olacak**) |

### 3. Çalıştırma

Actions → 3x-ui VLESS Node → Run workflow (parametreler isteğe bağlı, secret'leri otomatik kullanır).

Loglarda tam `vless://` bağlantısını bulup v2rayN / v2rayNG / Streisand / NekoBox içine aktarın.

## Otomatik yenileme

- Her çalıştırma `run_minutes` dakika sürer (varsayılan 290), bitiminde otomatik olarak bir sonrakini tetikler
- Alan adı ve UUID sabittir, istemciyi bir kez yapılandırmanız yeterli
- Durdurmak için Actions'ta Cancel'e basın — iptalden sonra kendiliğinden yeniden başlamaz

## Notlar

- **Hız**: trafik Cloudflare → GitHub veri merkezleri (ABD/Avrupa) üzerinden geçer, yalnızca geçici / hafif kullanım için uygundur
- **Kısa kesintiler**: her 5 saatteki otomatik yeniden başlatmada yaklaşık 3-5 dakikalık kesinti olur (yeni makinenin başlaması, 3x-ui kurulumu, tünelin yeniden bağlanması), istemci otomatik yeniden bağlanır
- **Uyumluluk**: GitHub Actions'ta proxy çalıştırmak ToS'yi ihlal edebilir ve hesap yasağına yol açabilir, riski kendiniz değerlendirin
- `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` değerlerini herkese açık yerlerde **paylaşmayın**
