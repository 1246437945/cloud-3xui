# cloud-3xui

用 GitHub Actions 免费服务器一键搭建 3x-ui 节点（VLESS + WebSocket + TLS）。

## 工作原理

每次运行都在 GitHub 的 Ubuntu 服务器上自动安装 3x-ui 面板，通过面板 API 创建 VLESS+WS 入站，再用两条 Cloudflare 隧道分别暴露面板和节点，最后在日志里输出可直接导入客户端的 `vless://` 链接。

## 使用方法

### 第 1 步：运行工作流

1. 打开本仓库的 **Actions** 标签页
2. 左侧选择 **3x-ui VLESS Node**
3. 点击右侧 **Run workflow**
4. 填写参数：
   - **Panel password**：面板登录密码（必填，自己设一个）
   - **Run duration**：运行时长（分钟，默认 300，最多 350）
5. 点击 **Run workflow** 开始

### 第 2 步：获取节点

等待约 2-3 分钟，点进正在运行的任务，展开 **Setup Cloudflare tunnels** 步骤的日志，找到：

```
Panel: https://xxx.trycloudflare.com/panel  (admin / 你的密码)

VLESS node:
vless://xxx@xxx.trycloudflare.com:443?encryption=none&security=tls&...
```

### 第 3 步：导入客户端

复制 `vless://` 开头的完整链接，粘贴到 v2rayN / v2rayNG / Streisand / NekoBox 等客户端即可使用。

### 面板管理

用上面的 Panel 地址 + 用户名 `admin` + 你设的密码登录，可在面板里查看流量、管理客户端、添加更多入站。

## 节点说明

- 协议：VLESS
- 传输：WebSocket（路径 `/ws`）
- 加密：TLS（由 Cloudflare 提供）
- 每次运行自动生成新的 UUID，无需手动配置

## 注意事项

- **有时限**：GitHub 单次运行最长约 6 小时，到时节点自动失效；隧道地址每次运行都会变，不要收藏
- **速度**：流量经 Cloudflare 边缘 → GitHub 机房（美欧），延迟和带宽仅适合临时使用
- **合规**：在 GitHub Actions 上运行代理服务存在违反服务条款被封号的风险，请自行评估
- **用完即关**：回到 Actions 页面点 **Cancel run** 可提前结束，节省额度

## 常见问题

**Q: 日志里没有节点链接？**
A: 检查 **Create VLESS+WS inbound via API** 步骤是否报错，3x-ui 安装失败时不会生成节点。

**Q: 客户端测试不通（-1）？**
A: 确认客户端版本较新（v2rayN 7.x+），并检查隧道地址是否复制完整。
