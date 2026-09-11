# kaliapk
🎯 Kali Mobile — Linux Workstation on Android

📋 MASTER PRD — الإصدار V3

---

1. الرؤية

أريد بناء تطبيق Android يحوّل الهاتف إلى Linux Workstation حقيقي داخل Android، بحيث تكون تجربة المستخدم الأساسية:

Open App
   ↓
Start Linux
   ↓
Linux Environment Ready
   ↓
Terminal + Linux Filesystem + Desktop

تكون Kali Linux هي التوزيعة الأولى والرئيسية، لكن لا أريد بناء المشروع بطريقة تجعل كل شيء مرتبطًا بـ Kali فقط.

الهدف المعماري طويل المدى هو إنشاء:

«Android Linux Runtime Platform»

يمكنها تشغيل Kali Linux أولًا، ثم دعم توزيعات أخرى مستقبلًا.

التطبيق ليس مجرد:

- Launcher
- Terminal wrapper
- VNC viewer
- NetHunter frontend
- واجهة لـ Termux فقط

بل يجب أن يكون طبقة تشغيل وإدارة متكاملة لبيئة Linux داخل Android.

---

2. الجهاز المستهدف

الجهاز المرجعي الأساسي:

- Android 15
- ARM64
- Snapdragon 7s Gen 4 أو منصة مشابهة
- RAM حوالي 12 GB
- بدون Root كخيار افتراضي
- HyperOS أو Android OEM مشابه

لكن:

«لا تفترض أن أي تقنية تعمل على الجهاز لمجرد أنها تعمل على Linux أو جهاز Android آخر.»

يجب إثبات التوافق بالتجربة أو بمصدر موثوق.

---

3. المبدأ المعماري الأساسي

يجب التعامل مع المشروع باعتباره:

Android Linux Runtime Platform
        │
        ├── Runtime Backend
        │
        ├── Linux Root Filesystem
        │
        ├── Process / Session Manager
        │
        ├── Graphics Backend
        │
        ├── Network Layer
        │
        ├── Storage Layer
        │
        ├── Android Integration
        │
        └── Distro Manager
                │
                └── Kali Linux

يجب أن تكون Kali:

«First-Class Distro»

وليست مجرد حالة خاصة داخل الكود.

---

4. القواعد الأساسية للمشروع

القاعدة 1 — لا تفترض

لا تفترض مسبقًا أن:

- Termux مناسب
- PRoot هو الأفضل
- NetHunter هو الأفضل
- KeX هو الأفضل
- XFCE هو الأفضل
- X11 هو الأفضل
- Wayland هو الأفضل
- VNC هو الأفضل
- namespaces تعمل بالكامل
- containers تعمل
- systemd يعمل
- GPU acceleration تعمل
- Turnip يعمل
- Zink يعمل
- Vulkan يعمل
- Root غير ضروري لكل شيء

كل ذلك يجب:

Research
↓
Compare
↓
Prototype
↓
Test
↓
Measure
↓
Decide

---

5. الأولوية

الأولوية ليست عدد الميزات.

الأولوية:

1. Stability
2. Correctness
3. Security
4. Compatibility
5. Performance
6. User Experience
7. Features

لا تضف ميزة إذا كانت ستجعل Runtime أقل استقرارًا.

---

6. الصدق الهندسي

ممنوع استخدام عبارات مثل:

- Should work
- Probably works
- Almost complete
- Production ready
- Native
- Fast
- Stable
- Fully supported

إلا إذا كان هناك دليل.

كل ادعاء تقني يجب تصنيفه.

استخدم الحالات:

NOT STARTED
RESEARCHED
PROTOTYPED
IMPLEMENTED
TESTED
VERIFIED
PARTIALLY SUPPORTED
UNSUPPORTED
BLOCKED
EXPERIMENTAL

---

7. Reuse Before Reimplement

قبل كتابة أي Runtime أو Linux subsystem من الصفر:

ابحث أولًا عن:

- Open-source projects
- Existing runtimes
- Existing rootfs managers
- Existing Android Linux solutions
- Existing graphical backends
- Existing IPC solutions
- Existing filesystem implementations
- Existing networking implementations
- Existing container/runtime projects

ثم اسأل:

«هل يمكن استخدام المشروع كما هو؟»

ثم:

«هل يمكن دمجه؟»

ثم:

«هل يمكن تعديله؟»

ثم فقط:

«هل نحتاج إلى إعادة بناء هذا الجزء؟»

لا تعيد اختراع شيء موجود إلا إذا كان هناك سبب تقني واضح.

---

8. مراحل المشروع

PHASE 0
Deep Research

        ↓

PHASE 0.5
Technical Proof of Concepts

        ↓

PHASE 1
Architecture Decision

        ↓

PHASE 2
MVP Definition

        ↓

PHASE 3
Core Runtime Engine

        ↓

PHASE 4
Android UI

        ↓

PHASE 5
Graphics + Linux Desktop

        ↓

PHASE 6
Full Testing

        ↓

PHASE 7+
Advanced Features

---

9. PHASE 0 — Deep Research

في هذه المرحلة:

ممنوع:

- كتابة التطبيق
- إنشاء Android project
- اختيار Architecture نهائية
- البدء بالبرمجة
- الادعاء بأن تقنية معينة هي الأفضل
- تنفيذ المشروع

الهدف هو البحث فقط.

---

10. المشاريع التي يجب دراستها

ابحث بعمق في:

Android Linux

- Termux
- PRoot
- PRoot-Distro
- UserLAnd
- Andronix
- AnLinux
- Linux Deploy
- Linux containers
- Android containers
- chroot
- namespaces
- unshare
- cgroups
- mount namespaces
- PID namespaces
- user namespaces

---

11. Kali Linux

ادرس:

- Kali NetHunter
- NetHunter Rootless
- NetHunter Lite
- NetHunter Full
- NetHunter Pro
- KeX
- Kali ARM64 rootfs
- Kali package repositories
- Kali desktop environments
- Kali Android integration

حدد بدقة:

ما يحتاج Root؟
ما لا يحتاج Root؟
ما يحتاج Kernel خاص؟
ما يعمل على Android 15؟
ما يعمل على ARM64؟
ما هو رسمي؟
ما هو community-maintained؟

---

12. Graphics Research

ادرس:

Display

- X11
- Termux:X11
- Wayland
- Weston
- VNC
- RDP
- SPICE

GPU

- Vulkan
- Mesa
- Turnip
- Zink
- VirGL
- Software Rendering
- llvmpipe
- Freedreno

Desktop

- XFCE
- LXQt
- MATE
- KDE Plasma
- Openbox
- i3

حدد:

- الأداء
- الاستقرار
- ARM64
- Android 15
- GPU support
- Touch support
- Keyboard
- Mouse
- Clipboard
- Audio
- Hardware acceleration

---

13. Virtualization / Compatibility Research

ادرس:

- QEMU
- KVM
- Limbo
- UTM
- FEX
- Box64
- Box86
- Wine
- Winlator

الهدف ليس استخدام Windows في MVP.

الهدف معرفة:

«ما التقنيات التي يمكن أن تساعد مستقبلًا في بناء Compatibility Layer.»

---

14. Linux Runtime Research

قارن بين:

A

PRoot

B

PRoot-Distro

C

chroot

D

Linux namespaces

E

containers

F

VM

G

hybrid architecture

لكل خيار:

- Root requirement
- Android compatibility
- SELinux interaction
- Kernel requirements
- filesystem behavior
- networking
- process isolation
- performance
- security
- stability
- complexity
- maintainability

---

15. PHASE 0 Research Criteria

لكل مشروع أو تقنية افحص:

- GitHub repository
- License
- Contributors
- Recent activity
- Releases
- Changelog
- Architecture
- ARM64
- Android 15
- Root requirement
- Kernel requirement
- GPU
- Networking
- USB
- Bluetooth
- Audio
- Clipboard
- Filesystem
- Performance
- Known issues
- Open issues
- Pull requests
- Active forks
- Dependent projects

---

16. مصادر البحث

استخدم بالترتيب:

1. Source code
2. Official documentation
3. Official repositories
4. Releases
5. Changelogs
6. Issues
7. Pull Requests
8. Wiki
9. Active forks
10. Projects that depend on the technology
11. Community reports

لا تعتمد على مقال واحد لاتخاذ قرار معماري.

---

17. تصنيف الأدلة

كل نتيجة بحث يجب تصنيفها:

