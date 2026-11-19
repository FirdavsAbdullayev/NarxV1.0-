# Narx v1.0 MVP — Telegram do'kon va avtomatik buyurtmalar tizimi

"Narx" — O'zbekistondagi kichik savdogarlar (Instagram va Telegram orqali savdo qiluvchilar) uchun mo'ljallangan SaaS platforma. Savdogar o'z mahsulotlarini boshqaruv paneliga kiritadi, noyob Telegram bot havolasini oladi (`https://t.me/BOT_USERNAME?start=shop_MERCHANT_ID`) va mijozlardan buyurtmalarni avtomatik yig'adi.

---

## Texnologiyalar

- **Frontend**: React 19, Vite, Tailwind CSS v4, React Router, Axios, Lucide Icons
- **Backend**: Node.js, Express.js, Telegraf (Telegram Bot API)
- **Ma'lumotlar bazasi**: MongoDB, Mongoose (agar `MONGODB_URI` ko'rsatilmasa yoki MongoDB ishlamayotgan bo'lsa, avtomatik ravishda mahalliy `.data/narx-db.json` fayl bazasiga o'tadi)
- **Autentifikatsiya**: JWT (`jsonwebtoken`), `bcryptjs`

---

## Loyiha tuzilishi

```text
├── server.ts                  # Express + Vite + Telegram bot asosiy kirish nuqtasi
├── server/
│   ├── config/env.ts          # Muhit o'zgaruvchilari
│   ├── db/store.ts            # MongoDB/Mongoose va mahalliy zaxira baza qatlami
│   ├── models/                # Mongoose modellari (Merchant, Product, Order, BotSession)
│   ├── middleware/auth.ts     # JWT tekshirish middleware'i
│   ├── routes/                # auth, products, orders API yo'llari
│   └── bot/                   # Telegraf bot va State Machine (engine.ts, telegram.ts)
└── src/
    ├── lib/api.ts             # Axios va JWT interceptor
    ├── context/AuthContext.tsx
    ├── components/            # Layout, AlertBanner, BotSimulatorDrawer
    └── pages/                 # AuthPage, DashboardPage, ProductsPage, OrdersPage, SettingsPage
```

---

## Ishga tushirish yo'riqnomasi

### 1. MongoDB o'rnatish (ixtiyoriy, ishlab chiqarish uchun tavsiya etiladi)
- Mahalliy kompyuterda [MongoDB Community Server](https://www.mongodb.com/try/download/community) o'rnating yoki [MongoDB Atlas](https://www.mongodb.com/atlas) dan bepul klaster oching.
- Eslatma: Agar `MONGODB_URI` bo'sh qoldirilsa, loyiha avtomatik mahalliy `.data/narx-db.json` fayl bazasida ham to'liq ishlayveradi.

### 2. `.env` faylini tayyorlash
`.env.example` faylidan nusxa olib `.env` faylini yarating:
```bash
cp .env.example .env
```

### 3. Telegram botlar va maxfiy kalitlar
1. **Telegram botlar**: `@BotFather` orqali token oling va `TELEGRAM_BOT_TOKEN` (savdogar), `CUSTOMER_BOT_TOKEN` (mijoz), `ADMIN_BOT_TOKEN` (admin) ga yozing.
2. **Majburiy maxfiy kalitlar (production)**: `JWT_SECRET` va `ADMIN_PASSWORD` o'rnatilishi shart. Standart (ochiq) qiymatlar bilan production serveri **ishga tushmaydi** — bu ataylab qilingan xavfsizlik chorasi.
   Tasodifiy kalit yaratish: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`
3. **Email xabarnomalari**: hozirda tizimga email yuborish ulanmagan. Barcha xabarnomalar **Xabarnomalar** bo'limida saqlanadi (ichki xabarnoma jurnali).

> Eslatma: Sahifalardagi "Email OTP" va SMTP sozlamalari hozircha faol emas. Bu funksiyalar kelajakda qo'shiladi.

### 4. Bog'liqliklarni o'rnatish
```bash
npm install
```

Production serverda ham `tsx` ishlatilgani sababli development dependency’larni o‘chirib tashlamang (`npm install --omit=dev` bilan o‘rnatmang).

### 5. Loyihani ishga tushirish (Backend + Frontend bitta portda: 3000)
```bash
npm run dev
```
Brauzerda oching: `http://localhost:3000`

### 6. Production build tayyorlash
```bash
npm run build
npm start
```

---

## Sinov ssenariysi (Tezkor tekshiruv)

1. Kirish sahifasida **Ro'yxatdan o'tish** orqali yangi do'kon oching (email, do'kon nomi, parol).
2. **Mahsulotlar** bo'limidan mahsulot qo'shing.
3. **Boshqaruv** sahifasidan **"Botni sinab ko'rish"** oynasini ochib buyurtma bering.
4. **Buyurtmalar** bo'limida buyurtma holatini o'zgartiring va **Xabarnomalar** bo'limida yozuvlarni tekshiring.

### API tutun testi (avtomatik)

