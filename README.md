# Fikr — mijoz fikri va filial sifati

Next.js 16 + React + JavaScript + MongoDB. Backend: `app/api/[...path]/route.js`. O‘zbekcha interfeys, uchta panel, filial QR-kodlari va statistika.

## Mavjud loyihani yangilash — 2026-10-23

1. Oldingi `npm start` va qo‘lda ochilgan cloudflared oynalarini `Ctrl+C` bilan to‘xtating.
2. Eski loyiha papkangizni zaxiralang. ZIP ichidagi `fikr-mvp` fayllarini mavjud `package.json` turgan papkaga almashtirib ko‘chiring.
3. O‘zingizdagi `.env.local` faylini saqlang. ZIP ichida shaxsiy sozlamalar va ma’lumotlar bazasi yo‘q. MongoDB bazasi va filial IDlarini o‘chirmang.
4. Shu papkada:

```powershell
npm install
npm start
```

Bundan keyin har safar **faqat `npm start`** yetadi. `npm run build`, cloudflared buyrug‘i yoki APP_URL tahriri alohida kerak emas.

Birinchi ishga tushirishda build qilinadi. Cloudflared mavjud bo‘lmasa rasmiy Cloudflare GitHub relizidan yuklanadi, SHA256 tekshiriladi va `.tools` papkasida saqlanadi. Internetdan yuklash ishlamasa, oldingi `cloudflared.exe` faylingizni `package.json` yoniga nusxalab `npm start` ni takrorlang. Kod o‘zgarmagan keyingi ishga tushirishlarda build qayta bajarilmaydi.

Terminalda `[Fikr] TAYYOR` chiqishini kuting. Kompyuterda http://localhost:3000 → biznes egasi paneli → Filiallar va QR → QRni oching va telefonda skaner qiling. Telefon Wi-Fi yoki mobil internetdan foydalanishi mumkin. Bir xil Wi-Fi shart emas.

**Kompyuter, MongoDB va shu terminal ishlayotgan bo‘lishi kerak.** VS Code yoki terminalni yopish saytni to‘xtatadi. Qayta kirganda `npm start` bosing. Internet aloqasi va Cloudflare xizmatining mavjudligi talab qilinadi.

### Eski saqlangan QR haqida

Quick Tunnel har ishga tushirishda yangi domen beradi. Paneldagi QR amaldagi manzilga avtomatik yangilanadi; ochiq QR oynasi manzilni har 10 soniyada va oynaga qaytganda tekshiradi. Eski rasm yoki qog‘ozdagi QR ichidagi manzilni dastur o‘zgartira olmaydi. Yangidan ochilgan paneldagi QRni skanerlang yoki yangi PNGni saqlang.

Chop etilgan QR bir necha kun yoki oy davomida o‘zgarmasligi kerak bo‘lsa, doimiy hosting/domen yoki Cloudflare named tunnel kerak. Bu ZIP doimiy domen bermaydi. Quick Tunnel bepul sinov uchun; uzluksiz ishlash kafolati yo‘q.

## Yangi o‘rnatish

Node.js 22.12+ yoki 24 va ishlaydigan MongoDB server kerak. Compass — bazani boshqarish dasturi; MongoDB serverning o‘zi ham ishlashi kerak.

```powershell
npm install
Copy-Item .env.example .env.local
notepad .env.local
```

`Copy-Item` ni faqat .env.local hali yo‘q bo‘lsa bajaring. Mavjud konfiguratsiyani almashtirmang.

Asosiy sozlamalar:

```dotenv
MONGODB_URI=mongodb://127.0.0.1:27017
MONGODB_DB=fikr
APP_URL=http://localhost:3000
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
COOKIE_SECURE=false
TRUST_PROXY=false
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=OZINGIZ_TANLAGAN_KUCHLI_PAROL
ADMIN_NAME=Administrator
```

Admin uchun o‘zingiz tanlagan kamida 12 belgili parolni qo‘ying, keyin:

```powershell
npm run admin
npm start
```

Mavjud adminni qayta yaratish kerak emas. Parol frontendda bo‘lmaydi. Biznes egasi /register orqali ro‘yxatdan o‘tadi; admin va biznes egasi /login orqali kiradi.

`npm start` `.env.local` faylini o‘zgartirmaydi. Joriy tunnel URLni faqat ishga tushirilgan server muhitiga beradi; eskirgan APP_URL avtomatik almashtiriladi. HTTP localhost va HTTPS internet manzili uchun tegishli cookie va aniq origin ruxsatlari qo‘llanadi. Brauzerlar/domenlarda login sessiyalari alohida.

Ixtiyoriy sozlamalar:

```dotenv
# O‘zingizdagi cloudflared joylashuvi; Windowsda / ishlating:
# CLOUDFLARED_PATH=C:/tools/cloudflared.exe
# PORT=3000
```

Cloudflared qidirish tartibi: CLOUDFLARED_PATH → loyiha papkasi → .tools → PATH → rasmiy yuklash. .tools, .fikr va .env.local ni Gitga yubormang.

## Uchta panel

