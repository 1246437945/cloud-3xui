[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# cloud-3xui

GitHub Actions സൗജന്യ സെർവറുകൾ ഉപയോഗിച്ച് 3x-ui നോഡ് (VLESS + WebSocket + TLS) നിർമ്മിക്കുക, സ്ഥിരമായ ഡൊമെയ്ൻ, ഓട്ടോമാറ്റിക് റീസ്റ്റാർട്ട്.

## പ്രവർത്തന രീതി

ഓരോ റണ്ണിലും GitHub Ubuntu സെർവറിൽ 3x-ui പാനൽ ഓട്ടോമാറ്റിക്കായി ഇൻസ്റ്റാൾ ചെയ്യുന്നു, panel API വഴി VLESS+WS ഇൻബൗണ്ട് സൃഷ്ടിക്കുന്നു, തുടർന്ന് Cloudflare **named tunnel** (സ്ഥിരമായ ഡൊമെയ്ൻ) വഴി പാനലും നോഡും എക്സ്പോസ് ചെയ്യുന്നു. ഓരോ റണ്ണും ഏകദേശം 5 മണിക്കൂർ നീണ്ടുനിൽക്കും, അവസാനിക്കുന്നതിന് മുമ്പ് അടുത്ത റൺ ഓട്ടോമാറ്റിക്കായി ആരംഭിക്കുന്നു, നോഡ് വിലാസം ഒരിക്കലും മാറില്ല.

## സ്ഥിര വിലാസങ്ങൾ

- നോഡ്: `vless.my-github.ccwu.cc:443` (VLESS + WS + TLS)
- പാനൽ: `https://panel.my-github.ccwu.cc/panel` (യൂസർനെയിം `admin`)

## ആദ്യ സജ്ജീകരണം

### 1. Cloudflare named tunnel

1. Cloudflare Dashboard → Zero Trust → Networks → Tunnels → Create tunnel (Cloudflared)
2. പേര് നൽകി token പകർത്തുക (`eyJ` എന്ന് തുടങ്ങുന്ന സ്ട്രിംഗ്)
3. Tunnel-ന്റെ Public hostnames-ൽ രണ്ട് വരികൾ ചേർക്കുക:
   - `vless.നിങ്ങളുടെഡൊമെയ്ൻ.com` → HTTP → `127.0.0.1:8080`
   - `panel.നിങ്ങളുടെഡൊമെയ്ൻ.com` → HTTP → `127.0.0.1:2053`

### 2. GitHub Secrets

Repo Settings → Secrets and variables → Actions, ഈ മൂന്നെണ്ണം ചേർക്കുക:

| Name | Value |
|------|-------|
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel token |
| `PANEL_PASSWORD` | പാനൽ ലോഗിൻ പാസ്‌വേഡ് |
| `VLESS_UUID` | നോഡ് UUID (സ്ഥിരം, ക്ലയന്റിൽ എന്നേക്കും സാധുവാണ്, https://1024tools.com/uuid/ എന്നതിൽ നിന്ന് ഉണ്ടാക്കുക, **ഈ UUID എഴുതി വെക്കുക, ലോഗുകളിൽ വ്യക്തമായി കാണിക്കില്ല, നോഡ് ലിങ്ക് കൂട്ടിച്ചേർക്കുമ്പോഴെല്ലാം ഇത് ആവശ്യമാണ്**) |

### 3. പ്രവർത്തിപ്പിക്കുക

Actions → 3x-ui VLESS Node → Run workflow (പാരാമീറ്ററുകൾ ഒഴിവാക്കാം, secret ഓട്ടോമാറ്റിക്കായി ഉപയോഗിക്കും).

ലോഗിൽ പൂർണ്ണമായ `vless://` ലിങ്ക് കാണിക്കും, v2rayN / v2rayNG / Streisand / NekoBox ക്ലയന്റിലേക്ക് ഇംപോർട്ട് ചെയ്യുക.

## ഓട്ടോമാറ്റിക് റീസ്റ്റാർട്ട്

- ഓരോ റണ്ണും `run_minutes` മിനിറ്റ് നീണ്ടുനിൽക്കും (സ്ഥിരം 290), അവസാനം അടുത്തത് ഓട്ടോമാറ്റിക്കായി ആരംഭിക്കും
- ഡൊമെയ്നും UUID-ഉം സ്ഥിരമാണ്, ക്ലയന്റ് ഒരിക്കൽ സജ്ജീകരിച്ചാൽ മതി
- നിർത്താൻ Actions-ൽ പോയി Cancel അമർത്തുക, റദ്ദാക്കിയാൽ പിന്നെ റീസ്റ്റാർട്ട് ഉണ്ടാകില്ല

## ശ്രദ്ധിക്കുക

- **വേഗത**: ട്രാഫിക് Cloudflare → GitHub സെർവറുകൾ (അമേരിക്ക/യൂറോപ്പ്) വഴിയാണ് പോകുന്നത്, താൽക്കാലിക/ലഘുവായ ഉപയോഗത്തിന് മാത്രം അനുയോജ്യം
- **ചെറിയ തടസ്സം**: ഓരോ 5 മണിക്കൂറിലും ഓട്ടോമാറ്റിക് റീസ്റ്റാർട്ട് സമയത്ത് ഏകദേശം 3-5 മിനിറ്റ് തടസ്സമുണ്ടാകും (പുതിയ മെഷീൻ ബൂട്ട്, 3x-ui ഇൻസ്റ്റാൾ, tunnel റീകണക്ട്), ക്ലയന്റ് ഓട്ടോമാറ്റിക്കായി വീണ്ടും കണക്ട് ചെയ്യും
- **നയം**: GitHub Actions-ൽ പ്രോക്സി പ്രവർത്തിപ്പിക്കുന്നത് ToS ലംഘനത്തിനും അക്കൗണ്ട് ബാൻ ചെയ്യപ്പെടാനും സാധ്യതയുണ്ട്, സ്വന്തം ഉത്തരവാദിത്തത്തിൽ ഉപയോഗിക്കുക
- `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` എന്നിവ പരസ്യമായ ഇടങ്ങളിൽ **പ്രസിദ്ധീകരിക്കരുത്**
