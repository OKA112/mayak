# 🌟 Маяк — дневник привычек

Персональный трекер привычек, статистики и дневник. Работает как приложение (PWA):
ставится на телефон и ноутбук, работает офлайн, данные хранятся локально, а по желанию
синхронизируются между устройствами через бесплатный Supabase.

- **Сегодня** — счётчики «дней без вредного», чек-лист дня, ритуал дня, заметка с настроением
- **Привычки** — таблица «привычка × день», добавление/настройка привычек
- **Зарядка** — 5 коротких комплексов без инвентаря, свой на каждый день
- **Чтение** — база из 38 бесплатных источников по 6 темам + заметки к каждой книге (👍/👎, мысли, где остановился)
- **Статистика** — серии, %, графики, сэкономленные деньги
- **Дневник** — записи по датам с настроением

## Установка на телефон
1. Открой адрес сайта в браузере телефона.
2. **iPhone (Safari):** Поделиться → «На экран „Домой“».
   **Android (Chrome):** меню ⋮ → «Установить приложение».

## Синхронизация между устройствами (по желанию)

Разовая настройка (~5 минут):

1. Зайди на [supabase.com](https://supabase.com), войди через GitHub, создай **New project**.
2. В **SQL Editor** выполни:

   ```sql
   create table if not exists public.mayak_state (
     user_id uuid primary key references auth.users(id) on delete cascade,
     data jsonb not null default '{}'::jsonb,
     updated_at timestamptz not null default now()
   );
   alter table public.mayak_state enable row level security;
   create policy "own_select" on public.mayak_state for select using (auth.uid() = user_id);
   create policy "own_insert" on public.mayak_state for insert with check (auth.uid() = user_id);
   create policy "own_update" on public.mayak_state for update using (auth.uid() = user_id);
   ```

3. **Authentication → URL Configuration:** в **Site URL** укажи адрес сайта,
   а в **Redirect URLs** добавь этот же адрес (и `http://localhost` для локальных тестов).
4. **Project Settings → API:** скопируй **Project URL** и ключ **anon public**.
5. В приложении: **☁ синхронизация** → вставь URL и ключ → «Сохранить» → введи email → войди по ссылке из письма.
6. Повтори вход на втором устройстве тем же email — данные объединятся.

## Напоминания в Telegram (по желанию)

Бот присылает 3 напоминания в день (10:00 / 15:00 / 20:00 по Москве) через бесплатный
планировщик GitHub Actions — работает, даже когда ноутбук и телефон выключены.
Нужен опубликованный на GitHub репозиторий (см. выше) и бот в Telegram.

1. В Telegram напиши **@BotFather** → `/newbot` → задай имя → получишь **токен** (`123456:ABC…`).
2. Узнай свой **chat_id**: напиши что-нибудь своему боту, затем открой в браузере
   `https://api.telegram.org/bot<ТОКЕН>/getUpdates` и найди `"chat":{"id": <ЧИСЛО>}`.
   (Или напиши боту **@userinfobot** — он пришлёт твой id.)
3. В репозитории на GitHub: **Settings → Secrets and variables → Actions → New repository secret**,
   добавь два секрета:
   - `TELEGRAM_BOT_TOKEN` = токен из шага 1
   - `TELEGRAM_CHAT_ID` = число из шага 2
4. Проверь: вкладка **Actions → «Маяк — напоминания» → Run workflow** — бот должен прислать сообщение.
5. Дальше сообщения приходят по расписанию сами.

Файл расписания — `.github/workflows/reminders.yml`. Время задано в **UTC** (Москва = UTC+3).
Чтобы поменять часы или пояс — правь строки `cron:` и `TZ` в этом файле, а адрес приложения — `APP_URL`.

> GitHub может задерживать запуск на несколько минут и отключает расписание,
> если в репозитории 60 дней нет активности (любой коммит включает обратно).

## Локальный запуск
```bash
python3 -m http.server 4599
# открыть http://localhost:4599
```

Данные приватны: код открыт, но твои записи лежат либо в браузере, либо в **твоём** Supabase
(доступ только у тебя, защита на уровне строк RLS).
