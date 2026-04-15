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
- ثبت لاگ امنیتی و Audit کامل
{
  "log": {},
  "inbounds": [
      {
        "sniffing": {
          "destOverride": [
            "tls",
            "http",
            "quic"
          ],
          "domainsExcluded": [
            "courier.push.apple.com"
          ],
          "enabled": true
        },
        "tag": "socks",
        "protocol": "socks",
        "listen": "127.0.0.1",
        "settings": {
          "udp": true,
          "auth": "noauth",
          "userLevel": 8
        },
        "port": 60028
      },
      {
        "tag": "directSocks",
        "settings": {
          "udp": true,
          "auth": "noauth",
          "userLevel": 8
        },
        "protocol": "socks",
        "listen": "127.0.0.1",
        "port": 1087
      },
      {
        "tag": "api",
        "settings": {
          "address": "[::1]"
        },
        "protocol": "dokodemo-door",
        "listen": "[::1]",
        "port": 60029
      }
    ],
  "outbounds": [
      {
        "settings": {
          "vnext": [
            {
              "address": "188.114.98.0",
              "users": [
                {
                  "email": "",
                  "flow": "",
                  "id": "57cea0ba-4cca-4612-a4f9-39be9d8a0c41",
                  "encryption": "none",
                  "level": 8
                }
              ],
              "port": 443
            }
          ]
        },
        "mux": {
          "concurrency": 50,
          "xudpProxyUDP443": "allow",
          "xudpConcurrency": 128,
          "enabled": false
        },
        "tag": "proxy",
        "protocol": "vless",
        "streamSettings": {
          "network": "ws",
          "tlsSettings": {
            "alpn": [],
            "fingerprint": "android",
            "serverName": "ad89767cfb9120f8.hozheen.com",
            "allowInsecure": false
          },
          "wsSettings": {
            "headers": {
              "Host": "madmax.hozheen.com"
            },
            "path": "/?ed=2048"
          },
          "security": "tls"
        }
      },
      {
        "tag": "fragment",
        "protocol": "freedom",
        "settings": {
          "fragment": {
            "packets": "tlshello",
            "length": "80-250",
            "interval": "10-100"
          },
          "userLevel": 8
        },
        "streamSettings": {
          "sockopt": {
            "tcpNoDelay": true
          }
        }
      }
    ],
  "api": {
      "services": [
        "StatsService"
      ],
      "tag": "api"
    },
  "dns": {
      "disableFallbackIfMatch": true,
      "tag": "dnsQuery",
      "queryStrategy": "UseIP",
      "hosts": {
        "dns.alidns.com": [
          "223.5.5.5",
          "223.6.6.6",
          "2400:3200::1",
          "2400:3200:baba::1"
        ],
        "one.one.one.one": [
          "1.1.1.1",
          "1.0.0.1",
          "2606:4700:4700::1111",
          "2606:4700:4700::1001"
        ],
        "cloudflare-dns.com": [
          "104.16.248.249",
          "104.16.249.249",
          "2606:4700::6810:f8f9",
          "2606:4700::6810:f9f9"
        ],
        "common.dot.dns.yandex.net": [
          "77.88.8.8",
          "77.88.8.1",
          "2a02:6b8::feed:0ff",
          "2a02:6b8:0:1::feed:0ff"
        ],
        "dot.pub": [
          "1.12.12.12",
          "120.53.53.53"
        ],
        "dns.quad9.net": [
          "9.9.9.9",
          "149.112.112.112",
          "2620:fe::fe",
          "2620:fe::9"
        ],
        "dns.google": [
          "8.8.8.8",
          "8.8.4.4",
          "2001:4860:4860::8888",
          "2001:4860:4860::8844"
        ],
        "dns.cloudflare.com": [
          "104.16.132.229",
          "104.16.133.229",
          "2606:4700::6810:84e5",
          "2606:4700::6810:85e5"
        ]
      },
      "disableFallback": true,
      "disableCache": true,
      "servers": [
        {
          "address": "8.8.8.8",
          "skipFallback": false
        }
      ]
    },
  "stats": {},
  "routing": {
        "domainStrategy": "AsIs",
        "rules": [
          {
            "inboundTag": [
              "api"
            ],
            "type": "field",
            "outboundTag": "api"
          },
          {
            "inboundTag": [
              "dnsQuery"
            ],
            "type": "field",
            "outboundTag": "proxy"
          },
          {
            "inboundTag": [
              "directSocks"
            ],
            "type": "field",
            "outboundTag": "direct"
          }
        ],
        "balancers": []
      },
  "policy": {
        "system": {
          "statsOutboundDownlink": true,
          "statsOutboundUplink": true,
          "statsInboundDownlink": true,
          "statsInboundUplink": true
        },
        "levels": {
          "8": {
            "bufferSize": 0,
            "handshake": 4,
            "connIdle": 30,
            "downlinkOnly": 1,
            "statsUserUplink": false,
            "uplinkOnly": 1,
            "statsUserDownlink": false
          }
        }
      },
  "transport": {}
}
