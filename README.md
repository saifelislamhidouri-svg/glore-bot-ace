# GLORE ACE ᵛˣ — Squad Bot + Dashboard

بوت سكواد Free Fire مع لوحة تحكم ويب احترافية.
Telegram: **@ace_vvxx**

---

## ✅ ما تم إصلاحه (What was fixed)

### 1) خطأ حرج في اللوحة (Critical dashboard bug)
كان المسار `TEMPLATE_PATH` يشير إلى مجلد غير موجود `XEROX_MODS/index.html`،
لذلك كانت اللوحة تعرض صفحة بديلة بسيطة (stub) بدل الواجهة الحقيقية.

**الحل:** الآن يتم البحث عن القالب في عدة مسارات تلقائيًا ويُخدَّم `index.html`
الحقيقي في كل طلب (بدون الحاجة لإعادة تشغيل السيرفر).

### 2) إعادة العلامة التجارية (Rebranding → GLORE ACE ᵛˣ)
- عنوان الصفحة والناڤبار والأسفل: **GLORE ACE ᵛˣ**
- شعار SVG محلي `logo.svg` (بدون الاعتماد على روابط خارجية)
- ردود الـ API: `BRAND`, `DEVLOVER`, `TELEGRAM`
- بانر الكونسول في `app.py`
- مفتاح الـ API الجديد: `GLORE_ACE` (مع دعم المفتاح القديم للتوافق)

### 3) تقوية البوت (Bot hardening)
- **Exponential backoff + jitter** لإعادة الاتصال بدل التأخير الثابت.
- **Pause-aware loops**: العامل يتوقف بانتظار الاستئناف دون أن يموت.
- **Watchdog supervisor**: يعيد تشغيل أي عامل مات تلقائيًا كل 30 ثانية.
- **start_worker_for()**: تشغيل العامل بشكل idempotent (بدون تكرار).
- **/health** endpoint للمراقبة.
- **خدمة الملفات الثابتة** (logo/favicon) بأمان ضد path traversal.
- واجهة إدارة حسابات كاملة: عرض / إيقاف / استئناف / إعادة تشغيل / حذف.

---

## 🎯 وضع كلاش سكواد مصنّف + جمع الڨلوري (Clash Squad Ranked + Glory Farm)

البوت الآن يلعب **كلاش سكواد مصنّف (Clash Squad Ranked)** افتراضيًا ويجمع **ڨلوري (Glory)**.

### كيف يعمل؟
- **محرّك الأنماط (Mode Engine):** دالة `mode_profile()` تُرجع معرّفات النمط الحالية
  (`mode_id`, `map_id`, `tag`, `ranked`, `glory`) وتُستخدم في كل مكان داخل البوت.
- **الافتراضي:** `GAME_MODE = "CLASH_SQUAD"` مع `CS_MODE_ID = 2` و `CS_MAP_ID = 1` (Bermuda).
- **الرجوع إلى الباتل رويال:** بدون تعديل كود، شغّل:
  ```bash
  GLORE_GAME_MODE=BATTLE_ROYALE python3 app.py
  ```
- **تغيير خريطة كلاش سكواد:** `GLORE_CS_MAP=KALAHARI` (الخيارات: `BERMUDA`,
  `BERMUDA_REMASTERED`, `KALAHARI`, `PURGATORY`, `NEXTERRA`).
- **حزمة البحث المخصّصة (اختياري):** ضع ملف `clash_squad_packet.hex` بجانب `app.py`
  ليستخدمه البوت كما هو في مطابقة كلاش سكواد (إن وُجد). بدون الملف يستخدم البوت
  قالب `StartUDPMatch` العام.

### جمع الڨلوري (Glory) — إنتاج مضاعف
- كل مباراة كلاش سكواد مكتملة تمنح **الأساس +32 ڨلوري** (`GLORY_PER_CS_MATCH`) ×
  **مضاعف المصنّف 1.5×** (`GLORY_RANKED_MULTIPLIER`) = **+48 ڨلوري مضمون لكل مباراة**.
- **مكافأة السلسلة (Streak):** +6 ڨلوري إضافية لكل مباراة متتالية (حتى +48).
- **مكافأة الأداء (Kills):** من +0 إلى +20 ڨلوري عشوائية لكل مباراة.
- مباراة باتل رويال تمنح **+8** (`GLORY_PER_BR_MATCH`).
- يُحفظ المجموع والسلسلة في `glory_cache.json` ويُعرض مباشرة في اللوحة:
  - بطاقة **✨ Glory Farmed** في نظرة عامة المحرّك (مع عرض السلسلة القصوى 🔥).
  - عمود **✨ Glory** داخل كل حساب.
- شارة النمط الحيّة أعلى الصفحة تُظهر الوضع الفعلي (Clash Squad Ranked · Bermuda).

