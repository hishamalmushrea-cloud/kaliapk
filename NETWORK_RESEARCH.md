# NETWORK_RESEARCH.md
## Phase 0 — بحث الشبكة والعزل (البند 24 و48)

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | بحث أولي — مع فصل صريح بين المستويات (كما يطلب البند 24) |
| التاريخ | 2026-09-12 |
| السياق | Rootless أولًا (البند 49) |

---

## 1. المبدأ: فصل المفاهيم (البند 24)

يجب **عدم** الخلط بين مستويات العزل. كل مستوى يُقيَّم مستقلًا:

| نوع العزل | المقصود | الحالة Rootless |
|---|---|---|
| Filesystem Isolation | هل ترى بيئة Linux ملفات النظام الحقيقية؟ | ⚠️ **ترجمة مسارات لا عزل kernelي** (PRoot) [1](https://github.com/termux/proot-distro) |
| Process Isolation | هل عمليات Linux معزولة عن النظام والتطبيقات؟ | ❌ **لا يوجد** (نفس UID ونفس النطاق) [1](https://github.com/termux/proot-distro) |
| Network Isolation | هل لبيئة Linux شبكة منفصلة؟ | ❌ **لا** — ترث شبكة التطبيق بالكامل |
| Kernel Isolation | هل للبيئة نواة منفصلة؟ | ❌ **لا** (نفس نواة Android) |

> 🔴 **استنتاج مباشر:** عبارة "VPNService تعني Network Isolation كامل" **خاطئة**، والبند 24 ينبه إليها صراحة. استخدام VpnService يمنحنا **توجيهًا/اعتراضًا للحزم**، لا عزلًا أمنيًا لبيئة Linux.

---

## 2. جدول قدرات الشبكة

| القدرة | Rootless | الحالة | الدليل / التعليل |
|---|---|---|---|
| إنترنت (HTTP/HTTPS) | ✅ | VERIFIED | كل توزيعات proot تستخدم مستودعات الحزم فعليًا |
| DNS من داخل Linux | ✅ | VERIFIED | يعتمد على `/etc/resolv.conf` داخل rootfs وإعدادات Android |
| TCP | ✅ | VERIFIED | ممارسة شائعة |
| UDP | ✅ | VERIFIED | ممارسة شائعة |
| localhost (داخل Linux) | ⚠️ | LIMITED | لا يوجد net namespace → `localhost` مشترك مع Android |
| Android ↔ Linux (IPC/منفذ محلي) | ⚠️ | LIMITED | نفس مساحة الشبكة؛ يجب تصميم المنافذ بعناية لتجنب التعارض |
| Linux ↔ إنترنت | ✅ | VERIFIED | — |
| مستودعات الحزم (apt) | ✅ | VERIFIED | أساس كل توزيعات proot |
| **raw sockets** | ❌ | UNSUPPORTED | يتطلب صلاحيات kernel/CAP_NET_RAW غير متاحة لتطبيق عادي |
| **monitor mode** | ❌ | UNSUPPORTED | يتطلب تعريف Wi-Fi ونواة تدعمه؛ ليس متاحًا Rootless |
| **packet injection** | ❌ | UNSUPPORTED | نفس السبب |
| **تحكم منخفض المستوى في Wi-Fi** | ❌ | UNSUPPORTED | محجوب بصلاحيات النظام |
| التقاط حزم (PCAP) | ⚠️ | LIMITED | ممكن فقط عبر **محاكاة VPN** (مثل PCAPdroid) [1](https://github.com/emanuele-f/PCAPdroid) — لا يعطي وصولًا للطبقة الثانية |

---

## 3. أثر قيود Android على أدوات Kali الشبكية

| الأداة/القدرة | التوقع Rootless | ملاحظة |
|---|---|---|
| `ping` (ICMP عبر socket عادي) | ⚠️ LIMITED | غالبًا يعمل عبر ICMP غير الخام أو يفشل حسب الإعداد — يحتاج اختبار |
| `netstat` / أدوات تقرأ `/proc/net` | ⚠️ LIMITED | الوصول إلى `/proc/net` مقيد لغير المروّتة [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) |
| `nmap` (connect scan) | ⚠️ LIMITED | المسح بنمط SYN الخام يحتاج raw sockets |
| `tcpdump` | ⚠️ LIMITED | يحتاج raw/packet socket |
| Metasploit | ⚠️ LIMITED | مصدر رسمي: يعمل لكن **بدون دعم قاعدة بيانات** في NetHunter Rootless [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) |
| `aircrack-ng` / حقن | ❌ UNSUPPORTED | مراقبة/حقن غير متاحين |
| `iptables` | ❌ UNSUPPORTED | لا وحدات kernel [1](https://github.com/termux/proot-distro) |

> ⚠️ **مهم للواجهة (البند 64):** يجب ألا تعرض الواجهة أي قدرة شبكية منخفضة المستوى على أنها "مدعومة" في الوضع Rootless. العرض الصادق: الحالة + السبب + البديل.

---

## 4. VpnService — ما يفعله وما لا يفعله

**ما يفعله:**
- يمنح التطبيق حق قراءة/تعديل حزم الشبكة الخاصة بالجهاز عبر واجهة VPN وهمية محلية (بدون خادم بعيد) [1](https://github.com/emanuele-f/PCAPdroid).
- يسمح بتنفيذ منطق ترشيح/توجيه داخل التطبيق.

**ما لا يفعله:**
- لا يعطي raw sockets على واجهة Wi-Fi الحقيقية.
- لا يعطي monitor mode أو حقنًا.
- لا يعزل بيئة Linux أمنيًا عن Android.
- لا يمنح وصولًا للطبقة الثانية (Ethernet/Wi-Fi frames).

**التصنيف:** LIMITED — أداة مراقبة/توجيه، **ليست** طبقة عزل.

---

## 5. خيارات معمارية محتملة (للبحث لا للقرار)

| الخيار | الوصف | الحالة | المخاطر |
|---|---|---|---|
| N-1 | لا شيء: Linux يرث شبكة التطبيق مباشرة | الأسهل، الأكثر استقرارًا | لا عزل (مقبول للـ MVP حسب البند 82) |
| N-2 | VpnService لتوجيه/مراقبة حزم Linux | LIMITED | استهلاك بطارية، تعارض مع تطبيقات VPN أخرى |
| N-3 | منفذ/جسر محلي عبر Unix socket أو منفذ TCP محلي | شائع (مثل خوادم VNC/RDP/PulseAudio) | تعارض المنافذ مع Android |
| N-4 | Namespace شبكة حقيقي | ❌ BLOCKED | يعتمد على user namespaces (POC-003) |

---

## 6. خطة اختبار الشبكة (POC-010)

| # | الاختبار | المتوقع | how to verify |
|---|---|---|---|
| T-01 | `curl https://kali.org` من داخل Linux | نجاح | سجل + كود استجابة |
| T-02 | `apt update` | نجاح | سجل |
| T-03 | تحليل DNS (`getent hosts`) | نجاح | سجل |
| T-04 | TCP محلي بين Android وLinux | يعمل | خادم/عميل تجريبي |
| T-05 | UDP اختباري | يعمل | `nc -u` |
| T-06 | `ping` | UNKNOWN | سجل النتيجة الحقيقية (نجاح أو فشل + السبب) |
| T-07 | قراءة `/proc/net` | UNKNOWN | سجل |
| T-08 | raw socket تجريبي | توقع الفشل | سجل رمز الخطأ |
| T-09 | الاستمرار بعد إيقاف الشاشة 10 دقائق | UNKNOWN | قياس إعادة الاتصال |
| T-10 | تبديل Wi-Fi ↔ بيانات الجوال أثناء الجلسة | UNKNOWN | سجل سلوك إعادة الاتصال |

---

## 7. التوصية المرحلية

1. **للـ MVP:** الخيار N-1 (شبكة موروثة) + N-3 للاتصال المحلي بين Android وLinux (terminal/desktop/audio).
2. **لا نضيف VpnService في الـ MVP** إلا لميزة واضحة (مثل مراقبة/توجيه) وبعد تقييم أثر البطارية والتعارض.
3. **نثبّت في الواجهة**: raw packets / monitor mode / injection = **UNSUPPORTED في الوضع Rootless**، مع توضيح أنها قد تصبح ممكنة في وضع Root بنواة مخصصة (Phase 7+) دون وعد.
