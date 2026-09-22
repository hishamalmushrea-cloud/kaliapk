# DEVICE_COMPATIBILITY.md
## Phase 0 — مصفوفة توافق الجهاز (البند 56)

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | **مسودة أولية** — لا تُملأ الخانات إلا بدليل (البند 56) |
| التاريخ | 2026-09-12 |

---

## 1. الجهاز المرجعي

| الخاصية | القيمة | الحالة |
|---|---|---|
| النظام | Android 15 | معلن من المستخدم |
| المعمارية | ARM64 | معلن من المستخدم |
| الشريحة | Snapdragon 7s Gen 4 | معلن من المستخدم |
| GPU | Adreno 810 (895 MHz، Vulkan 1.3، OpenGL ES 3.2) | DOCUMENTED من مواصفات منشورة [1](https://www.91mobiles.com/processor/qualcomm-snapdragon-7s-gen-4-pdp) |
| RAM | ~12 GB | معلن من المستخدم |
| طبقة الشركة | HyperOS (أو مشابه) | معلن من المستخدم |
| Root | لا (الوضع الافتراضي) | معلن من المستخدم |
| حجم صفحة الذاكرة | **UNKNOWN** (4 KB أم 16 KB؟) | يجب التحقق — يحدد متطلبات البناء |
| إصدار النواة | **UNKNOWN** | يجب التحقق |
| دعم user namespaces في النواة | **UNKNOWN** | POC-003 |

> ⚠️ **البند 81:** مواصفات الجهاز **ليست** دليلًا على دعم GPU، monitor mode، حقن الحزم، USB HID، Bluetooth منخفض المستوى، نواة مخصصة، namespaces، أو containers. كل واحدة يجب إثباتها.

---

## 2. المصفوفة الرئيسية

الرموز: ✅ مدعوم · ⚠️ جزئي · ❌ غير مدعوم · ❓ UNKNOWN (لم يُختبر)

| الميزة | Rootless | Root | نواة مخصصة | Android 15 | جهازنا | الحالة | دليل/ملاحظة |
|---|---|---|---|---|---|---|---|
| Kali ARM64 rootfs (تحميل + استخراج) | ✅ | ✅ | No | ⚠️ | ❓ | **UNKNOWN** | تجربة مطلوبة (POC-013) |
| تشغيل shell داخل Kali | ✅ | ✅ | No | ⚠️ | ❓ | **UNKNOWN** | POC-001 |
| تنفيذ binaries داخل rootfs | ⚠️ | ✅ | No | ⚠️ | ❓ | **UNKNOWN** | مقيد بـ W^X (C-01) → POC-002 |
| نظام الملفات (صلاحيات/symlinks//proc//dev//tmp) | ⚠️ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-004 |
| مدير الحزم (apt) | ✅ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-001 |
| Python | ✅ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-001 |
| Git | ✅ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-001 |
| إنترنت / DNS / TCP / UDP | ✅ | ✅ | No | ✅ | ❓ | **UNKNOWN** | POC-010 |
| مستودعات حزم Kali | ✅ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-010 |
| raw sockets | ❌ | ⚠️ | ⚠️ | ❌ | ❌ | **UNSUPPORTED** | قيد منصة (NETWORK_RESEARCH.md) |
| monitor mode | ❌ | ⚠️ | ✅ | ❌ | ❓ | **UNSUPPORTED** Rootless | يحتاج نواة/تعريف |
| حقن حزم | ❌ | ⚠️ | ✅ | ❌ | ❓ | **UNSUPPORTED** Rootless | |
| Terminal (لمس + لوحة مفاتيح) | ✅ | ✅ | No | ✅ | ❓ | **UNKNOWN** | POC-007 |
| ديسكتوب عبر Termux:X11 | ⚠️ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-007 |
| ديسكتوب عبر VNC | ⚠️ | ✅ | No | ✅ | ❓ | **UNKNOWN** | مرجع رسمي NetHunter [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) |
| لمس (touch) في الديسكتوب | ⚠️ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-007 |
| فأرة ولوحة مفاتيح خارجية | ⚠️ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-007 |
| حافظة (clipboard) | ⚠️ | ✅ | No | ❓ | ❓ | **UNKNOWN** | POC-007 |
| صوت Linux → Android | ⚠️ | ⚠️ | No | ❓ | ❓ | **UNKNOWN** | البند 26: ليس blocker |
| GPU — software rendering | ✅ | ✅ | No | ✅ | ❓ | **UNKNOWN** | POC-007 |
| GPU — VirGL | ⚠️ | ⚠️ | No | ❓ | ❓ | **EXPERIMENTAL** | [1](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md) |
| GPU — Turnip/Zink (Adreno 810) | ❓ | ❓ | No | ❓ | ❓ | **UNKNOWN** | لا دليل على دعم 810 [2](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/) |
| systemd | ❌ | ⚠️ | No | ❌ | ❌ | **UNSUPPORTED** | [1](https://github.com/termux/proot-distro) |
| Docker / containers حقيقية | ❌ | ⚠️ | ✅ | ❌ | ❓ | **BLOCKED** | [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c) |
| KVM / VM | ❌ | ⚠️ | ✅ | ❌ | ❓ | **UNSUPPORTED** | [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/) |
| USB متقدم / HID | ❌ | ⚠️ | ⚠️ | ❌ | ❓ | **UNSUPPORTED** Rootless | البند 27 |
| Bluetooth منخفض المستوى | ❌ | ⚠️ | ⚠️ | ❌ | ❓ | **UNSUPPORTED** Rootless | البند 27 |
| Foreground Service (بقاء الجلسة) | ✅ | ✅ | No | ✅ | ❓ | **UNKNOWN** | POC-011 |
| النجاة من قتل Android للعملية | ⚠️ | ⚠️ | No | ❓ | ❓ | **UNKNOWN** | POC-011 |

---

## 3. كيف تتحول الخانات من ❓ إلى حالة مثبتة

| الخطوة | العمل |
|---|---|
| 1 | تنفيذ الـ PoC المرتبط على الجهاز المرجعي |
| 2 | تسجيل: الجهاز · إصدار Android · إصدار النواة · إصدار البرمجيات · الخطوات · النتيجة الفعلية · السجلات |
| 3 | تحديث هذه المصفوفة + إضافة السطر إلى RESEARCH_SOURCES.md |
| 4 | إذا فشل: فتح قسم في ملف الفشل لاحقًا (البند 59) بالصيغة: Problem · Cause · Evidence · Attempts · Result · Alternative · Decision |

---

## 4. تحذيرات خاصة بالجهاز

1. **HyperOS (OEM):** قد يضيف قيودًا على العمليات في الخلفية وعلى الخدمات الأمامية تتجاوز سلوك AOSP. الحالة: **UNKNOWN** — يجب اختباره مبكرًا لأنه قد ينسف افتراضات التصميم.
2. **حجم صفحة الذاكرة:** إن كان 16 KB، فكل binary غير محاذٍ سيفشل. التحقق خطوة أولى إلزامية.
3. **Adreno 810:** لا يوجد دليل على دعمه في Turnip/Freedreno؛ الأداء "المقبول" غير مضمون.
4. **12 GB RAM:** لا تعني توفرها للتطبيق؛ Android يفرض حدودًا لكل عملية. يجب القياس.

---

## 5. ملخص حالة الجهاز الآن

| المؤشر | القيمة |
|---|---|
| عدد الخانات المثبتة (✅/❌/⚠️ بدليل) | ~14 (معظمها قيود سلبية مثبتة من مصادر عامة) |
| عدد الخانات UNKNOWN | ~22 |
| عدد الخانات التي تحتاج اختبارًا على الجهاز | **الكل تقريبًا** |
| جاهزية الانتقال لـ Phase 1 | ❌ **لا** (ينقص: PoC + قياسات) |
