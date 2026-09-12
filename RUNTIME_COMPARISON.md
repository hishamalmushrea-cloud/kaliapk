# RUNTIME_COMPARISON.md
## مقارنة محركات تشغيل Linux على Android (البند 14 من PRD)

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | مقارنة أولية موثقة — **بدون قرار نهائي** |
| التاريخ | 2026-09-12 |
| الجهاز المرجعي | Android 15 · ARM64 · Snapdragon 7s Gen 4 · بدون Root |
| ملاحظة | لا يوجد هنا "فائز"؛ القرار مؤجل إلى ما بعد PHASE 0.5 (البند 32) |

---

## 1. الخيارات قيد المقارنة

| الرمز | الخيار |
|---|---|
| A | PRoot |
| B | PRoot-Distro |
| C | chroot |
| D | Linux namespaces (unshare) |
| E | Containers (runc/LXC/Docker) |
| F | VM (KVM/QEMU/AVF) |
| G | Hybrid |

---

## 2. جدول المقارنة الشامل

المعايير مطلوبة في البند 14. الرموز: ✅ يعمل · ⚠️ جزئي/مشروط · ❌ لا يعمل · ❓ UNKNOWN

| المعيار | A: PRoot | B: PRoot-Distro | C: chroot | D: namespaces | E: containers | F: VM |
|---|---|---|---|---|---|---|
| **Root requirement** | ❌ لا يحتاج | ❌ لا يحتاج | ✅ يحتاج | ❌ نظريًا لا | ❌ نظريًا لا | ❌ نظريًا لا |
| **Android compatibility** | ✅ مثبت مجتمعيًا | ✅ مثبت | ⚠️ فقط بأجهزة مروّتة | ❌ `EPERM` على أغلب الأجهزة [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c) | ❌ يعتمد على D | ❌ لا KVM لطرف ثالث [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/) |
| **SELinux interaction** | ⚠️ يعمل داخل نطاق التطبيق | ⚠️ نفسه | ✅ يحتاج سياسة/تجاهل | ❌ سياسات تمنع | ❌ | ❌ |
| **Kernel requirements** | ❌ لا شيء | ❌ لا شيء | `CAP_SYS_ADMIN` | `CONFIG_USER_NS` + مسموح | namespaces + cgroups | KVM/AVF |
| **filesystem behavior** | ⚠️ ترجمة مسارات، قيود موثقة | ⚠️ نفسه | ✅ حقيقي | ✅ حقيقي | ✅ حقيقي | ✅ حقيقي |
| **networking** | ✅ يرث شبكة التطبيق | ✅ | ✅ | ⚠️ netns غير متاح | ⚠️ | ✅ (NAT) |
| **process isolation** | ❌ لا يوجد [1](https://github.com/termux/proot-distro) | ❌ | ⚠️ | ✅ | ✅ | ✅ |
| **performance** | ⚠️ ptrace overhead (غير مقيس) | ⚠️ نفسه | ✅ | ✅ | ✅ | ❌ TCG بطيء جدًا |
| **security** | ⚠️ عزل ملفات فقط | ⚠️ | ⚠️ | ✅ | ✅ | ✅ (أفضل عزل) |
| **stability** | ✅ واسع الاستخدام | ✅ | ⚠️ | ❌ | ❌ | ❌ |
| **complexity** | متوسطة | منخفضة (جاهزة) | منخفضة | عالية | عالية جدًا | عالية جدًا |
| **maintainability** | ⚠️ upstream بطيء (5.4.0 منذ 2023) [2](https://github.com/proot-me/PRoot/releases) | ✅ نشط في Termux | — | — | — | — |
| **ARM64** | ✅ | ✅ | ✅ | ❓ | ❌ | ❌ |
| **Android 15** | ⚠️ يحتاج 16 KB alignment | ⚠️ نفسه | ❓ | ❌ | ❌ | ❌ |
| **systemd** | ❌ [1](https://github.com/termux/proot-distro) | ❌ | ⚠️ | ✅ نظريًا | ✅ | ✅ |
| **GPU access** | ⚠️ عبر VirGL/Turnip (تجريبي) | ⚠️ | ✅ | ✅ | ✅ | ⚠️ |

---

## 3. تحليل كل خيار

### A — PRoot
**الآلية:** تطبيق في مساحة المستخدم لـ `chroot` و`mount --bind` و`binfmt_misc` عبر اعتراض نداءات النظام بـ `ptrace` وإعادة كتابة مسارات الملفات أثناء التنفيذ [1](https://github.com/termux/proot-distro).

**القيود الموثقة من المصدر الرسمي:**
- الأداء: كل نداء نظام يمر عبر PRoot.
- لا وحدات kernel: FUSE، أهداف iptables مخصصة، تسلسلات cgroup مخصصة.
- لا root حقيقي: إعادة تعيين UID/GID فقط.
- لا خدمات خلفية/ـinit: systemd وOpenRC غير ممكنين عمليًا.
- لا namespaces ولا cgroups.
- لا تداخل (لا proot داخل proot).
[1](https://github.com/termux/proot-distro)

**الحالة:** IMPLEMENTED مجتمعيًا · أعلى تقنية نضجًا في السياق Rootless.

### B — PRoot-Distro
**الآلية:** طبقة إدارة فوق PRoot (تحميل/تثبيت/إدارة صور التوزيعات، وحدات إعداد جاهزة).

**الإيجابيات:** يختصر وقت التطوير، مجتمع نشط، إصدارات منتظمة.
**السلبيات:** مرتبط ببيئة Termux (بعض الأعلام Termux-only) [1](https://github.com/termux/proot-distro)؛ واعتمادنا عليه يعني اعتمادًا على بيئة خارجية لا نتحكم في دورة حياتها (يتعارض مع هدف "One-Tap Startup" بدون Termux).

**الحالة:** VERIFIED · مناسب كمرجع تصميم وكأداة PoC، لا كاعتماد runtime نهائي إلا بقرار واضح.

### C — chroot
**الحالة:** **UNSUPPORTED** في الوضع Rootless (يحتاج `CAP_SYS_ADMIN`).
**مستقبلًا:** خيار مشروع في "Root Mode" (البند 49) — يعطي أداءً أصليًا ونظام ملفات حقيقيًا.

### D — Linux namespaces
**الدليل المتاح:** محاولات موثقة تُظهر:
```
unshare(CLONE_NEWUSER) = -1 EPERM (Operation not permitted)
clone(flags=CLONE_NEWUSER|SIGCHLD) = -1 EPERM
```
على Android غير مروّت [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c).

**الحالة:** **BLOCKED** (مرجّح) مع DOCUMENTED · يجب تأكيده على جهازنا في POC-003، لأن القرار النهائي يتوقف عليه.

### E — Containers
يعتمد كليًا على D. التقارير تُشير إلى أن الوضع Rootless يعتمد على user namespaces التي لا يكشفها صندوق التطبيق [5](https://www.sitepoint.com/hardened-mobile-dev-a-termux-docker-guide-for-grapheneos/).
**الحالة:** **BLOCKED** Rootless.

### F — VM
- `/dev/kvm` غير متاح لتطبيقات عادية.
- إطار AVF/pKVM يسمح فقط بصور موقّعة من Google أو الشركة المصنّعة، ولا يوفر وصولًا عامًا للـ hypervisor [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/).
- البديل Rootless هو QEMU بمحاكاة TCG (ترجمة تعليمات برمجية) — بطيء جدًا عمليًا.
**الحالة:** **UNSUPPORTED** Rootless.

### G — Hybrid
مثال: PRoot كأساس + chroot/VM عند توفر Root + طبقة عرض مستقلة.
**الحالة:** **PLAUSIBLE** — القرار مؤجل (البند 32 يسمح به صراحة إن أثبتته التجارب).

---

## 4. مصفوفة المخاطر

| الخيار | الخطر الأساسي | الاحتمال | الأثر | تخفيف مقترح |
|---|---|---|---|---|
| PRoot | أداء غير كافٍ للديسكتوب | متوسط | عالٍ | قياس مبكر (POC-005) قبل بناء الواجهة |
| PRoot | حد 32 عملية يقتل الجلسة | عالٍ [1](https://cosyra.com/guides/termux-signal-9-fix.html) | عالٍ | تقليل عدد العمليات + كشف + استرداد + إرشاد مستخدم |
| PRoot | 16 KB page size يكسر binaries | متوسط | عالٍ | بناء كل الكود الأصلي بمحاذاة 16 KB (POC-012) |
| PRoot | W^X يمنع التنفيذ | **مؤكد** [1](https://github.com/termux/termux-packages/wiki/Termux-execution-environment) | **حرج** | `system_linker_exec` (POC-002) |
| namespaces | نفترض التعطّل ثم يتبين العكس | منخفض | متوسط | POC-003 مبكر |
| VM | إضاعة وقت في مسار مسدود | منخفض (مستبعد مبكرًا) | متوسط | الاستبعاد الآن بناءً على الدليل المتاح |

---

## 5. الاستنتاج المؤقت (غير ملزم)

- **المرشح الوحيد القابل للتنفيذ Rootless اليوم: PRoot (A)**، مع إدارة rootfs خاصة بنا أو عبر PRoot-Distro في مرحلة الـ PoC فقط.
- **B/C/D/E/F/G كلها إما غير متاحة Rootless أو مؤجلة**، باستثناء G كمعمارية هجينة مستقبلية وC/D/E/F كخيارات في "Root Mode" (Phase 7+).
- ⚠️ **لا يعني هذا أن PRoot "الأفضل"** — بل أنه الوحيد المتبقي بعد إسقاط الخيارات الممنوعة kernelيًا. هذا تمييز مهم يجب ألا يُطمس في أي ADR لاحق (البند 6: الصدق الهندسي).

---

## 6. ما يحتاج قياسًا قبل القرار

| # | القياس | الطريقة | الأولوية |
|---|---|---|---|
| M-01 | overhead الـ ptrace على CPU وI/O | مقارنة نفس المهمة داخل وخارج PRoot | عالية |
| M-02 | زمن استخراج rootfs Kali | قياس على وحدة التخزين الداخلية | عالية |
| M-03 | زمن إقلاع shell | من أمر البدء حتى جاهزية الـ shell | عالية |
| M-04 | عدد العمليات الممكنة قبل القتل | اختبار تدريجي مع/بدون تعطيل القيد | عالية |
| M-05 | استهلاك RAM لجلسة + ديسكتوب | قياس فعلي | متوسطة |
