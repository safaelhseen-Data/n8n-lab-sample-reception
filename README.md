# نظام أتمتة استقبال العينات في معمل نفط
# Oil Lab Sample Reception Automation (n8n + PostgreSQL)

> مشروع شخصي ببيانات تجريبية (Synthetic Data) لبناء نظام أتمتة متكامل: فورم ← قاعدة بيانات ← إشعار تلقائي ← تحليل.
> A personal project using synthetic data: form → database → automatic notification → analytics.

---

## نظرة عامة | Overview

عند وصول عينة جديدة للمعمل، يقوم موظف الاستقبال بتعبئة فورم واحد، وبعدها يتولى النظام تلقائياً:

1. توليد رقم عينة ورقم مرجعي جديدين (مثال: `SMP-002001` و`OCL-2026-002001`).
2. تسجيل العينة في 3 جداول مترابطة (العينات، المالية، النتائج) داخل عملية واحدة.
3. حساب ضريبة القيمة المضافة (15%) والمبلغ الإجمالي.
4. إرسال إشعار بريد إلكتروني بالعربية يحتوي تفاصيل العينة.

When a new sample arrives, the receptionist fills in a single form and the workflow automatically generates IDs, writes to three linked tables in one atomic query, calculates 15% VAT, and sends an Arabic email notification.

## مخطط سير العمل | Workflow

```
Sample Reception Form  →  Save Sample (PostgreSQL)  →  Send a message (Gmail)
```

| الخطوة | الوصف |
|---|---|
| **Sample Reception Form** | فورم n8n بقوائم منسدلة مأخوذة من البيانات الفعلية: 10 معامل، 48 نوع عينة، 8 مناطق، 3 أنواع عملاء، 5 طرق دفع |
| **Save Sample** | استعلام SQL واحد (CTEs) يولّد الأرقام ويكتب في `samples` و`finance` و`results` معاً، فلا تُسجَّل عينة ناقصة عند حدوث خطأ |
| **Send a message** | إشعار بريد إلكتروني بالعربية يحتوي رقم العينة والمعمل والعميل والمبلغ وتاريخ النتيجة المتوقع |

## قاعدة البيانات | Database

PostgreSQL مستضافة على [Neon](https://neon.tech). الجداول الثلاثة مرتبطة بالعمود `sample_id`:

| الجدول | المحتوى |
|---|---|
| `samples` | بيانات العينة: المعمل، النوع، العميل، المنطقة، الكمية، التواريخ، السعر |
| `finance` | الفاتورة: سعر الوحدة، الضريبة، الإجمالي، المدفوع، المتبقي، حالة الدفع |
| `results` | النتيجة: حالة التقرير، التأخير، حالة العينة، حالة التحصيل |

حجم البيانات التجريبية: 2000 عينة في كل جدول (2020 – 2026).

## المنطق المطبّق | Business Logic

- **الترقيم التلقائي:** آخر رقم عينة + 1، بصيغة `SMP-000000` والمرجع `OCL-YYYY-000000`.
- **الضريبة:** 15%، والإجمالي = المبلغ قبل الضريبة × 1.15.
- **القيم الابتدائية للعينة الجديدة:** حالة الدفع `Unpaid`، حالة التقرير `Pending`، المدفوع صفر.
- **تاريخ النتيجة المتوقع:** تاريخ الاستلام + عدد أيام التنفيذ المُدخل في الفورم.

## الأدوات | Tech Stack

n8n · PostgreSQL (Neon) · SQL · Gmail API · Power BI · DBeaver · GitHub

## ملفات المشروع | Repository Contents

| الملف | الوصف |
|---|---|
| `Lab Sample Reception.json` | ملف الـ workflow القابل للاستيراد في n8n |
| `README.md` | هذا الملف |

## طريقة التشغيل | How to Run

1. أنشئ قاعدة PostgreSQL (مثلاً على Neon) وأنشئ الجداول `samples` و`finance` و`results`.
2. استورد البيانات من ملفات CSV (يمكن استخدام DBeaver).
3. في n8n: Create workflow ← القائمة ⋯ ← **Import from file** ← اختر ملف الـ JSON.
4. اربط Credential الخاص بـ **Postgres** (Host/Database/User/Password مع تفعيل SSL).
5. اربط Credential الخاص بـ **Gmail** وغيّر خانة `sendTo` إلى بريدك.
6. اضغط **Publish** واستخدم رابط الفورم الدائم.

> ملاحظة: ملف الـ JSON لا يحتوي على كلمات مرور أو مفاتيح، ويجب إعادة ربط الـ Credentials بعد الاستيراد.

## لقطات الشاشة | Screenshots

<!-- أضف الصور هنا بعد رفعها للـ repo -->
- [ ] الفورم
- [ ] الـ workflow في n8n
- [ ] إشعار البريد الإلكتروني
- [ ] الجداول في قاعدة البيانات

## المشروع المرتبط | Related Project

التحليل ولوحات Power BI مبنية على نفس البيانات التجريبية، وموجودة في مشروع منفصل:

[oil-central-lab-powerbi](https://github.com/safaelhseen-Data/oil-central-lab-powerbi)

The Power BI analysis and dashboards built on the same synthetic dataset live in a separate repository (link above).

## ملاحظات | Notes

- جميع البيانات تجريبية ولا تخص أي جهة حقيقية.
- خطوات مستقبلية: فورم لتحديث نتيجة العينة وتسجيل الدفع، وربط Power BI بقاعدة البيانات مباشرة.
