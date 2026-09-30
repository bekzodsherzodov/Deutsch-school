# Deutsch School — boshqaruv paneli

Sinflar, o'quvchilar, dars jadvali, haftalik natijalar, reyting, to'lovlar va davomat.
**Frontend** (`public/index.html`) + **backend** (`server/server.js`, Node 22, bog'liqliksiz, SQLite).

## Tezkor ishga tushirish (kompyuterda)
```bash
cp .env.example .env     # ADMIN_USER, ADMIN_PASSWORD, SESSION_SECRET ni to'ldiring
npm start                # http://localhost:3000
npm test                 # 16 ta avtomatik test
```
Node **22.5+** kerak. `npm install` shart emas (hech qanday paket ishlatilmaydi).

## Arxitektura
| Qatlam | Nima | Fayl |
|---|---|---|
| Frontend | Bitta sahifali ilova | `public/index.html` |
| Backend | `node:http` server, statik fayl + JSON API | `server/server.js` |
| Baza | SQLite (WAL). Holat `kv` jadvalida, akkauntlar alohida jadvalda | `data/school.db` |
| Auth | Cookie-sessiya (HttpOnly, SameSite=Strict, imzolangan), parollar scrypt | |
| API | `/api/auth/{login,logout,me,password}`, `/api/state` (GET/PUT), `/healthz` | |

## Rollar va xavfsizlik
- **Administrator** — hamma narsa. **O'qituvchi** — faqat o'z (sinf, fan) juftliklari bo'yicha haftalik natijalarni yozadi;
  to'lov, davomat va o'quvchi to'lov summalarini **ko'rmaydi** (server yubormaydi).
- Parol xeshlari brauzerga hech qachon qaytmaydi. Login urinishlari cheklangan (8 ta xato / 15 daqiqa).
- CSRF (Origin tekshiruvi), CSP, HSTS (production), `X-Frame-Options: DENY`.
- Administrator va o'qituvchi natijalari bir vaqtda yozilsa, **yangisi yutadi** (eski nusxa bosib ketmaydi).

## Deploy (Render.com — tavsiya)
1. Kodni GitHub'ga yuklang (pastda).
2. Render → **New → Blueprint** → repo'ni tanlang (`render.yaml` hamma narsani sozlaydi).
3. `ADMIN_USER` va `ADMIN_PASSWORD` ni Render panelida kiriting. `SESSION_SECRET` avtomatik yaratiladi.
4. Doimiy disk (`/data`) **pullik reja** talab qiladi. Bepul rejada har deployda ma'lumot o'chadi — ishlatmang.

Boshqa joy (VPS/Fly/Railway):
`docker build -t deutsch-school . && docker run -p 3000:3000 -v school-data:/data -e SESSION_SECRET=... -e ADMIN_USER=... -e ADMIN_PASSWORD=... deutsch-school`

**Faqat HTTPS** orqali ishlating (o'qituvchi parollari brauzerda `crypto.subtle` bilan xeshlanadi — u faqat HTTPS/localhost'da ishlaydi).

## GitHub'ga yuklash
```bash
cd deutsch-school
git remote add origin https://github.com/<login>/<repo>.git
git push -u origin main
```
`.env` va `data/` `.gitignore`da — parol GitHub'ga **chiqmaydi**. Push'dan so'ng GitHub Actions testlarni avtomatik yuritadi.

## Zaxira nusxa
`npm run backup` → `backups/` (oxirgi 14 ta saqlanadi). Render'da `/data` diskini kunlik snapshot qiling yoki cron orqali ishga tushiring.

## Ma'lum cheklovlar
- Butun holat bitta JSON sifatida saqlanadi (kichik maktab uchun yetarli; yuzlab o'quvchi). Katta hajmda jadvallarga ajratish kerak bo'ladi.
- Bitta server nusxasi (SQLite). Bir nechta nusxaga kengaytirish uchun PostgreSQL'ga o'tish kerak.
- Real brauzerda qo'lda to'liq sinovdan o'tkazilmagan: API va rollar avtomatik testlangan, interfeys o'zgartirilmagan (faqat sinxronlash va parol oqimi yamalgan).
