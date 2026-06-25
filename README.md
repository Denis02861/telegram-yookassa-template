# Telegram bot + YooKassa payments — шаблон n8n

Минимальный рабочий шаблон для приёма оплат в Telegram-боте через YooKassa. **15 нод суммарно в двух workflow.** Без БД. Без Telegram Payments (в РФ заблокирован) — прямая интеграция с YooKassa API.

## Что внутри

- `workflows/01-bot-create-payment.json` — бот: /pay → меню тарифов → создание платежа в YK → ссылка пользователю (9 нод)
- `workflows/02-yookassa-webhook.json` — webhook от YK: проверка статуса → сообщение пользователю о успехе (6 нод)

## Как работает

```
Пользователь → /pay → бот показывает тарифы
       ↓
   Клик «Купить»
       ↓
бот → YooKassa API создаёт payment
       ↓
бот → присылает кнопку «Оплатить» с YK-ссылкой
       ↓
пользователь оплачивает на странице YK
       ↓
YooKassa → POST на /webhook/yookassa-webhook
       ↓
webhook → проверяет статус → шлёт пользователю «Оплата прошла»
```

Состояние не хранится в БД. Метаданные (chat_id, tier) передаются через `metadata` в самом платеже YK и возвращаются обратно в webhook.

## Установка (10 шагов)

1. **Создайте Telegram-бота** через @BotFather, сохраните токен
2. **Получите доступ к YooKassa**: kassa.yookassa.ru → магазин → API ключи. Нужны: shopId + secretKey
3. **В n8n создайте 2 credentials:**
   - `Your Telegram Bot` (Telegram API, токен из шага 1)
   - `YooKassa (shopId + secretKey)` (HTTP Basic Auth: username = shopId, password = secretKey)
4. **Импортируйте оба workflow** из папки `workflows/` в n8n
5. **В импортированных workflow замените credentials** на свои (везде где `REPLACE`)
6. **В `01-bot-create-payment.json` → нода `Build YK Payment`:**
   - Замените `return_url` на ссылку вашего бота: `https://t.me/YOUR_BOT_USERNAME`
   - Замените email-fallback `@yourbot.example` на свой домен или соберите реальный email с пользователя (ФЗ-54)
   - Под себя поправьте тарифы в ноде `Router` (объект `TARIFFS`)
7. **Активируйте оба workflow**
8. **Откройте webhook-URL** в `02-yookassa-webhook.json` (Production URL), скопируйте
9. **В личном кабинете YooKassa** → Интеграция → Уведомления HTTP → добавьте URL вебхука. Подпишитесь на события: `payment.succeeded` (минимум), `payment.canceled` (опционально)
10. **Тест:** напишите боту `/pay`, нажмите «Купить», оплатите тестовой картой YK. Бот должен прислать «Оплата прошла»

## Что НЕ покрыто этим шаблоном

- БД для хранения подписок и истории платежей
- Возвраты (можно добавить второй webhook с событием `refund.succeeded`)
- Idempotence на уровне БД (есть только на уровне создания YK payment через 5-минутное окно)
- Реальный сбор email пользователя для чека (используется fallback)
- Многоразовые подписки и rebill

## Источник

Шаблон выжат из реального бота [@CheknikartyBot](https://t.me/) (Review Defender), который обрабатывает >5 платежей в день. В шаблон попали только базовые элементы для быстрого старта.

## Лицензия

MIT — используйте, переделывайте, продавайте.
