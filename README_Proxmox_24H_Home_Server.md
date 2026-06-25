# Proxmox VE 24/7 Cloud Server Guide

**Prepared by: izzaldeen mansour**  

هذا الملف يشرح فكرة استخدام **Proxmox VE** كمنصة تشغيل دائمة 24/7 على جهاز مستقل، مثل كمبيوتر قديم في البيت أو هارد/SSD مخصص، بهدف تشغيل Virtual Machines و Containers ومشاريع شبكات أو سيرفرات بشكل دائم.

---

## ⚠️ تحذير مهم جدًا قبل البدء

> **لا تثبّت Proxmox VE على جهازك الأساسي الذي عليه Windows مباشرة.**  
> تثبيت Proxmox يتم كنظام تشغيل كامل على الجهاز، وقد يقوم بحذف البيانات والأنظمة الموجودة على القرص الذي تختاره أثناء التثبيت.

استخدم هذا الدليل فقط في الحالات التالية:

- عندك جهاز قديم منفصل في البيت.
- عندك هاردسك أو SSD فاضي ومخصص للتجربة.
- عندك Mini PC أو Desktop تريد تحويله إلى Home Server.
- فاهم أن الجهاز بعد تثبيت Proxmox سيصبح سيرفر وليس جهاز Windows عادي.

**إذا عندك ملفات مهمة على القرص، انسخها قبل أي خطوة.**  
**لا تضغط Install قبل أن تتأكد 100% أنك اخترت القرص الصحيح.**

---

## 1. ما هو Proxmox VE؟

Proxmox VE هو نظام تشغيل مخصص للسيرفرات يسمح لك بتشغيل:

- Virtual Machines
- Linux Containers
- شبكات افتراضية
- أنظمة Linux/Windows داخل السيرفر
- خدمات ومشاريع تعمل 24/7

الفكرة أن الجهاز يصبح منصة مركزية. تدخل عليه من المتصفح وتدير كل شيء من Web Interface.

---

## 2. لماذا نستخدم Proxmox؟

نستخدم Proxmox عندما نريد جهازًا يعمل طول الوقت، وليس فقط أثناء فتح اللابتوب.

مثال:

```text
Old Desktop / Mini PC
        |
        Proxmox VE
        |
   ┌────┼────┐
 Ubuntu VM   Network Lab VM   Docker VM
```

يعني بدل ما تشغل كل مشروع يدويًا على Windows أو WSL، تخلي عندك سيرفر دائم في البيت.

---

## 3. متى أستخدم Proxmox ومتى لا؟

### استخدم Proxmox إذا:

- عندك جهاز منفصل.
- تريد تشغيل خدمات 24/7.
- تريد تجربة أكثر من VM أو Container.
- تريد بيئة قريبة من السيرفرات الحقيقية.
- تريد تشغيل مشاريع تخرج أو مختبر شبكات أو سيرفرات تجريبية.

### لا تستخدم Proxmox إذا:

- الجهاز هو لابتوبك الأساسي وعليه Windows وملفاتك.
- ما عندك نسخة احتياطية.
- ما تعرف أي قرص تختار أثناء التثبيت.
- هدفك فقط تجربة Docker بسيطة؛ هنا WSL + Ubuntu + Docker يكفي.

---

## 4. المتطلبات المقترحة

### الحد الأدنى للتجربة

- CPU يدعم 64-bit.
- دعم Virtualization: Intel VT-x أو AMD-V.
- RAM: يفضل 8GB على الأقل للتجارب المريحة.
- Storage: SSD أو HDD فاضي.
- Network card.
- USB flash drive لتثبيت Proxmox.

### الأفضل لمختبر منزلي

- RAM: 16GB أو أكثر.
- SSD للنظام.
- HDD/SSD إضافي للتخزين.
- Ethernet cable بدل Wi-Fi.
- جهاز يقدر يظل شغال 24/7.

---

## 5. الفكرة العامة للتثبيت

الخطوات العامة تكون كالتالي:

```text
Download Proxmox ISO
        ↓
Burn ISO to USB
        ↓
Boot old PC from USB
        ↓
Install Proxmox on empty disk
        ↓
Open Proxmox Web Interface
        ↓
Create VM or Container
        ↓
Run Ubuntu / Docker / Network Projects
```

---

## 6. تحميل Proxmox ISO

ادخل إلى الموقع الرسمي لـ Proxmox وحمل ملف ISO الخاص بـ Proxmox VE.

ابحث عن:

```text
Proxmox VE ISO Installer
```

يفضل دائمًا التحميل من الموقع الرسمي فقط.

---

## 7. حرق ISO على USB