VERIFIED
DOCUMENTED
PLAUSIBLE
EXPERIMENTAL
LIMITED
UNSUPPORTED
BLOCKED

مثال:

Feature:
GPU acceleration

Status:
EXPERIMENTAL

Evidence:
...

Device:
Snapdragon ...

Android:
15

Reason:
...

---

18. الوثائق المطلوبة في Phase 0

أنشئ:

RESEARCH.md
OPEN_SOURCE_COMPARISON.md
RESEARCH_SOURCES.md
KNOWN_TECHNICAL_CONSTRAINTS.md
DEVICE_COMPATIBILITY.md
SECURITY_RESEARCH.md
GRAPHICS_RESEARCH.md
RUNTIME_COMPARISON.md
NETWORK_RESEARCH.md

لا تكتب Architecture نهائية بعد.

---

19. PHASE 0.5 — Technical Proof of Concepts

هذه المرحلة مهمة جدًا.

قبل اختيار Architecture النهائية، يجب بناء تجارب صغيرة مستقلة.

ليس التطبيق الكامل.

الهدف:

«إثبات أن التقنيات الأساسية تعمل فعليًا على الجهاز المستهدف.»

---

20. PoC — Runtime

اختبر:

- PRoot + Kali
- PRoot-Distro
- namespaces
- unshare
- rootless runtime
- container approach
- filesystem mounting
- process execution

اختبر:

Launch shell
↓
Execute Linux commands
↓
Install package
↓
Run Python
↓
Run Git
↓
Run basic Kali tools

---

21. PoC — Filesystem

اختبر:

- read/write
- permissions
- symlinks
- "/proc"
- "/sys"
- "/dev"
- "/tmp"
- home directory
- package manager
- large file operations
- many small files
- extraction speed

قِس الأداء.

---

22. PoC — Graphics

اختبر بشكل مستقل:

Kali
↓
Desktop
↓
Display Backend

قارن:

- Termux:X11
- X11
- Wayland
- VNC
- RDP
- Software Rendering
- Vulkan
- Mesa
- Turnip
- Zink
- VirGL

لا تفترض أن GPU acceleration ضرورية للـ MVP.

إذا كان:

Software Rendering
+
XFCE
+
Touch
+
Keyboard
+
Mouse

يعمل بثبات، فهذا مقبول للـ MVP.

---

23. PoC — Desktop

قارن:

- XFCE
- LXQt
- MATE
- KDE

حسب:

- RAM
- CPU
- startup time
- responsiveness
- touch
- keyboard
- mouse
- clipboard
- audio
- stability

---

24. PoC — Networking

اختبر:

- internet
- DNS
- TCP
- UDP
- localhost
- Android ↔ Linux
- Linux ↔ internet
- package repositories

لا تفترض أن VPNService تعني تلقائيًا:

«Network Isolation كامل»

يجب فصل:

Filesystem Isolation
Process Isolation
Network Isolation
Kernel Isolation

والتحقق من كل واحد بشكل مستقل.

---

25. PoC — Android Integration

اختبر:

- app lifecycle
- backgrounding
- foreground service
- notification
- screen off
- app restart
- process death
- memory pressure
- orientation
- permissions
- storage
- clipboard
- keyboard
- touch

---

26. PoC — Audio

اختبر:

- Linux → Android audio
- microphone
- system audio
- desktop sounds

إذا لم يكن مستقرًا:

«لا تجعل Audio blocker للـ MVP.»

---

27. PoC — USB / Bluetooth

اختبر فقط ما هو ممكن فعليًا.

حدد:

Supported
Partially Supported
Root Required
Kernel Required
Unsupported

لا تجعل USB/Bluetooth المتقدم شرطًا للـ MVP.

---

28. PoC Evidence Procedure

كل PoC يجب أن يحتوي:

Objective
Device
Android Version
Software Version
Setup
Test Steps
Expected Result
Actual Result
Logs
Performance
Problems
Conclusion

مثال:

PoC ID: POC-001

Technology:
PRoot + Kali ARM64

Result:
PASS

Startup:
...

RAM:
...

Package installation:
PASS

Python:
PASS

Git:
PASS

Networking:
PASS

Limitations:
...

---

29. Benchmark Rules

ممنوع قول:

«هذا أسرع»

بدون benchmark.

يجب قياس قدر الإمكان:

- startup time
- RAM
- CPU
- disk I/O
- package installation
- extraction
- desktop responsiveness
- process startup
- network throughput
- battery impact

---

30. PHASE 1 — Architecture Decision

بعد Phase 0 و0.5 فقط.

قارن:

Architecture A

Android
+
Termux
+
PRoot
+
Kali
+
XFCE
+
X11

Architecture B

Android
+
NetHunter Rootless
+
KeX

Architecture C

Android
+
Root
+
chroot
+
Kali

Architecture D

Android
+
Container
+
Kali

Architecture E

Android
+
VM
+
Kali

Architecture F

Hybrid

---

31. Architecture Decision Record

اكتب:

ADR-001-runtime.md
ADR-002-graphics.md
ADR-003-desktop.md
ADR-004-network.md
ADR-005-storage.md
ADR-006-security.md

كل قرار يجب أن يشرح:

Problem
Options
Evidence
Trade-offs
Decision
Why
Rejected Alternatives
Risks
Future Re-evaluation

---

32. لا يوجد Architecture مقدسة

إذا أثبتت التجارب أن:

PRoot

أفضل:

استخدمه.

إذا أثبتت:

Container

أفضل:

استخدمه.

إذا أثبت:

Hybrid

أفضل:

استخدم Hybrid.

إذا فشلت فكرة:

«يجب التخلي عنها.»

الفشل نتيجة هندسية صحيحة وليس مشكلة.

---

33. PHASE 2 — MVP

الـ MVP يجب أن يحقق:

Installation

Install APK
↓
Open
↓
Setup
↓
Download Kali
↓
Verify
↓
Install

---

34. Kali RootFS

لا تضع rootfs داخل APK.

استخدم:

Downloader
↓
Official/Trusted Source
↓
SHA256 Verification
↓
Extraction
↓
Configuration
↓
Validation

يجب دعم:

- resume
- retry
- integrity check
- storage check
- version
- cleanup
- update strategy

---

35. One-Tap Startup

بعد الإعداد:

Open App
↓
Start Kali
↓
Runtime Ready

لا تطلب من المستخدم كتابة أوامر Terminal لكي يبدأ Linux في الاستخدام الطبيعي.

---

36. Clarification — No Manual Commands

عبارة:

«No manual commands»

تعني:

«المستخدم العادي لا يحتاج إلى تنفيذ أوامر Terminal لإدارة التطبيق.»

لكن أثناء التطوير يجوز استخدام:

- ADB
- Logcat
- Debug builds
- Developer tools
- profiling
- shell

للتشخيص والاختبار.

---

37. MVP Features

يجب أن يحتوي MVP على:

Runtime

- Start
- Stop
- Restart
- Status

Terminal

- Linux shell
- command execution
- environment variables

Linux

- filesystem
- package manager
- Python
- Git
- basic Kali tools

Networking

- internet
- DNS
- package repositories

Desktop

- Desktop environment واحد
- touch
- keyboard
- mouse
- clipboard

Android

- foreground service
- lifecycle handling
- crash recovery
- notifications

Storage

- Linux home
- user files
- import/export

Backup

- basic backup
- restore

Settings

- distro
- storage
- display
- desktop
- performance
- lifecycle

---

38. Backup / Restore

لا تستخدم مفهوم:

«RAM checkpoint»

كحل أساسي.

النسخة الأولى من Backup يجب أن تحفظ:

- user files
- Linux home
- configuration
- installed package list
- application settings
- environment configuration

ثم:

Restore
↓
Recreate Runtime
↓
Restore Files
↓
Restore Configuration
↓
Restore Packages

ليس مطلوبًا في MVP حفظ RAM/process state.

---

39. Session Recovery

عند:

- Android killing process
- application crash
- memory pressure
- screen off
- app restart

يجب أن يستطيع التطبيق:

Detect Failure
↓
Check Runtime State
↓
Cleanup
↓
Restart Runtime
↓
Restore Environment

ولا يجب افتراض إمكانية استعادة العمليات القديمة من RAM.

---

40. PHASE 3 — Core Engine

استخدم:

- Kotlin
- Gradle
- Clean Architecture
- MVVM أو Architecture مناسبة مثبتة بالأدلة

المكونات المقترحة:

Linux Environment Manager
Runtime Backend
Distro Manager
RootFS Manager
Session Manager
Process Manager
IPC Layer
Storage Manager
Network Manager
Graphics Manager
Lifecycle Manager
Recovery Manager
Diagnostics Manager

لكن هذه ليست قرارات نهائية إذا أثبت البحث غير ذلك.

---

41. Core Runtime API

يجب أن يكون هناك abstraction يسمح مستقبلًا:

RuntimeBackend

مثل:

PRootBackend
ContainerBackend
RootBackend
VMBackend

حتى لا تصبح واجهة Android مرتبطة مباشرة بمحرك واحد.

---

42. Distro Abstraction

صمم النظام مستقبلًا ليستوعب:

Kali
Ubuntu
Debian
Arch
Fedora
Alpine

لكن:

«لا تنفذ Multi-Distro في MVP.»

---

43. PHASE 4 — Android UI

استخدم:

- Jetpack Compose
- Material 3 أو تصميم مخصص مناسب
- Dark/Light
- RTL
- Accessibility

الشاشات الأساسية:

Home
Setup
Linux Status
Terminal
Desktop
Files
Settings
Diagnostics
About

---

44. Home Screen

يجب أن تكون بسيطة.

مثال:

┌─────────────────────────┐
│       Kali Mobile       │
│                         │
│    ● Linux Stopped      │
│                         │
│      [ START KALI ]     │
│                         │
│ Storage: 12 GB          │
│ RAM: 3.2 GB             │
│ Runtime: Ready          │
│                         │
│ Terminal    Desktop     │
│ Files       Settings    │
└─────────────────────────┘

---

45. PHASE 5 — Graphics + Desktop

اختيار:

X11
Wayland
VNC
RDP

يجب أن يعتمد على نتائج Phase 0.5.

لا تختار بناءً على الشهرة.

---

46. GPU

GPU acceleration:

«ليست شرطًا للـ MVP.»

لكن يجب البحث والتجربة لاحقًا في:

- Vulkan
- Mesa
- Turnip
- Zink
- VirGL

إذا نجحت:

«أضفها.»

إذا كانت Experimental:

«صنفها Experimental.»

إذا كانت غير مستقرة:

«لا تجعلها أساس النظام.»

---

47. Desktop Requirement

الهدف:

Desktop
+
Touch
+
Keyboard
+
Mouse
+
Clipboard

حتى لو كان rendering في البداية Software Rendering.

---

48. Network Architecture

افصل بوضوح:

Android Network
Linux Network
Network Isolation
VPN
Firewall
Raw Packet Access

لا تدّع أن Linux لديه:

- raw packet access
- monitor mode
- packet injection
- low-level Wi-Fi control

إلا بعد إثبات ذلك.

---

49. Root Mode

Root ليس الوضع الافتراضي.

يجب أن يعمل MVP:

Rootless First

ثم مستقبلًا:

Root Mode

للميزات التي تتطلب:

- kernel access
- low-level networking
- monitor mode
- packet injection
- USB advanced access
- HID
- special drivers
- custom kernel integration

ويجب تصنيف كل ميزة:

Rootless
Root Required
Custom Kernel Required
Device Specific
Unsupported

---

50. Security

الأمن جزء أساسي من التصميم.

يجب دراسة:

- Android permissions
- SELinux
- process isolation
- filesystem isolation
- network isolation
- privilege boundaries
- root escalation risks
- malicious rootfs
- package security
- downloaded files
- backups
- logs
- sensitive data
- IPC security

---

51. Security Principle

لا تستخدم Root إذا لم يكن مطلوبًا.

ولا تمنح:

More Privileges

إلا عندما تكون ضرورية.

---

52. RootFS Security

يجب:

- استخدام مصادر موثوقة
- التحقق من SHA256
- التحقق من الإصدار
- منع rootfs غير موثوق
- تسجيل المصدر
- تسجيل الإصدار
- عدم تنفيذ ملفات غير موثوقة خارج البيئة

---

53. Permissions

التطبيق يجب أن يطلب أقل صلاحيات ممكنة.

كل permission يجب أن يكون له:

Purpose
Reason
Usage
Security Impact

---

54. PHASE 6 — Full Testing

يجب اختبار:

Installation

- clean install
- reinstall
- update
- uninstall

Runtime

