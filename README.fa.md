[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

ساخت نود 3x-ui (VLESS + WebSocket + TLS) روی سرورهای رایگان GitHub Actions با دامنه ثابت و راه‌اندازی مجدد خودکار.

## نحوه کار

هر اجرا به‌صورت خودکار پنل 3x-ui را روی سرور Ubuntu گیت‌هاب نصب می‌کند، یک inbound از نوع VLESS+WS از طریق API پنل می‌سازد و پنل و نود را با **تونل نام‌دار** Cloudflare (دامنه ثابت) در دسترس قرار می‌دهد. هر اجرا حدود ۵ ساعت طول می‌کشد و پیش از پایان، اجرای بعدی را خودکار شروع می‌کند، پس آدرس نود هیچ‌وقت تغییر نمی‌کند.

## آدرس‌های ثابت

- نود: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- پنل: `https://panel.my-github.ccwu.cc/panel` (نام کاربری `admin`)

## راه‌اندازی اولیه

### ۱. تونل نام‌دار Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. تونل را نام‌گذاری کنید و token را کپی کنید (رشته‌ای که با `eyJ` شروع می‌شود)
3. دو public hostname در تونل اضافه کنید:
   - `vless.your-domain.com` → HTTP → `127.0.0.1:8080`
   - `panel.your-domain.com` → HTTP → `127.0.0.1:2053`

### ۲. GitHub Secrets

در مخزن: Settings → Secrets and variables → Actions، سه مورد اضافه کنید:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | توکن تونل Cloudflare |
| `PANEL_PASSWORD` | رمز ورود به پنل |
| `VLESS_UUID` | UUID نود (ثابت، در کلاینت دائمی معتبر، خودتان در https://1024tools.com/uuid/ بسازید) |

### ۳. اجرا

Actions → 3x-ui VLESS Node → Run workflow (پارامترها می‌توانند خالی باشند، secrets خودکار استفاده می‌شوند).

لاگ لینک کامل `vless://` را چاپ می‌کند که در v2rayN / v2rayNG / Streisand / NekoBox قابل وارد کردن است.

## راه‌اندازی مجدد خودکار

- هر اجرا `run_minutes` دقیقه طول می‌کشد (پیش‌فرض 290) و در پایان خودکار اجرای بعدی را شروع می‌کند
- دامنه و UUID ثابت‌اند، تنظیمات کلاینت برای همیشه معتبر است
- برای توقف: در Actions روی Cancel بزنید؛ اجراهای لغوشده خودکار ری‌استارت نمی‌شوند

## نکات

- **سرعت**: ترافیک از Cloudflare → دیتاسنترهای گیت‌هاب (آمریکا/اروپا) می‌گذرد، فقط برای استفاده موقت/سبک مناسب است
- **قطعی کوتاه**: در هر راه‌اندازی مجدد خودکار حدود ۳ تا ۵ دقیقه قطعی (بالا آمدن ماشین جدید، نصب 3x-ui، اتصال مجدد تونل)؛ کلاینت خودکار وصل می‌شود
- **انطباق**: اجرای پروکسی روی GitHub Actions ممکن است خلاف ToS باشد و ریسک تعلیق حساب دارد، خودتان ارزیابی کنید
- `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` را در جای عمومی منتشر **نکنید**