### ⚡ زيادة الحركة والنشاط داخل المباراة (Activity Boost)
البوت الآن **يتحرّك بنشاط ~92% من وقت المباراة** بدل الوقوف الخامل، عبر ثلاث عمّال متزامنة:
- **حركة مستمرة (`movement_worker`):** حزمة الموقع `2001` تُرسل كل **160ms** بسرعة `1.25–1.75×`
  ونصف قطر حركة أوسع (`220`) مع **اهتزاز مسار (jitter)** حتى لا تتكرر الحركة بشكل آلي.
  توزيع الحالات: مشي 30% · جري 38% · عدْو 18% · قفز 6% · انخفاض 4% · بروك 2% · سكون 2% فقط.
- **مضاد الخمول (`anti_afk_worker`):** حزم إطلاق/إيقاف نار كل **0.7–1.7s**.
- **اندفاعة قتال (`combat_burst_worker`):** رشقة نار سريعة (6 طلقات) كل **4–9s** لتبدو الشخصية
  منخرطة في القتال فعليًا.
- عدّادات حيّة **⚡ Live Activity** في اللوحة تُظهر عدد حزم الحركة وعدد الطلقات.

### الإعدادات (Config) في `app.py`
| المتغيّر | القيمة الافتراضية | الوصف |
|---------|------------------|-------|
| `GAME_MODE` | `CLASH_SQUAD` | النمط النشط |
| `CS_MODE_ID` | `2` | معرّف نمط كلاش سكواد |
| `CS_MAP_ID` | `1` | خريطة كلاش سكواد الافتراضية |
| `CS_RANKED` | `True` | مطابقة مصنّفة |
| `CS_MAPS` | Bermuda…NEXTerra | جدول خرائط كلاش سكواد |
| `GLORY_PER_CS_MATCH` | `32` | ڨلوري الأساس لكل مباراة كلاش سكواد |
| `GLORY_RANKED_MULTIPLIER` | `1.5` | مضاعف المباريات المصنّفة |
| `GLORY_STREAK_BONUS` | `6` | مكافأة السلسلة لكل مباراة متتالية |
| `GLORY_KILL_BONUS_MAX` | `20` | أقصى مكافأة أداء (kills) |
| `GLORY_PER_BR_MATCH` | `8` | ڨلوري لكل مباراة باتل رويال |
| `MOVEMENT_TICK_MS` | `160` | الفاصل بين حزم الحركة (ms) |
| `MOVEMENT_SPEED_MIN/MAX` | `1.25/1.75` | سرعة الحركة |
| `MOVEMENT_STATE_IDLE_CHANCE` | `0.02` | احتمال السكون (منخفض جدًا) |
| `ANTI_AFK_MIN/MAX_INTERVAL` | `0.7/1.7` | فاصل طلقات مضاد الخمول (ثانية) |
| `COMBAT_BURST_SHOTS` | `6` | عدد طلقات الاندفاعة القتالية |
| `ENABLE_COMBAT_BURST` | `True` | تفعيل اندفاعات القتال الدورية |

> ⚠️ ملاحظة: قيم `mode_id`/`map_id` خاصة ببروتوكول اللعبة. إن كان سيرفرك يتوقّع
> قيمًا مختلفة، عدّلها من الإعدادات أو مرّر حزمة بحث جاهزة عبر `clash_squad_packet.hex`.

---

## 📁 الملفات المعدّلة (Modified files)
| الملف | الوصف |
|------|--------|
| `app.py` | محرك البوت — تقوية اللوبات + watchdog + العلامة + **محرّك أنماط كلاش سكواد/ڨلوري + زيادة الحركة والقتال** |
| `dashboard_server.py` | سيرفر اللوحة — إصلاح القالب + العلامة + /health + static + **تتبّع الڨلوري والنمط** |
| `index.html` | الواجهة الجديدة بالكامل GLORE ACE ᵛˣ + إدارة الحسابات + **بطاقة الڨلوري وشارة النمط** |
| `logo.svg` | شعار جديد محلي |
| `README.md` | هذا الملف |

---

## 🚀 التشغيل (Run)

```bash
pip install -r requirements.txt
python3 app.py
```

ثم افتح: `http://localhost:20333`

### نقاط الـ API
| Method | Path | الوصف |
|--------|------|--------|
| GET | `/` | لوحة التحكم |
| GET | `/health` | فحص الحالة |
| GET | `/api/stats` | إحصائيات + الحسابات |
| GET | `/api/logs` | السجلات |
| GET | `/api/guildinfo?guild_id=&region=&key=GLORE_ACE` | معلومات الجيلد |
| POST | `/api/clanjoin` | طلب الانضمام |
| POST | `/api/account/add` | إضافة حساب |
| POST | `/api/account/delete` | حذف حساب |
| POST | `/api/account/pause` | إيقاف/استئناف حساب |
| POST | `/api/account/pause_all` | إيقاف/استئناف الكل |
| POST | `/api/account/restart` | إعادة تشغيل حساب |
| POST | `/api/logs/clear` | مسح السجلات |

---

## ⚙️ الإعدادات (Config)
- المنفذ: `WEB_PORT = 20333` في `app.py`
- مفتاح الـ API: `API_KEY = "GLORE_ACE"` في `dashboard_server.py`
- الحسابات: `accounts.json`
- الكاش: `token_cache.json`

---

**GLORE ACE ᵛˣ** • Telegram [@ace_vvxx](https://t.me/ace_vvxx)
