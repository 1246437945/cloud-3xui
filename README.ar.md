[简体中文](README.md) | [English](README.en.md) | [Русский](README.ru.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [العربية](README.ar.md) | [Čeština](README.cs.md) | [Dansk](README.da.md) | [Deutsch](README.de.md) | [Esperanto](README.eo.md) | [فارسی](README.fa.md) | [Suomi](README.fi.md) | [Français](README.fr.md) | [Ελληνικά](README.gr.md) | [Magyar](README.hu.md) | [Bahasa Indonesia](README.id.md) | [Italiano](README.it.md) | [മലയാളം](README.ml.md) | [Nederlands](README.nl.md) | [Norsk](README.no.md) | [Polski](README.pl.md) | [Português](README.ptbr.md) | [Română](README.ro.md) | [Türkçe](README.tr.md) | [Українська](README.ua.md) | [Tiếng Việt](README.vn.md)

# ☁️ cloud-3xui — عقدة VLESS مجانية على GitHub

أنشئ تلقائيًا لوحة 3x-ui وعقدة VLESS على الخوادم المجانية لـ GitHub Actions، مع اسم نطاق ثابت وتجديد تلقائي عند انقطاع الاتصال.

## مخطط المعمارية

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

## عملية التجديد التلقائي

```mermaid
flowchart TD
    A["手动 Run workflow\n(或上一轮自动触发)"] --> B["安装 3x-ui"]
    B --> C["API 创建 VLESS+WS 入站"]
    C --> D["启动 Cloudflare 命名隧道"]
    D --> E["输出节点链接"]
    E --> F["إبقاء نشطًا N دقيقة<br/>(إدخال يدوي، الافتراضي 290)"]
    F --> G["استدعاء API لتشغيل الجولة التالية<br/>(باستخدام القيمة الافتراضية 290)"]
    G --> A

    style G fill:#f6821f,color:#fff
```

## التجديد التلقائي

- **مدة الإبقاء تحددها أنت**: عند النقر على Run workflow املأ `run_minutes` يدويًا، ويُشغَّل بقدر ما تدخل من دقائق؛ وإذا تركته فارغًا تُستخدم القيمة الافتراضية 290
- **التشغيل التلقائي يستخدم القيمة الافتراضية**: عند نهاية كل جولة يُستدعى API لتشغيل الجولة التالية، واستدعاء API لا يحمل معلمات، لذلك كل جولة من جولات التجديد التلقائي هي 290 دقيقة
- للاختبار السريع املأ `5`، وللاستخدام اليومي اتركه فارغًا ليُستخدم الافتراضي 290
- النطاق وUUID ثابتان، فإعداد العميل مرة واحدة يبقى صالحًا دائمًا
- للإيقاف انتقل إلى Actions وانقر Cancel، ولن يُعاد التشغيل تلقائيًا بعد الإلغاء

## عناوين ثابتة

| الاستخدام | العنوان |
|------|------|
| العقدة | `vless.my-github.ccwu.cc:443` (VLESS + WebSocket + TLS) |
| اللوحة | `https://panel.my-github.ccwu.cc/panel` (اسم المستخدم `admin`) |

> ⚠️ أسماء النطاقات أعلاه مجرد أمثلة، استبدلها بنطاقك الخاص.

## خطوات التكوين التفصيلية

### الخطوة 1: إنشاء نفق مسمّى في Cloudflare


الموقع الرسمي لـ Cloudflare: https://www.cloudflare-cn.com (الصين) / https://www.cloudflare.com (دولي)
لوحة التحكم: https://dash.cloudflare.com

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

1. افتح [Cloudflare Dashboard](https://dash.cloudflare.com)، ومن القائمة اليسرى انتقل إلى **Zero Trust**
2. من اليسار **Networks** → **Tunnels**، ثم انقر **Create a tunnel** أعلى اليمين
3. اختر النوع **Cloudflared**، ثم انقر التالي
4. أعطِ النفق اسمًا (مثل `github-3xui`)، ثم احفظ
5. ستعرض الصفحة أمر تثبيت على الشكل التالي:
   ```
   cloudflared.exe service install eyJhIjoiYzEyNGI2...
   ```
   انسخ السلسلة الطويلة التي تبدأ بـ `eyJ`، وهي **Tunnel Token**، واحتفظ بها جيدًا
6. انقر التالي للانتقال إلى **Public hostnames** (مسارات التطبيقات المنشورة)، وانقر **Add a public hostname** لإضافة مسارين:

   **الأول (العقدة):**
   - Subdomain (النطاق الفرعي): `vless`
   - Domain (النطاق): اختر نطاقك
   - Service Type (نوع الخدمة): `HTTP`
   - URL: `127.0.0.1:8080`

   **الثاني (اللوحة):**
   - Subdomain (النطاق الفرعي): `panel`
   - Domain (النطاق): اختر نطاقك
   - Service Type (نوع الخدمة): `HTTP`
   - URL: `127.0.0.1:2053`

7. احفظ. سينشئ Cloudflare سجلات DNS تلقائيًا دون الحاجة إلى إضافة يدوية.

### الخطوة 2: توليد UUID

1. افتح https://1024tools.com/uuid/
2. انسخ الـ UUID المولّد (على الشكل `cbfc229f-9d34-4fef-9990-8baa82a7327b`)
3. **احفظ هذا الـ UUID جيدًا**، فلن يظهر بشكل صريح في سجلات سير العمل، وستحتاجه في كل مرة لتركيب رابط العقدة

### الخطوة 3: إضافة Secrets إلى GitHub

1. ادخل إلى هذا المستودع، وانقر على تبويب **Settings**
2. من اليسار **Secrets and variables** → **Actions**
3. انقر **New repository secret** أعلى اليمين، وأضف الثلاثة التالية بالترتيب:

   | Name | Secret (القيمة) | الوصف |
   |------|-------------|------|
   | `CF_TUNNEL_TOKEN` | الـ Tunnel Token المنسوخ من الخطوة 1 | بيانات اعتماد نفق Cloudflare |
   | `PANEL_PASSWORD` | كلمة المرور التي حددتها للوحة | كلمة مرور تسجيل الدخول إلى لوحة 3x-ui |
   | `VLESS_UUID` | الـ UUID المولّد في الخطوة 2 | UUID العقدة، ثابت لا يتغير |

### الخطوة 4: تشغيل سير العمل

1. انقر على تبويب **Actions** أعلى المستودع
2. من اليسار اختر **3x-ui VLESS Node**
3. انقر زر **Run workflow** على اليمين
4. في النموذج المنبثق:
   - **Panel password**: اتركه فارغًا (سيستخدم `PANEL_PASSWORD` secret تلقائيًا)
     كلمة مرور اللوحة (اتركها فارغة لاستخدام PANEL_PASSWORD secret)
   - **Run duration**: الافتراضي `290` (دقيقة)، وللاختبار الأول يمكن كتابة `5` للتحقق السريع؛ وللاستخدام اليومي اتركه فارغًا
     مدة التشغيل (بالدقائق، الافتراضي 290، بحد أقصى 340)
5. انقر الزر الأخضر **Run workflow** لبدء التشغيل

### الخطوة 5: الحصول على رابط العقدة

1. انتظر حوالي 2-3 دقائق، وانقر على المهمة الجارية للانتقال إلى صفحة التفاصيل
2. وسّع خطوة **Start named Cloudflare tunnel**، واطلع على مخرجات السجل:
   ```
   ==================================================
     面板 Panel: https://panel.my-github.ccwu.cc/panel  (admin / 你的密码 your password)

     VLESS 节点 node:
     vless://YOUR_UUID@vless.my-github.ccwu.cc:443?encryption=none&security=tls&sni=vless.my-github.ccwu.cc&type=ws&path=%2Fws#GitHub-3xui

     把 YOUR_UUID 换成你在 VLESS_UUID secret 里填的 UUID
     Replace YOUR_UUID with the UUID from your VLESS_UUID secret
   ==================================================
   ```
3. انسخ رابط `vless://`، واستبدل `YOUR_UUID` بـ UUID الخاص بك
4. الصقه في عميل v2rayN / v2rayNG / Streisand / NekoBox وغيرها لبدء الاستخدام

### الخطوة 6: التحقق من التجديد التلقائي (اختياري)

1. في التشغيل الأول، اضبط المدة على `5` دقائق
2. بعد 5 دقائق، اطلع على سجل الخطوة الأخيرة **Trigger next run**، ويجب أن يظهر:
   ```
   HTTP 204
   已触发下一次运行 Next run triggered
   ```
3. ارجع إلى قائمة Actions، ويجب أن يظهر تشغيل جديد مُفعَّل تلقائيًا
4. بعد التأكد، أعد التشغيل وأعد المدة إلى `290` للاستخدام العادي

## شرح خطوات سير العمل

| الخطوة | العربي | English | الوصف |
|------|------|---------|------|
| Install 3x-ui | تثبيت 3x-ui | Install 3x-ui | تثبيت اللوحة بشكل غير تفاعلي بالسكربت الرسمي |
| Create VLESS+WS inbound via API | إنشاء مدخل VLESS+WS عبر API | Create VLESS+WS inbound via API | تسجيل الدخول برمز API وإنشاء المدخل تلقائيًا |
| Start named Cloudflare tunnel | تشغيل نفق Cloudflare المسمّى | Start named Cloudflare tunnel | الاتصال بالنفق ذي النطاق الثابت باستخدام الرمز |
| Keep alive | الحفاظ على التشغيل | Keep alive | السكون لعدد الدقائق المحدد مع مراقبة حالة الخدمات |
| Trigger next run | تشغيل الدورة التالية | Trigger next run | استدعاء API لبدء الدورة التالية تلقائيًا |

## الأسئلة الشائعة

**Q: الـ UUID في رابط العقدة يظهر كـ `***` في السجلات؟**
A: هذه آلية حماية الـ secrets في GitHub. استبدل `YOUR_UUID` بالـ UUID الذي أدخلته في `VLESS_UUID` secret.

**Q: اختبار العميل يفشل (-1)؟**
A: تحقق من ثلاثة أمور: ① هل تم استبدال الـ UUID بشكل صحيح؛ ② هل إصدار العميل حديث (v2rayN 7.x+)؛ ③ هل نفق Cloudflare متصل بشكل طبيعي (اطلع على السجلات).

**Q: لم يتم تشغيل إعادة التشغيل التلقائية؟**
A: اطلع على سجل خطوة **Trigger next run**. إذا ظهر رمز HTTP غير 204، تحقق من إعدادات صلاحيات Actions في المستودع.

**Q: هل من الطبيعي أن ينقطع الاتصال لبضع دقائق كل 5 ساعات؟**
A: نعم، هذا طبيعي. من نهاية الجولة السابقة إلى اتصال النفق في الجولة التالية يستغرق حوالي 3-5 دقائق، وسيعيد العميل الاتصال تلقائيًا.

**Q: أريد إيقاف التجديد التلقائي؟**
A: اذهب إلى صفحة Actions وألغِ (Cancel) التشغيل الحالي، ولن يتم التشغيل تلقائيًا بعد ذلك.

## ملاحظات

- **السرعة**: تمر الحركة عبر Cloudflare → خوادم GitHub (أمريكا/أوروبا)، وهي مناسبة للاستخدام المؤقت/الخفيف فقط
- **انقطاع قصير**: عند إعادة التشغيل التلقائية كل 5 ساعات ينقطع الاتصال حوالي 3-5 دقائق (تشغيل جهاز جديد، تثبيت 3x-ui، إعادة اتصال النفق)، وسيعيد العميل الاتصال تلقائيًا
- **الامتثال**: تشغيل وكيل على GitHub Actions قد ينتهك شروط الاستخدام ويؤدي إلى إغلاق الحساب، قيّم ذلك بنفسك
- **الأمان**: لا تنشر `CF_TUNNEL_TOKEN` / `PANEL_PASSWORD` / `VLESS_UUID` في أماكن عامة
