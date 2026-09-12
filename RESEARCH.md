# RESEARCH.md
## Phase 0 — Deep Research | النتائج الرئيسية

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 — DEEP RESEARCH |
| الحالة | **NOT STARTED → IN PROGRESS** (بحث أولي مكتمل، بحث جهازي مطلوب) |
| التاريخ | 2026-09-12 |
| الجهاز المرجعي | Android 15 · ARM64 · Snapdragon 7s Gen 4 (Adreno 810) · ~12 GB RAM · بدون Root |
| مستوى الأدلة | مزيج من **VERIFIED** (مصدر رسمي/كود) و **DOCUMENTED** و **UNKNOWN** (يحتاج اختبار جهاز) |
| المخرجات المرتبطة | OPEN_SOURCE_COMPARISON.md · RUNTIME_COMPARISON.md · GRAPHICS_RESEARCH.md · NETWORK_RESEARCH.md · KNOWN_TECHNICAL_CONSTRAINTS.md · DEVICE_COMPATIBILITY.md · SECURITY_RESEARCH.md · RESEARCH_SOURCES.md |

> ⚠️ **قاعدة هذا المستند (البند 6 و73 من PRD):** لا يُستخدم وصف "يعمل" إلا مع دليل. كل ما لم يُختبر على الجهاز المرجعي يُكتب **UNKNOWN** مع "كيف نتحقق".

---

## 1. الملخص التنفيذي

الخلاصة المختصرة بعد البحث الأولي:

