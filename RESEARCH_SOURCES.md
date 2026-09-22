# RESEARCH_SOURCES.md
## Phase 0 — مصادر البحث (البند 16 و17)

| الحقل | القيمة |
|---|---|
| المرحلة | PHASE 0 |
| الحالة | جولة بحث أولى (2026-09-12) |
| ترتيب المصادر المُتبع | كود مصدري → وثائق رسمية → مستودعات → إصدارات → changelogs → issues → PRs → wiki → forks → مشاريع تعتمد → تقارير مجتمع |

---

## 1. تصنيف موثوقية المصادر

| المستوى | النوع | كيف نستخدمه |
|---|---|---|
| S1 | كود مصدري / مستودع رسمي | دليل قوي |
| S2 | وثائق رسمية (Android Developers، kali.org) | دليل قوي |
| S3 | issues/PRs في المستودعات الرسمية | مؤشر + سياق |
| S4 | إصدارات/changelogs | دليل على النشاط والتطور |
| S5 | تقارير مجتمع (مدونات، Reddit، منتديات) | **مؤشر فقط** — لا يُبنى عليه قرار معماري وحده |

---

## 2. المصادر حسب المجال

### 2.1 Runtime / PRoot
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| R-01 | https://github.com/termux/proot-distro | مستودع رسمي | S1 | آلية عمل PRoot + **القائمة الرسمية للقيود** [1](https://github.com/termux/proot-distro) |
| R-02 | https://github.com/proot-me/PRoot/releases | إصدارات | S4 | آخر إصدار upstream v5.4.0 (2023-05-13) [2](https://github.com/proot-me/PRoot/releases) |
| R-03 | https://launchpad.net/ubuntu/+source/proot | حزمة توزيعة | S4 | تتبع الإصدارات (5.4.0-3) |
| R-04 | https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c | تقرير تقني | S5 | `unshare(CLONE_NEWUSER) = EPERM` على Android غير مروّت [1](https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c) |
| R-05 | https://www.sitepoint.com/hardened-mobile-dev-a-termux-docker-guide-for-grapheneos/ | مقال تقني | S5 | عدم إمكانية Docker Rootless + تقديرات أداء غير مقيسة |
| R-06 | https://termuxtools.com/proot-distro-linux-termux/ | مقال | S5 | ادعاءات أداء (~5% overhead) — **غير موثوقة، UNVERIFIED** |

