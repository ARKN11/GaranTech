# نمودار موجودیت‌ـ‌رابطه (ERD) — GaranTech

<div dir="rtl">

**مبنای نمودار:** `docs/design/data-model.md`  
**تعداد موجودیت‌ها:** 13  
**نوع نمودار:** Logical ERD

این نمودار روابط، کلیدهای اصلی، کلیدهای خارجی و کاردینالیتی‌های مدل منطقی GaranTech را نمایش می‌دهد. نام موجودیت‌ها و ویژگی‌ها انگلیسی نگه داشته شده‌اند تا با نام‌گذاری فنی و پیاده‌سازی آینده سازگار باشند.

</div>

## 1. نمودار کامل

```mermaid
erDiagram
    CUSTOMER ||--o{ REPAIR_REQUEST : submits
    DEVICE ||--o{ REPAIR_REQUEST : subject_of

    REPAIR_REQUEST ||--o| DEVICE_RECEIPT : has
    STAFF ||--o{ DEVICE_RECEIPT : receives

    REPAIR_REQUEST ||--o| INSPECTION : has
    STAFF ||--o{ INSPECTION : performs
    INSPECTION ||--|{ INSPECTION_DEFECT : identifies

    INSPECTION ||--o| REPAIR_PROPOSAL : may_produce
    REPAIR_PROPOSAL ||--o| CUSTOMER_DECISION : receives

    REPAIR_REQUEST ||--o| REPAIR_JOB : has
    STAFF ||--o{ REPAIR_JOB : assigned_to
    REPAIR_JOB ||--|{ REPAIR_JOB_OPERATION : contains

    REPAIR_JOB ||--o{ REPAIR_TEST : tested_by
    STAFF ||--o{ REPAIR_TEST : records

    REPAIR_REQUEST ||--|{ STATUS_HISTORY : records
    STAFF o|--o{ STATUS_HISTORY : may_initiate

    CUSTOMER {
        string customer_id PK
        string full_name
        string phone_number UK
        datetime created_at
    }

    DEVICE {
        string device_id PK
        string category
        string brand "optional"
        string model
        string serial_number
        datetime created_at
    }

    REPAIR_REQUEST {
        string request_id PK
        string tracking_code UK
        string customer_id FK
        string device_id FK
        string initial_description
        string current_status
        datetime submitted_at
        datetime updated_at
        datetime cancelled_at
        string cancellation_reason
    }

    STAFF {
        string staff_id PK
        string full_name
        string phone
        string email UK
        string username UK
        string password_hash
        string role
        boolean is_active
        datetime created_at
    }

    DEVICE_RECEIPT {
        string receipt_id PK
        string request_id FK, UK
        string received_by_staff_id FK
        datetime received_at
        string notes
    }

    INSPECTION {
        string inspection_id PK
        string request_id FK, UK
        string service_expert_id FK
        string warranty_result
        string warranty_rejection_reason
        string recommended_action
        string inspection_notes
        datetime inspected_at
    }

    INSPECTION_DEFECT {
        string defect_id PK
        string inspection_id FK
        string title
        string description
    }

    REPAIR_PROPOSAL {
        string proposal_id PK
        string inspection_id FK, UK
        decimal estimated_total_cost
        string notes
        datetime created_at
    }

    CUSTOMER_DECISION {
        string decision_id PK
        string proposal_id FK, UK
        string decision
        datetime decided_at
        string customer_note
    }

    REPAIR_JOB {
        string repair_job_id PK
        string request_id FK, UK
        string technician_id FK
        datetime started_at
        datetime completed_at
    }

    REPAIR_JOB_OPERATION {
        string job_operation_id PK
        string repair_job_id FK
        string operation_type
        string description
        string part_description
        string operation_status
        datetime completed_at
    }

    REPAIR_TEST {
        string repair_test_id PK
        string repair_job_id FK
        string tested_by_staff_id FK
        string result
        string test_notes "optional"
        datetime tested_at
    }

    STATUS_HISTORY {
        string history_id PK
        string request_id FK
        string actor_type
        string changed_by_staff_id FK
        string previous_status
        string new_status
        datetime changed_at
        string note
    }
```

## 2. توضیحات اختیاری بودن ویژگی‌ها

