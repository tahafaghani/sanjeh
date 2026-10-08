# فاز اول پروژه — اعتبارسنجی انطباق محصول با استاندارد

موتور **قطعی (deterministic)** برای مقایسهٔ ردیف‌های Excel محصولات با جدول استانداردها.  
هر پارامتر محصول با بازهٔ `Min`–`Max` همان دسته چک می‌شود و نتیجه به‌صورت **تأیید (APPROVED)** یا **رد (REJECTED)** همراه دلیل گزارش می‌شود.

```
Excel Loader  →  Schema Mapper  →  Rule Engine  →  Reporter
```

- قوانین از **فایل استاندارد** خوانده می‌شوند، نه از داخل کد  
- نام دسته و پارامتر **hardcode** نیست  
- از مدل زبانی / یادگیری ماشین برای قبول یا رد استفاده **نمی‌شود**

---

## قابلیت‌ها

| قابلیت | توضیح |
|--------|--------|
| اعتبارسنجی بازه‌ای | `Min ≤ مقدار ≤ Max` (مرزها شامل می‌شوند) |
| چند محصول / چند پارامتر | هر ردیف محصول در برابر استاندارد همان `Category` |
| گزارش Excel | شیت‌های Summary، Details و Overview |
| رابط وب | آپلود فایل، دیتاست نمونه، دانلود نتیجه |
| CLI | اجرا از خط فرمان بدون مرورگر |
| پیام دوزبانه | دلیل رد به انگلیسی و فارسی |

---

## ساختار پروژه

```
sanjeh/
├── validator/              # هستهٔ موتور پایتون
│   ├── types.py            # ساختارهای داده
│   ├── schema.py           # نگاشت ستون‌ها و شیت‌ها
│   ├── normalize.py        # نرمال‌سازی هدر و اعداد
│   ├── loader.py           # خواندن Excel
│   ├── parse.py            # تبدیل ردیف‌ها به مدل داخلی
│   ├── engine.py           # Rule Engine (بازه inclusive)
│   ├── messages.py         # پیام‌های خطا (EN / FA)
│   ├── reporter.py         # ساخت validation_results.xlsx
│   ├── run.py              # هماهنگی کل pipeline
│   ├── serialize.py        # خروجی JSON برای وب
│   ├── samples.py          # مسیر دیتاست‌های نمونه
│   └── __main__.py         # نقطه ورود CLI
├── webapp/                 # رابط وب
│   ├── main.py             # FastAPI
│   ├── static/             # app.js ، styles.css
│   └── templates/          # index.html
├── tests/                  # تست‌های pytest
├── samples/                # Products.xlsx و Standards.xlsx
├── requirements.txt
└── README.md
```

---

## پیش‌نیاز

- Python **3.10+**
- سیستم‌عامل: ویندوز / مک / لینوکس

---

## نصب

```bash
# کلون یا دانلود پروژه
cd sanjeh

# (پیشنهادی) محیط مجازی
python -m venv .venv

# فعال‌سازی
# ویندوز PowerShell:
.\.venv\Scripts\Activate.ps1
# مک / لینوکس:
source .venv/bin/activate

# وابستگی‌ها
pip install -r requirements.txt
```

---

## اجرا با خط فرمان (CLI)

```bash
# دیتاست نمونه Products
python -m validator --demo products --out validation_results.xlsx

# فایل‌های خودتان
python -m validator --products path/to/Products.xlsx --standards path/to/Standards.xlsx --out validation_results.xlsx

# خروجی JSON در ترمینال
python -m validator --demo products --json
```

---

## اجرا با رابط وب

```bash
# از ریشهٔ پروژه
# ویندوز PowerShell:
$env:PYTHONPATH = "."
uvicorn webapp.main:app --host 127.0.0.1 --port 8080

# مک / لینوکس:
export PYTHONPATH=.
uvicorn webapp.main:app --host 127.0.0.1 --port 8080
```

سپس در مرورگر:

```
http://127.0.0.1:8080
```

در وب می‌توانید:

1. دیتاست نمونه **Products** را اجرا کنید  
2. دو فایل Excel خودتان را آپلود کنید (نوار وضعیت سبز / قرمز)  
3. جدول تأیید و رد را ببینید  
4. فایل `validation_results.xlsx` را دانلود کنید  