### 2.2 قيود منصة Android
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| A-01 | https://github.com/termux/termux-packages/wiki/Termux-execution-environment | wiki رسمي | S2 | قيد W^X على `targetSdk >= 29` [1](https://github.com/termux/termux-packages/wiki/Termux-execution-environment) |
| A-02 | https://www.reddit.com/r/androiddev/comments/b2inbu/psa_android_q_blocks_executing_binaries_in_your/ | نقاش + رابط AOSP | S3/S5 | توضيح رسمي من Google بأن المنع working-as-intended (commit `0dd738d8`) [1](https://www.reddit.com/r/androiddev/comments/b2inbu/psa_android_q_blocks_executing_binaries_in_your/) |
| A-03 | https://github.com/termux/termux-exec-package | مستودع رسمي | S1 | حل `system_linker_exec` عبر `/system/bin/linker*` (إصدار 2.5.0) [1](https://github.com/termux/termux-exec-package) |
| A-04 | https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/ | مقال | S5 | استراتيجية Termux بالبقاء على `targetSdk 28` + تقييد `/proc/net` [1](https://www.xda-developers.com/termux-terminal-linux-google-play-updates-stopped/) |
| A-05 | https://android-developers.googleblog.com/2024/08/adding-16-kb-page-size-to-android.html | وثيقة رسمية | S2 | متطلب 16 KB page size لكل تطبيق فيه كود أصلي [5](https://android-developers.googleblog.com/2024/08/adding-16-kb-page-size-to-android.html) |
| A-06 | https://github.com/termux/termux-app/issues/4185 | issue | S3 | أثر 16 KB على Termux [3](https://github.com/termux/termux-app/issues/4185) |
| A-07 | https://github.com/termux/termux-packages/issues/21688 | issue | S3 | نفس المسألة على مستوى الحزم [4](https://github.com/termux/termux-packages/issues/21688) |
| A-08 | https://cosyra.com/guides/termux-signal-9-fix.html | دليل | S5 | حد 32 عملية + طرق التعطيل حسب إصدار Android [1](https://cosyra.com/guides/termux-signal-9-fix.html) |
| A-09 | https://github.com/sabamdarif/termux-desktop/blob/main/docs/disable-phantom-process-killing.md | توثيق مجتمعي | S5 | أوامر تعطيل قاتل العمليات الوهمية [2](https://github.com/sabamdarif/termux-desktop/blob/main/docs/disable-phantom-process-killing.md) |
| A-10 | https://ivonblog.com/en-us/posts/fix-termux-signal9-error/ | دليل | S5 | `DEFAULT_MAX_PHANTOM_PROCESSES = 32` (لكل النظام) [3](https://ivonblog.com/en-us/posts/fix-termux-signal9-error/) |

### 2.3 Kali / NetHunter
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| K-01 | https://www.kali.org/docs/nethunter/nethunter-rootless/ | **وثيقة رسمية** | S2 | بنية NetHunter Rootless + أوامر KeX + القيود المعلنة [1](https://www.kali.org/docs/nethunter/nethunter-rootless/) |
| K-02 | https://www.kali.org/docs/development/kali-linux-arm-chroot/ | **وثيقة رسمية** | S2 | طريقة بناء rootfs ARM مخصص [2](https://www.kali.org/docs/development/kali-linux-arm-chroot/) |
| K-03 | https://github.com/zulfi0/install_rootfs_android | طرف ثالث | S5 | روابط rootfs Kali (kalifs-arm64-full/minimal) — **يحتاج تحققًا رسميًا** [1](https://github.com/zulfi0/install_rootfs_android) |
| K-04 | https://deepwiki.com/LinuxDroidMaster/Termux-Desktops/6.5-kali-nethunter-proot-setup | توثيق مجتمعي | S5 | مثال عملي لأوامر proot لتشغيل NetHunter [3](https://deepwiki.com/LinuxDroidMaster/Termux-Desktops/6.5-kali-nethunter-proot-setup) |

### 2.4 الرسوميات
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| G-01 | https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md | توثيق مجتمعي | S5 | مسارات VirGL/Zink/Turnip + شرط DRI3 [1](https://github.com/cheadrian/termux-chroot-proot-wine-box86_64/blob/main/Hardware_Acceleration_Resources.md) |
| G-02 | https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/ | تقرير مجتمع | S5 | دعم Adreno 830/840 + أرقام أداء غير رسمية [2](https://www.reddit.com/r/termux/comments/1qhy64o/freedreno_and_turnip_drivers_now_support_adreno/) |
| G-03 | https://ivonblog.com/en-us/posts/termux-x11/ | دليل | S5 | متطلبات Termux:X11 + تاريخ التنفيذ (XWayland → XCB) [3](https://ivonblog.com/en-us/posts/termux-x11/) |
| G-04 | https://termux-x11.en.uptodown.com/android | متجر بديل | S5 | إصدار Termux:X11 المرصود (1.03.01) [2](https://termux-x11.en.uptodown.com/android) |
| G-05 | https://vuink.com/post/tvguho-d-dpbz/termux/termux-x11 | نسخة مستودع | S5 | متطلب Android 8+ + التوزيع عبر nightly [1](https://vuink.com/post/tvguho-d-dpbz/termux/termux-x11) |

### 2.5 الجهاز
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| D-01 | https://www.91mobiles.com/processor/qualcomm-snapdragon-7s-gen-4-pdp | مواصفات | S5 | GPU = Adreno 810، Vulkan 1.3، GLES 3.2 [1](https://www.91mobiles.com/processor/qualcomm-snapdragon-7s-gen-4-pdp) |
| D-02 | https://inquisitiveuniverse.com/2025/09/17/snapdragon-7s-gen-4-specs-and-benchmarks-review/ | مراجعة | S5 | مستوى أداء Adreno 810 النسبي [2](https://inquisitiveuniverse.com/2025/09/17/snapdragon-7s-gen-4-specs-and-benchmarks-review/) |
| D-03 | https://nanoreview.net/en/soc/qualcomm-snapdragon-7s-gen-4 | مواصفات | S5 | تأكيد Adreno 810 [3](https://nanoreview.net/en/soc/qualcomm-snapdragon-7s-gen-4) |

### 2.6 الشبكة
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| N-01 | https://github.com/emanuele-f/PCAPdroid | مستودع رسمي | S1 | التقاط حزم Rootless عبر محاكاة VPN (LIMITED) [1](https://github.com/emanuele-f/PCAPdroid) |

### 2.7 مشاريع مشابهة
| # | المصدر | النوع | المستوى | ما أفادنا به |
|---|---|---|---|---|
| P-01 | https://grokipedia.com/page/UserLAnd_Technologies | مصدر ثانوي | S5 | UserLAnd: MIT، خليفة GNURoot Debian [1](https://grokipedia.com/page/UserLAnd_Technologies) |
| P-02 | https://alternativeto.net/software/userland/about/ | دليل برمجيات | S5 | UserLAnd: CypherpunkArmory، مفتوح المصدر |
| P-03 | https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/ | نقاش | S5 | قيود AVF/pKVM لتطبيقات طرف ثالث [2](https://www.reddit.com/r/termux/comments/1f4c4bs/what_is_android_virtualization_framework_and_pkvm/) |

### 2.8 محاكاة/توافق (مؤجل)
| # | المصدر | النوع | المستوى |
|---|---|---|---|
| E-01 | https://byteiota.com/fex-emu-x86-arm64-linux/ | مقال | S5 |
| E-02 | https://www.murtpoiss.ee/fex-emu-run-x86-applications-on-arm64-linux-devices/ | مقال | S5 |
| E-03 | https://emulation.gametechwiki.com/index.php/FEX-Emu | wiki | S5 |

---

## 3. نقاط الضعف في مصادر هذه الجولة (اعتراف منهجي)

| # | الضعف | الأثر | المعالجة |
|---|---|---|---|
| W-01 | كثافة الاعتماد على مصادر S5 (مدونات/Reddit) في الجرافيكس والأداء | قرارات غير قابلة للبناء عليها | سدّها بمصادر S1/S2 قبل Phase 1 |
| W-02 | تراخيص Termux:X11 وPRoot وUserLAnd غير متحقق منها من ملف LICENSE | مخاطر امتثال (البند 61) | مهام V-09..V-12 |
| W-03 | لا يوجد مصدر رسمي مباشر لروابط rootfs Kali في هذه الجولة | خطر أمني | مهمة S-01 |
| W-04 | لا وجود لأي قياس على الجهاز المرجعي | كل أرقام الأداء مفقودة | موجة PoC الأولى |
| W-05 | معلومات HyperOS/OEM مفقودة تمامًا | قد تنسف افتراضات التصميم | اختبار مبكر في POC-011 |

---

## 4. مهام التحقق المفتوحة (Backlog)

| # | المهمة | الأولوية |
|---|---|---|
| V-01 | الحصول على مصدر رسمي (kali.org/kali.download) لـ rootfs Kali ARM64 + SHA256/توقيع | **عالية** |
| V-02 | التحقق من إصدار وترخيص ومتطلبات Termux:X11 من مستودعه الرسمي | **عالية** |
| V-03 | التحقق من دعم Termux:X11 لـ Android 15 و16 KB pages | **عالية** |
| V-04 | تقييم إمكانية تضمين X server داخل تطبيقنا (بدل APK منفصل) | متوسطة |
| V-05 | حالة Wayland/Weston العملية على Android 15 | منخفضة |
| V-06 | دعم Adreno 810 في Mesa/Freedreno upstream | متوسطة |
| V-07 | ترخيص ونشاط Andronix | متوسطة |
| V-08 | ترخيص ونشاط AnLinux | متوسطة |
| V-09 | ملف LICENSE الخاص بـ PRoot (تحديد GPL v2 أم أحدث) | **عالية** |
| V-10 | ملف LICENSE الخاص بـ Termux:X11 | **عالية** |
| V-11 | ملف LICENSE الرسمي لـ UserLAnd (من المستودع لا من مصدر ثانوي) | متوسطة |
| V-12 | تراخيص Mesa / virglrenderer / X.Org المكوّنات | متوسطة |
| V-13 | حجم صفحة الذاكرة على الجهاز المرجعي (4 أم 16 KB) | **عالية** |
| V-14 | إصدار نواة الجهاز + إعدادات namespaces | **عالية** |
