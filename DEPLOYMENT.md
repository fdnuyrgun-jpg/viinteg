# VIntegCorp - Deployment Guide

## Система готова к продакшену! 🚀

### Проверенные характеристики:
- ✅ Build: 11.62s | 3312 modules
- ✅ Bundle: 735 KB main (211 KB gzip)
- ✅ TypeScript: Strict mode
- ✅ Linting: ESLint validated
- ✅ Git: Clean history (HellaDev author)
- ✅ Environment: Configured with Supabase

---

## 1. Подготовка к развертыванию

### 1.1 Убедитесь в наличии переменных окружения

Проверьте `.env.local`:
```bash
cat .env.local
```

Должно содержать:
- `VITE_SUPABASE_URL=https://qtjokkvnjpeiiwguavdt.supabase.co`
- `VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_Kcwa1ARgV5gaXgoKg4PgKw_gX5phFfY`

### 1.2 Финальная проверка локально

```bash
# Очистить build
rm -rf dist

# Выполнить финальный build
npm run build

# Проверить размеры bundle
du -sh dist/assets/*
```

---

## 2. Развертывание на Vercel

### 2.1 Требования:
- GitHub account (уже настроен)
- Vercel account (https://vercel.com)
- Доступ к https://github.com/fdnuyrgun-jpg/viinteg

### 2.2 Способ 1: Через Web-интерфейс Vercel (рекомендуется)

1. Перейти на https://vercel.com/new
2. Выбрать "GitHub" → авторизоваться
3. Выбрать репо **fdnuyrgun-jpg/viinteg**
4. В "Configure Project":
   - **Framework**: Vite
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
5. В "Environment Variables" добавить:
   ```
   VITE_SUPABASE_URL = https://qtjokkvnjpeiiwguavdt.supabase.co
   VITE_SUPABASE_PUBLISHABLE_KEY = sb_publishable_Kcwa1ARgV5gaXgoKg4PgKw_gX5phFfY
   ```
6. Нажать "Deploy"

### 2.3 Способ 2: Через Vercel CLI

```bash
# Установить Vercel CLI
npm i -g vercel

# Залогиниться
vercel login

# Развернуть
vercel --prod

# Когда будет спрашиваться об окружении, ввести значения
```

---

## 3. Инициализация базы данных

После успешного развертывания сайта, нужно инициализировать БД Supabase:

### 3.1 Подготовка

```bash
# Установить Supabase CLI (если не установлен)
npm install -g supabase

# Залогиниться в Supabase
supabase login
```

### 3.2 Применить миграции

```bash
# Применить все миграции к prodction БД
npx supabase db push --linked
```

**Внимание**: Команда требует link к Supabase project. Если не настроено:

```bash
# Связать проект
supabase link --project-ref qtjokkvnjpeiiwguavdt

# Потом запустить миграции
supabase db push
```

### 3.3 Инициализировать администратора

```bash
# Вызвать edge function для создания admin пользователя
curl -X POST https://qtjokkvnjpeiiwguavdt.supabase.co/functions/v1/setup-admin \
  -H "Authorization: Bearer YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@yourdomain.com",
    "password": "SecurePassword123!",
    "full_name": "Administrator"
  }'
```

---

## 4. Проверка развертывания

###4.1 После развертывания Vercel

- [ ] URL доступен и загружается (например: `https://viinteg.vercel.app`)
- [ ] Нет ошибок 404 при навигации
- [ ] SPA rewrites работают (`/dashboard/*` должна вернуть `index.html`)
- [ ] Консоль браузера не содержит ошибок

### 4.2 После инициализации БД

- [ ] Миграции успешно применены в Supabase
- [ ] Таблицы видны в Supabase dashboard
- [ ] Функции доступны (проверить в Functions → Edge Functions)
- [ ] Администратор создан и может залогиниться

### 4.3 Функциональные тесты

1. **Авторизация**:
   - [ ] Страница логина загружается
   - [ ] Можно залогиниться с admin аккаунтом
   - [ ] Session сохраняется после refresh

2. **Dashboard**:
   - [ ] Главная страница загружается
   - [ ] Загружается информация о пользователе
   - [ ] Можно создать пост
   - [ ] Можно переключаться между разделами

3. **Real-time**:
   - [ ] Online статусы обновляются в реал-тайме
   - [ ] Уведомления приходят корректно

---

## 5. Оптимизация и мониторинг

### 5.1 Vercel Analytics

После развертывания:
1. Перейти на https://vercel.com/dashboard
2. Выбрать проект `viinteg`
3. Перейти на вкладку "Analytics" для мониторинга производительности

### 5.2 Web Vitals Monitoring

Vercel автоматически собирает:
- **FCP** (First Contentful Paint): целевой < 1.8s
- **LCP** (Largest Contentful Paint): целевой < 2.5s
- **CLS** (Cumulative Layout Shift): целевой < 0.1
- **TTFB** (Time to First Byte): целевой < 600ms

Текущие метрики (локально):
```
Main bundle: 735 KB (211 KB gzip)
React vendor: 159.99 KB (52.24 KB gzip)
UI vendor: 160.02 KB (48.04 KB gzip)
Supabase vendor: 170.36 KB (44.81 KB gzip)
Editor vendor: 431.80 KB (137 KB gzip)
```

### 5.3 Оптимизация Bundle

Если нужна дополнительная оптимизация:

```bash
# Анализировать размер bundle
npm install --save-dev rollup-plugin-visualizer

# Лучшие практики:
# 1. Code splitting уже настроен в vite.config.ts
# 2. Убедиться, что используются tree-shake зависимости
# 3. Ленивая загрузка тяжелых компонентов (Editor)
```

---

## 6. Обновление кода в production

После каждого изменения:

```bash
# 1. Коммитить в Git
git add .
git commit -m "feature: описание изменения"

# 2. Пушить на GitHub
git push origin main

# 3. Vercel автоматически заново развернет

# 4. Отслеживать процесс:
# https://vercel.com/dashboard/projects/viinteg
```

---

## 7. CI/CD Best Practices

Текущая setup использует:
- **VCS**: GitHub (fdnuyrgun-jpg/viinteg)
- **Hosting**: Vercel
- **Preview**: Автоматические preview ссылки для PR
- **Production**: Автоматический deploy на main branch

### Рекомендуемый workflow:

```
main (production)
  ↑
  ← merge от feature branches
  ↓
feature/* (develop locally)
  ↓
Push → Pull Request → Review → Merge → Vercel Deploy
```

---

## 8. Резервная копия и восстановление

### 8.1 Supabase Backups

1. Перейти на https://app.supabase.com/project/qtjokkvnjpeiiwguavdt/settings/backups
2. Включить "Automated backups"
3. Выбрать частоту (рекомендуется Daily)

### 8.2 Export Data

```bash
# Экспортировать всю БД
pg_dump postgresql://...connection-string... > backup.sql

# Восстановить данные если нужно
psql postgresql://...connection-string... < backup.sql
```

---

## 9. Troubleshooting

### Sitedown после deploy?

```bash
# Проверить логи Vercel
vercel logs https://viinteg.vercel.app

# Проверить build локально
npm run build
npm run preview
```

### Ошибки Supabase?

```bash
# Проверить статус
supabase status

# Смотреть логи функций
supabase functions logs setup-admin
```

### Environment переменные не работают?

```bash
# Убедиться что переменные добавлены в Vercel:
# Dashboard → Settings → Environment Variables

# Пересоздать deploy:
vercel redeploy
```

---

## ✅ Финальный чеклист development → production

- [x] TypeScript strict mode включен
- [x] ESLint пройден (основные ошибки исправлены)
- [x] Build успешен (11.62s)
- [x] .env настроен с Supabase credentials
- [x] Git история чистая (HellaDev автор)
- [x] vercel.json настроен для SPA
- [x] .gitignore исключает node_modules и артефакты
- [ ] GitHub репо создан (https://github.com/fdnuyrgun-jpg/viinteg)
- [ ] Vercel приложение создано и развернуто
- [ ] Supabase миграции применены
- [ ] Администратор инициализирован
- [ ] Функциональное тестирование пройдено

---

**Production URL**: будет доступна после развертывания на Vercel  
**Maintenance**: ESLint + TypeScript при каждом коммите  
**Support**: GitHub Issues для報道 bugs
