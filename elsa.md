# Elsa Automation

Софт для автоматизации проекта HeyElsa.
Выполняет 2 ежедневные задачи:

-   [`ElsaBridgeQuestTask` - выполнение квеста "Bridge Builder" на платформе Elsa](#elsabridgequesttask)
-   [`ElsaDailyVolumeQuestTask` - получение объёма транзакций на платформе Elsa](#elsadailyvolumequesttask)

# 🚀 Начало работы

## 1. 🔧 Установка

Скачайте и установите python 3.11.

_Можно использовать любую версию из серии 3.11 (например, 3.11.6)_

https://www.python.org/downloads/release/python-3115/

-   Windows - Windows installer (64-bit) / Windows installer (32-bit)

-   Mac - macOS 64-bit universal2 installer

    После установки:

    Откройте папку установки питона(Например, Applications/Python 3.11)
    Запустите файл `Install Certificates.command`

## 2. ⚙️ Конфигурация

### 1. Заполните файлы данных (находятся в папке проекта):

-   `private_keys.txt` - приватные ключи кошельков (1 ключ на строку)
-   `proxies.txt` - прокси в формате `логин:пароль@ip:порт` (опционально - 1 прокси на строку)

    🚨 Запуск возможен только в том случае, когда количество проксей больше или равно количеству аккаунтов, либо прокси отсутствуют вообще
    🚨 Очки начисляются проектом только за качественные по их мнению промпты

_При запуске софт синхронизирует базу данных с файлами `private_keys.txt` и `proxies.txt`._

-   Порядок ключей и проксей играет роль.
-   Связка аккаунт - прокси определяется номером строки в соответствующем файле.
-   Если вы поменяете местами 2 прокси, они заменятся у двух соответствующих аккаунтов.

### 2. В файле `.env` настройте следующие параметры:

| Переменная                       | Описание                                                                                                            | Пример/Значение по умолчанию  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| `DEBUG`                          | Уровень логирования: 0 — INFO, 1 — DEBUG, 2 — TRACE                                                                 | 1                             |
| `MODE`                           | Режим работы планировщика: `run` — прогон, `farm` — ферма [Подробнее](#⏰-механизм-планирования-задач)              | run                           |
| `ACCOUNT_DELAY_SECONDS_INTERVAL` | Интервал задержки между аккаунтами в секундах (для режима "run"). Формат: `[min, max]` или `null`                   | null (по умолчанию 10-20 сек) |
| `WORK_HOURS`                     | Рабочие часы для режима "farm". Формат: `HH:MM-HH:MM` или `null`                                                    | 12:30-19:15                   |
| `WORKERS_NUMBER`                 | Количество параллельных воркеров (1-32)                                                                             | 4                             |
| `COMPLETE_BASE_BRIDGE_TASK`      | Выполнять ли задачу по бриджу в Base ([ElsaBridgeQuestTask](#elsabridgequesttask)): 1 — да, 0 — нет                 | 1                             |
| `DAILY_VOLUME_USD`               | Ежедневный целевой объём транзакций в USD (50-500) для задачи [ElsaDailyVolumeQuestTask](#elsadailyvolumequesttask) | 50                            |
| `TG_BOT_TOKEN`                   | Токен Telegram-бота для уведомлений                                                                                 | null                          |
| `TG_ID`                          | ID чата/пользователя для уведомлений                                                                                | null                          |
| `NOTIFICATION_INTERVAL_MINUTES`  | Интервал отправки информационных уведомлений в Telegram (минуты)                                                    | 10                            |
| `SSL_VERIFY`                     | Верификация SSL-сертификатов (1 — включено, 0 — отключено) - не рекомендуется отключать                             | 1                             |

**Примечания:**

-   Если `TG_BOT_TOKEN` и `TG_ID` не заданы, Telegram-уведомления отключены.
-   Для режима "farm" важно корректно задать `WORK_HOURS`, иначе задачи будут распределяться на весь день.
-   `ACCOUNT_DELAY_SECONDS_INTERVAL` в формате `[min, max]` (например, `[120, 300]` для задержки 2-5 минут).


## 3. 🚦 Запуск и остановка

-   Откройте папку проекта через терминал.
-   Создайте и активируйте виртуальное окружение.
-   Установите пакеты используя requirements.txt:
    ```bash
    pip install -r requirements.txt
    ```
-   Запустите команду:

    -   Windows: `python main.py`
    -   Mac/Linux: `python3 main.py`

-   Для остановки программы нажмите сочетание клавиш Ctrl + C в терминале.

_(подробная инструкция по работе с окружением и пакетами есть в файле `python.md`)_

## 4. 📲 Телеграм уведомления

Для получения уведомлений о ходе работы и статусе аккаунтов используйте Telegram-бота.

### Как настроить уведомления:

1. **Создайте Telegram-бота через [@BotFather](https://t.me/BotFather):**
    - Введите команду `/newbot` и следуйте инструкциям.
    - Получите токен бота (`TG_BOT_TOKEN`).
2. **Узнайте Telegram ID для чата уведомлений:**
    - Напишите [@UserInfoToBot](https://t.me/UserInfoToBot) или аналогичному боту.
    - Получите ID и укажите его в `TG_ID`.
    - Если нужно уведомлять группу, то добавьте бота в нужную группу и дайте ему права на отправку сообщений.
    - Если нужно, чтобы бот отправлял уведомления пользователю, пользователь должен предварительно отправить любое сообщение боту.
3. **Укажите параметры в .env:**
    - `TG_BOT_TOKEN` — токен вашего Telegram-бота
    - `TG_ID` — ваш Telegram user ID или ID группы/чата
    - `NOTIFICATION_INTERVAL_MINUTES` — как часто отправлять сводку (по умолчанию 10 минут)

**Виды уведомлений:**

-   О старте и завершении работы.
-   О статусе аккаунтов (выполнено, запущено, запланировано, ошибка).
-   О критических ошибках (например, недостаточно средств).

**Пример уведомления:**

```
ℹ️ Аккаунты:
Выполнены: 3
Запущены: 1
Запланированы: 2
Ошибка: 0
```

# ✍️ Логирование

Лог создаётся для каждого запуска программы. В названии файла лога отмечено время старта.

-   Общие логи работы - `/logs/info/run_<дата>.log`
-   Подробные логи работы - `/logs/tracing/run_<дата>.log`
-   Лог о выполнении аккаунтов - `/logs/completions.log`
-   Технические логи работы - `/logs/internals/run_<дата>.json`

# ⏰ Механизм планирования задач

Планировщик задач для автоматизации работает в двух режимах: "run" и "farm".

## Режим "run" - ПРОГОН (последовательное выполнение)

1. Характеристики:

    - Задачи выполняются последовательно для всех аккаунтов
    - Между запусками аккаунтов есть случайная задержка (по умолчанию 10-20 секунд).
        - _Определяется параметром `ACCOUNT_DELAY_SECONDS_INTERVAL` в файле `.env_
    - Не учитывает рабочие часы (work_hours)
    - Задачи запускаются сразу после завершения предыдущих

2. Логика работы:

    - Получает список всех аккаунтов
    - Для каждого аккаунта:
        - Проверяет, есть ли активная сессия
        - Получает список задач, которые нужно выполнить сегодня
        - Планирует выполнение с задержкой относительно предыдущего аккаунта

3. Использование:
    - Подходит для быстрого выполнения задач на всех аккаунтах
    - Когда временные ограничения не критичны

## Режим "farm" - ФЕРМА (распределенное выполнение)

1. Характеристики:

    - Задачи распределяются случайным образом в течение дня
    - Учитывает рабочие часы (work_hours), если они заданы
        - _Определяется параметром `WORK_HOURS` в файле `.env_
    - Учитывает предполагаемое время выполнения задач
    - Позволяет имитировать "естественное" поведение пользователей

2. Логика работы:

    - Определяет временное окно для выполнения (рабочие часы или весь день)
    - Рассчитывает самое позднее время начала, чтобы уложиться в окно выполнения
    - Для каждого аккаунта:
        - Генерирует случайное время начала в допустимом диапазоне
        - Проверяет, есть ли активная сессия
        - Получает список задач, которые нужно выполнить сегодня
        - Планирует выполнение в случайное время
        - Подходит для имитации человеческого поведения и избегания подозрительной активности

3. Использование:
    - Когда нужно имитировать поведение реальных пользователей
    - Когда важно распределить нагрузку в течение дня
    - При работе с ограниченными временными окнами (рабочие часы)

# 📋 Описание задач

# ElsaBridgeQuestTask

### Назначение

Задача `ElsaBridgeQuestTask` автоматизирует выполнение квеста "Bridge Builder" на платформе Elsa. Цель - выполнить бридж ETH на сумму ~$20-21 из сетей Arbitrum или Optimism в сеть Base.
Задачу можно отключить с помощью настройки в файле `.env` - `COMPLETE_BASE_BRIDGE_TASK`

### Ключевые константы и параметры

#### Сети

-   **Отправляющие сети:** Arbitrum, Optimism
-   **Целевая сеть:** Base
-   **Множитель расходов:** 1.005 (0.5% на комиссии)

#### Суммы

-   **Минимальная сумма бриджа:** $20.5 USD
-   **Максимальная сумма бриджа:** $21 USD
-   **С учетом расходов:** $20.6 - $21.1 USD
-   **Минимальная сумма пополнения:** $20.7 USD (с учетом двойных расходов)

### Алгоритм выполнения

#### 1. Проверка статуса квеста

-   Получение списка квестов с дашборда
-   Поиск квеста "Bridge Builder"
-   Если квест уже выполнен - завершение

#### 2. Анализ балансов

-   Получение балансов ETH во всех сетях
-   Проверка достаточности средств в отправляющих сетях

#### 3. Логика принятия решений

##### Сценарий A: Прямой бридж

-   Если в Arbitrum или Optimism достаточно ETH (≥$20.6)
-   Выполняется бридж случайной суммы $20.6-$21.1 в Base

##### Сценарий B: Пополнение + бридж

-   Если в отправляющих сетях недостаточно средств
-   Проверяется баланс в Base
-   Если в Base достаточно средств (≥$20.7):
    -   Случайно выбирается сеть для пополнения (Arbitrum или Optimism)
    -   Выполняется бридж из Base в выбранную сеть на $20.7-$21.1
    -   После пополнения выполняется бридж обратно в Base на $20.6-$21.1

#### 4. Обработка ошибок

-   Если недостаточно средств нигде - выбрасывается ошибка `StopExecutionException` с уведомлением

#### Особенности:

-   **Случайные суммы** в заданных диапазонах
-   **Округление** до 2 знаков после запятой
-   **Генерация естественных сообщений** для чата
-   **Поллинг статуса транзакций** в реальном времени
-   **Автоматическая подпись и отправка транзакций** через Web3

### Детали констант

#### Сети

-   **SENDER_BRIDGE_CHAINS:** ["arbitrum", "optimism"]
-   **TARGET_BRIDGE_CHAIN:** "base"

#### Суммы (USD)

-   **MIN_TARGET_BRIDGE_AMOUNT_USD:** 20.5
-   **MAX_TARGET_BRIDGE_AMOUNT_USD:** 21.0
-   **EXPENSES_MULTIPLIER:** 1.005 (0.5% комиссии)
-   **MIN_TARGET_BRIDGE_AMOUNT_USD_WITH_EXPENSES:** 20.6
-   **MAX_TARGET_BRIDGE_AMOUNT_USD_WITH_EXPENSES:** 21.1
-   **MIN_TOP_UP_AMOUNT_USD_WITH_EXPENSES:** 20.7

#### Логика расчета сумм

1. **Прямой бридж:** $20.6 - $21.1 (с учетом комиссий)
2. **Пополнение:** $20.7 - $21.1 (с учетом двойных комиссий)
3. **Целевой бридж после пополнения:** $20.6 - $21.1

# ElsaDailyVolumeQuestTask

### Назначение

`ElsaDailyVolumeQuestTask` — автоматизация ежедневного квеста по объёму транзакций на платформе Elsa. Цель — достичь заданного объёма swap-операций в сети Base.
Необходимый объём устанавливается в файле `.env` - `DAILY_VOLUME_USD` - от $50 до $500

_Программа будет стараться свапнуть стейблы в ETH, так чтобы баланс аккаунта преимущественно состоял из ETH_

### Ключевые параметры

-   **Целевая сеть:** Base
-   **Целевой объём:** задаётся через `DAILY_VOLUME_USD` в .env (от $50 до $500)
-   **Минимальный swap:** $10
-   **Токены:** ETH (native), USDC, DAI
-   **Комиссионный множитель:** 0.005 (0.5%)
-   **Стратегия:** оптимальный подбор последовательности swap-операций для достижения объёма с учётом балансов и комиссий

### Алгоритм выполнения

1. **Проверка прогресса квеста**

    - Получение текущего прогресса по квесту "Volume Sprinter".
    - Если объём уже достигнут — задача завершается.

2. **Анализ балансов**

    - Получение балансов ETH, USDC, DAI в сети Base.
    - Если нет минимального баланса для swap — задача останавливается с ошибкой.

3. **Поиск swap-стратегии**
    - Используется модуль swap_strategy_finder (реализован на Go).
    - Подбирается последовательность swap-операций (например, ETH→USDC, USDC→DAI и т.д.), чтобы суммарный объём был не меньше требуемого.
    - Учитываются комиссии, ограничения по минимальной сумме, остатки на счетах.

#### Подробнее о логике поиска swap-стратегии

-   Используется алгоритм **beam search** (широкий поиск с отсечением худших вариантов на каждом шаге).
-   На каждом шаге перебираются все возможные swap-операции между токенами (ETH, USDC, DAI), с разными суммами (от максимального баланса до минимального swap).
-   Для каждого состояния рассчитывается **эвристика** (насколько оно близко к целевому объёму, сколько осталось native-токена, сколько swap-ов между стейблами и т.д.).
-   В приоритете:
    1. Минимальное количество swap-ов (меньше комиссий и транзакций)
    2. Максимальный остаток в ETH (native)
    3. Точное попадание в целевой объём
    4. Использование swap-ов между стейблами
-   Стратегия строится так, чтобы после достижения объёма все средства по возможности вернулись в ETH.
-   Если не удаётся найти стратегию с заданными параметрами (например, не хватает баланса) — задача завершается с ошибкой.

4. **Выполнение swap-операций**

    - Для каждой операции из найденной стратегии выполняется swap.
    - Между swap-ами делается небольшая задержка (5-7 секунд).
    - После каждого swap логируется результат.

5. **Завершение**
    - После достижения объёма задача логирует успех.

#### Особенности:

-   Минимальный swap — $10.
-   Учитываются комиссии (0.5%).
-   Стратегия swap-ов минимизирует избыточный объём и комиссии.
-   Все действия логируются, при ошибках отправляются уведомления в Telegram (если настроено).

---
