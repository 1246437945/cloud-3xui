[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

GitHub Actions の無料サーバーで 3x-ui ノード（VLESS + WebSocket + TLS）を構築。固定ドメイン＋自動更新。

## 仕組み

実行のたびに GitHub の Ubuntu サーバーへ 3x-ui パネルを自動インストールし、パネル API で VLESS+WS インバウンドを作成、Cloudflare の名前付きトンネル（固定ドメイン）でパネルとノードを公開します。1 回の実行は約 5 時間で、終了前に次回を自動起動するため、ノードアドレスは変わりません。

## 固定アドレス

- ノード: `vless.my-github.ccwu.cc:443`（VLESS + WS + TLS）
- パネル: `https://panel.my-github.ccwu.cc/panel`（ユーザー名 `admin`）

## 初期設定

### 1. Cloudflare 名前付きトンネル

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel（Cloudflared）
2. 名前を付けて token（`eyJ` で始まる長い文字列）をコピー
3. トンネルの Public hostnames に 2 件追加:
   - `vless.yourdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.yourdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

リポジトリ Settings → Secrets and variables → Actions で 3 件追加:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare トンネルの token |
| `PANEL_PASSWORD` | パネルのログインパスワード |
| `VLESS_UUID` | ノードの UUID（固定、クライアント側で恒久有効。https://1024tools.com/uuid/ で生成） |

### 3. 実行

Actions → 3x-ui VLESS Node → Run workflow（パラメータは省略可、secret が自動使用されます）。

ログに完全な `vless://` リンクが出力されるので、v2rayN / v2rayNG / Streisand / NekoBox にインポートしてください。

## 自動更新

- 1 回の実行は `run_minutes` 分（デフォルト 290）、終了時に次回を自動起動
- ドメイン・UUID ともに固定、一度設定すれば恒久利用可能
- 停止したい場合は Actions で Cancel（自動再起動しません）

## 注意事項

- **速度**: トラフィックは Cloudflare → GitHub データセンター（米/欧）経由、一時利用・軽量利用向け
- **短時間の切断**: 5 時間ごとの自動再起動の引き継ぎ時に約 3〜5 分切断（新マシン起動、3x-ui インストール、トンネル再接続）、クライアントは自動再接続します
- **コンプライアンス**: GitHub Actions でのプロキシ運用は ToS 違反でアカウント停止のリスクあり、自己責任で
- `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` を公開の場に出さないこと
