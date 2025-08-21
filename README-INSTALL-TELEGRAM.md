# Готовый пакет для Telegram WebApp + БД (Prisma + SQLite)

Это уже подготовленная версия твоего проекта из архива. Дальше — шаги очень мелко, под Windows.

## 0) Что внутри добавлено
- `server/index.js` — Express API, проверка Telegram WebApp, CRUD по карточкам
- `prisma/schema.prisma` — модель `Card`
- `.env.example` — образец переменных окружения
- `src/lib/api.ts` — мини-клиент для запросов
- `src/hooks/useCards.ts` — переделан с моков на API
- `index.html` — подключен Telegram SDK
- `src/main.tsx` — инициализация Telegram WebApp
- `Caddyfile.sample` — шаблон для публикации на домене `bskdnwgl.duckdns.org`
- `package.json` — добавлены скрипты и зависимости сервера/Prisma

## 1) Установка зависимостей
1. Установи Node.js LTS 20+: https://nodejs.org
2. Открой PowerShell в папке проекта.
3. Выполни:
   ```powershell
   npm i
   ```

## 2) Создай .env
1. Скопируй файл `.env.example` → `.env`.
2. Вставь свой токен бота в `BOT_TOKEN=`.

> Пока `SKIP_AUTH=1` — это режим разработки (без проверки подписи Telegram). В проде поменяешь на `0`.

## 3) Инициализируй БД
```powershell
npx prisma generate
npx prisma migrate dev --name init
```

## 4) Запусти API
```powershell
npm run server:dev
```
Ожидается: `API running on http://localhost:3000`

## 5) Собери фронт
```powershell
npm run build
```
Папка `dist/` появится в корне.

## 6) Публикация через Caddy и домен
1. Скачай Caddy (Windows exe): https://caddyserver.com/download
2. Скопируй `Caddyfile.sample` в `C:\caddy\Caddyfile` и **исправь путь** `root * C:\path\to\card-flow-sync-main\dist` на твой реальный путь.
3. Пробрось на роутере порты **80 и 443** на локальный IP ноутбука.
4. Запусти Caddy в админском PowerShell:
   ```powershell
   cd C:\caddy
   .\caddy.exe run --config C:\caddy\Caddyfile
   ```
5. Открой в браузере `https://bskdnwgl.duckdns.org` — должен открыться сайт.  
   Если не открывается извне, вероятно CGNAT у провайдера — тогда лучше VPS.

## 7) Кнопка в боте (пример на Node/telegraf)
```js
import { Telegraf } from "telegraf";
const bot = new Telegraf(process.env.BOT_TOKEN);
bot.start((ctx) => {
  ctx.reply("Открыть WebApp", {
    reply_markup: {
      keyboard: [[{ text: "Открыть приложение", web_app: { url: "https://bskdnwgl.duckdns.org" } }]],
      resize_keyboard: true
    }
  });
});
bot.launch();
```

## 8) Переключение на прод-режим защиты
Когда всё заработало по домену внутри Telegram — в `.env` поставь:
```
SKIP_AUTH=0
```
Перезапусти `npm run server:dev` (или `npm start`). Теперь API принимает запросы только из Telegram WebApp.

## 9) Частые проблемы
- **Пусто на странице карточек** — проверь, что API на `:3000` жив и `Caddyfile` проксирует `/api/*`.
- **401 Unauthorized** — в проде `SKIP_AUTH=0`, но открываешь через обычный браузер. Открывай только кнопкой в боте (вебвью).
- **Сертификат не выдался** — не проброшены 80/443 или CGNAT.
- **Папки/пути с пробелами** — кавычки в путях Windows обязательны.

Удачи! Если захочешь — помогу оформить автозапуск API/Caddy как сервис.