| Rol | Manzil | Imkoniyatlar |
| --- | --- | --- |
| Administrator | /admin | Bizneslar, filial va murojaat sonlari, biznesni to‘xtatish/faollashtirish |
| Biznes egasi | /dashboard | Filiallar, QR, fikrlar, javoblar, holat boshqaruvi, statistika |
| Mijoz | /f/filial-id | 1–5 yulduz, shikoyat, maqtov, taklif, mavzu, voqea vaqti, ixtiyoriy aloqa |
| Mijoz kuzatuvi | /mijoz, /track/maxsus-kod | Murojaat holati, biznes javobi va tarix |

Har filial UUID orqali alohida identifikator oladi, MongoDB unique index takrorlanishdan himoya qiladi. Filial tahririda ID saqlanadi. QR PNG 1000×1000 px; server haqiqiy URLni kodlaydi.

Mijozga akkaunt shart emas. Murojaat MongoDB’da saqlanadi. Maxsus kuzatish havolasini bilgan odam murojaatni ko‘ra oladi; havolani maxfiy saqlang. Aloqa ma’lumoti ommaviy kuzatish javobida berilmaydi.

## Statistika

Statistika MongoDB’dagi barcha tegishli yozuvlardan hisoblanadi, oxirgi 500 ta bilan cheklanmaydi. Murojaatlar ro‘yxati esa oxirgi 500 tani ko‘rsatadi.

- Barcha vaqt yoki oxirgi 7/30/90 kun: yuborilgan vaqt bo‘yicha.
- Filiallar o‘rtacha bahosi, fikrlar soni, shikoyat/maqtov/taklif sonlari.
- Ijobiy foiz = 4–5 yulduzli fikrlar / barcha fikrlar.
- Hal qilish foizi = hal etilgan shikoyatlar / barcha shikoyatlar.
- Ochiq shikoyatlar, baholar taqsimoti, oylar va mavzular tahlili.
- 5 tadan kam fikr — dastlabki natija. Eng yuqori/past kartalari kamida 5 fikrli filiallarni taqqoslaydi; bu statistik ishonchlilik kafolati emas.
- Nol fikrda baho va foizlar “—”. Natijalar fikr bildirgan mijozlarga tegishli, barcha tashriflarni ifodalamaydi.
- Eski “Minnatdorchilik” yozuvlari maqtovga kiradi. Migratsiya yoki bazani tozalash talab qilinmaydi.

## Ishga tushirishdagi xatolar

- **3000-port band:** oldingi sayt terminalida Ctrl+C bosing. Dastur boshqa jarayonlarni o‘zi o‘chirmaydi.
- **Cloudflared yuklanmadi:** internetni tekshiring yoki mavjud cloudflared.exe ni package.json yoniga nusxalang. Token/Cloudflare akkaunti quick rejimga kerak emas.
- **Tunnel ulanmadi/to‘xtadi:** terminaldagi sababni tekshiring. Internetni tiklab npm start ni qayta bosing. Joriy QRni paneldan oching.
- **MongoDB aloqa xatosi:** MongoDB xizmatini ishga tushiring, MONGODB_URI va MONGODB_DB ni tekshiring. MONGODB_DB oddiy baza nomi (fikr), URL emas.
- **Telefonda eski 1033/NXDOMAIN:** saqlangan eski QRni emas, ayni ishga tushirishdagi panel QRni tekshiring.
- **Yuklanmoqda:** QR manzilini telefon brauzerida oching; terminaldagi xatoni tekshiring. Sayt API so‘rovlari 20 soniyada tugatiladi, cheksiz kutish o‘rniga xato ko‘rsatiladi.

MongoDB holatini Windows PowerShell’da tekshirish:

```powershell
Get-Service *Mongo*
Test-NetConnection 127.0.0.1 -Port 27017
```

## Doimiy hosting yoki faqat lokal server

`npm start` kompyuterda avtomatik quick tunnel uchun. O‘z doimiy domeningiz/hostingingiz yoki alohida boshqarilgan tunnel bo‘lsa:

```powershell
npm run build
npm run start:server
```

Bu rejimda APP_URLni haqiqiy doimiy HTTPS manzilga sozlang va COOKIE_SECURE=true qo‘ying. Server va MongoDBni hostingda boshqarish kerak. TRUST_PROXY=true faqat ishonchli hosting proksisi X-Forwarded-For ni o‘zi qayta yozsa ishlatiladi. `.env.local` o‘rniga hosting environment variables ham qo‘llanadi.

Development: `npm run dev` faqat dasturlash uchun. Telefon orqali namoyishda `npm start` ishlating.

## Tekshiruv

```powershell
npm run build
npm test
```

API integratsiya sinovlari vaqtinchalik MongoDB va production Next serverda bajariladi. MongoDB test binary birinchi safar yuklanishi mumkin. Launcher sinovlari lokal yordamchi jarayonlar bilan URL almashishi, eski env saqlanishi, port bandligi, to‘xtash va SHA256 xatolarini tekshiradi. Haqiqiy Cloudflare/Windows/telefon sinovi alohida talab qilinadi.

## MVP chegaralari

To‘lov, Telegram, media yuklash, email tasdiqlash va parolni tiklash bu versiyada yo‘q. Parollar scrypt bilan hash qilinadi; sessiyalar HttpOnly cookie, ma’lumot ajratilishi va origin tekshiruvi serverda bajariladi. Login va murojaatlar uchun rate limit bor. MongoDB ma’lumotlarini muntazam zaxiralang.
#   f i k r  
 