[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Monta un nodo 3x-ui (VLESS + WebSocket + TLS) en servidores gratuitos de GitHub Actions, con dominio fijo y renovación automática.

## Cómo funciona

Cada ejecución instala automáticamente el panel 3x-ui en un servidor Ubuntu de GitHub, crea un inbound VLESS+WS mediante la API del panel y expone el panel y el nodo a través de un túnel nombrado de Cloudflare (dominio fijo). Cada ejecución dura unas 5 horas y antes de terminar dispara automáticamente la siguiente, por lo que la dirección del nodo nunca cambia.

## Direcciones fijas

- Nodo: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (usuario `admin`)

## Configuración inicial

### 1. Túnel nombrado de Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Ponle un nombre y copia el token (la cadena larga que empieza por `eyJ`)
3. Añade dos entradas en Public hostnames del túnel:
   - `vless.tudominio.com` → HTTP → `127.0.0.1:8080`
   - `panel.tudominio.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Settings del repositorio → Secrets and variables → Actions, añade estos tres:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token del túnel de Cloudflare |
| `PANEL_PASSWORD` | Contraseña del panel |
| `VLESS_UUID` | UUID del nodo (fijo, válido permanentemente para los clientes; genera uno en https://1024tools.com/uuid/, **Asegúrate de guardar este UUID: no aparecerá en texto plano en los registros y lo necesitarás cada vez que armes el enlace del nodo**) |

### 3. Ejecutar

Actions → 3x-ui VLESS Node → Run workflow (los parámetros son opcionales, los secrets se usan automáticamente).

El log mostrará el enlace `vless://` completo; impórtalo en v2rayN / v2rayNG / Streisand / NekoBox.

## Renovación automática

- Cada ejecución dura `run_minutes` minutos (290 por defecto) y al terminar dispara la siguiente
- Dominio y UUID fijos; configura el cliente una vez y funciona para siempre
- Para detenerlo, haz clic en Cancel en Actions; no se reiniciará solo

## Notas

- **Velocidad**: el tráfico pasa por Cloudflare → centros de datos de GitHub (EE. UU./UE); solo para uso temporal/ligero
- **Interrupción breve**: en cada traspaso del reinicio automático hay un corte de 3-5 minutos (arranque de la nueva máquina, instalación de 3x-ui, reconexión del túnel); los clientes se reconectan solos
- **Cumplimiento**: usar un proxy en GitHub Actions puede violar los ToS y suponer la suspensión de la cuenta; evalúalo por tu cuenta
- **No** publiques `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` en sitios públicos
