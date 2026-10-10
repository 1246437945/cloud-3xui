[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

Dựng node 3x-ui (VLESS + WebSocket + TLS) trên máy chủ miễn phí của GitHub Actions, tên miền cố định, tự động gia hạn.

## Nguyên lý hoạt động

Mỗi lần chạy sẽ tự động cài đặt panel 3x-ui trên máy chủ Ubuntu của GitHub, tạo inbound VLESS+WS qua API của panel, rồi dùng tunnel đặt tên của Cloudflare (tên miền cố định) để công khai panel và node. Mỗi lần chạy khoảng 5 giờ, trước khi kết thúc sẽ tự động kích hoạt lần chạy tiếp theo — địa chỉ node không bao giờ thay đổi.

## Địa chỉ cố định

- Node: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- Panel: `https://panel.my-github.ccwu.cc/panel` (tên đăng nhập `admin`)

## Cấu hình lần đầu

### 1. Tunnel đặt tên của Cloudflare

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. Sau khi đặt tên, sao chép token (chuỗi bắt đầu bằng `eyJ`)
3. Trong tab Public hostnames của tunnel, thêm hai bản ghi:
   - `vless.ten-mien-cua-ban.com` → HTTP → `127.0.0.1:8080`
   - `panel.ten-mien-cua-ban.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Trong repo: Settings → Secrets and variables → Actions, thêm ba secret:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Token tunnel Cloudflare |
| `PANEL_PASSWORD` | Mật khẩu đăng nhập panel |
| `VLESS_UUID` | UUID của node (cố định, hiệu lực vĩnh viễn trong client, tự tạo tại https://1024tools.com/uuid/, **Hãy nhớ lưu UUID này, nó sẽ không hiển thị dưới dạng văn bản rõ trong nhật ký, bạn sẽ cần nó mỗi khi ghép liên kết node**) |

### 3. Chạy

Actions → 3x-ui VLESS Node → Run workflow (tham số có thể bỏ trống, tự dùng secret).

Trong log sẽ có link `vless://` đầy đủ, import vào v2rayN / v2rayNG / Streisand / NekoBox là dùng được.

## Tự động gia hạn

- Mỗi lần chạy `run_minutes` phút (mặc định 290), khi kết thúc tự động kích hoạt lần tiếp theo
- Tên miền và UUID đều cố định, cấu hình client một lần dùng mãi mãi
- Muốn dừng thì vào Actions bấm Cancel — sau khi hủy sẽ không tự chạy lại

## Lưu ý

- **Tốc độ**: lưu lượng đi qua Cloudflare → trung tâm dữ liệu GitHub (Mỹ/châu Âu), chỉ phù hợp dùng tạm thời / nhẹ
- **Gián đoạn ngắn**: mỗi lần tự khởi động lại sau 5 giờ sẽ ngắt khoảng 3-5 phút (máy mới khởi động, cài 3x-ui, tunnel kết nối lại), client sẽ tự kết nối lại
- **Tuân thủ**: chạy proxy trên GitHub Actions có thể vi phạm ToS và bị khóa tài khoản, tự đánh giá rủi ro
- **Đừng** đăng `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` ở nơi công khai