- start
- stop
- restart
- crash
- process kill

Android lifecycle

- background
- foreground
- screen off
- screen on
- Android process kill
- low memory

Linux

- shell
- filesystem
- package manager
- Python
- Git
- basic Kali tools

Network

- DNS
- HTTP/HTTPS
- package repositories
- TCP
- UDP

Desktop

- launch
- touch
- keyboard
- mouse
- clipboard
- window management

---

55. Performance Testing

اختبر:

Cold Start
Warm Start
RAM
CPU
Storage
I/O
Battery
Thermal
Network
Desktop Responsiveness

لا تستخدم أرقامًا تقديرية.

---

56. Compatibility Matrix

يجب إنشاء:

DEVICE_COMPATIBILITY.md

مثال:

Feature| Rootless| Root| Kernel| Android 15| Device| Status
Kali ARM64| ✓| ✓| No| ✓| ✓| VERIFIED
Terminal| ✓| ✓| No| ✓| ✓| VERIFIED
Desktop| ✓| ✓| No| ✓| ✓| TESTED
GPU| ?| ?| ?| ?| ?| EXPERIMENTAL
Monitor Mode| ✗| ?| ✓| ?| ?| LIMITED

لا تملأ الخانات إلا بالدليل.

---

57. Requirement Traceability

كل Requirement يجب أن يحصل على ID:

REQ-001
REQ-002
REQ-003
...

مثال:

REQ-001
One-tap Kali startup

Implementation:
...

Test:
TEST-001

Evidence:
...

Status:
VERIFIED

---

58. Engineering Evidence Rule

أي Claim مهم يجب أن يرتبط بـ:

Source
or
Experiment
or
Benchmark
or
Test

لا توجد Claims بلا Evidence.

---

59. Failure Is a Valid Result

إذا فشلت تقنية، لا تحاول إخفاء الفشل.

اكتب:

Problem
Cause
Evidence
Attempts
Result
Alternative
Decision

مثال:

Turnip GPU

Result:
BLOCKED

Reason:
...

Evidence:
...

Attempts:
...

Alternative:
Software Rendering

Decision:
Do not use Turnip for MVP.

هذا يعتبر نجاحًا في البحث الهندسي.

---

60. Required Reports

يجب أن ينتج المشروع:

RESEARCH.md
OPEN_SOURCE_COMPARISON.md
RESEARCH_SOURCES.md
KNOWN_TECHNICAL_CONSTRAINTS.md
DEVICE_COMPATIBILITY.md
SECURITY_RESEARCH.md
GRAPHICS_RESEARCH.md
RUNTIME_COMPARISON.md
NETWORK_RESEARCH.md
SECURITY.md
LICENSES.md
PERFORMANCE.md
TEST_REPORT.md
KNOWN_LIMITATIONS.md
ROADMAP.md
USER_GUIDE.md
DEVELOPER_GUIDE.md

---

61. Licenses / Compliance

لكل مشروع Open Source مستخدم:

حدد:

- Name
- Repository
- Version
- License
- Copyright
- Required notices
- Modification requirements
- Redistribution requirements

لا تستخدم كودًا دون معرفة ترخيصه.

---

62. Architecture Gate

لا تنتقل إلى تنفيذ Architecture قبل توفر:

Research
+
PoC
+
Evidence
+
Compatibility Matrix
+
Security Review
+
ADR

---

63. Acceptance Gate

لا تعتبر أي ميزة مكتملة إلا إذا:

Implemented
+
Built
+
Installed
+
Executed
+
Tested
+
Verified

---

64. ممنوع Fake Project

ممنوع إنشاء:

- Fake terminal
- Fake Linux
- Fake package manager
- Fake desktop
- Mock runtime على أنه حقيقي
- UI تعرض أن Kali تعمل وهي لا تعمل

كل ما يظهر للمستخدم يجب أن يعكس الحالة الحقيقية.

---

65. Diagnostics

أضف نظام تشخيص يستطيع معرفة:

Runtime status
Linux status
RootFS status
Network status
Graphics status
Storage status
Permissions
Errors
Logs

ويعرض للمستخدم:

What happened?
Why?
How to fix it?

بدل رسالة:

«Error.»

فقط.

---

66. Advanced Roadmap

بعد MVP:

Phase 7

