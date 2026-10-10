[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# ☁️ cloud-3xui — Gratis VLESS-node på GitHub

Opsætter automatisk et 3x-ui-panel + VLESS-node på GitHub Actions' gratis servere med fast domæne og automatisk fornyelse ved afbrydelse.

## Arkitekturdiagram

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

## Automatisk fornyelsesproces

```mermaid
flowchart TD
    A["手动 Run workflow\n(或上一轮自动触发)"] --> B["安装 3x-ui"]
    B --> C["API 创建 VLESS+WS 入站"]
    C --> D["启动 Cloudflare 命名隧道"]
    D --> E["输出节点链接"]
    E --> F["Hold i live i N minutter<br/>(manuel indtastning, standard 290)"]
    F --> G["Kald API for at udløse næste runde<br/>(med standardværdien 290)"]
    G --> A

    style G fill:#f6821f,color:#fff
```

## Automatisk fornyelse

- **Varigheden bestemmer du**: når du klikker på Run workflow, udfyld manuelt `run_minutes` – det kører det antal minutter, du indtaster; lader du det stå tomt, bruges standardværdien 290
- **Automatisk udløsning bruger standardværdien**: ved slutningen af hver runde kaldes API'et for at starte næste runde, og API-kaldet medtager ingen parametre, så hver automatisk fornyet runde er 290 minutter
- Til en hurtig test skriv `5`; til daglig brug lad det stå tomt og brug standard 290
- Domænet og UUID er faste, så klientkonfigurationen gælder permanent efter én opsætning
- Vil du stoppe, så gå til Actions og klik på Cancel – efter annullering genstartes det ikke automatisk

## Faste adresser

| Formål | Adresse |
|------|------|
| Node | `vless.my-github.ccwu.cc:443` (VLESS + WebSocket + TLS) |
| Panel | `https://panel.my-github.ccwu.cc/panel` (brugernavn `admin`) |

> ⚠️ Domænerne ovenfor er eksempler – udskift dem med dit eget domæne.

## Detaljerede konfigurationstrin

### Gratis domæne (intet domæne? start her)

Har du ikke dit eget domæne? Få et gratis på https://my.dnshe.com/ (f.eks. `.ccwu.cc`, gyldigt i 10 år), og peg derefter domænets NS-poster til Cloudflare.

### Trin 1: Opret navngivet tunnel i Cloudflare


Cloudflares officielle website: https://www.cloudflare-cn.com (Kina) / https://www.cloudflare.com (international)
Kontrolpanel: https://dash.cloudflare.com

```mermaid
flowchart TD
    A["控制台 → Zero Trust<br/>→ 网络（Networks）→ 连接器（Tunnels）"] --> B["右上角 创建隧道"]
    B --> C["选 Cloudflared<br/>(推荐) → 选择 Cloudflared"]
    C --> D["输入隧道名称<br/>(例：github-3xui)<br/>→ 保存隧道"]
    D --> E["安装并运行连接器页<br/>复制 Token<br/>(eyJ 开头的一长串)"]
    E --> F["下一步 → 路由隧道<br/>→ 已发布应用程序路由"]
    F --> G["添加已发布应用程序路由<br/>子域 vless + 你的域<br/>类型 HTTP → URL 127.0.0.1:8080"]
    F --> H["添加已发布应用程序路由<br/>子域 panel + 你的域<br/>类型 HTTP → URL 127.0.0.1:2053"]
    G --> I["保存完成"]
    H --> I

    style E fill:#f6821f,color:#fff
    style I fill:#1f9d55,color:#fff
```

Detaljerede trin:

1. Åbn [Cloudflare Dashboard](https://dash.cloudflare.com) og gå til **Zero Trust** → **Networks** → **Tunnels** i menuen til venstre
2. Klik på **Create a tunnel** øverst til højre
3. **Vælg tunneltype**: vælg **Cloudflared** (anbefales), og klik på **Select Cloudflared**
4. **Giv tunnelen et navn**: indtast et navn (f.eks. `github-3xui`), og klik på **Save tunnel**
5. **Installer og kør connectoren**: siden viser en installationskommando — kopiér den lange **token**, der starter med `eyJ`, og gem den (du skal bruge den som GitHub Secret senere). Du behøver **ikke** installere noget lokalt; fortsæt blot
6. **Rout tunnelen**: gå til fanen **Public hostnames**, klik på **Add a public hostname**, og tilføj to poster:

   **Første (node):**
   - Underdomæne: `vless`
   - Domæne: vælg dit domæne
   - Type: `HTTP`
   - URL: `127.0.0.1:8080`

   **Anden (panel):**
   - Underdomæne: `panel`
   - Domæne: vælg dit domæne
   - Type: `HTTP`
   - URL: `127.0.0.1:2053`
7. Gem. Cloudflare opretter DNS-poster automatisk. Status **Healthy** på Tunnels-listen betyder, at opsætningen lykkedes.


### Trin 2: Generér UUID

1. Åbn https://1024tools.com/uuid/
2. Kopiér det genererede UUID (i formen `cbfc229f-9d34-4fef-9990-8baa82a7327b`)
3. **Sørg for at gemme dette UUID**, det vises ikke i klartekst i workflow-loggen, og du skal bruge det hver gang du samler node-linket

### Trin 3: Tilføj Secrets på GitHub

1. Gå til dette repository, og klik på fanen **Settings**
2. Til venstre: **Secrets and variables** → **Actions**
3. Klik på **New repository secret** øverst til højre, og tilføj de tre i rækkefølge:

   | Name | Secret (værdi) | Beskrivelse |
   |------|-------------|------|
   | `CF_TUNNEL_TOKEN` | Tunnel Token kopieret i trin 1 | Cloudflare-tunnelens legitimationsoplysninger |
   | `PANEL_PASSWORD` | Den panel-adgangskode du har valgt | Login-adgangskode til 3x-ui-panelet |
   | `VLESS_UUID` | UUID'et genereret i trin 2 | Nodens UUID, ændres ikke |

### Trin 4: Kør workflowet

1. Klik på fanen **Actions** øverst i repositoryet
2. Vælg **3x-ui VLESS Node** til venstre
3. Klik på knappen **Run workflow** til højre
4. I den pop op-form:
   - **Panel password**: lad stå tomt (bruger automatisk `PANEL_PASSWORD`-secretet)
     Panel-adgangskode (tom = bruger PANEL_PASSWORD-secret)
   - **Run duration**: standard `290` (minutter), ved første test kan du skrive `5` for hurtig verifikation; lad det stå tomt til daglig brug
     Kørselsvarighed (minutter, standard 290, maks. 340)
5. Klik på den grønne **Run workflow** for at starte

### Trin 5: Få node-linket

1. Vent ca. 2-3 minutter, klik på den kørende opgave for at åbne detaljesiden
2. Udfold trinnet **Start named Cloudflare tunnel**, og se log-outputtet:
   ```
   ==================================================
     面板 Panel: https://panel.my-github.ccwu.cc/panel  (admin / 你的密码 your password)

     VLESS 节点 node:
     vless://YOUR_UUID@vless.my-github.ccwu.cc:443?encryption=none&security=tls&sni=vless.my-github.ccwu.cc&type=ws&path=%2Fws#GitHub-3xui

     把 YOUR_UUID 换成你在 VLESS_UUID secret 里填的 UUID
     Replace YOUR_UUID with the UUID from your VLESS_UUID secret
   ==================================================
   ```
3. Kopiér `vless://`-linket, og erstat `YOUR_UUID` med dit UUID
4. Indsæt det i v2rayN / v2rayNG / Streisand / NekoBox m.fl. for at bruge det

### Trin 6: Verificér automatisk fornyelse (valgfrit)

1. Ved første kørsel sættes varigheden til `5` minutter
2. Efter 5 minutter ses loggen for det sidste trin **Trigger next run**, som bør vise:
   ```
   HTTP 204
   已触发下一次运行 Next run triggered
   ```
3. Gå tilbage til Actions-listen – der bør nu være en ny automatisk udløst kørsel
4. Når det er bekræftet, kør igen og sæt varigheden tilbage til `290` til normal brug

## Forklaring af workflow-trin

| Trin | Dansk | English | Forklaring |
|------|------|---------|------|
| Install 3x-ui | Installér 3x-ui | Install 3x-ui | Installerer panelet med det officielle script uden interaktion |
| Create VLESS+WS inbound via API | Opret VLESS+WS inbound via API | Create VLESS+WS inbound via API | Logger ind med API-token og opretter inbound automatisk |
| Start named Cloudflare tunnel | Start navngivet Cloudflare-tunnel | Start named Cloudflare tunnel | Forbinder tunnelen med fast domæne via token |
| Keep alive | Hold i live | Keep alive | Sover i det angivne antal minutter og overvåger tjenesternes status |
| Trigger next run | Udløs næste kørsel | Trigger next run | Kalder API'et og starter automatisk næste runde |

## Ofte stillede spørgsmål

**Q: UUID'et i node-linket vises som `***` i loggen?**
A: Det er GitHubs secret-beskyttelsesmekanisme. Erstat `YOUR_UUID` med det UUID, du indtastede i `VLESS_UUID`-secretet.

**Q: Klienttesten fejler (-1)?**
A: Tjek tre ting: ① om UUID'et er erstattet korrekt; ② om klientversionen er ny nok (v2rayN 7.x+); ③ om Cloudflare-tunnelen er forbundet korrekt (se loggen).

**Q: Den automatiske genstart blev ikke udløst?**
A: Se loggen for trinnet **Trigger next run**. Hvis der vises en anden HTTP-statuskode end 204, tjek Actions-rettighedsindstillingerne i repositoryet.

**Q: Er det normalt, at forbindelsen afbrydes i et par minutter hver 5. time?**
A: Ja, det er normalt. Fra forrige rundes afslutning til tunnelen er forbundet i næste runde går der ca. 3-5 minutter, og klienten genopretter automatisk forbindelsen.

**Q: Vil du stoppe den automatiske fornyelse?**
A: Gå til Actions-siden og annuller (Cancel) den aktuelle kørsel – så udløses der ikke flere automatisk.

## Bemærkninger

- **Hastighed**: trafikken går via Cloudflare → GitHubs datacentre (USA/Europa), kun egnet til midlertidig/let brug
- **Kort afbrydelse**: ved den automatiske genstart hver 5. time afbrydes forbindelsen i ca. 3-5 minutter (ny maskine starter, 3x-ui installeres, tunnelen genforbindes), klienten genopretter automatisk
- **Overholdelse**: at køre proxy på GitHub Actions kan overtræde ToS og føre til kontosuspension – vurder selv
- **Sikkerhed**: offentliggør ikke `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID`