در موجودیت DEVICE، ویژگی `brand` اختیاری است؛ ویژگی‌های `category` و `model` الزامی‌اند و `serial_number` نیز در صورت نبود شماره سریال می‌تواند خالی باشد. در REPAIR_TEST، ویژگی `test_notes` اختیاری است؛ توضیح برای نتیجه FAILED در لایه برنامه الزامی و برای PASSED اختیاری است.

## 3. راهنمای کاردینالیتی

| نماد Mermaid | معنا |
|---|---|
| `||` | دقیقاً یک |
| `o|` | صفر یا یک |
| `o{` | صفر یا چند |
| `|{` | یک یا چند |

برای نمونه، رابطه Customer و RepairRequest نشان می‌دهد هر درخواست دقیقاً متعلق به یک مشتری است، اما یک مشتری می‌تواند هیچ درخواست یا چند درخواست داشته باشد.

## 3. نکات مهم در خواندن نمودار

1. **Inspection و InspectionDefect:** ایرادهای تشخیص‌داده‌شده به کارشناسی متصل‌اند، نه مستقیماً به درخواست؛ شرح اولیه مشتری در RepairRequest باقی می‌ماند.
2. **Inspection و RepairProposal:** پیشنهاد هزینه اختیاری است؛ فقط برای درخواست خارج از گارانتی ایجاد می‌شود. درخواست تحت گارانتی مستقیماً آماده تعمیر می‌شود.
3. **RepairProposal و CustomerDecision:** تصمیم مشتری فقط برای پیشنهاد هزینه خارج از گارانتی ثبت می‌شود.
4. **RepairJob و RepairJobOperation:** عملیات واقعی مستقل از پیشنهاد هزینه است؛ بنابراین تعمیر تحت گارانتی نیز بدون ProposalItem قابل ثبت است.
5. **RepairJob و RepairTest:** هر فرایند تعمیر می‌تواند چند نتیجه آزمایش داشته باشد. آزمایش ناموفق درخواست را در IN_REPAIR نگه می‌دارد.
6. **StatusHistory:** هر درخواست یک رکورد اولیه وضعیت دارد. changed_by_staff_id برای رویدادهای با actor_type برابر STAFF مقدار دارد و برای CUSTOMER یا SYSTEM خالی است.

## 4. وضعیت‌های درخواست

مقادیر فنی مورد استفاده در RepairRequest.current_status و StatusHistory:

| مقدار | عنوان فارسی |
|---|---|
| REGISTERED | ثبت‌شده |
| RECEIVED | دریافت‌شده |
| INSPECTED | بررسی‌شده |
| AWAITING_CUSTOMER_DECISION | در انتظار تصمیم مشتری |
| READY_FOR_REPAIR | آماده تعمیر |
| IN_REPAIR | در حال تعمیر |
| COMPLETED | تکمیل‌شده |
| CANCELLED | لغوشده |

مقدار APPROVED در نسخه اولیه نمودار استفاده نمی‌شود؛ وضعیت درست طبق SRS و UC-12 برابر READY_FOR_REPAIR است.

## 5. قواعدی که با خط رابطه به‌تنهایی نمایش داده نمی‌شوند

ERD ساختار و کاردینالیتی را نمایش می‌دهد؛ اما برخی قواعد کسب‌وکار باید در مدل داده و منطق برنامه نیز enforce شوند:

- RepairProposal فقط وقتی مجاز است که Inspection.warranty_result برابر NOT_ELIGIBLE باشد.
- برای ELIGIBLE، RepairProposal و CustomerDecision نباید ایجاد شوند.
- برای NOT_ELIGIBLE، پیش از شروع RepairJob باید پیشنهاد هزینه و تصمیم ACCEPTED وجود داشته باشد.
- فقط کارشناس خدمات می‌تواند DeviceReceipt و Inspection را ثبت کند.
- فقط تعمیرکار می‌تواند RepairJob را اجرا و RepairTest را ثبت کند.
- RepairJob فقط پس از READY_FOR_REPAIR آغاز می‌شود.
- FAILED وضعیت را تغییر نمی‌دهد؛ PASSED باعث COMPLETED شدن درخواست توسط سامانه می‌شود.
- CANCELLED و COMPLETED وضعیت‌های نهایی‌اند.
- current_status باید با آخرین رکورد StatusHistory سازگار باشد.

</div>
