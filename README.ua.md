[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# ☁️ cloud-3xui — Безкоштовний VLESS-вузол на GitHub

Автоматично розгортайте панель 3x-ui + VLESS-вузол на безкоштовних серверах GitHub Actions: фіксований домен, автоматичне відновлення при розриві.

## Схема архітектури

```mermaid
flowchart LR
    subgraph GitHub["GitHub Actions Runner (Ubuntu)"]
        XUI["3x-ui 面板\n:2053"]
        XRAY["Xray VLESS+WS\n:8080"]
    end
    subgraph CF["Cloudflare"]
        TUN["命名隧道\n(固定域名)"]
    end
    USER["用户客户端\nv2rayN / NekoBox 等"]

    XUI --> TUN
    XRAY --> TUN
    TUN --> USER

    style TUN fill:#f6821f,color:#fff
    style USER fill:#1f9d55,color:#fff
```

## Процес автоматичного відновлення

```mermaid
flowchart TD
    A["手动 Run workflow\n(或上一轮自动触发)"] --> B["安装 3x-ui"]
    B --> C["API 创建 VLESS+WS 入站"]
    C --> D["启动 Cloudflare 命名隧道"]
    D --> E["输出节点链接"]
    E --> F["保活 N 分钟<br/>(手动输入，默认 290)"]
    F --> G["调用 API 触发下一轮<br/>(用默认值 290)"]
    G --> A

    style G fill:#f6821f,color:#fff
```

## Автоматичне поновлення

- **Тривалість визначаєш ти**: під час Run workflow вручну заповни `run_minutes`, працюватиме стільки хвилин, скільки введеш; порожнє поле — 290 за замовчуванням
- **Автозапуск використовує значення за замовчуванням**: наприкінці кожного раунду API запускає наступний без параметрів, тож кожен автоматичний раунд триває 290 хвилин
- Для швидкого тесту введи `5`, для щоденного користування лиши порожнім (290)
- Домен та UUID фіксовані, клієнт налаштовується один раз
- Щоб зупинити, натисни Cancel в Actions, автоматично не перезапуститься

## Фіксовані адреси

| Призначення | Адреса |
|------|------|
| Вузол | `vless.my-github.ccwu.cc:443`（VLESS + WebSocket + TLS） |
| Панель | `https://panel.my-github.ccwu.cc/panel`（ім'я користувача `admin`） |

> ⚠️ Наведені вище домени — приклади, замініть їх власним доменом.

## Детальні кроки налаштування

### Безкоштовний домен (немає домену?)

Немає власного домену? Зареєструйте безкоштовний на https://my.dnshe.com/ (напр. `.ccwu.cc`, дійсний 10 років), а потім спрямуйте NS-сервери домену на Cloudflare.

### Крок 1: Створіть Named Tunnel у Cloudflare

Офіційний сайт Cloudflare: https://www.cloudflare-cn.com (внутрішній) / https://www.cloudflare.com (міжнародний)

Консоль: https://dash.cloudflare.com

```mermaid
flowchart TD
    A["打开 Cloudflare Dashboard"] --> B["左侧 Zero Trust"]
    B --> C["Networks → Tunnels"]
    C --> D["右上角 Create a tunnel"]
    D --> E["选 Cloudflared → 下一步"]
    E --> F["起名（例：github-3xui）→ 保存"]
    F --> G["复制 Token<br/>(eyJ 开头的一长串)"]
    G --> H["下一步 → Public hostnames"]
    H --> I["添加第一条<br/>vless.你的域名.com → HTTP → 127.0.0.1:8080"]
    H --> J["添加第二条<br/>panel.你的域名.com → HTTP → 127.0.0.1:2053"]
    I --> K["保存完成"]
    J --> K

    style G fill:#f6821f,color:#fff
    style K fill:#1f9d55,color:#fff
```

1. Відкрийте [Cloudflare Dashboard](https://dash.cloudflare.com), у лівому меню перейдіть до **Zero Trust**
2. Ліворуч **Networks** → **Tunnels**, натисніть **Create a tunnel** у верхньому правому куті
3. Оберіть тип **Cloudflared**, натисніть далі
4. Дайте тунелю ім'я (наприклад `github-3xui`), збережіть
5. На сторінці з'явиться команда встановлення, наприклад:
   ```
   cloudflared.exe service install eyJhIjoiYzEyNGI2...
   ```
   Скопіюйте довгий рядок символів, що починається з `eyJ`, — це **Tunnel Token**, збережіть його
6. Натисніть далі, щоб перейти до **Public hostnames** (Опубліковані маршрути застосунків), натисніть **Add a public hostname** і додайте два:

   **Перший (вузол):**
   - Subdomain (піддомен): `vless`
   - Domain (домен): оберіть ваш домен
   - Service Type (тип служби): `HTTP`
   - URL: `127.0.0.1:8080`

   **Другий (панель):**
   - Subdomain (піддомен): `panel`
   - Domain (домен): оберіть ваш домен
   - Service Type (тип служби): `HTTP`
   - URL: `127.0.0.1:2053`

7. Збережіть. Cloudflare автоматично створить DNS-записи, додавати вручну не потрібно.

### Крок 2: Згенеруйте UUID

1. Відкрийте https://1024tools.com/uuid/
2. Скопіюйте згенерований UUID (наприклад `cbfc229f-9d34-4fef-9990-8baa82a7327b`)
3. **Обов'язково збережіть цей UUID**; у журналі workflow він не відображається у відкритому вигляді, він потрібен щоразу для побудови посилання на вузол

### Крок 3: Додайте Secrets у GitHub

1. Увійдіть до цього репозиторію, натисніть вкладку **Settings**
2. Ліворуч **Secrets and variables** → **Actions**
3. Натисніть **New repository secret** у верхньому правому куті та додайте три по черзі:

   | Name | Secret (значення) | Опис |
   |------|-------------|------|
   | `CF_TUNNEL_TOKEN` | Tunnel Token, скопійований на кроці 1 | Облікові дані тунелю Cloudflare |
   | `PANEL_PASSWORD` | Встановлений вами пароль панелі | Пароль входу до панелі 3x-ui |
   | `VLESS_UUID` | UUID, згенерований на кроці 2 | UUID вузла, фіксований і незмінний |

### Крок 4: Запустіть workflow

1. Натисніть вкладку **Actions** у верхній частині репозиторію
2. Ліворуч оберіть **3x-ui VLESS Node**
3. Натисніть кнопку **Run workflow** праворуч
4. У формі, що з'явиться:
   - **Panel password**: залиште порожнім (автоматично використовується secret `PANEL_PASSWORD`)
     Пароль панелі (якщо порожньо — використовується secret PANEL_PASSWORD)
   - **Run duration**: за замовчуванням `290` (хвилин), для першого тесту можна ввести `5` для швидкої перевірки
     Тривалість роботи (хвилин, за замовчуванням 290, максимум 340)
5. Натисніть зелену кнопку **Run workflow**, щоб почати

### Крок 5: Отримайте посилання на вузол

1. Зачекайте приблизно 2-3 хвилини, натисніть на запущене завдання, щоб відкрити сторінку деталей
2. Розгорніть крок **Start named Cloudflare tunnel**, перегляньте вивід журналу:
   ```
   ==================================================
     面板 Panel: https://panel.my-github.ccwu.cc/panel  (admin / 你的密码 your password)

     VLESS 节点 node:
     vless://YOUR_UUID@vless.my-github.ccwu.cc:443?encryption=none&security=tls&sni=vless.my-github.ccwu.cc&type=ws&path=%2Fws#GitHub-3xui

     把 YOUR_UUID 换成你在 VLESS_UUID secret 里填的 UUID
     Replace YOUR_UUID with the UUID from your VLESS_UUID secret
   ==================================================
   ```
3. Скопіюйте посилання `vless://` і замініть `YOUR_UUID` на ваш UUID
4. Вставте його в клієнти на кшталт v2rayN / v2rayNG / Streisand / NekoBox — і користуйтеся

### Крок 6: Перевірте автоматичне відновлення (необов'язково)

1. Під час першого запуску встановіть тривалість `5` хвилин
2. Через 5 хвилин перегляньте журнал останнього кроку **Trigger next run**, має відобразитися:
   ```
   HTTP 204
   已触发下一次运行 Next run triggered
   ```
3. Поверніться до списку Actions, там має з'явитися новий автоматично запущений запуск
4. Після підтвердження запустіть знову, повернувши тривалість `290` для звичайного використання

## Опис кроків workflow

| Крок | Українська | English | Опис |
|------|------------|---------|------|
| Install 3x-ui | Встановіть 3x-ui | Install 3x-ui | Встановлення панелі офіційним скриптом у неінтерактивному режимі |
| Create VLESS+WS inbound via API | Створіть VLESS+WS inbound через API | Create VLESS+WS inbound via API | Вхід за API Token, автоматичне створення inbound |
| Start named Cloudflare tunnel | Запустіть Named Tunnel Cloudflare | Start named Cloudflare tunnel | Підключення тунелю з фіксованим доменом за Token |
| Keep alive | Підтримка активності | Keep alive | Очікування вказаної кількості хвилин, моніторинг стану служб |
| Trigger next run | Запустіть наступний запуск | Trigger next run | Виклик API для автоматичного запуску наступного раунду |

## Часті запитання

**З: UUID у посиланні на вузол у журналі відображається як `***`?**
В: Це механізм захисту секретів GitHub. Просто замініть `YOUR_UUID` на UUID, який ви вказали в секреті `VLESS_UUID`.

**З: Клієнт не проходить тест (-1)?**
В: Перевірте три моменти: ① чи правильно замінено UUID; ② чи нова версія клієнта (v2rayN 7.x+); ③ чи нормально підключено тунель Cloudflare (дивіться журнал).

**З: Автоматичний перезапуск не спрацював?**
В: Перегляньте журнал кроку **Trigger next run**. Якщо відображається HTTP-статус, відмінний від 204, перевірте налаштування дозволів Actions репозиторію.

**З: Це нормально — розрив на кілька хвилин кожні 5 годин?**
В: Нормально. Від завершення попереднього раунду до підключення тунелю наступного раунду минає близько 3-5 хвилин, клієнт автоматично перепідключиться.

**З: Хочете зупинити автоматичне відновлення?**
В: Перейдіть на сторінку Actions і зробіть Cancel поточного запуску — автоматично більше не запускатиметься.

## Зауваження

- **Швидкість**: трафік іде через Cloudflare → сервери GitHub (США/Європа), підходить лише для тимчасового/легкого використання
- **Короткі перерви**: кожні 5 годин під час автоматичного перезапуску з'єднання переривається приблизно на 3-5 хвилин (запуск нової машини, встановлення 3x-ui, перепідключення тунелю), клієнт автоматично перепідключиться
- **Відповідність**: запуск проксі в GitHub Actions може порушувати ToS і призвести до блокування акаунта — оцінюйте самостійно
- **Безпека**: не публікуйте `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` у відкритих місцях