> دسترسی فقط روی همان سیستمی است که سرویس را اجرا کرده. برای دسترسی دیگران از اینترنت باید روی هاست یا با تونل (مثل ngrok) مستقر شود.

---

## تست

```bash
python -m pytest tests -q
```

---

## قالب فایل‌های ورودی

### محصولات (`Products` sheet)

| ستون نمونه | نقش |
|------------|-----|
| ID | شناسه محصول |
| Product / Name | نام |
| Category | دسته (باید در استانداردها تعریف شده باشد) |
| Weight_g, Length_cm, … | پارامترهای عددی |

### استانداردها (`Standards` sheet — long format)

| ستون | نقش |
|------|-----|
| Category | دسته |
| Parameter | نام پارامتر (مثلاً `Weight_g`) |
| Min_Value | حداقل مجاز |
| Max_Value | حداکثر مجاز |
| Unit | واحد (اختیاری؛ مثلاً `g`, `cm`, `score`) |

قانون اصلی:

```text
Min_Value ≤ مقدار محصول ≤ Max_Value   →  PASS
در غیر این صورت                        →  FAIL
```

---

## خروجی Excel

فایل `validation_results.xlsx` معمولاً شامل:

1. **Summary** — وضعیت هر محصول، تعداد مغایرت، دلایل  
2. **Details** — نتیجهٔ هر پارامتر (PASS / FAIL)  
3. **Overview** — آمار کلی و پرتکرارترین مغایرت‌ها  

---

## توسعه‌پذیری

پارامتر جدید **بدون تغییر کد**:

1. ستون جدید در شیت محصولات  
2. ردیف(های) جدید در شیت استانداردها با همان نام پارامتر و بازهٔ Min/Max  

موتور ستون‌ها را از روی هدر پیدا می‌کند؛ نیازی به ویرایش `engine.py` نیست.

---

## وابستگی‌های اصلی

| بسته | کاربرد |
|------|--------|
| `fastapi` | API و سرو وب |
| `uvicorn` | سرور ASGI |
| `openpyxl` | خواندن و نوشتن Excel |
| `pytest` | تست |

همه متن‌باز و رایگان هستند. برای تصمیم تأیید/رد هیچ API پولی‌ای فراخوانی نمی‌شود.

---

<!-- ## نکات مهم

- تصمیم‌ها **قطعی** هستند؛ با ورودی یکسان همیشه خروجی یکسان است.  
- این پروژه جایگزین LLM نیست؛ یک Rule Engine ساده و قابل توضیح است.  
- برای استفاده سازمانی روی وب، بهتر است پشت reverse proxy و با HTTPS مستقر شود. -->

<!-- --- -->


<!-- ============ -->
<!-- # سنجه — اعتبارسنجی انطباق محصول با استاندارد

موتور قطعی (deterministic) برای مقایسهٔ ردیف‌های Excel محصولات با جدول استاندارد long-format.

```
Excel Loader  →  Schema Mapper  →  Rule Engine  →  Reporter
```

قوانین از فایل خوانده می‌شوند، نه از کد. نام دسته و پارامتر hardcode نشده است.

## ساختار پروژه

```
sanjeh/
├── validator/          # موتور پایتون (هسته اصلی)
│   ├── types.py
│   ├── schema.py
│   ├── normalize.py
│   ├── loader.py
│   ├── parse.py
│   ├── engine.py
│   ├── messages.py
│   ├── reporter.py
│   ├── run.py
│   ├── serialize.py
│   ├── samples.py
│   └── __main__.py
├── webapp/             # رابط وب FastAPI
│   ├── main.py
│   ├── static/
│   └── templates/
├── tests/              # تست‌ها
├── samples/            # فایل‌های نمونه Excel
├── requirements.txt
└── README.md
```

## اجرا

```bash
# نصب وابستگی‌ها
pip install -r requirements.txt

# CLI
python -m validator --demo document --out validation_results.xlsx
python -m validator --products samples/products.xlsx --standards samples/standards.xlsx

# تست
python -m pytest tests -q

# وب
PYTHONPATH=. uvicorn webapp.main:app --host 0.0.0.0 --port 8080
```

## توسعه‌پذیری

پارامتر جدید = ستون جدید در محصولات + ردیف جدید در استانداردها. بدون تغییر کد. -->
