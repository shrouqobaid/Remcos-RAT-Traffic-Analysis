# Malware Traffic Analysis: Remcos RAT Case Study

![Wireshark](https://img.shields.io/badge/Tool-Wireshark-blue?style=flat-square&logo=wireshark)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)
![Category](https://img.shields.io/badge/Category-Malware%20Analysis-red?style=flat-square)

## نبذة عن المشروع
هذا المشروع ليس مجرد توثيق لخطوات فنية، بل هو محاكاة لتحقيق جنائي رقمي قمت به لتحليل حركة شبكة مشبوهة تتبع دورة حياة هجوم برمجية **Remcos RAT**. استعرضت فيه منهجية ربط الأدلة الرقمية واستخراج مؤشرات الاختراق (IOCs) مع تطبيق إطار MITRE ATT&CK.

---

## تفاصيل بيئة التحقيق
* **الأداة الأساسية:** Wireshark (على بيئة Kali Linux).
* **المصادر الاستخباراتية:** VirusTotal & Malware-Traffic-Analysis.net.

---

## التسلسل الزمني والتحليل الفني (Investigation Steps)

### 1️⃣ تحديد الضحية ونقطة الدخول (Initial Access)
أثناء مراقبة أول اتصال صادر، لفت نظري استعلام DNS متبوعاً بطلب `HTTP GET` لموقع **OneDrive**. 
* **ملاحظة شخصية:** استغلال المنصات الموثوقة مثل OneDrive هو تكتيك ذكي لتجاوز أنظمة الـ Proxy، وهذا ما جعلني أركز على هذا المسار بالتحديد.
* **الضحية (Victim):** `10.5.29.101`

![Victim Identification](screenshots/victim_identification.png)

### 2️⃣ تحليل الحزمة وتحليل الروابط (Packet Inspection)
بالغوص في تفاصيل البروتوكول، استخرجت الرابط المباشر المستخدم لجلب الملف الخبيث.
* **النتيجة:** الرابط يظهر استخدام مفاتيح برمجية مثل `authkey` داخل الرابط، وهي تقنية تستخدم لضمان وصول الضحية المستهدفة فقط للحمولة (Payload).

![Artifact Recovery](screenshots/artifact_recovery.png)

### 3️⃣ كشف سيرفر التحكم (C2 Beaconing)
هنا كانت المرحلة الحاسمة؛ لاحظت تكرار اتصال (Beaconing) بتردد منتظم باتجاه عنوان IP في فرنسا.
* **سيرفر المهاجم (C2):** `146.70.158.105` عبر المنفذ `9138`.
* **التحليل الفني:** استخدام منفذ غير قياسي مثل `9138` زاد من شكوكي، وبعد مطابقة البيانات مع قواعد بيانات Threat Intelligence، تأكدت من ارتباطه ببرمجية Remcos RAT.

![C2 Beaconing](screenshots/c2_beaconing.png)

---

## MITRE ATT&CK Mapping
لتعميق فهم سلوك المهاجم، قمت بربط الملاحظات بإطار MITRE ATT&CK:

| ID | Tactic | Technique | Description |
| :--- | :--- | :--- | :--- |
| **T1566** | Phishing | Spearphishing Link | استخدام روابط ملغمة (OneDrive) لإيصال البرمجية. |
| **T1071** | Command and Control | Application Layer Protocol | استخدام بروتوكولات طبقة التطبيق للتواصل مع سيرفر C2. |
| **T1547** | Persistence | Boot or Logon Autostart | تكتيك يضمن بقاء البرمجية تعمل حتى بعد إعادة التشغيل. |

---

## مؤشرات الاختراق المستخرجة (Captured IOCs)

| النوع (Type) | القيمة (Value) | الوصف (Description) |
| :--- | :--- | :--- |
| **C2 Server** | `146.70.158.105` | سيرفر التحكم والمسيطرة (Command & Control) |
| **C2 Port** | `9138` | المنفذ المستخدم لنشاط الـ Beaconing |
| **Malware** | `Remcos RAT` | نوع البرمجية الخبيثة المكتشفة |
| **Password** | `infected` | كلمة مرور ملف الـ ZIP المرفق للأدلة |

---
## فرص الكشف والاصطياد (Detection & Hunting Opportunities)
بناءً على هذا التحليل، يمكن لفرق الـ SOC اكتشاف نشاط مشابه عبر مراقبة المعايير التالية:

* **الاتصالات الخارجية المشبوهة (Suspicious Outbound):** تتبع تحميل ملفات تنفيذية من روابط سحابية مثل OneDrive متبوعة بنشاط غير معتاد.
* **المنافذ غير القياسية (Unusual Ports):** تفعيل تنبيهات عند رصد تواصل عبر المنفذ `9138`.
* **أنماط المناداة المشفرة (Encrypted Beaconing):** البحث عن اتصالات دورية (Periodic communication) ذات حجم بيانات ثابت.
* **ثبات النظام (Registry Persistence):** مراقبة التغييرات في مفاتيح سجل النظام (Auto-run keys) التي تستخدمها برمجية Remcos.
* **استخدام الـ DNS الديناميكي:** مراقبة استعلامات الـ DNS لخدمات مثل `geoplugin` بشكل مفاجئ.

---

## دروس مستفادة وتحديات (Personal Reflection)
* **تحدي واجهته:** في البداية، كان من الصعب تمييز حركة الـ C2 عن حركة الشبكة العادية، لكن باستخدام فلاتر Wireshark المتقدمة (مثل تحليل الـ SYN Packets وتردد الـ Beaconing)، استطعت حصر السيرفر المشبوه.
* **الدرس المستفاد:** أهمية مراقبة المنافذ غير المعروفة (Non-standard ports) حتى لو كان الاتصال يبدو صغيراً في الحجم، فغالباً ما تخفي خلفها أوامر التحكم (C2 Commands).
---

## محتويات المستودع
* [screenshots/](./screenshots/) : لقطات شاشة لخطوات التحليل موثقة من بيئة العمل.
* [pcap_files/](./pcap_files/) : الأدلة الجنائية (PCAP) مضغوطة بكلمة مرور `infected`.
* [README.md](./README.md) : هذا التقرير التحليلي.

---
| مسار محلل SOC*
