# cloud-3xui

用 GitHub Actions 免费服务器搭建 3x-ui 节点（VLESS + WebSocket + TLS），固定域名，自动续命。

## 工作原理

每次运行都在 GitHub 的 Ubuntu 服务器上自动安装 3x-ui 面板，通过面板 API 创建 VLESS+WS 入站，再用 Cloudflare **命名隧道**（固定域名）暴露面板和节点。每次运行约 5 小时，结束前自动触发下一次运行，节点地址永久不变。

## 固定地址

- 节点：`vless.my-github.ccwu.cc:443`（VLESS + WS + TLS）
- 面板：`https://panel.my-github.ccwu.cc/panel`（用户名 `admin`）

## 首次配置

### 1. Cloudflare 命名隧道

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel（Cloudflared）
2. 命名后复制 token（`eyJ` 开头那串）
3. 在隧道的 Public hostnames 加两条：
   - `vless.你的域名.com` → HTTP → `127.0.0.1:8080`
   - `panel.你的域名.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

仓库 Settings → Secrets and variables → Actions，加三个：

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare 隧道 token |
| `PANEL_PASSWORD` | 面板登录密码 |
| `VLESS_UUID` | 节点 UUID（固定，客户端永久有效，自己去 https://1024tools.com/uuid/ 生成一个） |

### 3. 运行

Actions → 3x-ui VLESS Node → Run workflow（参数可空，自动用 secret）。

日志里会输出完整的 `vless://` 链接，导入 v2rayN / v2rayNG / Streisand / NekoBox 即可。

## 自动续命

- 每次运行 `run_minutes` 分钟（默认 290），结束时自动触发下一次
- 域名、UUID 都固定，客户端配置一次永久有效
- 想停就去 Actions 点 Cancel，再取消不会自动重启

## 注意事项

- **速度**：流量经 Cloudflare → GitHub 机房（美欧），仅适合临时/轻度使用
- **短暂中断**：每 5 小时自动重启交接时约断 3-5 分钟（新机器启动、装 3x-ui、隧道重连），客户端会自动重连恢复
- **合规**：在 GitHub Actions 跑代理有违反 ToS 被封号的风险，自行评估
- **不要**把 `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` 发到公开地方
