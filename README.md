# سامانه گردش پرونده هیئت (نسخه اولیه)

این پروژه یک نسخه اولیه (MVP) برای ثبت و رهگیری جابه‌جایی پرونده‌ها در بخش **هیئت** اداره ثبت اسناد و املاک است.

هدف اصلی: ثبت سریع عملیات توسط کاربر با حداقل کلیک:
- وارد کردن کد پرونده (سال + شماره)
- انتخاب حالت `دریافت` یا `ارسال`
- در حالت ارسال: انتخاب مقصد
- ثبت عملیات

## امکانات نسخه اولیه

- ثبت گردش پرونده با API بک‌اند
- ایجاد خودکار پرونده در اولین ثبت (شروع از هیئت)
- نگهداری تاریخچه کامل گردش برای هر پرونده
- نمایش لیست گردش‌های اخیر
- جستجوی تاریخچه بر اساس شماره پرونده
- پشتیبانی از کد پرونده دو بخشی:
  - `caseYear`: سال 4 رقمی
  - `caseSerial`: شماره 1 تا 4 رقمی (با صفرگذاری خودکار)
- تعریف بخش‌های جدید با رنگ اختصاصی:
  - کارشناسی (`KARSHENASI`)
  - نقشه برداری (`NAGHSHE_BARDARI`)
  - هیئت (`HEIAT`)
  - جمشیدی (`JAMSHIDI`)
  - ریاست (`RIASAT`)
  - حسابداری (`HESABDARI`)
  - ارباب رجوع (`ARBAB_ROJOO`)
  - ابلاغ رای (`EBLAGH_RAY`)
- بسته‌شدن پرونده با انتخاب مقصد `ابلاغ رای` و جلوگیری از عملیات بعدی
- گزینه `بدون بخش` برای ثبت عملیات بدون مبدا/مقصد

## پشته فناوری

- Frontend: `Next.js + React + TypeScript`
- Backend: `ElysiaJS (Bun runtime)`
- Database: `PostgreSQL`
- ORM: `Prisma`

## ساختار پروژه

```text
Sabt asnad/
├── front/                 # Next.js UI
│   ├── app/
│   ├── .env.example
│   └── README.md
├── back/                  # Elysia API + Prisma
│   ├── prisma/
│   │   └── schema.prisma
│   ├── src/
│   │   └── index.ts
│   ├── .env.example
│   └── README.md
└── README.md
```

## پیش‌نیازها

- Node.js 20+
- Bun 1.1+
- PostgreSQL 14+

## راه‌اندازی محلی

### 1) Backend

```bash
cd back
bun install
cp .env.example .env
bun run prisma:generate
bun run prisma:push
bun run dev
```

Backend روی `http://localhost:4000` اجرا می‌شود.

### 2) Frontend

```bash
cd front
npm install
cp .env.example .env.local
npm run dev
```

Frontend روی `http://localhost:3000` اجرا می‌شود.

## تنظیمات محیطی

### `back/.env`

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/sabt_asnad"
PORT=4000
```

### `front/.env.local`

```env
NEXT_PUBLIC_API_BASE_URL="/api"
API_PROXY_TARGET="http://localhost:4000"
```

## استقرار روی شبکه داخلی (بدون اینترنت)

سناریوی پیشنهادی:
- یک سیستم مرکزی با IP ثابت (مثلاً `192.168.1.50`)
- دیتابیس + بک‌اند + فرانت روی همین سیستم اجرا شوند
- کلاینت‌ها از طریق مرورگر به IP ثابت وصل شوند

پیکربندی نمونه:
- `back/.env`:
  - `PORT=4000`
  - `DATABASE_URL=...`
- `front/.env.local`:
  - `NEXT_PUBLIC_API_BASE_URL="/api"`
  - `API_PROXY_TARGET="http://192.168.1.50:4000"`

اجرای Production پیشنهادی:

```bash
# backend
cd back
bun run prisma:generate
bun run prisma:push
bun run start

# frontend
cd front
npm run build
npm run start -- -H 0.0.0.0 -p 3000
```

## مدل داده

### Department
- `id`
- `code` (unique)
- `name`
- `colorHex`

### CaseFile
- `id`
- `caseNumber` (unique)
- `currentDepartmentId` (nullable)
- `isClosed`
- `closedAt` (nullable)
- `createdAt`
- `updatedAt`

### Movement
- `id`
- `type` (`RECEIVE` | `SEND`)
- `caseFileId`
- `fromDepartmentId` (nullable)
- `toDepartmentId` (nullable)
- `actorName` (nullable)
- `note` (nullable)
- `createdAt`

## APIهای اصلی

Base URL: `http://localhost:4000`

### Health Check
- `GET /health`

### لیست بخش‌ها
- `GET /departments`

### ثبت گردش پرونده
- `POST /movements`

نمونه Body برای ارسال:

```json
{
  "caseYear": "1404",
  "caseSerial": "12",
  "type": "SEND",
  "destinationDepartmentId": "cmf... (اختیاری)",
  "actorName": "کارمند الف",
  "note": "ارسال جهت بررسی"
}
```

نمونه Body برای دریافت:

```json
{
  "caseYear": "1404",
  "caseSerial": "12",
  "type": "RECEIVE",
  "actorName": "کارمند ب"
}
```

### گردش‌های اخیر
- `GET /movements/recent?limit=15`

### تاریخچه پرونده
- `GET /cases/:caseNumber/history`

## گردش کار کاربر در UI

- ورود سال + شماره پرونده
- انتخاب نوع عملیات:
  - `دریافت`
  - `ارسال`
- در حالت ارسال: انتخاب مقصد یا `بدون بخش`
- در حالت دریافت: پرونده وارد هیئت می‌شود و مبدا از وضعیت قبلی محاسبه می‌شود
- ثبت
- مشاهده نتیجه و تاریخچه

## نکات نسخه اولیه

- احراز هویت کاربران هنوز اضافه نشده است.
- گزارش‌گیری پیشرفته، چاپ، و سطح دسترسی نقش‌ها در فاز بعدی پیشنهاد می‌شود.
- برای استفاده اداری، حتماً بکاپ روزانه PostgreSQL تنظیم شود.

## گام‌های پیشنهادی فاز بعد

- افزودن ورود کاربران و نقش‌ها (اپراتور، مدیر)
- گزارش روزانه/هفتگی بر اساس بخش و کاربر
- قفل/اعتبارسنجی بیشتر روی شماره پرونده
- داشبورد مدیریتی ساده
- ثبت لاگ امنیتی و Audit کا

  
```
@echo off
setlocal
cd /d %~dp0

if not exist data mkdir data
if not exist ".env" copy ".env.example" ".env" >nul

echo Starting SabtAsnad...
SabtAsnad.exe
echo.
echo ExitCode: %errorlevel%
pause
```
--------------------------

```
SabtAsnad.exe > run.log 2>&1
echo %errorlevel%
type run.log
```