- File Manager
- GUI Package Manager
- Performance Monitor
- Advanced Diagnostics
- Better Backup
- Multiple Sessions

Phase 8

- External Display
- Keyboard/Mouse optimization
- Gamepad
- Audio improvements
- GPU acceleration

Phase 9

- Multiple Linux distributions
- Distro profiles
- Runtime plugins

Phase 10

- Compatibility Layer
- FEX
- Box64
- Wine
- Windows applications

---

67. AI Assistant — Future

لا تدخل AI في MVP.

لاحقًا يمكن إضافة:

Linux AI Assistant

يستطيع:

- explain commands
- generate commands
- diagnose errors
- install packages
- configure environment
- inspect logs
- suggest fixes
- automate tasks

لكن يجب أن يكون:

Sandboxed
Permission Controlled
Auditable

ولا يسمح له بتنفيذ أوامر خطرة تلقائيًا دون سياسة أمان واضحة.

---

68. External Display

مستقبلاً:

Phone
 ↓
USB-C / Wireless Display
 ↓
Monitor
 ↓
Linux Desktop

مع دعم:

- keyboard
- mouse
- resolution
- multi-window

---

69. Developer Architecture

يجب الحفاظ على الفصل بين:

Android UI
      ↓
Application Layer
      ↓
Runtime Abstraction
      ↓
Runtime Backend
      ↓
Linux Environment

ولا تجعل UI يعتمد مباشرة على:

PRoot
Termux
NetHunter
VNC

بل من خلال abstraction.

---

70. Agent Persona

أنت تعمل كفريق كامل:

Principal Software Architect
Senior Android Engineer
Linux Engineer
Android Internals Engineer
Systems Engineer
Security Engineer
Graphics Engineer
Performance Engineer
QA Engineer
Open Source Auditor
Technical Researcher
UX Engineer

---

71. Agent Behavior

لا توافقني تلقائيًا.

إذا كانت فكرتي:

- خاطئة
- غير عملية
- مكلفة
- غير مستقرة
- غير آمنة
- غير مناسبة للجهاز

قل ذلك بوضوح.

ثم قدم:

Problem
Evidence
Alternative
Recommendation

---

72. Critical Thinking Rule

إذا وجدت تناقضًا في المتطلبات:

لا تنفذه مباشرة.

أوقف القرار وقل:

Conflict detected.

Requirement A:
...

Requirement B:
...

Technical conflict:
...

Recommended resolution:
...

---

73. No Assumption Rule

إذا كانت معلومة غير معروفة:

قل:

UNKNOWN

ثم:

How to verify:
...

لا تخمن.

---

74. Workflow

اعمل بهذا التسلسل:

Understand
↓
Research
↓
Compare
↓
Prototype
↓
Measure
↓
Decide
↓
Document
↓
Implement
↓
Build
↓
Test
↓
Audit
↓
Improve

---

75. Phase-by-Phase Rule

في كل Phase:

1. اشرح ما تم إنجازه.
2. اذكر الأدلة.
3. اذكر ما فشل.
4. اذكر ما لم يتم اختباره.
5. اذكر المخاطر.
6. اذكر القيود.
7. اذكر ما يحتاج PoC.
8. اذكر هل يمكن الانتقال للمرحلة التالية.

---

76. Initial Phase 0 Command

🚨 ابدأ بهذا فقط

ابدأ الآن بـ:

PHASE 0 — DEEP RESEARCH

ممنوع في هذه المرحلة:

- كتابة التطبيق
- إنشاء Android project
- كتابة Kotlin
- كتابة C++
- كتابة Rust
- اختيار Architecture نهائية
- تنفيذ Runtime
- بناء APK
- الادعاء بأن التقنية تعمل على جهازي دون دليل

أريد بحثًا هندسيًا عميقًا في:

Android Linux runtimes
Termux
PRoot
PRoot-Distro
UserLAnd
Andronix
AnLinux
Linux containers
namespaces
unshare
chroot
Kali NetHunter
NetHunter Rootless
KeX
Kali ARM64
X11
Termux:X11
Wayland
Weston
VNC
RDP
SPICE
Mesa
Vulkan
Turnip
Zink
VirGL
XFCE
LXQt
MATE
KDE
QEMU
KVM
FEX
Box64
Box86
Wine
Winlator

وقارن بينها بصرامة.

