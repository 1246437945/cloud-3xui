[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Сборка узла 3x-ui (VLESS + WebSocket + TLS) на бесплатных серверах GitHub Actions, с фиксированным доменом и автоматическим продлением.

## Как это работает

Каждый запуск автоматически устанавливает панель 3x-ui на сервере GitHub Ubuntu, создаёт VLESS+WS inbound через API панели, затем открывает панель и узел через именованный туннель Cloudflare (фиксированный домен). Каждый запуск длится около 5 часов и перед завершением автоматически запускает следующий, поэтому адрес узла никогда не меняется.

## Фиксированные адреса

- Узел: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Панель: `https://panel.my-github.ccwu.cc/panel` (имя пользователя `admin`)

## Первоначальная настройка

### 1. Именованный туннель Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Назовите его и скопируйте token (длинная строка, начинающаяся с `eyJ`)
3. Добавьте две записи в Public hostnames туннеля:
   - `vless.yourdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.yourdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Settings репозитория → Secrets and variables → Actions, добавьте три:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token туннеля Cloudflare |
| `PANEL_PASSWORD` | Пароль для входа в панель |
| `VLESS_UUID` | UUID узла (фиксированный, действует для клиентов постоянно; сгенерируйте на https://1024tools.com/uuid/, **Обязательно сохраните этот UUID — он не отображается в логах открытым текстом, и он нужен каждый раз при сборке ссылки узла**) |

### 3. Запуск

Actions → 3x-ui VLESS Node → Run workflow (параметры необязательны, secrets используются автоматически).

В логе будет напечатана полная ссылка `vless://`; импортируйте её в v2rayN / v2rayNG / Streisand / NekoBox.

## Автоматическое продление

- Каждый запуск длится `run_minutes` минут (по умолчанию 290), затем автоматически запускает следующий
- Домен и UUID фиксированы; настройте клиент один раз — работает всегда
- Чтобы остановить, нажмите Cancel в Actions; автоматически не перезапустится

## Примечания

- **Скорость**: трафик идёт через Cloudflare → дата-центры GitHub (США/ЕС); только для временного/лёгкого использования
- **Краткое прерывание**: при каждой передаче на автоперезапуск — разрыв около 3-5 минут (загрузка новой машины, установка 3x-ui, переподключение туннеля); клиенты переподключатся автоматически
- **Соответствие правилам**: запуск прокси в GitHub Actions может нарушать ToS и грозить блокировкой аккаунта; оцените риски сами
- **Не** публикуйте `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` в открытых местах
