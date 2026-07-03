# 🌟 Маяк — дневник привычек

Персональный трекер привычек, статистики и дневник. Работает как приложение (PWA):
ставится на телефон и ноутбук, работает офлайн, данные хранятся локально, а по желанию
синхронизируются между устройствами через бесплатный Supabase.

- **Сегодня** — счётчики «дней без вредного», чек-лист дня, заметка с настроением
- **Привычки** — таблица «привычка × день», добавление/настройка привычек
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

## Локальный запуск
```bash
python3 -m http.server 4599
# открыть http://localhost:4599
```

Данные приватны: код открыт, но твои записи лежат либо в браузере, либо в **твоём** Supabase
(доступ только у тебя, защита на уровне строк RLS).
