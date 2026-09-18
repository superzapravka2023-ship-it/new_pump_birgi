# Bybit → Bitget · LONG Continuation Bot

Следит за **Bybit** (там объёмы и импульс приходят раньше), ловит аномальный
рост, проверяет что монета есть на **Bitget** и там **ещё отстаёт**, и шлёт в
Telegram сигнал на **LONG (продолжение)**. Сделку открываешь на Bitget.

Публичные API обеих бирж — ключи бирж не нужны. Нужен только Telegram-бот и chat_id.

## Логика сигнала

Монета попадает в сигнал, только если **всё** совпало:
- есть и на Bybit (linear USDT-перп), и на Bitget (USDT-FUTURES);
- оборот 24ч на Bybit ≥ `MIN_TURNOVER_24H` (5M);
- монета старше `MIN_AGE_DAYS` (30д);
- рост на Bybit ≥ `PUMP_THRESHOLD_PCT`% (5%) за `PUMP_LOOKBACK_MIN` мин (5);
- **Bitget отстаёт** от Bybit минимум на `MIN_BITGET_GAP_PCT`% (1%) — это и есть «запас хода»;
- **балл качества ≥ `MIN_SCORE`** (2).

Балл качества (+1 за каждое): всплеск объёма (`×VOL_SURGE_MULT`), рост открытого
интереса на Bybit, пробой локального хая, комфортный запас на Bitget
(`≥GOOD_GAP_PCT`), здоровый RSI (60–88). Штраф −1 за истощение (большой верхний
фитиль + перегретый RSI). Антиспам: одна монета не чаще раза в `COOLDOWN_MIN` мин.

Сигнал приходит с кнопкой **«Открыть на Bitget»** — сразу в терминал.

## Деплой GitHub → Railway

1. Залей файлы в новый GitHub-репозиторий:
   ```bash
   git init && git add . && git commit -m "bybit->bitget long bot"
   git branch -M main
   git remote add origin https://github.com/USERNAME/REPO.git
   git push -u origin main
   ```
2. Токен у [@BotFather](https://t.me/BotFather) (`/newbot`), chat_id у
   [@userinfobot](https://t.me/userinfobot). Один раз напиши боту `/start`.
3. [railway.app](https://railway.app) → New Project → Deploy from GitHub repo →
   вкладка **Variables** → впиши переменные из `.env.example` (минимум
   `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`). Деплой стартует сам.

Это worker (без веб-порта), `restartPolicyType: ALWAYS` поднимает после падения.

## Команды
`/start` — настройки · `/status` — состояние · `/help` — справка

## Тюнинг под себя (переменные Railway)
- Меньше, но качественнее сигналов → `MIN_SCORE=3`, `MIN_BITGET_GAP_PCT=1.5`.
- Больше сигналов → `MIN_SCORE=1`, `PUMP_THRESHOLD_PCT=4`, `SCAN_INTERVAL=30`.
- Строже к «свежести» цены на Bitget → подними `MIN_BITGET_GAP_PCT`.

## Честно про ограничения
- Это **сигнальный** бот: продолжение импульса — вероятность, а не гарантия.
  Часть сигналов развернётся; когда памп откатывает, он бывает резким. Риск и
  объём — на твоей стороне (тейки/стопы в сигнал намеренно не пишутся).
- «Запас на Bitget» измеряется за то же окно, что и памп на Bybit — это
  оценка отставания, а не обещание, что Bitget его отработает.

## Автоторговля (по желанию, позже)
Добавить `pybit`-аналог для Bitget нельзя из коробки — Bitget торгуется через
свой подписанный API (`/api/v2/mix/order/place-order`, HMAC-SHA256 с
API_KEY/SECRET/PASSPHRASE). Если решишь автоматизировать вход — скажи, добавлю
модуль с расчётом объёма от риска на сделку и обязательным стопом.
