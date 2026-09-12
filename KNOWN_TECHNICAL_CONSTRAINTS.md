# KNOWN_TECHNICAL_CONSTRAINTS.md
## Phase 0 — القيود التقنية المعروفة

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | سجل حي يُحدَّث بعد كل PoC |
| التاريخ | 2026-09-12 |
| القاعدة | كل قيد يجب أن يحمل: الوصف · المصدر · الأثر · الأمل في التخفيف · حالة التحقق على جهازنا |

---

## 1. قيود منصة Android

### C-01 — W^X: منع تنفيذ ملفات داخل بيانات التطبيق
| الحقل | القيمة |
|---|---|
| الوصف | على Android >= 10، التطبيق الذي يستهدف `targetSdkVersion >= 29` لا يمكنه `exec()` ملفات داخل مجلد بياناته (`/data/data/...`) |
| المصدر | سياسة SELinux (commit `0dd738d8`) [1](https://www.reddit.com/r/androiddev/comments/b2inbu/psa_android_q_blocks_executing_binaries_in_your/) · توثيق Termux [1](https://github.com/termux/termux-packages/wiki/Termux-execution-environment) |
| التصنيف | **VERIFIED** |
| الأثر | **حرج** — يمنع تنفيذ أي binary داخل rootfs Kali التي ننزلها وقت التشغيل |
| مسارات التخفيف | (1) التنفيذ عبر `/system/bin/linker*` — مُطبَّق في `termux-exec` ويعمل مع `targetSdk >= 28` على Android >= 10 [1](https://github.com/termux/termux-exec-package) · (2) شحن binaries كـ native libs داخل APK (مجلد للقراءة فقط) · (3) خفض targetSdk — **مرفوض** لأنه يتعارض مع متطلبات المتجر والأمن |
| التحقق على جهازنا | **UNKNOWN** → POC-002 |
| ملاحظة | الاستراتيجية التي تعتمدها Termux هي البقاء على `targetSdk 28` [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) — خيار غير متاح لتطبيق يستهدف التوزيع عبر المتجر |

### C-02 — قاتل العمليات الوهمية (Phantom Process Killer)
| الحقل | القيمة |
|---|---|
| الوصف | Android 12+ يفرض حدًا أقصى للعمليات المتفرعة عن التطبيقات (`DEFAULT_MAX_PHANTOM_PROCESSES = 32`) ويقتل العمليات التي تستهلك CPU بكثافة. الحد **لكل النظام** لا لكل تطبيق |
| المصدر | [1](https://cosyra.com/guides/termux-signal-9-fix.html) · [2](https://github.com/sabamdarif/termux-desktop/blob/main/docs/disable-phantom-process-killing.md) |
| التصنيف | **VERIFIED** |
| الأثر | **حرج** — بيئة Linux كاملة (shell + خدمات + ديسكتوب) تتجاوز 32 عملية بسهولة |
| مسارات التخفيف | (1) تعطيل عبر Developer Options على Android 14+: "Disable child process restrictions" · (2) عبر ADB: `settings put global settings_enable_monitor_phantom_procs false` · (3) تصميم التطبيق لتقليل عدد العمليات · (4) كشف القتل + استرداد (البند 39) · (5) إرشاد المستخدم |
| التحقق على جهازنا | **UNKNOWN** → POC-006 |
| ⚠️ خطر تصميمي | إذا تطلب الحل تدخلًا يدويًا من المستخدم (Developer Options)، فإن ذلك **يتعارض جزئيًا** مع "One-Tap Startup" (البند 35). يجب معالجة هذا صراحةً في PHASE 1: إما التخفيف البرمجي أو توثيق الخطوة كجزء من الإعداد |

### C-03 — 16 KB Page Size
| الحقل | القيمة |
|---|---|
| الوصف | Android 15 يدعم تشغيل النظام بصفحات ذاكرة 16 KB؛ كل تطبيق يحتوي كودًا أصليًا يجب أن يُعاد بناؤه بمحاذاة 16 KB وإلا لا يعمل على الأجهزة ذات 16 KB |
| المصدر | [5](https://android-developers.googleblog.com/2024/08/adding-16-kb-page-size-to-android.html) |
| التصنيف | **VERIFIED** |
| الأثر | كل binary نَشحنه (proot، X server، مساعدات) يجب أن يكون page-size agnostic |
| مسارات التخفيف | البناء بـ `-Wl,-z,max-page-size=16384` وNDK حديث · اختبار على محاكي/جهاز 16 KB |
| التحقق على جهازنا | **UNKNOWN** → POC-012 (والتحقق من حجم صفحة جهازنا فعلًا) |
| ملاحظة | Termux نفسها واجهت هذه المسألة في issues معلومة [3](https://github.com/termux/termux-app/issues/4185) [4](https://github.com/termux/termux-packages/issues/21688) |

### C-04 — user namespaces غير متاحة
| الحقل | القيمة |
|---|---|
| الوصف | `unshare(CLONE_NEWUSER)` يرجع `EPERM` على أجهزة Android غير مروّتة |
| المصدر | [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c) |
| التصنيف | **DOCUMENTED** (يحتاج تأكيد جهازي) |
| الأثر | يُسقط كل خيارات الـ containers الحقيقية |
| التحقق على جهازنا | **UNKNOWN** → POC-003 |

### C-05 — لا KVM/AVF لتطبيقات طرف ثالث
| الحقل | القيمة |
|---|---|
| الوصف | الوصول المباشر إلى hypervisor مقيد؛ AVF يسمح فقط بصور موقّعة من Google/الشركة المصنّعة |
| المصدر | [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/) |
| التصنيف | **DOCUMENTED** |
| الأثر | مسار VM مغلق Rootless |

### C-06 — تقييد `/proc/net`
| الحقل | القيمة |
|---|---|
| الوصف | المستخدم غير المروّت لا يستطيع الوصول إلى `/proc/net` |
| المصدر | [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) |
| التصنيف | **DOCUMENTED** |
| الأثر | أدوات مثل `netstat` تتأثر |

---

## 2. قيود PRoot (من المصدر الرسمي)

| # | القيد | الأثر | التخفيف الممكن |
|---|---|---|---|
| C-07 | اعتراض كل نداء نظام عبر `ptrace` | أعباء أداء في الأعمال كثيفة الملفات | قياس مبكر؛ تجنب أعباء غير ضرورية |
| C-08 | لا وحدات kernel (FUSE، iptables targets، cgroups مخصّصة) | أدوات معطّلة | توثيق القيود للمستخدم |
| C-09 | لا root حقيقي (UID/GID remap فقط) | `mount`، `iptables`، `sudo` تفشل | توثيق + بدائل |
| C-10 | لا systemd/OpenRC | لا إدارة خدمات بنظام init | إدارة عمليات خفيفة خاصة بنا (Process Manager) |
| C-11 | لا namespaces/cgroups | لا عزل ولا تحديد موارد | قبول القيد في MVP (البند 82) |
| C-12 | لا تداخل (proot داخل proot) | لا "container داخل container" | تصميم مسطح للجلسات |
| C-13 | سرعة تطوير upstream منخفضة (5.4.0 منذ 2023) [2](https://github.com/proot-me/PRoot/releases) | مخاطر صيانة طويلة الأمد | تقييم البدائل (مثل بدائل LD_PRELOAD) + تجريد الطبقة خلف `RuntimeBackend` |

المصدر لكل ما سبق: [1](https://github.com/termux/proot-distro)

---

## 3. قيود نظام الملفات / التخزين

| # | القيد | الحالة | الملاحظة |
|---|---|---|---|
| C-14 | وحدة التخزين الخارجية/SD أبطأ بكثير وأحيانًا بلا صلاحيات تنفيذ | DOCUMENTED | يُنصح بالتخزين الداخلي [2](https://termuxtools.com/proot-distro-linux-termux/) |
| C-15 | أنظمة ملفات Android قد لا تدعم symlinks/صلاحيات Unix الكاملة | UNKNOWN | يجب اختباره (POC-004) قبل الاعتماد على `tar` لاستعادة النسخ |
| C-16 | حجم rootfs Kali (full ~1.7 GB / minimal ~135 MB) | DOCUMENTED | يفرض متطلبات مساحة واضحة في واجهة الإعداد [1](https://github.com/zulfi0/install_rootfs_android) |
| C-17 | ضغط/فك ضغط rootfs يستهلك وقتًا ومساحة مؤقتة | UNKNOWN | قياس في POC-013 |

---

## 4. قيود دورة الحياة (Android Lifecycle)

| # | القيد | الحالة | الملاحظة |
|---|---|---|---|
| C-18 | Android قد يقتل عملية التطبيق تحت ضغط الذاكرة | VERIFIED كسلوك منصة | يتطلب Recovery Manager (البند 39) |
| C-19 | خدمات الخلفية مقيدة؛ الخدمة الأمامية (Foreground Service) مطلوبة للبقاء | VERIFIED كسلوك منصة | إشعار دائم إلزامي |
| C-20 | قيود إضافية من طبقة OEM (HyperOS) على العمليات الطويلة | **UNKNOWN** | يجب اختباره مبكرًا — قد يكون عامل حاسم |
| C-21 | إيقاف الشاشة قد يجمّد/يبطئ التنفيذ | UNKNOWN | POC-011 |

---

## 5. قيود العرض والصوت

| # | القيد | الحالة |
|---|---|---|
| C-22 | لا تسريع GPU مضمون (Adreno 810 غير مثبت الدعم) | UNKNOWN → POC-008 |
| C-23 | الصوت من Linux إلى Android غير مضمون الاستقرار | UNKNOWN (البند 26: لا نجعله blocker) |
| C-24 | دعم الشاشة الخارجية/الدوران | UNKNOWN |

---

## 6. قيود الأمن (ملخص — التفاصيل في SECURITY_RESEARCH.md)

| # | القيد | الحالة |
|---|---|---|
| C-25 | لا عزل أمني حقيقي بين "Linux" وبقية التطبيق | VERIFIED [1](https://github.com/termux/proot-distro) |
| C-26 | المستخدم غير root يظهر كـ root داخل proot | VERIFIED رسميًا [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) |
| C-27 | rootfs مُنزَّلة من الإنترنت = سطح هجوم | يتطلب تحقق SHA256/توقيع (البند 52) |

---

## 7. سجل التحديثات

| التاريخ | التغيير |
|---|---|
| 2026-09-12 | إنشاء السجل من نتائج البحث الأولي (27 قيدًا مسجلة) |
| — | سيُحدَّث بعد كل PoC: كل قيد يثبت أو يُنفى يُحوَّل إلى VERIFIED/UNSUPPORTED مع دليل ومصدر |