Server ishlayotgan paytda boshqa terminalda:
```bash
npm run test:smoke                       # standart: http://localhost:3000
BASE_URL=https://sizning-domen.uz npm run test:smoke
```
Test ro'yxatdan o'tish, kirish, mahsulot CRUD, ochiq buyurtma, stok, promo-kod va ruxsat (403/401) tekshiruvlarini bajaradi va yaratgan ma'lumotlarini o'chiradi.

---

## Vercel ga deploy qilish

Loyiha Vercel uchun to'liq moslashtirilgan (`vercel.json` va `api/index.ts`):

1. Loyihani GitHub repozitoriyaga yuklang va Vercel'da import qiling (**Framework Preset: Vite** avtomatik aniqlanadi).
2. **Environment Variables** bo'limiga quyidagilarni kiriting:
   - `MONGODB_URI` — MongoDB Atlas ulanish manzili. **Muhim:** Vercel'da bo'sh bo'lsa ma'lumotlar `/tmp` da saqlanadi va vaqtinchalik bo'ladi.
   - `JWT_SECRET` — kuchli tasodifiy maxfiy kalit
   - `TELEGRAM_BOT_TOKEN` va `TELEGRAM_BOT_USERNAME` (va mijoz/admin botlar uchun `CUSTOMER_BOT_TOKEN`, `ADMIN_BOT_TOKEN`)
   - `JWT_SECRET` va `ADMIN_PASSWORD` — **majburiy**, aks holda production serveri ishga tushmaydi
3. Deploy tugagach, haqiqiy Telegram bot uchun webhook o'rnating:
   quyidagi buyruq bilan webhooklarni ulang (`WEBHOOK_SETUP_SECRET` o'rnatilgan bo'lishi kerak):
```bash
curl -X POST "https://<SIZNING-LOYIHA>.vercel.app/api/setup-webhooks?secret=<WEBHOOK_SETUP_SECRET>"
```
Webhooklar Telegram maxfiy kalitini (`secret_token`) bilan ro'yxatdan o'tadi. Hozircha bu amal faqat API orqali bajariladi; boshqaruv panelida tugma yo'q.

---

## Railway / Render platformalariga deploy qilish

1. Loyihani GitHub repozitoriyaga yuklang.
2. [Render.com](https://render.com) yoki [Railway.app](https://railway.app) da **New Web Service** yarating.
3. **Build Command**: `npm install && npm run build`
4. **Start Command**: `npm start` (yoki `NODE_ENV=production npm start`)
5. **Environment Variables** bo'limiga quyidagilarni kiriting:
   - `NODE_ENV=production`
   - `MONGODB_URI` (MongoDB Atlas ulanish manzili)
   - `JWT_SECRET` (kuchli tasodifiy satr)
   - `TELEGRAM_BOT_TOKEN`
   - `TELEGRAM_BOT_USERNAME`
   - `JWT_SECRET` va `ADMIN_PASSWORD` (majburiy)
   - `TRUST_PROXY_HOPS=1` (Render/Railway kabi reverse-proxy ortida; login va buyurtma chegaralagichi haqiqiy mijoz IP sini ishlatishi uchun)
6. Deploy'dan keyin webhooklarni qayta ulang: `curl -X POST "https://<domen>/api/setup-webhooks?secret=<WEBHOOK_SETUP_SECRET>"`. Eski webhooklar maxfiy kalitsiz ro'yxatdan o'tgan bo'lishi mumkin; ular yangilanmaguncha bot yangilanishlarini rad etadi.

---

## Xavfsizlik eslatmalari

- **Arxiv tozaligi:** `.data/` ichidagi lokal ma’lumotlar bazasi va `dist/` build fayllari repozitoriy/arxivga qo‘shilmasin. Build deploy vaqtida yangidan yaratiladi.
- **Production baza:** Vercel/Railway/Render production’da MongoDB Atlas (`MONGODB_URI`) ni sozlang. JSON fallback serverless/ephemeral diskda ma’lumotni doimiy saqlash kafolatini bermaydi; fallback faqat local/dev uchun.

- **Maxfiy kalitlarni almashtiring:** repozitoriyada yoki arxivda `.env` fayl bo'lsa, undagi Telegram tokenlar, MongoDB parolı, `JWT_SECRET` va `ADMIN_PASSWORD` ochiq hisoblanadi — ularni darhol yangilang.
- **Chegaralagichlar:** kirish, ro'yxatdan o'tish, ochiq buyurtma va promo-kod tekshiruvi IP bo'yicha cheklangan. Proksi ortida ishlaganda `TRUST_PROXY_HOPS` ni to'g'ri sozlang; aks holda chegaralagich noto'g'ri IP ni ko'radi.
- **Chegaralagich holati:** u jarayon xotirasida saqlanadi. Ko'p instansiyali (serverless) muhitda har bir instansiya o'z hisobini yuritadi.
- **To'lovlar:** tarif yangilash hozircha to'lov provayderiga ulanmagan (sinov). Haqiqiy to'lov uchun to'lov shlyuzini ulash va karta ma'lumotlarini server orqali o'tkazmaslik kerak.

