# GRAPHICS_RESEARCH.md
## Phase 0 — بحث طبقة العرض والرسوميات (البند 12 و22 و45 و46)

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | بحث أولي — **لا قرار نهائي قبل Phase 0.5** |
| التاريخ | 2026-09-12 |
| الجهاز المرجعي | Android 15 · ARM64 · **Adreno 810** (Snapdragon 7s Gen 4) · بدون Root |
| هدف الـ MVP (البند 47) | Desktop + Touch + Keyboard + Mouse + Clipboard — **ولو بـ Software Rendering** |

---

## 1. طبقة العرض (Display Backend)

### 1.1 جدول المقارنة

| الخيار | الحالة | الأداء | الاستقرار | ARM64 | Android 15 | لمس | لوحة مفاتيح/فأرة | حافظة | صوت | تسريع عتادي |
|---|---|---|---|---|---|---|---|---|---|
| **Termux:X11 (XCB)** | IMPLEMENTED مجتمعيًا | جيد (أصلي، بدون شبكة) | جيد | ✅ | ❓ (يعلن Android 8+) [3](https://ivonblog.com/en-us/posts/termux-x11/) | ✅ | ✅ | ✅ | ⚠️ عبر PulseAudio | ⚠️ عبر DRI3 (تجريبي) |
| X11 (عام) | VERIFIED تقنيًا | — | — | ✅ | ❓ | ✅ | ✅ | ✅ | ⚠️ | ⚠️ |
| Wayland / Weston | EXPERIMENTAL على Android | ❓ | ❓ | ✅ | ❓ | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ⚠️ |
| VNC (KeX / TigerVNC) | VERIFIED رسميًا (NetHunter) [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) | أقل (ترميز + شبكة محلية) | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ | ❌ | ❌ (غالبًا) |
| RDP (xrdp) | LIMITED | متوسط | ⚠️ | ✅ | ❓ | ⚠️ | ✅ | ✅ | ⚠️ | ❌ |
| SPICE | LIMITED/غير شائع على Android | ❓ | ❓ | ✅ | ❓ | ❓ | ❓ | ❓ | ⚠️ | ⚠️ |

### 1.2 ملاحظات موثقة
- **Termux:X11**: يتطلب Android 8 أو أحدث؛ يتكوّن من تطبيق Android + حزمة مرافقة في Termux؛ بدأ تنفيذه على XWayland ثم استُبدل بـ XCB؛ يُوزَّع عبر إصدارات nightly في المستودع [3](https://ivonblog.com/en-us/posts/termux-x11/) [1](https://vuink.com/post/tvguho-d-dpbz/termux/termux-x11).
- **VNC هو أساس KeX** في NetHunter Rootless رسميًا، ويحتاج عميل KeX منفصل [1](https://www.kali.org/docs/nethunter/nethunter-rootless/).
- **DRI3** شرط لتسريع GPU عبر Termux:X11 — أي أن نسخة Termux:X11 المستخدمة يجب أن تدعم DRI3 [3](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md).

> 🔎 **فجوة بحثية:** لم نتحقق بعد من: (1) إصدار Termux:X11 الدقيق المتاح اليوم، (2) دعمه الرسمي لـ Android 15، (3) ترخيصه وسياسة التوزيع. الحالة: **UNKNOWN** → مهام V-02 وV-03.

---

## 2. GPU (البند 12 و46)

### 2.1 التقنيات

| التقنية | الوظيفة | الحالة على Android Rootless | الأدلة |
|---|---|---|---|
| **VirGL (virglrenderer)** | OpenGL برمجي/مُرحَّل: Android GL/ES → خادم VirGL في Termux → برنامج `virpipe` داخل proot | **EXPERIMENTAL/LIMITED** لكنه المسار الأكثر نضجًا | [1](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md) |
| **Zink** | طبقة OpenGL فوق Vulkan في Mesa | **EXPERIMENTAL** | [3](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md) |
| **Turnip (freedreno)** | برنامج تشغيل Vulkan مفتوح المصدر لـ Adreno 6xx/7xx، يصل إلى KGSL مباشرة من داخل proot | **EXPERIMENTAL** — يعطي Vulkan 1.3 / OpenGL 4.6 / GLES 3.2 عند دمجه مع Zink | [1](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/) |
| **Mesa + llvmpipe/softpipe** | تصيير برمجي | **VERIFIED** كخيار أساسي | ممارسة شائعة |
| **Vulkan (Android)** | متاح للتطبيق المضيف | ✅ مدعوم على Adreno 810 (Vulkan 1.3) [1](https://www.91mobiles.com/processor/qualcomm-snapdragon-7s-gen-4-pdp) | مواصفات الشريحة |

### 2.2 أداء Turnip المُعلن
- تحسين **2.5x إلى 4–5x** مقابل `virglrenderer-android` وZink+Turnip بدون DRI3 (قياس `glmark2`) [1](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/).
- ⚠️ هذه أرقام **من تقارير مجتمع على أجهزة أخرى**، وليست قياسًا على جهازنا. التصنيف: **UNVERIFIED**.

### 2.3 وضع جهازنا تحديدًا (Adreno 810)
- Adreno 810 ينتمي لعائلة Adreno 800، بمستوى أداء قريب من Adreno 640/644 تقريبيًا [2](https://inquisitiveuniverse.com/2025/09/17/snapdragon-7s-gen-4-specs-and-benchmarks-review/).
- دعم Freedreno/Turnip أُضيف حديثًا لـ **Adreno 830/840** عبر patches غير رسمية (KGSL + DRI3) مع دعم "غير رسمي" لـ 830 بمعاملات 840 [2](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/).
- **الاستنتاج:** لا يوجد أي دليل على دعم Adreno 810 تحديدًا. الحالة: **UNKNOWN** → POC-008.

> ⚠️ **مهم (البند 46):** GPU acceleration ليست شرطًا للـ MVP. إذا فشلت أو كانت غير مستقرة: **نصنفها EXPERIMENTAL ولا نجعلها أساس النظام**.

---

## 3. بيئات الديسكتوب (البند 23)

| البيئة | الحالة على ARM64/Android | تقدير RAM/الأداء | لمس | حافظة | ملاحظة |
|---|---|---|---|---|---|
| XFCE | شائع جدًا في هذا السياق | ❓ | ⚠️ يحتاج إعداد | ⚠️ | مرشح أول |
| LXQt | خفيف | ❓ | ⚠️ | ⚠️ | مرشح (Qt) |
| MATE | متوسط | ❓ | ⚠️ | ⚠️ | أثقل من LXQt غالبًا |
| KDE Plasma | ثقيل | ❓ | ✅ (أفضل دعم لمسي) | ⚠️ | يُستبعد مبدئيًا للـ MVP |
| Openbox / i3 | خفيف جدًا | ❓ | ❌ ضعيف | ⚠️ | مناسب كـ WM لا كـ DE |

> 🔴 **لا توجد أرقام مقيسة.** أي ادعاء بأن XFCE "أخف" من LXQt يجب أن يُثبت بقياس على الجهاز (POC-009). القاعدة من البند 29: **ممنوع قول "أسرع" بدون benchmark.**

---

## 4. معايير القبول لطبقة العرض في الـ MVP

مطلوب إثباتها كلها في POC-007/POC-009 قبل أي اعتماد:

| # | المعيار | الحالة |
|---|---|---|
| G-01 | بدء الديسكتوب من داخل Kali Rootless | UNKNOWN |
| G-02 | لمس (نقرة، سحب، تكبير) | UNKNOWN |
| G-03 | لوحة مفاتيح (برمجية + خارجية) | UNKNOWN |
| G-04 | فأرة (مؤشر + أزرار) | UNKNOWN |
| G-05 | حافظة ثنائية الاتجاه Android ↔ Linux | UNKNOWN |
| G-06 | استقرار جلسة ≥ 30 دقيقة دون انهيار | UNKNOWN |
| G-07 | استهلاك RAM ضمن حدود آمنة | UNKNOWN |
| G-08 | سلوك الشاشة عند الإيقاف/الدوران | UNKNOWN |

---

## 5. التوصية المرحلية (غير نهائية)

1. **للـ MVP:** Termux:X11 كطبقة عرض + تصيير برمجي (Software Rendering / llvmpipe) + بيئة ديسكتوب خفيفة (XFCE أو LXQt بعد القياس).
2. **كطبقة ثانية (اختيارية):** VNC كمسار بديل/احتياطي للتشخيص والوصول عن بعد — لأنه الأكثر استقرارًا تاريخيًا والأساس الرسمي لـ KeX.
3. **مؤجل:** GPU عبر VirGL ثم Turnip/Zink — يُضاف فقط إذا ثبت بالقياس (البند 46).
4. **مستبعد للـ MVP:** SPICE، Wayland/Weston كأساس، KDE Plasma.

---

## 6. ما لم يُبحث بعد (فجوات يجب سدّها)

| # | الفجوة | كيف نسدّها |
|---|---|---|
| V-02 | إصدار Termux:X11 الحالي، ترخيصه، متطلباته الدقيقة، سياسة التوزيع | قراءة مستودع GitHub الرسمي + LICENSE + Releases |
| V-03 | دعم Termux:X11 المعلن لـ Android 15 و16 KB pages | فحص Issues/Releases |
| V-04 | بديل داخلي: هل نستطيع تضمين X server داخل تطبيقنا (بدل APK منفصل)؟ | دراسة ترخيص وبنية Termux:X11 + XWayland |
| V-05 | حالة Wayland/Weston العملية على Android 15 | دراسة + تجربة |
| V-06 | دعم Adreno 810 في Mesa/Freedreno upstream | فحص شيفرة Mesa + قوائم البريد |
