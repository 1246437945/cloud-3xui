[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Construisez un nœud 3x-ui (VLESS + WebSocket + TLS) sur les serveurs gratuits de GitHub Actions, avec domaine fixe et redémarrage automatique.

## Principe de fonctionnement

Chaque exécution installe automatiquement le panneau 3x-ui sur le serveur Ubuntu de GitHub, crée un inbound VLESS+WS via l'API du panneau, puis expose le panneau et le nœud via un **tunnel nommé** Cloudflare (domaine fixe). Chaque exécution dure environ 5 heures et déclenche automatiquement la suivante avant de se terminer, l'adresse du nœud ne change donc jamais.

## Adresses fixes

- Nœud : `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panneau : `https://panel.my-github.ccwu.cc/panel` (nom d'utilisateur `admin`)

## Configuration initiale

### 1. Tunnel nommé Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Nommez le tunnel et copiez le token (la chaîne commençant par `eyJ`)
3. Ajoutez deux public hostnames dans le tunnel :
   - `vless.votre-domaine.com` → HTTP → `127.0.0.1:8080`
   - `panel.votre-domaine.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Dans le dépôt : Settings → Secrets and variables → Actions, ajoutez-en trois :

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token du tunnel Cloudflare |
| `PANEL_PASSWORD` | Mot de passe de connexion au panneau |
| `VLESS_UUID` | UUID du nœud (fixe, valable en permanence dans le client, générez-en un vous-même sur https://1024tools.com/uuid/) |

### 3. Exécution

Actions → 3x-ui VLESS Node → Run workflow (les paramètres peuvent rester vides, les secrets sont utilisés automatiquement).

Le journal affiche le lien `vless://` complet, importable dans v2rayN / v2rayNG / Streisand / NekoBox.

## Redémarrage automatique

- Chaque exécution dure `run_minutes` minutes (290 par défaut) et déclenche automatiquement la suivante à la fin
- Le domaine et l'UUID sont fixes, la configuration du client reste valable en permanence
- Pour arrêter : cliquez sur Cancel dans Actions, les exécutions annulées ne redémarrent pas automatiquement

## Remarques

- **Vitesse** : le trafic passe par Cloudflare → centres de données GitHub (USA/UE), convient uniquement à un usage temporaire/léger
- **Courte interruption** : environ 3 à 5 minutes d'interruption à chaque redémarrage automatique (démarrage d'une nouvelle machine, installation de 3x-ui, reconnexion du tunnel), le client se reconnecte automatiquement
- **Conformité** : exécuter un proxy sur GitHub Actions peut enfreindre les ToS avec risque de suspension du compte, à évaluer par vous-même
- **Ne partagez pas** `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` publiquement