1. **لا يوجد اليوم مسار "container حقيقي" بدون Root على Android.** محاولات `unshare(CLONE_NEWUSER)` ترجع `EPERM` على أجهزة غير مُروّتة، لأن سياسات SELinux و/أو ضبط النواة تمنع إنشاء user namespaces من تطبيق عادي. الحالة: **DOCUMENTED** مع حاجة تأكيد على الجهاز [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c).
2. **PRoot هو المسار الواقعي الوحيد المُثبت مجتمعيًا لتشغيل rootfs توزيعة كاملة بدون Root**، لكنه ترجمة مسارات عبر `ptrace` وليس عزلًا kernelيًا: لا namespaces، لا cgroups، لا systemd، لا FUSE، لا iptables حقيقي [1](https://github.com/termux/proot-distro).
3. **أكبر عقبة هندسية ليست PRoot بل قيود Android نفسها:** (أ) منع `exec()` لملفات داخل بيانات التطبيق على `targetSdkVersion >= 29` (W^X)، (ب) قاتل العمليات الوهمية (Phantom Process Killer) بحد 32 عملية، (ج) متطلب 16 KB page size للتطبيقات ذات الكود الأصلي على Android 15.
4. **للعقبة (أ) حل موثق رسميًا في Termux:** تنفيذ ملفات ELF بتمريرها إلى `/system/bin/linker*` لتجاوز منع التنفيذ — مُطبَّق في `termux-exec` (إصدار 2.5.0) ويعمل مع `targetSdkVersion >= 28` على Android >= 10 [1](https://github.com/termux/termux-exec-package).
5. **الجرافيكس:** المسار الأكثر واقعية للـ MVP هو **Termux:X11 (XCB) +软件 Rendering**، مع بقاء تسريع GPU (Turnip/Zink/VirGL عبر KGSL) في خانة **EXPERIMENTAL** [1](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md) [2](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/).
6. **الشبكة:** التطبيق يستطيع الإنترنت/DNS/TCP/UDP بشكل طبيعي، لكن **raw sockets وmonitor mode وحقن الحزم غير متاحة Rootless** — ويجب نفيها صراحة في الواجهة لا الادعاء بها (البند 48 و64).

---

## 2. منهجية البحث

ترتيب المصادر المُتبع (حسب البند 16):

1. الكود المصدري والمستودعات الرسمية
2. الوثائق الرسمية (Android Developers، Kali.org، مستودعات المشاريع)
3. الإصدارات والتغييرات (Releases/Changelogs)
4. Issues وPull Requests المفتوحة
5. تقارير المجتمع (تُستخدم فقط كمؤشر، لا كدليل نهائي)

تصنيف الأدلة المُستخدم (البند 17):

| التصنيف | المعنى |
|---|---|
| VERIFIED | مُثبت بمصدر رسمي أو كود أو اختبار منشور |
| DOCUMENTED | مذكور في وثائق رسمية دون إثبات على جهازنا |
| PLAUSIBLE | منطقي لكن بلا مصدر كافٍ |
| EXPERIMENTAL | يعمل لدى البعض، غير مستقر أو غير مدعوم رسميًا |
| LIMITED | يعمل جزئيًا أو بشروط |
| UNSUPPORTED | غير مدعوم |
| UNKNOWN | لا دليل — يحتاج اختبارًا |
| BLOCKED | ممنوع تقنيًا في السياق الحالي |

---

## 3. نتائج البحث حسب المجال

### 3.1 Runtime (محرك التشغيل)

| التقنية | الحالة | الأدلة | الملاحظة |
|---|---|---|---|
| PRoot (proot-me) | **IMPLEMENTED** مجتمعيًا | v5.4.0 آخر إصدار upstream (2023-05-13) [2](https://github.com/proot-me/PRoot/releases) | سرعة تطوير منخفضة؛ حزم Debian تتابع 5.4.0 |
| PRoot-Distro | **VERIFIED** | وثائق القيود الرسمية [1](https://github.com/termux/proot-distro) | أداة إدارة صور/ـrootfs فوق proot |
| chroot | **UNSUPPORTED** Rootless | يتطلب `CAP_SYS_ADMIN` | متاح فقط في Root Mode |
| namespaces / unshare | **BLOCKED** Rootless (مرجّح) | `unshare(CLONE_NEWUSER) = EPERM` [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c) | يحتاج تأكيدًا على الجهاز (POC-003) |
| containers (runc/LXC/Docker) | **BLOCKED** Rootless | يعتمد على namespaces + cgroups | Docker daemon يحتاج صلاحيات kernel [5](https://www.sitepoint.com/hardened-mobile-dev-a-termux-docker-guide-for-grapheneos/) |
| VM / KVM / QEMU | **LIMITED** | `/dev/kvm` محجوب؛ AVF يسمح فقط بصور موقّعة من Google/الشركة المصنّعة [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/) | Rootless = QEMU TCG (محاكاة بطيئة جدًا) |
| AVF / Microdroid / pKVM | **UNSUPPORTED** لطرف ثالث | ليس متاحًا لتطبيقات عادية | قد يُدرس مستقبلًا فقط بصلاحيات نظام |

**قيود PRoot المؤكدة (من المصدر الرسمي):**
- الأداء: اعتراض كل نداء نظام عبر `ptrace` → أعباء واضحة في الأعمال كثيفة الملفات.
- لا وحدات kernel (FUSE، أهداف iptables مخصصة، تسلسلات cgroup مخصصة).
- لا root حقيقي (إعادة تعيين UID/GID فقط) → `mount` الحقيقي و`iptables` و`sudo` تفشل.
- لا systemd/OpenRC.
- لا namespaces ولا cgroups.
- لا تداخل: PRoot يرفض العمل داخل PRoot آخر.
[1](https://github.com/termux/proot-distro)

> 🔎 **قياس الأداء متضارب في المصادر الثانوية:** ادعاءات "~5% overhead" [2](https://termuxtools.com/proot-distro-linux-termux/) مقابل "10–15% على العمليات كثيفة I/O" [5](https://www.sitepoint.com/hardened-mobile-dev-a-termux-docker-guide-for-grapheneos/) — كلاهما **غير مقيس على جهازنا**. التصنيف: **UNKNOWN** → POC-005 (benchmark إلزامي قبل أي ادعاء).

### 3.2 قيود منصة Android (الأكثر تأثيرًا على المعمارية)

| القيد | الحالة | الدليل | الأثر على المشروع |
|---|---|---|---|
| W^X: منع `exec()` لملفات داخل بيانات التطبيق عند `targetSdk >= 29` | **VERIFIED** | سياسة SELinux (commit `0dd738d8`) [1](https://www.reddit.com/r/androiddev/comments/b2inbu/psa_android_q_blocks_executing_binaries_in_your/) و[1](https://github.com/termux/termux-packages/wiki/Termux-execution-environment) | **حرج:** يمنع تنفيذ أي binary داخل rootfs المحمّلة |
| الحل: التنفيذ عبر `/system/bin/linker*` | **VERIFIED** | `termux-exec` 2.5.0 يضيف `system_linker_exec` [1](https://github.com/termux/termux-exec-package) | مسار اعتماد محتمل — يجب POC |
| استراتيجية Termux: البقاء على `targetSdk 28` | **VERIFIED** | إشعار رسمي من فريق Termux [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) | **غير متاح لنا:** Play يفرض target API حديثًا |
| Phantom Process Killer (حد 32 عملية) | **VERIFIED** | حد `DEFAULT_MAX_PHANTOM_PROCESSES = 32` (لكل النظام لا لكل تطبيق) + تعطيل عبر Developer Options على Android 14+ [1](https://cosyra.com/guides/termux-signal-9-fix.html) | **حرج:** بيئة Linux تخلق عشرات العمليات |
| 16 KB page size على Android 15 | **VERIFIED** | كل تطبيق فيه كود أصلي يجب إعادة بنائه بمحاذاة 16 KB [5](https://android-developers.googleblog.com/2024/08/adding-16-kb-page-size-to-android.html) | يؤثر على كل binary نُشفّعه (proot، X server) |
| Termux و16 KB | **DOCUMENTED** | issues مفتوحة/مغلقة في termux-app وtermux-packages [3](https://github.com/termux/termux-app/issues/4185) [4](https://github.com/termux/termux-packages/issues/21688) | مؤشر على حجم العمل المطلوب |
| تقييد الوصول إلى `/proc/net` للمستخدم غير المروّت | **DOCUMENTED** | `netstat` وأدوات مشابهة لا تعمل [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) | يؤثر على أدوات Kali الشبكية |

### 3.3 Kali Linux

| البند | النتيجة | الحالة |
|---|---|---|
| NetHunter Rootless | رسمي من Offensive Security؛ يعمل فوق Termux + proot؛ سطح المكتب عبر **KeX** (أساسه VNC) [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) | **VERIFIED** (رسمي) |
| قيود NetHunter Rootless المعلنة | بعض الأدوات تعمل بقيود (metasploit بدون دعم قاعدة بيانات)؛ `top` لا يعمل على جهاز غير مروّت؛ مستخدم غير root يظل "root" داخل proot | **VERIFIED** (مصدر رسمي) |
| rootfs رسمي ARM64 | `https://kali.download/nethunter-images/current/rootfs/kalifs-arm64-full.tar.xz` (~1.7 GB) و `kalifs-arm64-minimal.tar.xz` (~135 MB) [1](https://github.com/zulfi0/install_rootfs_android) | **DOCUMENTED** — يجب التحقق من SHA256/التوقيع من مصدر رسمي |
| NetHunter Pro / Full | يستهدف أجهزة مروّتة بنواة/صورة مخصصة | خارج نطاق MVP |
| بناء rootfs مخصص | موثق رسميًا عبر `debootstrap` على ARM [2](https://www.kali.org/docs/development/kali-linux-arm-chroot/) | **VERIFIED** (رسمي) — بديل للمصادر الجاهزة |

> ⚠️ **تنبيه أمني (البند 52):** رابط rootfs أعلاه مأخوذ من مستودع طرف ثالث، وليس من صفحة Kali الرسمية مباشرة. قبل الاعتماد يجب الحصول على الرابط والتحقق (SHA256/توقيع) من نطاق `kali.org` / `kali.download` الرسمي. الحالة الحالية: **UNKNOWN** → مهمة تحقق (V-01).

### 3.4 الرسوميات (ملخص — التفاصيل في GRAPHICS_RESEARCH.md)

- **Termux:X11**: يتطلب Android 8+؛ مكوّن من تطبيق Android + حزمة مرافقة؛ بُني بدايةً على XWayland ثم انتقل إلى XCB؛ يُوزَّع عبر nightly releases [3](https://ivonblog.com/en-us/posts/termux-x11/) [2](https://termux-x11.en.uptodown.com/android). الحالة: **IMPLEMENTED** مجتمعيًا.
- **VirGL**: مسار ناضج نسبيًا: Android GL/ES → خادم VirGL في Termux → برنامج `virpipe` داخل proot [1](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md).
- **Turnip + Zink عبر KGSL**: وصول مباشر لـ Adreno من داخل proot بدون VirGL؛ أدى إلى Vulkan 1.3 / OpenGL 4.6 / GLES 3.2 مع تحسين 2.5x إلى 4-5x مقابل virglrenderer — لكنه **يعتمد على patches غير رسمية (KGSL + DRI3 في Termux:X11)** وبرامج تشغيل مبنية مسبقًا [1](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/) [3](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md).
- **جهازنا (Adreno 810)**: الانتماء لعائلة Adreno 8xx [1](https://www.91mobiles.com/processor/qualcomm-snapdragon-7s-gen-4-pdp). دعم Turnip لأحدث 830/840 أُضيف حديثًا وبشكل **غير رسمي** [2](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/). دعم Adreno 810 تحديدًا: **UNKNOWN** → POC-008.

### 3.5 الشبكة (ملخص — التفاصيل في NETWORK_RESEARCH.md)

- الإنترنت/DNS/TCP/UDP من داخل Linux Rootless: **VERIFIED** عمليًا (كل توزيعات proot تعمل على مستودعات الحزم).
- raw sockets / monitor mode / حقن الحزم: **UNSUPPORTED** Rootless — ليست مسألة تنفيذ بل مسألة صلاحيات kernel وتعريفات Wi-Fi.
- التقاط الحزم Rootless ممكن فقط عبر محاكاة VPN (مثل PCAPdroid) [1](https://github.com/emanuele-f/PCAPdroid) → **LIMITED**، ولا يعطي وصولًا للطبقة الثانية.

### 3.6 المحاكاة والتوافق (Phase 10 مستقبلًا)

- **FEX-Emu** و**Box64/Box86**: طبقات ترجمة x86/x86_64 في مساحة المستخدم [2](https://byteiota.com/fex-emu-x86-arm64-linux/) [3](https://www.murtpoiss.ee/fex-emu-run-x86-applications-on-arm64-linux-devices/). الحالة على Android: **EXPERIMENTAL**؛ تُستخدم في مشاريع مثل Winlator. ليست ضمن MVP (البند 82).

---

## 4. إجابات Phase 0 (البند 78)

| السؤال | الإجابة المختصرة | الحالة |
|---|---|---|
| أفضل طريقة لتشغيل Linux على Android 15 ARM64 بدون Root؟ | PRoot (مع إدارة rootfs منفصلة) — لا يوجد بديل kernelي مثبت | **VERIFIED** كواقع، لا كأفضلية مطلقة |
| أفضل طريقة لتشغيل Kali ARM64؟ | rootfs رسمي Kali + PRoot؛ NetHunter Rootless هو السابقة الرسمية | **DOCUMENTED** |
| القيود الحقيقية لـ PRoot؟ | ptrace overhead، لا namespaces/cgroups/systemd/FUSE/iptables/root حقيقي/تداخل | **VERIFIED** [1](https://github.com/termux/proot-distro) |
| هل namespaces قابلة للاستخدام؟ | الراجح: لا على جهاز غير مروّت (`EPERM`) | **DOCUMENTED** + يحتاج POC |
| هل يمكن تشغيل container حقيقي بدون Root؟ | لا، بالمعنى kernelي | **BLOCKED** |
| أفضل طريقة لديكستوب حقيقي؟ | X11 عبر Termux:X11 (XCB) + software rendering للـ MVP | **DOCUMENTED** + POC |
| هل تسريع GPU ممكن؟ | ممكن تجريبيًا عبر Turnip/Zink/VirGL على Adreno، باعتماد patches غير رسمية | **EXPERIMENTAL** |
| أفضل ديسكتوب للـ MVP؟ | الأرجح XFCE أو LXQt — **لم يُقس بعد** | **UNKNOWN** → POC-009 |
| ما يمكن/لا يمكن Rootless في الشبكة؟ | TCP/UDP/DNS/HTTP نعم؛ raw sockets/monitor mode/حقن لا | **VERIFIED** كحدود منصة |
| حدود العزل؟ | عزل ملفات (مسارات) فقط؛ لا عزل عمليات/kernel/شبكة حقيقي | **VERIFIED** [1](https://github.com/termux/proot-distro) |
| الاختناقات المتوقعة؟ | I/O على نظام ملفات Android، ptrace، استخراج rootfs، عدد العمليات | **UNKNOWN** (قياس مطلوب) |
| ماذا يحدث عند قتل Android للعملية؟ | قتل شجرة العمليات؛ يجب كشف الحالة + تنظيف + إعادة إقلاع | **UNKNOWN** → POC-011 |
| أفضل طريقة لإدارة rootfs؟ | تحميل متدرّج + تحقق SHA256 + فك ضغط تدريجي على وحدة تخزين داخلية (لا SD) | **DOCUMENTED** (قياس مطلوب) |
| ما سيبقى مستحيلًا Rootless؟ | raw packets، monitor mode، حقن، USB HID/متقدم، تعريفات kernel، systemd | **VERIFIED** كحدود منصة |

---

## 5. التقرير النهائي لـ Phase 0 (البند 80)

### A — ما نعرفه (What We Know)
1. PRoot يعمل Rootless ويدعم rootfs توزيعات كاملة — مثبت مجتمعيًا وموثق رسميًا في PRoot-Distro.
2. قيود PRoot موثقة من مصدرها الرسمي (قائمة كاملة أعلاه).
3. NetHunter Rootless سابقة **رسمية** من Kali لتشغيل Kali Rootless فوق proot، بواجهة KeX (VNC).
4. قيود Android (W^X، phantom processes، 16 KB pages) موثقة رسميًا ولها آثار معمارية مباشرة.
5. يوجد حل موثق ومنشور لمشكلة W^X: التنفيذ عبر `/system/bin/linker*`.
6. لا KVM/AVF لتطبيقات طرف ثالث → مسار VM مغلق Rootless.

### B — ما لا نعرفه (What We Don't Know)
1. ما إذا كان `unshare`/namespaces معطّلًا على **نواة جهازنا تحديدًا** (OEM/HyperOS).
2. الأداء الفعلي لـ PRoot على Snapdragon 7s Gen 4 (لا أرقام مقيسة).
3. ما إذا كان حل `system_linker_exec` يعمل على Android 15 + HyperOS (التوثيق يغطي Android >= 10).
4. حالة دعم Turnip/KGSL لـ Adreno 810 تحديدًا.
5. استهلاك RAM الفعلي لديسكتوب Kali على الجهاز.
6. سلوك النظام عند قتل العملية تحت ضغط الذاكرة على HyperOS.
7. المصدر الرسمي الدقيق وSHA256 لـ rootfs Kali ARM64 الموصى به.
8. ما إذا كانت سياسات OEM (HyperOS/Xiaomi) تضيف قيودًا إضافية على العمليات الطويلة/الخدمات الأمامية.

### C — ما يحتاج PoC (What Requires PoC)
- POC-001: تشغيل shell داخل PRoot + Kali على الجهاز.
- POC-002: اختبار حل تجاوز W^X عبر `/system/bin/linker64` مع `targetSdk 35`.
- POC-003: اختبار `unshare -U` / namespaces على الجهاز (توقع: فشل).
- POC-004: نظام الملفات (صلاحيات، symlinks، /proc، /dev، /tmp، سرعة).
- POC-005: benchmark أداء PRoot مقابل الأصلي.
- POC-006: اختبار حد 32 عملية والعمليات الطويلة.
- POC-007: Termux:X11 + software rendering.
- POC-008: Turnip/Zink/VirGL على Adreno 810.
- POC-009: مقارنة XFCE/LXQt/MATE من حيث RAM وبدء التشغيل.
- POC-010: الشبكة (DNS، TCP، UDP، localhost، مستودعات الحزم).
- POC-011: دورة حياة Android (foreground service، قتل، شاشة مطفأة).
- POC-012: 16 KB page alignment لكل binary نَشحنه.
- POC-013: تنزيل/تحقق/استخراج rootfs مع قياس الزمن والمساحة.

### D — افتراضات محفوفة بالمخاطر (Risky Assumptions)
| الافتراض | لماذا هو خطر |
|---|---|
| "Termux:X11 سيكون مستقرًا على جهازي" | لم يُختبر؛ يعتمد على APK منفصل/مصدر خارجي |
| "HyperOS يتصرف كـ AOSP" | شركات OEM تضيف قيودًا على الخلفية والعمليات |
| "12 GB RAM تكفي لديكستوب كامل" | لم يُقس؛ Android يفرض حدودًا لكل تطبيق |
| "rootfs Kali يعمل كما هو" | NetHunter يستخدم rootfs معدّلة؛ الفروق غير مفحوصة |
| "الأداء مقبول" | لا يوجد benchmark واحد على هذا الجهاز |

### E — تقنيات واعدة (Promising)
- PRoot + إدارة rootfs مخصصة (بدل الاعتماد على PRoot-Distro كتطبيق).
- `system_linker_exec` كآلية تجاوز W^X.
- Termux:X11 (XCB) كطبقة عرض.
- VirGL كمسار تسريع "الأقل خطورة".
- NetHunter Rootless كمرجع تصميم (لا كتبعية).

### F — تقنيات يجب تجنبها (Avoid for MVP)
- Docker/runc/LXC Rootless (ممنوع kernelيًا).
- QEMU/KVM Full VM (لا KVM، TCG بطيء جدًا).
- AVF/Microdroid (غير متاح لطرف ثالث).
- systemd كنظام إدارة خدمات داخلي.
- Turnip كـ "أساس" النظام (EXPERIMENTAL، ب patches غير رسمية).

### G — تقنيات تحتاج اختبار جهاز (Need Device Testing)
PRoot · namespaces · حل W^X · Termux:X11 · Turnip/Zink/VirGL · كل بيئات الديسكتوب · دورة حياة Android · شريحة تخزين الجهاز.

### H — مناسبة للـ MVP (Suitable)
PRoot (runtime) · rootfs رسمي Kali مع تحقق · X11 عبر Termux:X11 · software rendering · XFCE أو LXQt (بعد قياس) · Foreground Service · Terminal · تحميل متدرّج للحزم.

### I — الأفضل تأجيلها (Deferred)
تسريع GPU · صوت Linux↔Android · USB متقدم · Bluetooth منخفض المستوى · مراقبة الشبكة/حقن الحزم · Multi-distro · FEX/Box64/Wine · شاشة خارجية · AI.

### J — خطة PoC الموصى بها (مبدئية)
ثلاث موجات:
1. **موجة الإثبات (Runtime):** POC-001 → 005 → 004 → 013 (هل يمكن تشغيل Kali أصلًا؟ بأي أداء؟)
2. **موجة العرض (Graphics):** POC-007 → 009 → 008 (هل الديسكتوب عملي؟)
3. **موجة المتانة (Lifecycle):** POC-006 → 011 → 010 → 012 (هل يعيش التطبيق في ظروف Android القاسية؟)

---

## 6. حالة الانتقال للمرحلة التالية

**لا يمكن الانتقال إلى PHASE 1 (Architecture Decision) بعد** — حسب البند 62 (Architecture Gate) ينقصنا: PoC + Evidence + مصفوفة توافق مبنية على قياس + مراجعة أمنية + ADR.

ما أُنجز في هذا الملف: **بحث أولي موثق (مصادر رسمية + مجتمعية)**.
ما يجب قبل أي قرار معماري: **الـ 13 PoC أعلاه على الجهاز المرجعي**.
