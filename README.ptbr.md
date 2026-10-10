[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# ☁️ cloud-3xui — Nó VLESS gratuito com GitHub

Implante automaticamente um painel 3x-ui + nó VLESS usando os servidores gratuitos do GitHub Actions, com domínio fixo e renovação automática após queda de conexão.

## Diagrama de arquitetura

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

## Fluxo de renovação automática

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

## Renovação automática

- **A duração é você quem decide**: ao executar Run workflow preencha `run_minutes` manualmente, dura os minutos informados; vazio usa o padrão 290
- **O gatilho automático usa o padrão**: ao fim de cada ciclo a API inicia o próximo sem parâmetros, então cada ciclo automático dura 290 minutos
- Para teste rápido digite `5`, para uso diário deixe vazio (290)
- Domínio e UUID são fixos, configure o cliente uma única vez
- Para parar clique Cancel em Actions, não reiniciará sozinho

## Endereços fixos

| Finalidade | Endereço |
|------|------|
| Nó | `vless.my-github.ccwu.cc:443`（VLESS + WebSocket + TLS） |
| Painel | `https://panel.my-github.ccwu.cc/panel`（nome de usuário `admin`） |

> ⚠️ Os domínios acima são exemplos — substitua-os pelo seu próprio domínio.

## Etapas detalhadas de configuração

### Domínio grátis (sem domínio?)

Não tem domínio próprio? Registre um gratuitamente em https://my.dnshe.com/ (ex.: `.ccwu.cc`, válido por 10 anos) e depois aponte os NS do domínio para a Cloudflare.

### Etapa 1: Criar um túnel nomeado no Cloudflare

Site oficial da Cloudflare: https://www.cloudflare-cn.com (nacional) / https://www.cloudflare.com (internacional)

Painel: https://dash.cloudflare.com

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

1. Abra o [painel do Cloudflare](https://dash.cloudflare.com) e, no menu à esquerda, acesse **Zero Trust**
2. À esquerda, vá em **Networks** → **Tunnels** e clique em **Create a tunnel** no canto superior direito
3. Escolha o tipo **Cloudflared** e clique em avançar
4. Dê um nome ao túnel (por exemplo, `github-3xui`) e salve
5. A página exibirá um comando de instalação, por exemplo:
   ```
   cloudflared.exe service install eyJhIjoiYzEyNGI2...
   ```
   Copie a longa sequência de caracteres que começa com `eyJ` — este é o **Tunnel Token** — e guarde-a bem
6. Clique em avançar para ir até **Public hostnames** (rotas de aplicativo publicadas) e clique em **Add a public hostname** para adicionar duas:

   **Primeira (nó):**
   - Subdomain (subdomínio): `vless`
   - Domain (domínio): escolha o seu próprio domínio
   - Service Type (tipo de serviço): `HTTP`
   - URL: `127.0.0.1:8080`

   **Segunda (painel):**
   - Subdomain (subdomínio): `panel`
   - Domain (domínio): escolha o seu próprio domínio
   - Service Type (tipo de serviço): `HTTP`
   - URL: `127.0.0.1:2053`

7. Salve. O Cloudflare criará os registros DNS automaticamente — não é preciso adicionar nada manualmente.

### Etapa 2: Gerar um UUID

1. Abra https://1024tools.com/uuid/
2. Copie o UUID gerado (por exemplo, `cbfc229f-9d34-4fef-9990-8baa82a7327b`)
3. **Guarde bem este UUID** — ele não aparece em texto puro nos logs do workflow, e você precisará dele toda vez que montar o link do nó

### Etapa 3: Adicionar Secrets no GitHub

1. Abra este repositório e clique na aba **Settings**
2. À esquerda, **Secrets and variables** → **Actions**
3. Clique em **New repository secret** no canto superior direito e adicione os três em sequência:

   | Name | Secret (valor) | Descrição |
   |------|-------------|------|
   | `CF_TUNNEL_TOKEN` | Tunnel Token copiado na etapa 1 | Credencial do túnel Cloudflare |
   | `PANEL_PASSWORD` | A senha do painel definida por você | Senha de login do painel 3x-ui |
   | `VLESS_UUID` | UUID gerado na etapa 2 | UUID do nó, fixo e imutável |

### Etapa 4: Executar o workflow

1. Clique na aba **Actions** no topo do repositório
2. À esquerda, selecione **3x-ui VLESS Node**
3. Clique no botão **Run workflow** à direita
4. No formulário exibido:
   - **Panel password**：deixe em branco (usa automaticamente o secret `PANEL_PASSWORD`)
     Senha do painel (em branco = usa o secret `PANEL_PASSWORD`)
   - **Run duration**：padrão `290` (minutos); para o primeiro teste, você pode preencher `5` para uma verificação rápida
     Duração da execução (minutos, padrão 290, máximo 340)
5. Clique no botão verde **Run workflow** para iniciar

### Etapa 5: Obter o link do nó

1. Aguarde cerca de 2–3 minutos e clique na tarefa em execução para abrir os detalhes
2. Expanda a etapa **Start named Cloudflare tunnel** e veja a saída do log:
   ```
   ==================================================
     Painel Panel: https://panel.my-github.ccwu.cc/panel  (admin / sua senha your password)

     Nó VLESS node:
     vless://YOUR_UUID@vless.my-github.ccwu.cc:443?encryption=none&security=tls&sni=vless.my-github.ccwu.cc&type=ws&path=%2Fws#GitHub-3xui

     Substitua YOUR_UUID pelo UUID do seu secret VLESS_UUID
     Replace YOUR_UUID with the UUID from your VLESS_UUID secret
   ==================================================
   ```
3. Copie o link `vless://` e substitua `YOUR_UUID` pelo seu próprio UUID
4. Cole em v2rayN / v2rayNG / Streisand / NekoBox ou outro cliente e já pode usar

### Etapa 6: Verificar a renovação automática (opcional)

1. Na primeira execução, defina a duração como `5` minutos
2. Após 5 minutos, verifique o log da última etapa **Trigger next run** — ele deve mostrar:
   ```
   HTTP 204
   Próxima execução disparada Next run triggered
   ```
3. Volte à lista de Actions — uma nova execução disparada automaticamente deve aparecer
4. Após confirmar, execute novamente e volte a duração para `290` para uso normal

## Descrição das etapas do workflow

| Etapa | Português | English | Descrição |
|------|------|---------|------|
| Install 3x-ui | Instalar o 3x-ui | Install 3x-ui | Instala o painel de forma não interativa com o script oficial |
| Create VLESS+WS inbound via API | Criar inbound VLESS+WS via API | Create VLESS+WS inbound via API | Login com o API Token e criação automática do inbound |
| Start named Cloudflare tunnel | Iniciar o túnel nomeado do Cloudflare | Start named Cloudflare tunnel | Conecta ao túnel de domínio fixo usando o Token |
| Keep alive | Manter ativo | Keep alive | Aguarda o número especificado de minutos e monitora o status dos serviços |
| Trigger next run | Disparar a próxima execução | Trigger next run | Chama a API para iniciar automaticamente a próxima rodada |

## Perguntas frequentes

**P: O UUID do link do nó aparece como `***` no log?**
R: Este é o mecanismo de proteção de secrets do GitHub. Substitua `YOUR_UUID` pelo UUID que você mesmo preencheu no secret `VLESS_UUID`.

**P: O teste do cliente falha (-1)?**
R: Verifique três pontos: ① se o UUID foi substituído corretamente; ② se a versão do cliente está atualizada (v2rayN 7.x+); ③ se o túnel do Cloudflare está conectado normalmente (veja o log).

**P: A reinicialização automática não foi disparada?**
R: Veja o log da etapa **Trigger next run**. Se mostrar um código de status HTTP diferente de 204, verifique as configurações de permissão de Actions do repositório.

**P: É normal a conexão cair por alguns minutos a cada 5 horas?**
R: Normal. Do fim da rodada anterior até a conexão do novo túnel leva cerca de 3–5 minutos; o cliente se reconecta automaticamente.

**P: Quer parar a renovação automática?**
R: Na página de Actions, cancele (Cancel) a execução atual — nenhuma nova será disparada automaticamente.

## Observações

- **Velocidade**: o tráfego passa pelo Cloudflare → datacenters do GitHub (EUA/Europa), adequado apenas para uso temporário/leve
- **Interrupções breves**: a cada reinicialização automática (5 horas), a conexão cai por cerca de 3–5 minutos (nova máquina inicia, 3x-ui é instalado, túnel se reconecta); o cliente se reconecta automaticamente
- **Conformidade**: rodar um proxy no GitHub Actions pode violar os ToS, com risco de banimento da conta — avalie por conta própria
- **Segurança**: não publique `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` em locais públicos