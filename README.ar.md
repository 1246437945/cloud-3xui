[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

أنشئ عقدة 3x-ui (VLESS + WebSocket + TLS) على خوادم GitHub Actions المجانية، بنطاق ثابت وتجديد تلقائي.

## كيف يعمل

كل تشغيل يثبّت لوحة 3x-ui تلقائيًا على خادم GitHub Ubuntu، وينشئ inbound من نوع VLESS+WS عبر API اللوحة، ثم يعرض اللوحة والعقدة عبر نفق Cloudflare مسمّى (نطاق ثابت). يستمر كل تشغيل حوالي 5 ساعات ويشغّل التشغيل التالي تلقائيًا قبل انتهائه، لذا لا يتغير عنوان العقدة أبدًا.

## عناوين ثابتة

- العقدة: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- اللوحة: `https://panel.my-github.ccwu.cc/panel` (اسم المستخدم `admin`)

## الإعداد لأول مرة

### 1. نفق Cloudflare المسمّى

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. سمِّه ثم انسخ الـ token (السلسلة الطويلة التي تبدأ بـ `eyJ`)
3. أضف مدخلين في Public hostnames للنفق:
   - `vless.yourdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.yourdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

إعدادات المستودع Settings → Secrets and variables → Actions، أضف هذه الثلاثة:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | token نفق Cloudflare |
| `PANEL_PASSWORD` | كلمة مرور اللوحة |
| `VLESS_UUID` | UUID العقدة (ثابت، صالح للعملاء دائمًا؛ ولّد واحدًا من https://1024tools.com/uuid/، **تأكد من حفظ هذا UUID، فلن يظهر كنص عادي في السجلات، وستحتاجه في كل مرة تركّب فيها رابط العقدة**) |

### 3. التشغيل

Actions → 3x-ui VLESS Node → Run workflow (المعاملات اختيارية، تُستخدم الـ secrets تلقائيًا).

سيطبع السجل رابط `vless://` الكامل؛ استورده في v2rayN / v2rayNG / Streisand / NekoBox.

## التجديد التلقائي

- يستمر كل تشغيل `run_minutes` دقيقة (290 افتراضيًا) ثم يشغّل التالي تلقائيًا
- النطاق و UUID ثابتان؛ اضبط العميل مرة واحدة ويعمل للأبد
- للإيقاف اضغط Cancel في Actions؛ لن يُعاد التشغيل تلقائيًا

## ملاحظات

- **السرعة**: يمر الترافيك عبر Cloudflare ← مراكز بيانات GitHub (الولايات المتحدة/الاتحاد الأوروبي)؛ مناسب للاستخدام المؤقت/الخفيف فقط
- **انقطاع قصير**: عند كل تسليم لإعادة التشغيل التلقائي ينقطع حوالي 3-5 دقائق (إقلاع الجهاز الجديد، تثبيت 3x-ui، إعادة اتصال النفق)؛ يعيد العملاء الاتصال تلقائيًا
- **الامتثال**: تشغيل بروكسي على GitHub Actions قد يخالف ToS ويعرّض الحساب للإيقاف؛ قيّم بنفسك
- **لا** تنشر `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` في أماكن عامة
