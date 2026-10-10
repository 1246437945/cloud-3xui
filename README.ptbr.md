[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Crie um nó 3x-ui (VLESS + WebSocket + TLS) nos servidores gratuitos do GitHub Actions, com domínio fixo e renovação automática.

## Como funciona

Cada execução instala automaticamente o painel 3x-ui em um servidor Ubuntu do GitHub, cria um inbound VLESS+WS via API do painel e expõe o painel e o nó por meio de um túnel nomeado da Cloudflare (domínio fixo). Cada execução dura cerca de 5 horas e, antes de terminar, dispara automaticamente a próxima — o endereço do nó nunca muda.

## Endereços fixos

- Nó: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Painel: `https://panel.my-github.ccwu.cc/panel` (usuário `admin`)

## Configuração inicial

### 1. Túnel nomeado da Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Após nomear, copie o token (a sequência que começa com `eyJ`)
3. Na aba Public hostnames do túnel, adicione duas entradas:
   - `vless.seu-dominio.com` → HTTP → `127.0.0.1:8080`
   - `panel.seu-dominio.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

No repositório: Settings → Secrets and variables → Actions, adicione três:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token do túnel Cloudflare |
| `PANEL_PASSWORD` | Senha de login do painel |
| `VLESS_UUID` | UUID do nó (fixo, válido permanentemente no cliente, gere em https://1024tools.com/uuid/, **Certifique-se de salvar este UUID, ele não será exibido em texto claro nos logs, você precisará dele toda vez que montar o link do nó**) |

### 3. Executar

Actions → 3x-ui VLESS Node → Run workflow (parâmetros opcionais, usa os secrets automaticamente).

Nos logs você encontra o link `vless://` completo, importe no v2rayN / v2rayNG / Streisand / NekoBox.

## Renovação automática

- Cada execução dura `run_minutes` minutos (padrão 290) e ao final dispara automaticamente a próxima
- Domínio e UUID são fixos, configure o cliente uma vez e use para sempre
- Para parar, clique em Cancel em Actions — após cancelar não reinicia sozinho

## Observações

- **Velocidade**: o tráfego passa pela Cloudflare → datacenters do GitHub (EUA/Europa), adequado apenas para uso temporário / leve
- **Interrupções breves**: a cada reinício automático de 5 horas há uma interrupção de 3-5 minutos (inicialização da nova máquina, instalação do 3x-ui, reconexão do túnel), o cliente reconecta automaticamente
- **Conformidade**: rodar proxy no GitHub Actions pode violar os ToS e levar ao banimento da conta, avalie por conta própria
- **Não** publique `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` em locais públicos