استخدم أداة مثل:

- Rufus
- Balena Etcher

الخطوات العامة:

1. وصل USB بالجهاز.
2. افتح Rufus أو Balena Etcher.
3. اختر ملف Proxmox ISO.
4. اختر USB.
5. اضغط Start / Flash.
6. انتظر حتى ينتهي.

> ملاحظة: هذه العملية ستحذف محتوى USB.

---

## 8. الإقلاع من USB

على الجهاز القديم أو الجهاز المخصص للسيرفر:

1. وصل USB.
2. شغل الجهاز.
3. ادخل Boot Menu.
4. اختر USB.
5. ستظهر شاشة تثبيت Proxmox.

أزرار Boot Menu تختلف حسب الجهاز:

```text
F12
F11
F9
ESC
DEL
```

---

## 9. تثبيت Proxmox VE

بعد الإقلاع من USB:

1. اختر Install Proxmox VE.
2. وافق على الشروط.
3. اختر القرص الذي تريد تثبيت Proxmox عليه.
4. اختر الدولة والوقت ولوحة المفاتيح.
5. أدخل كلمة مرور قوية وبريد إلكتروني.
6. اختر Network Interface.
7. حدد Hostname و IP.

مثال:

```text
Hostname: proxmox-home.local
IP Address: 192.168.1.50
Gateway: 192.168.1.1
DNS: 1.1.1.1
```

---

## ⚠️ ملاحظة خطيرة عند اختيار القرص

أثناء التثبيت ستصل إلى خطوة اختيار القرص.

تأكد من الآتي:

- لا تختار قرص عليه Windows.
- لا تختار قرص عليه ملفات مهمة.
- لا تثبت على قرص لابتوبك الأساسي.
- إذا كان عندك أكثر من قرص، افصل الأقراص المهمة قبل التثبيت إن أمكن.

Proxmox سيستخدم القرص المختار كنظام تشغيل للسيرفر، وقد يتم حذف ما عليه.

---

## 10. الدخول إلى واجهة Proxmox

بعد انتهاء التثبيت سيظهر لك رابط مثل:

```text
https://192.168.1.50:8006
```

افتح هذا الرابط من متصفح على جهاز آخر داخل نفس الشبكة.

قد تظهر رسالة تحذير SSL في المتصفح، اضغط Advanced ثم Continue.

بيانات الدخول:

```text
Username: root
Password: كلمة المرور التي اخترتها أثناء التثبيت
Realm: Linux PAM
```

---

## 11. أول إعدادات بعد التثبيت

بعد الدخول إلى Proxmox:

1. تأكد أن الشبكة تعمل.
2. تأكد من الوقت والتاريخ.
3. ارفع ISO خاص بـ Ubuntu Server.
4. أنشئ أول VM.
5. جرّب تشغيل VM والتأكد من اتصالها بالإنترنت.

---

## 12. إنشاء أول Ubuntu VM

من واجهة Proxmox:

1. اضغط على Datacenter.
2. اختر node الخاص بك.
3. اذهب إلى local storage.
4. Upload ISO.
5. ارفع Ubuntu Server ISO.
6. اضغط Create VM.
7. اختر ISO.
8. حدد CPU/RAM/Disk.
9. Start VM.
10. افتح Console.
11. ثبت Ubuntu داخل VM.

اقتراح إعدادات بسيطة:

```text
CPU: 2 cores
RAM: 2GB - 4GB
Disk: 20GB - 40GB
Network: Default bridge vmbr0
```

---

## 13. تشغيل Docker داخل Ubuntu VM

بعد تثبيت Ubuntu Server داخل VM، يمكنك تثبيت Docker داخلها مثل بيئة Linux عادية.

مثال مختصر:

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

ثم أضف Docker repository وثبت Docker حسب توثيق Docker الرسمي.

بعدها:

```bash
docker --version
docker ps
```

---

## 14. أين يأتي دور Containerlab؟

الترتيب المقترح:

```text
Proxmox Server
    └── Ubuntu VM
            └── Docker
                    └── Containerlab
                            └── Network Labs
```

يعني Containerlab لا يشتغل مباشرة بدل Proxmox.  
الأفضل تشغيله داخل Ubuntu VM على Proxmox.

---

## 15. الهدف من التشغيل 24/7

الفائدة الأساسية من Proxmox أن الجهاز يبقى شغالًا طوال الوقت، مثل سيرفر منزلي.

مثلاً:

- مشروع ويب للتجربة.
- Docker containers.
- Ubuntu Server.
- Network lab.
- Database.
- Monitoring tools.
- Graduation project prototype.

