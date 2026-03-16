---
description: MikoPBX REST API v3 — Руководство по Python-интеграции
---

# API ключи (IN DEV)

В этой статье описана работа с REST API MikoPBX на примере базовых практических задач: создание сотрудников и SIP-провайдеров, получение истории звонков, мониторинг состояния станции в реальном времени.

{% hint style="info" %}
Для корректной работы у Вас должен быть действующий https сертификат. Один из простых способов для выпуска доверенного сертификата - [Let's encrypt](../../modules/miko/module-get-ssl-lets-encrypt.md).
{% endhint %}

### Подготовка: API-ключ и окружение

#### Создание API-ключа

1. Перейдите в раздел : "**Система" → "API ключи".**&#x20;

<figure><img src="../../.gitbook/assets/APIKeysSection.png" alt=""><figcaption><p>Раздел "Cистема" -> "API ключи"</p></figcaption></figure>

2. Нажмите **"Добавить API ключ".**

* Заполните поле **Описание** (например: `CRM Integration`)
* Скопируйте сгенерированный API-ключ — он отображается **только один раз.**
* **Сетевой фильтр:** ограничьте доступ по IP или выберите «Разрешены с любых адресов»

Выберите права по ресурсам: **«Чтение»** (GET) или **«Чтение и запись»** (POST/PUT/DELETE)

В зависимости от задачи переключите тумблер **«Полные права доступа»** или настройте права вручную для каждого ресурса. Придерживайтесь принципа минимальных привилегий (Least Privilege) — каждый ключ должен иметь доступ только к тем ресурсам, которые реально нужны.

{% hint style="info" %}
API-ключ используется только для получения JWT-токена через `POST /auth`. В последующих запросах передаётся Bearer-токен, а не сам ключ.
{% endhint %}

<figure><img src="../../.gitbook/assets/APIKeyBasicSettings.png" alt=""><figcaption><p>Базовые настройки API ключа</p></figcaption></figure>

В этой инструкции в качестве примеров будут приведены следующие действия с помощью REST API:

* Создать сотрудника - необходимо разрешить доступ к интерфейсу "Employees Management" на уровне "Чтение и запись".
* Создать провайдера - необходимо разрешить доступ к интерфейсу "Providers" на уровне "Чтение и запись".
* Получить статусы регистрации сотрудников и провайдеров - необходимо разрешить доступ к интерфейсу "SIP" на уровне "Чтение".
* Получить историю звонков - необходимо разрешить доступ к интерфейсу "Call Records" на уровне "Чтение".

<mark style="color:$warning;">Так же выдайте права на интерфейс "Authentication" на уровне "Чтение и запись".</mark>

<figure><img src="../../.gitbook/assets/APIKeyCDR&#x26;Auth.png" alt=""><figcaption><p>Пример с выдачей доступа к интерфейсу "Call Records"</p></figcaption></figure>

В этой статье, мы будем работать с Python, поэтому необходимо установить все зависимости:

```bash
pip install requests
```

###

| Поле                          | Обяз. | Тип / ограничения                      | Описание                                            |
| ----------------------------- | ----- | -------------------------------------- | --------------------------------------------------- |
| `number`                      | ✅     | string, 2–8 цифр                       | Добавочный номер                                    |
| `user_username`               | ✅     | string, 1–100 символов                 | ФИО сотрудника                                      |
| `sip_secret`                  | ✅     | string, 5–100 символов                 | Пароль SIP-аккаунта                                 |
| `user_email`                  | —     | string email, ≤255                     | Email для уведомлений                               |
| `mobile_number`               | —     | string E.164, ≤50                      | Мобильный (+7...) для переадресации                 |
| `mobile_dialstring`           | —     | string, ≤255                           | Строка набора мобильного                            |
| `sip_transport`               | —     | `udp` / `tcp` / `tls` / `udp,tcp`      | Транспорт SIP (по умолч.: `udp`)                    |
| `sip_dtmfmode`                | —     | `auto` / `rfc4733` / `inband` / `info` | Режим DTMF (по умолч.: `auto`)                      |
| `sip_enableRecording`         | —     | boolean                                | Запись разговоров (по умолч.: `true`)               |
| `sip_networkfilterid`         | —     | number \| `"none"`                     | ID сетевого фильтра                                 |
| `sip_manualattributes`        | —     | string, ≤1024                          | Дополнительные SIP-параметры                        |
| `fwd_ringlength`              | —     | integer, ≤180                          | Время дозвона до переадресации (сек, по умолч.: 45) |
| `fwd_forwarding`              | —     | number \| `hangup` \| `busy`           | Безусловная переадресация                           |
| `fwd_forwardingonbusy`        | —     | number \| `hangup` \| `busy`           | Переадресация при занятости                         |
| `fwd_forwardingonunavailable` | —     | number \| `hangup` \| `busy`           | Переадресация при недоступности                     |

#### Создание сотрудника

```python
def create_employee(
    number: str,
    name: str,
    sip_secret: str,
    email: str = '',
    mobile: str = '',
    record_calls: bool = True,
    fwd_ringlength: int = 45,
) -> dict:
    payload = {
        'number':              number,
        'user_username':       name,
        'sip_secret':          sip_secret,
        'sip_enableRecording': record_calls,
        'fwd_ringlength':      fwd_ringlength,
    }
    if email:  payload['user_email']    = email
    if mobile: payload['mobile_number'] = mobile

    result = client.post('/employees', json=payload)

    if result.get('result'):
        emp_id = result['data']['id']
        print(f'✅ Создан: {number} ({name}), id={emp_id}')
    else:
        errors = result.get('messages', {}).get('error', [])
        print(f'❌ Ошибка: {errors}')
    return result


# Минимальный пример
create_employee(
    number='201',
    name='Иванов Иван',
    sip_secret='Secure#Pass9201',
)

# Полный пример
create_employee(
    number='202',
    name='Петрова Анна',
    sip_secret='Secure#Pass9202',
    email='anna@company.ru',
    mobile='79001234567',
    record_calls=True,
    fwd_ringlength=30,
)
```

#### Ответ API (HTTP 201)

```json
{
  "result": true,
  "data": {
    "number": "201",
    "user_username": "Иванов Иван",
    "sip_secret": "Secure#Pass9201",
    "sip_dtmfmode": "auto",
    "sip_transport": "udp",
    "sip_enableRecording": true,
    "sip_networkfilterid": "none",
    "fwd_ringlength": 45,
    "id": "1",
    "extensions_length": 3
  },
  "messages": {"error": [], "info": [], "warning": []}
}
```

#### Коды ответов

| Код | Описание                                                             |
| --- | -------------------------------------------------------------------- |
| 201 | Сотрудник успешно создан                                             |
| 400 | Ошибка валидации (слабый пароль <5 символов, неверный формат номера) |
| 401 | Токен истёк — нужно обновить через `/auth`                           |
| 403 | Нет прав на запись для ресурса `/employees`                          |
| 409 | Конфликт — номер уже занят                                           |

#### Список сотрудников

```python
def list_employees(search: str = '', limit: int = 100, offset: int = 0) -> list:
    params = {'limit': limit, 'offset': offset}
    if search: params['search'] = search
    result = client.get('/employees', params=params)
    return result.get('data', result) if isinstance(result, dict) else result

for emp in list_employees():
    print(f"  {emp.get('number'):>6}  {emp.get('user_username', '')}")
```

#### Массовое создание

```python
import time

employees = [
    {'number': '201', 'name': 'Иванов Иван',  'secret': 'Pass#9201'},
    {'number': '202', 'name': 'Петрова Анна', 'secret': 'Pass#9202'},
    {'number': '203', 'name': 'Сидоров Пётр', 'secret': 'Pass#9203'},
]

created, failed = [], []
for emp in employees:
    result = client.post('/employees', json={
        'number':        emp['number'],
        'user_username': emp['name'],
        'sip_secret':    emp['secret'],
    })
    if result.get('result'):
        created.append(emp['number'])
        print(f"✅ {emp['number']} {emp['name']}")
    else:
        failed.append(emp['number'])
        print(f"❌ {emp['number']}: {result.get('messages', {}).get('error', [])}")
    time.sleep(0.2)

print(f'Создано: {len(created)}, Ошибок: {len(failed)}')
```

***

### 5. Управление SIP-провайдерами

**Для CRUD:** `POST /sip-providers`\
**Для просмотра всех (SIP + IAX):** `GET /providers` (только чтение)

#### Создание SIP-провайдера

```python
def create_sip_provider(
    uniqid: str,
    host: str,
    username: str,
    password: str,
    description: str = '',
    qualify: bool = True,
) -> dict:
    payload = {
        'uniqid':            uniqid,
        'description':       description or f'{username}@{host}',
        'host':              host,
        'username':          username,
        'secret':            password,
        'qualify':           qualify,
        'registration_type': 'inbound+outbound',
    }
    result = client.post('/sip-providers', json=payload)
    if result.get('result'):
        print(f'✅ Провайдер создан: {description or uniqid}')
    else:
        print(f'❌ Ошибка: {result.get("messages", {}).get("error", [])}')
    return result


create_sip_provider(
    uniqid='SIP-ZADARMA-001',
    host='sip.zadarma.com',
    username='316811',
    password='mysecretpass',
    description='Zadarma',
)
```

#### Список всех провайдеров

```python
def list_providers() -> list:
    result = client.get('/providers')
    return result.get('data', result) if isinstance(result, dict) else result

for prov in list_providers():
    print(f"  {prov.get('id'):<20} {prov.get('description', '')}  [{prov.get('type', '')}]")
```

***

### 6. История звонков (CDR)

**Эндпоинт:** `GET /cdr` — только чтение.\
Поддерживает фильтрацию по дате, номеру, статусу, длительности.

#### Базовый запрос с фильтрацией

```python
def get_cdr(
    offset: int = 0,
    limit: int = 100,
    date_from: str = None,    # формат: 'YYYY-MM-DD HH:MM:SS'
    date_to: str = None,
    src_num: str = None,
    dst_num: str = None,
    disposition: str = None,  # ANSWERED / NO ANSWER / BUSY / FAILED
) -> list:
    params = {'offset': offset, 'limit': min(limit, 1000)}
    if date_from:   params['date_from']   = date_from
    if date_to:     params['date_to']     = date_to
    if src_num:     params['src_num']     = src_num
    if dst_num:     params['dst_num']     = dst_num
    if disposition: params['disposition'] = disposition

    result = client.get('/cdr', params=params)
    return result.get('data', result) if isinstance(result, dict) else result


# Последние 20 звонков
for row in get_cdr(limit=20):
    print(
        str(row.get('start', ''))[:16],
        row.get('src_num', ''), '→', row.get('dst_num', ''),
        row.get('disposition', ''), row.get('billsec', 0), 'с'
    )
```

#### Статистика за период

```python
from datetime import datetime, timedelta

def cdr_stats(days: int = 1) -> dict:
    now  = datetime.now()
    then = now - timedelta(days=days)
    records = get_cdr(
        date_from=then.strftime('%Y-%m-%d %H:%M:%S'),
        date_to=now.strftime('%Y-%m-%d %H:%M:%S'),
        limit=1000
    )
    answered  = [r for r in records if r.get('disposition') == 'ANSWERED']
    missed    = [r for r in records if r.get('disposition') == 'NO ANSWER']
    total_dur = sum(r.get('billsec', 0) for r in answered)
    return {
        'total':    len(records),
        'answered': len(answered),
        'missed':   len(missed),
        'avg_sec':  total_dur // len(answered) if answered else 0,
    }

stats = cdr_stats(days=7)
print(f"Звонков за 7 дней: {stats['total']}")
print(f"Отвечено:          {stats['answered']}")
print(f"Пропущено:         {stats['missed']}")
print(f"Средняя длит.:     {stats['avg_sec']}с")
```

#### Поля CDR-записи

| Поле            | Тип      | Описание                                     |
| --------------- | -------- | -------------------------------------------- |
| `start`         | datetime | Время начала звонка                          |
| `answer`        | datetime | Время ответа (пусто — пропущен)              |
| `endtime`       | datetime | Время завершения                             |
| `src_num`       | string   | Номер звонящего                              |
| `dst_num`       | string   | Номер назначения                             |
| `disposition`   | string   | `ANSWERED` / `NO ANSWER` / `BUSY` / `FAILED` |
| `billsec`       | int      | Длительность разговора (секунды)             |
| `duration`      | int      | Полная длительность (включая дозвон)         |
| `recordingfile` | string   | Путь к MP3-записи разговора                  |

> 📼 Стриминг записей: `GET /cdr/{id}/recording` — поддерживает HTTP Range.

***

### 7. Мониторинг: статусы SIP и активные звонки

#### Статусы SIP-устройств

**Эндпоинт:** `GET /sip` — только мониторинг.

```python
def get_sip_status() -> dict:
    return client.get('/sip')

sip = get_sip_status()

# Статусы сотрудников
peers = sip.get('peers', sip.get('data', []))
print('\n── Сотрудники ──────────────────────────────')
for p in peers:
    icon = '🟢' if p.get('state') == 'OK' else '🔴'
    print(f"  {icon}  {p.get('id'):>6}  {p.get('state', '')}")

# Статусы провайдеров
registry = sip.get('registry', [])
print('\n── Провайдеры (регистрация) ─────────────────')
for entry in registry:
    icon = '🟢' if entry.get('state') == 'OK' else '🔴'
    print(f"  {icon}  {entry.get('username', '?')}@{entry.get('host', '?')}  [{entry.get('state')}]")
```

#### Активные звонки в реальном времени

**Эндпоинт:** `GET /pbx-status`

```python
def get_active_calls() -> dict:
    return client.get('/pbx-status')

status   = get_active_calls()
calls    = status.get('calls', [])
channels = status.get('channels', [])

print(f'Активных звонков: {len(calls)}')
print(f'Активных каналов: {len(channels)}')

for call in calls:
    print(f"  📞 {call.get('src_num', '?')} → {call.get('dst_num', '?')}  ({call.get('duration', 0)}с)")
```

#### Значения поля `state`

| Значение      | Описание                                               |
| ------------- | ------------------------------------------------------ |
| `OK`          | Устройство зарегистрировано и доступно                 |
| `UNKNOWN`     | Не зарегистрировано (оффлайн)                          |
| `Unreachable` | Было зарегистрировано, но не отвечает на QUALIFY-пинги |
| `OFF`         | Регистрация отключена в настройках провайдера          |

### 9. Полный скрипт мониторинга

```python
"""
mikopbx_monitor.py — мониторинг MikoPBX REST API v3
Запуск: python mikopbx_monitor.py
"""
from mikopbx_client import MikoPBXClient
from datetime import datetime, timedelta

BASE_URL = 'https://your-mikopbx.com'
API_KEY  = 'YOUR_API_KEY'

client = MikoPBXClient(BASE_URL, API_KEY)


def section(title: str, width: int = 48):
    pad = '─' * (width - len(title) - 2)
    print(f'\n── {title} {pad}')


def show_sip():
    section('SIP-устройства и провайдеры')
    data     = client.get('/sip')
    peers    = data.get('peers', data.get('data', []))
    registry = data.get('registry', [])

    online  = [p for p in peers if p.get('state') == 'OK']
    offline = [p for p in peers if p.get('state') != 'OK']
    print(f'  Сотрудники — 🟢 Онлайн: {len(online)}  🔴 Оффлайн: {len(offline)}')
    for p in offline:
        print(f"    🔴 {p.get('id'):>6}  [{p.get('state')}]")

    print('  Провайдеры:')
    for e in registry:
        icon = '🟢' if e.get('state') == 'OK' else '🔴'
        print(f"    {icon}  {e.get('username', '?')}@{e.get('host', '?')}  [{e.get('state')}]")


def show_active_calls():
    section('Активные звонки')
    status = client.get('/pbx-status')
    calls  = status.get('calls', [])
    if not calls:
        print('  (нет активных звонков)')
        return
    for c in calls:
        print(f"  📞  {c.get('src_num', '?')} → {c.get('dst_num', '?')}  {c.get('duration', 0)}с")


def show_cdr_summary():
    section('Статистика за последние 24 часа')
    now  = datetime.now()
    then = now - timedelta(days=1)
    rows = client.get('/cdr', params={
        'date_from': then.strftime('%Y-%m-%d %H:%M:%S'),
        'date_to':   now.strftime('%Y-%m-%d %H:%M:%S'),
        'limit': 1000
    })
    records  = rows.get('data', rows) if isinstance(rows, dict) else rows
    answered = [r for r in records if r.get('disposition') == 'ANSWERED']
    missed   = [r for r in records if r.get('disposition') == 'NO ANSWER']

    print(f'  Всего звонков: {len(records)}')
    print(f'  Отвечено:      {len(answered)}')
    print(f'  Пропущено:     {len(missed)}')

    print('  Последние 5 звонков:')
    for r in records[:5]:
        t    = str(r.get('start', ''))[:16]
        icon = '✅' if r.get('disposition') == 'ANSWERED' else '❌'
        print(f"    {icon} {t}  {r.get('src_num', '')} → {r.get('dst_num', '')}  {r.get('billsec', 0)}с")


if __name__ == '__main__':
    print(f'MikoPBX Monitor [{datetime.now().strftime("%Y-%m-%d %H:%M:%S")}]')
    print(f'Станция: {BASE_URL}')
    show_sip()
    show_active_calls()
    show_cdr_summary()
```

***

### 10. Справочник эндпоинтов

Базовый путь: `https://YOUR_HOST/pbxcore/api/v3/`

> ⚠️ «Чтение» = GET. «Чтение и запись» = GET + POST + PUT + DELETE.\
> Права настраиваются отдельно для каждого ресурса в разделе «API ключи».

#### Телефония и маршрутизация

| Путь                     | Методы | Описание                                             |
| ------------------------ | ------ | ---------------------------------------------------- |
| `/employees`             | CRUD   | Сотрудники (внутренние номера)                       |
| `/extensions`            | GET    | Все номера: сотрудники, IVR, очереди — только чтение |
| `/sip-providers`         | CRUD   | SIP-провайдеры                                       |
| `/iax-providers`         | CRUD   | IAX-провайдеры                                       |
| `/providers`             | GET    | Все провайдеры (SIP + IAX) — только чтение           |
| `/call-queues`           | CRUD   | Очереди вызовов                                      |
| `/ivr-menu`              | CRUD   | IVR-меню                                             |
| `/incoming-routes`       | CRUD   | Входящие маршруты (DID)                              |
| `/outbound-routes`       | CRUD   | Исходящие маршруты                                   |
| `/off-work-times`        | CRUD   | Расписание нерабочего времени                        |
| `/conference-rooms`      | CRUD   | Конференц-залы                                       |
| `/dialplan-applications` | CRUD   | Приложения диалплана                                 |

#### Мониторинг и статистика

| Путь                  | Методы | Описание                                    |
| --------------------- | ------ | ------------------------------------------- |
| `/sip`                | GET    | Статусы SIP-устройств (peers + registry)    |
| `/iax`                | GET    | Статусы IAX-соединений                      |
| `/pbx-status`         | GET    | Активные звонки и каналы в реальном времени |
| `/cdr`                | GET    | История звонков с фильтрацией и пагинацией  |
| `/cdr/{id}/recording` | GET    | Стриминг записи разговора (HTTP Range)      |
| `/advice`             | GET    | Системные рекомендации и предупреждения     |
| `/sysinfo`            | GET    | Аппаратная информация о сервере             |
| `/syslog`             | GET    | Системные логи и диагностика                |

#### Аутентификация и управление доступом

| Путь                   | Методы   | Описание                                    |
| ---------------------- | -------- | ------------------------------------------- |
| `/auth`                | POST     | Получить JWT access\_token по API-ключу     |
| `/auth/refresh`        | POST     | Обновить access\_token через refresh\_token |
| `/api-keys`            | CRUD     | Управление API-ключами                      |
| `/asterisk-managers`   | CRUD     | Пользователи AMI                            |
| `/asterisk-rest-users` | CRUD     | Пользователи ARI (для ARI-приложений)       |
| `/users`               | CRUD     | Пользователи веб-интерфейса                 |
| `/passkeys`            | CRUD     | WebAuthn / FIDO2 passkeys                   |
| `/passwords`           | GET/POST | Генерация и валидация паролей               |
| `/network-filters`     | GET      | Сетевые фильтры для UI-списков              |

#### Системные настройки

| Путь                | Методы   | Описание                                |
| ------------------- | -------- | --------------------------------------- |
| `/system`           | GET/POST | Ping, reboot, update, factory reset     |
| `/general-settings` | GET/PUT  | Общие настройки АТС                     |
| `/time-settings`    | GET/PUT  | Часовой пояс и NTP                      |
| `/network`          | GET/PUT  | Сетевые интерфейсы, IP, DNS, NAT        |
| `/firewall`         | CRUD     | Правила файервола                       |
| `/fail2ban`         | GET/PUT  | Политики блокировки IP                  |
| `/mail-settings`    | GET/PUT  | Настройки SMTP / OAuth2                 |
| `/storage`          | GET/PUT  | Управление дисками и хранилищем         |
| `/s3-storage`       | GET/PUT  | S3-облачное хранилище записей           |
| `/modules`          | CRUD     | Модули расширения                       |
| `/license`          | GET/POST | Управление лицензиями                   |
| `/sound-files`      | CRUD     | Звуковые файлы (IVR, MOH, объявления)   |
| `/files`            | CRUD     | Управление файлами (chunked upload)     |
| `/custom-files`     | CRUD     | Пользовательские конфигурационные файлы |
| `/search`           | GET      | Глобальный поиск по всем сущностям      |
| `/openapi`          | GET      | OpenAPI/Swagger спецификация            |

> 📖 Интерактивная Swagger-документация: `https://your-mikopbx.com/pbxcore/api/v3/openapi`
