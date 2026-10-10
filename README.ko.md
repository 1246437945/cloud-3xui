[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

GitHub Actions 무료 서버에 3x-ui 노드(VLESS + WebSocket + TLS) 구축. 고정 도메인 + 자동 갱신.

## 동작 방식

실행할 때마다 GitHub Ubuntu 서버에 3x-ui 패널을 자동 설치하고, 패널 API로 VLESS+WS 인바운드를 생성한 뒤, Cloudflare 네임드 터널(고정 도메인)으로 패널과 노드를 노출합니다. 1회 실행은 약 5시간이며, 종료 전에 다음 실행을 자동 트리거하므로 노드 주소가 바뀌지 않습니다.

## 고정 주소

- 노드: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- 패널: `https://panel.my-github.ccwu.cc/panel` (사용자명 `admin`)

## 최초 설정

### 1. Cloudflare 네임드 터널

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. 이름을 지정하고 token(`eyJ`로 시작하는 긴 문자열)을 복사
3. 터널의 Public hostnames에 2개 추가:
   - `vless.yourdomain.com` → HTTP → `127.0.0.1:8080`
   - `panel.yourdomain.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

저장소 Settings → Secrets and variables → Actions에서 3개 추가:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare 터널 token |
| `PANEL_PASSWORD` | 패널 로그인 비밀번호 |
| `VLESS_UUID` | 노드 UUID (고정, 클라이언트에서 영구 유효. https://1024tools.com/uuid/ 에서 생성, **이 UUID를 반드시 메모해 두세요. 로그에 평문으로 표시되지 않으며, 노드 링크를 조립할 때마다 필요합니다**) |

### 3. 실행

Actions → 3x-ui VLESS Node → Run workflow (파라미터 생략 가능, secret 자동 사용).

로그에 전체 `vless://` 링크가 출력되며, v2rayN / v2rayNG / Streisand / NekoBox에 가져오면 됩니다.

## 자동 갱신

- 1회 실행은 `run_minutes`분(기본 290), 종료 시 다음 실행 자동 트리거
- 도메인과 UUID 모두 고정, 한 번 설정하면 영구 사용
- 중지하려면 Actions에서 Cancel (자동 재시작 안 됨)

## 주의사항

- **속도**: 트래픽은 Cloudflare → GitHub 데이터센터(미국/유럽) 경유, 임시/가벼운 용도에 적합
- **짧은 끊김**: 5시간마다 자동 재시작 전환 시 약 3~5분 끊김 (새 머신 부팅, 3x-ui 설치, 터널 재연결), 클라이언트는 자동 재연결됨
- **준수**: GitHub Actions에서 프록시 운영은 ToS 위반으로 계정 정지 위험, 본인 책임
- `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID`를 공개된 곳에 올리지 마세요
