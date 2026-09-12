# worker-deplex — Cloudflare Pages proxy front

## ساختار پروژه
```
worker-deplex/
├── functions/
│   └── _middleware.js   # منطق پراکسی به سمت Deploxo (سرور اصلی)
├── public/
│   └── index.html       # فقط پرکننده - هیچ‌وقت واقعاً سرو نمی‌شه
└── wrangler.toml
```

`_middleware.js` روی **همه‌ی مسیرها** (`/`, `/ws`, `/xhttp`, ...) قبل از سرو فایل‌های
استاتیک اجرا می‌شه، پس محتوای `public/` عملاً هیچ‌وقت دیده نمی‌شه — فقط برای اینه که
Cloudflare Pages به یه output directory غیرخالی نیاز داره.

---

## روش ۱: Deploy از طریق داشبورد (بدون نیاز به CLI)

1. برو به **Cloudflare Dashboard → Workers & Pages → Create → Pages**
2. اگه پروژه رو از گیت‌هاب/گیت‌لب می‌سازی: ریپازیتوری رو وصل کن، **Build output directory**
   رو `public` بذار (فایل build command لازم نیست، خالی بذار).
3. اگه می‌خوای مستقیم آپلود کنی (Direct Upload): کل پوشه‌ی `worker-deplex/` رو
   (شامل `functions/` و `public/`) zip کن و آپلود کن.
4. بعد از deploy، آدرس پیش‌فرض چیزی مثل `worker-deplex.pages.dev` می‌شه.
5. برو `https://worker-deplex.pages.dev/` رو تست کن — باید `OK` ببینی.

## روش ۲: Deploy از طریق Wrangler CLI

```bash
npm install -g wrangler
wrangler login

cd worker-deplex
wrangler pages deploy public --project-name=worker-deplex
```

`functions/` به‌صورت خودکار توسط wrangler شناسایی و attach می‌شه، نیازی به دستور
جدا نیست.

---

## بعد از deploy موفق روی worker-deplex.pages.dev

مرحله‌ی بعدی وصل کردن دامنه‌ی آروان (`iu.hamkm.ir`) به این پروژه‌ست:

1. توی **Cloudflare Pages → پروژه‌ی worker-deplex → Custom domains**، می‌تونی یه
   دامنه‌ی سفارشی اضافه کنی — اما چون DNS اصلی دست آروانه نه Cloudflare، این روش
   مستقیم کار نمی‌کنه؛ باید از طریق **Origin/Backend روی پنل آروان** به آدرس
   `worker-deplex.pages.dev` اشاره کنی (Reverse Proxy تنظیمات آروان).
2. روی آروان: SSL mode باید `Full` یا `Full (Strict)` باشه، و WebSocket proxying
   فعال باشه.
3. کش رو روی مسیرهای `/ws` و `/xhttp` غیرفعال کن.

بعد از این، تست کن:
```bash
curl -v https://iu.hamkm.ir/
```
باید `OK` برگردونه. اگه اینجا هم جواب گرفتی، کانفیگ‌های vless (که قبلاً دادم) باید
وصل بشن.
