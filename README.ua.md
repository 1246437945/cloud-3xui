[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Зберіть вузол 3x-ui (VLESS + WebSocket + TLS) на безкоштовних серверах GitHub Actions, зі сталим доменом та автоматичним поновленням.

## Як це працює

Кожен запуск автоматично встановлює панель 3x-ui на сервері Ubuntu в GitHub, створює inbound VLESS+WS через API панелі, а потім відкриває панель і вузол через іменований тунель Cloudflare (сталий домен). Кожен запуск триває близько 5 годин і перед завершенням автоматично запускає наступний — адреса вузла ніколи не змінюється.

## Сталі адреси

- Вузол: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Панель: `https://panel.my-github.ccwu.cc/panel` (користувач `admin`)

## Початкове налаштування

### 1. Іменований тунель Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Після найменування скопіюйте токен (рядок, що починається з `eyJ`)
3. У вкладці Public hostnames тунелю додайте два записи:
   - `vless.ваш-домен.com` → HTTP → `127.0.0.1:8080`
   - `panel.ваш-домен.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

У репозиторії: Settings → Secrets and variables → Actions, додайте три:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Токен тунелю Cloudflare |
| `PANEL_PASSWORD` | Пароль входу до панелі |
| `VLESS_UUID` | UUID вузла (сталий, дійсний у клієнті постійно, згенеруйте на https://1024tools.com/uuid/, **Обов'язково збережіть цей UUID, він не відображатиметься у відкритому вигляді в логах, він знадобиться щоразу під час складання посилання вузла**) |

### 3. Запуск

Actions → 3x-ui VLESS Node → Run workflow (параметри необов'язкові, секрети підхопляться автоматично).

У логах знайдіть повне посилання `vless://` та імпортуйте його у v2rayN / v2rayNG / Streisand / NekoBox.

## Автоматичне поновлення

- Кожен запуск триває `run_minutes` хвилин (за замовчуванням 290), наприкінці автоматично запускає наступний
- Домен і UUID сталі, клієнт налаштовується один раз
- Щоб зупинити, натисніть Cancel в Actions — після скасування сам не перезапуститься

## Зауваження

- **Швидкість**: трафік іде через Cloudflare → дата-центри GitHub (США/Європа), підходить лише для тимчасового / легкого використання
- **Короткі перерви**: під час кожного автоперезапуску раз на 5 годин буває перерва 3-5 хвилин (старт нової машини, встановлення 3x-ui, перепідключення тунелю), клієнт перепідключиться автоматично
- **Відповідність**: запуск проксі в GitHub Actions може порушувати ToS і призвести до блокування акаунта, оцінюйте ризик самостійно
- **Не** публікуйте `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` у відкритих місцях