طالما الجهاز شغال والإنترنت شغال، الخدمات تبقى تعمل.

---

## 16. هل يمكن الدخول عليه من خارج البيت؟

نعم، لكن لا تفتح Proxmox مباشرة على الإنترنت.

الأفضل:

- VPN مثل Tailscale أو WireGuard.
- Cloudflare Tunnel للخدمات الويب فقط.
- عدم فتح Port 8006 للعالم مباشرة.

للتجربة داخل البيت، يكفي الدخول من نفس شبكة Wi-Fi أو LAN.

---

## 17. ملاحظات أمان مهمة

- لا تستخدم كلمة مرور ضعيفة.
- لا تفتح لوحة Proxmox مباشرة على الإنترنت.
- حدّث النظام باستمرار.
- لا تعطي حساب root لأي شخص.
- اعمل backup للـ VMs المهمة.
- استخدم UPS إذا كان السيرفر مهمًا.
- راقب الحرارة إذا الجهاز قديم.

---

## 18. الفرق بين WSL و Proxmox

| المقارنة | WSL + Ubuntu | Proxmox |
|---|---|---|
| يعمل داخل Windows | نعم | لا |
| يحتاج جهاز مستقل | لا | يفضل نعم |
| مناسب للتعلم السريع | نعم | نعم |
| مناسب 24/7 | ليس الأفضل | نعم |
| يشغل VMs و Containers من واجهة Web | لا | نعم |
| خطر حذف Windows أثناء التثبيت | لا | نعم إذا ثبتته على نفس القرص |
| مناسب للمشاريع المنزلية الطويلة | متوسط | ممتاز |

---

## 19. متى أستخدم WSL بدل Proxmox؟

استخدم WSL إذا:

- تريد تجربة Docker بسرعة.
- تعمل على لابتوبك.
- لا تريد حذف Windows.
- لا تحتاج سيرفر 24/7.
- تريد تطوير مشروع محلي.

---

## 20. متى أستخدم Proxmox بدل WSL؟

استخدم Proxmox إذا:

- عندك جهاز قديم مستقل.
- تريد تشغيل خدمات دائمًا.
- تريد إدارة VMs و Containers.
- تريد بيئة أقرب للسيرفرات.
- تريد مختبر منزلي لمشروع تخرج أو شبكات.

---

## 21. Checklist قبل تثبيت Proxmox

قبل أن تبدأ، تأكد من هذه النقاط:

- [ ] الجهاز ليس جهازك الأساسي.
- [ ] لا توجد ملفات مهمة على القرص المختار.
- [ ] عندك Backup لأي شيء مهم.
- [ ] Virtualization مفعلة من BIOS/UEFI.
- [ ] الجهاز متصل بالإنترنت عبر Ethernet إن أمكن.
- [ ] عندك USB عليه Proxmox ISO.
- [ ] تعرف كيف تدخل Boot Menu.
- [ ] فاهم أن Proxmox سيصبح نظام التشغيل الأساسي للجهاز.
- [ ] هدفك تشغيل السيرفر 24/7.

---

## 22. أخطاء شائعة

### لا أستطيع الدخول إلى Web Interface

تأكد أنك على نفس الشبكة، وتأكد من IP الظاهر على شاشة Proxmox.

### نسيت IP

ادخل من شاشة الجهاز واكتب:

```bash
ip a
```

### المتصفح يقول الاتصال غير آمن

هذا طبيعي بسبب شهادة SSL محلية. اضغط Advanced ثم Continue.

### لا أرى الإنترنت داخل VM

تأكد من إعدادات bridge ومن أن الجهاز الأساسي متصل بالشبكة.

### الجهاز يسخن

نظف الغبار، تأكد من المراوح، ولا تشغل VMs أكثر من قدرة الجهاز.

---

## 23. الخلاصة

Proxmox VE مناسب لتحويل جهاز قديم إلى سيرفر منزلي يعمل 24/7.  
لا يستخدم عادة داخل Windows مثل برنامج عادي، بل يثبت كنظام تشغيل كامل على جهاز مستقل أو قرص مخصص.

إذا كان هدفك تجربة سريعة على لابتوبك، استخدم WSL + Ubuntu + Docker.  
إذا كان هدفك سيرفر دائم، استخدم Proxmox على جهاز منفصل.

---

## Official References

- Proxmox VE Get Started: https://www.proxmox.com/en/products/proxmox-virtual-environment/get-started
- Proxmox VE Requirements: https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements
- Proxmox VE Documentation: https://pve.proxmox.com/pve-docs/
- Docker Engine on Ubuntu: https://docs.docker.com/engine/install/ubuntu/