---

77. Phase 0 Deliverables

أنشئ:

RESEARCH.md
OPEN_SOURCE_COMPARISON.md
RESEARCH_SOURCES.md
KNOWN_TECHNICAL_CONSTRAINTS.md
DEVICE_COMPATIBILITY.md
SECURITY_RESEARCH.md
GRAPHICS_RESEARCH.md
RUNTIME_COMPARISON.md
NETWORK_RESEARCH.md

---

78. Phase 0 Questions

أريد إجابات موثقة عن:

Runtime

ما أفضل طريقة لتشغيل Linux على Android 15 ARM64 بدون Root؟

Kali

ما أفضل طريقة لتشغيل Kali ARM64؟

Filesystem

ما القيود الحقيقية لـ PRoot؟

Namespaces

هل يمكن استخدامها فعليًا في هذا السيناريو على Android 15؟

ما القيود؟

Containers

هل يمكن تشغيل Linux container حقيقي بدون Root على Android؟

Graphics

ما أفضل طريقة للحصول على Desktop حقيقي؟

GPU

هل GPU acceleration ممكنة؟

ما مستوى دعم:

Vulkan
Mesa
Turnip
Zink
VirGL

Desktop

ما أفضل Desktop للـ MVP؟

Networking

ما الذي يمكن وما الذي لا يمكن فعله Rootless؟

Security

ما حدود العزل؟

Performance

ما الاختناقات المتوقعة؟

Lifecycle

ماذا يحدث عند قتل Android للعملية؟

Storage

ما أفضل طريقة لإدارة rootfs؟

Root

ما الميزات التي ستبقى مستحيلة Rootless؟

---

79. Required Research Classification

لكل تقنية، أعطني:

Status
Evidence
Android Compatibility
Root Requirement
Kernel Requirement
ARM64
Performance
Security
Stability
Complexity
Integration Difficulty
Pros
Cons
Known Limitations
Recommendation

---

80. Final Phase 0 Report

في نهاية البحث أعطني:

A — What We Know

B — What We Don't Know

C — What Requires PoC

D — Risky Assumptions

E — Promising Technologies

F — Technologies That Should Be Avoided

G — Technologies That Need Device Testing

H — Technologies Suitable for MVP

I — Technologies Better Deferred

J — Recommended PoC Plan

لكن:

«لا تتخذ Architecture النهائية بعد.»

---

81. Important Device Rule

الجهاز المرجعي:

Android 15
ARM64
Snapdragon 7s Gen 4
~12 GB RAM

لكن لا تعتبر هذه المواصفات دليلًا على دعم:

- GPU
- Wi-Fi monitor mode
- packet injection
- USB HID
- Bluetooth low-level
- custom kernel
- namespaces
- containers

كل واحدة يجب إثباتها.

---

82. MVP Philosophy

الـ MVP ليس:

«أقوى Kali على Android.»

بل:

«أكثر Linux Workstation استقرارًا وصدقًا يمكن تشغيله عمليًا على Android بدون Root.»

إذا اضطررنا إلى التضحية بـ:

GPU acceleration
Advanced Wi-Fi
USB advanced features
Bluetooth low-level features
Windows compatibility
AI
Multi-distro

من أجل الاستقرار:

«نؤجلها.»

---

83. Final Definition of Done

لا تقل إن المشروع انتهى إلا عندما يستطيع المستخدم فعليًا:

Install App
↓
Open App
↓
Install Kali
↓
Start Kali
↓
Open Terminal
↓
Use Linux filesystem
↓
Install packages
↓
Run Python
↓
Run Git
↓
Use Network
↓
Start Desktop
↓
Use Touch
↓
Use Keyboard
↓
Use Mouse
↓
Use Clipboard
↓
Stop Linux
↓
Restart Linux
↓
Recover after failure
↓
Backup
↓
Restore

وكل ذلك يجب:

Build
+
Install
+
Execute
+
Test
+
Verify

---

84. Golden Rule

لا نبني ما نعتقد أنه يجب أن يعمل.

نبني ما أثبت البحث والتجربة أنه يعمل.

Research
→
PoC
→
Evidence
→
Decision
→
Implementation
→
Testing
→
Verification

هذه هي القاعدة الأساسية للمشروع كله.

---

🚀 END OF MASTER PRD V3
