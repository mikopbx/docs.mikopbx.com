---
description: Инструкция с примерами по созданию и использованию API-ключей
---

# API ключи (IN DEV)

В этой статье описана работа с REST API MikoPBX на примере базовых практических задач: создание сотрудников и SIP-провайдеров, получение истории звонков, мониторинг состояния станции в реальном времени.

{% hint style="info" %}
Если у вас отсутствует доверенный сертификат — добавьте `verify=False` в каждый запрос и отключите предупреждения:

```python
import urllib3
urllib3.disable_warnings()
```

Настоятельно рекомендуется выпустить доверенный сертификат. Самый простой способ селать это - с помощью модуля [Let's encrypt](../../modules/miko/module-get-ssl-lets-encrypt.md).
{% endhint %}

### Подготовка: API-ключ и окружение

**Создание API-ключа**

1. Перейдите в раздел: **"Система" → "API ключи".**

<figure><img src="../../.gitbook/assets/APIKeysSection.png" alt=""><figcaption><p>Раздел "Система" -> "API ключи"</p></figcaption></figure>

2. Нажмите **"Добавить API ключ".**

* Заполните поле **Описание** (например: `CRM Integration`)
* Скопируйте сгенерированный API-ключ — он отображается **только один раз**
* **Сетевой фильтр:** ограничьте доступ по IP или выберите «Разрешены с любых адресов»

В зависимости от задачи переключите тумблер **«Полные права доступа»** или настройте права вручную для каждого ресурса. Придерживайтесь принципа минимальных привилегий (Least Privilege) — каждый ключ должен иметь доступ только к тем ресурсам, которые реально нужны.

<figure><img src="../../.gitbook/assets/APIKeyBasicSettings.png" alt=""><figcaption><p>Базовые настройки API ключа</p></figcaption></figure>

В этой инструкции будут рассмотрены следующие задачи:

* **Создать сотрудника** — разрешите доступ к ресурсу **"Employees Management"** на уровне **"Чтение и запись"**
* **Создать SIP-провайдера** — разрешите доступ к ресурсу **"SIP Providers"** на уровне **"Чтение и запись"**
* **Получить статусы регистрации** сотрудников и провайдеров — разрешите доступ к ресурсу **"SIP"** на уровне **"Чтение"**
* **Получить историю звонков** — разрешите доступ к ресурсу **"Call Records"** на уровне **"Чтение"**

<figure><img src="../../.gitbook/assets/APIKeyCallRecords.png" alt=""><figcaption><p>Пример настройки прав доступа</p></figcaption></figure>

В этой статье, мы будем работать с Python, поэтому необходимо установить все необходимые зависимости:

```bash
pip install requests
```

### **Подключение**

Ниже приведен скрипт для подключения к станции через API-ключ. Этот шаблон необходимо использовать перед всеми скриптами, описанными в этой инструкции.

API-ключ и URL для API запросов передаются напрямую в заголовке запроса - никакой дополнительной аутентификации не требуется:

```python
import requests

BASE_URL = 'https://your-mikopbx.com/pbxcore/api/v3'
API_KEY  = 'ваш-api-ключ'

HEADERS = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type':  'application/json',
}
```

{% hint style="info" %}
В шаблоне замените следующие параметры:

* "**your-mikopbx.com**" на IP адрес или URL Вашей станции.
* "**ваш-api-ключ**" на ранее созданный API-ключ с необходимыми правами.
{% endhint %}

### Работа с сотрудниками

**Эндпоинт:** `POST /pbxcore/api/v3/employees`

Ниже приведена таблица с параметрами (полями) для такого запроса, которые Вы можете использовать.

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

#### **Создание одного сотрудника**

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

    r = requests.post(f'{BASE_URL}/employees', headers=HEADERS, json=payload)
    result = r.json()
    if result.get('result'):
        print(f" Создан: {number} ({name}), id={result['data']['id']}")
    else:
        print(f" Ошибка: {result.get('messages', {}).get('error', [])}")
    return result


# Минимальный пример (только обязательные поля)
create_employee(
    number='243',
    name='Иванов Иван',
    sip_secret='Secure#Pass9201',
)

# Полный пример
create_employee(
    number='244',
    name='Петрова Анна',
    sip_secret='Secure#Pass9202',
    email='anna@company.ru',
    mobile='79001234567',
    record_calls=True,
    fwd_ringlength=30,
)
```

Ниже приведен пример ответа API (HTTP 201) на такой запрос:

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

Возможные коды ответов:

| Код | Описание                                                             |
| --- | -------------------------------------------------------------------- |
| 201 | Сотрудник успешно создан                                             |
| 400 | Ошибка валидации (слабый пароль <5 символов, неверный формат номера) |
| 401 | Неверный или отсутствующий API-ключ                                  |
| 403 | Нет прав на запись для ресурса `/employees`                          |
| 409 | Конфликт — номер уже занят                                           |

В случае успешного выполнения запроса Вы увидите следующий вывод в консоль:

{% code overflow="wrap" %}
```python
 Создан: 243 (Иванов Иван), id=113
 Создан: 244 (Петрова Анна), id=114

Process finished with exit code 0
```
{% endcode %}

На станции будут созданы сотрудники 243 и 244.

<figure><img src="../../.gitbook/assets/createdExtensionsWithAPI.png" alt=""><figcaption><p>Созданные сотрудники с помощью REST API</p></figcaption></figure>

#### Получение списка сотрудников

```python
def list_employees(search: str = '', limit: int = 100, offset: int = 0) -> list:
    params = {'limit': limit, 'offset': offset}
    if search: params['search'] = search
    r = requests.get(f'{BASE_URL}/employees', headers=HEADERS, params=params)
    return r.json().get('data', {}).get('data', []) 

for emp in list_employees():
    print(f"  {emp.get('number'):>6}  {emp.get('user_username', '')}")
```

Будет выведен список сотрудников в следующем формате:

{% code overflow="wrap" %}
```python
     202  Brown Brandon
     203  Collins Melanie
     201  Smith James
     243  Иванов Иван
     244  Петрова Анна

Process finished with exit code 0
```
{% endcode %}

#### **Массовое создание сотрудников**

```python
import time

employees = [
    {'number': '251', 'name': 'Иванов Иван',  'secret': 'Pass#9201'},
    {'number': '252', 'name': 'Петрова Анна', 'secret': 'Pass#9202'},
    {'number': '253', 'name': 'Сидоров Пётр', 'secret': 'Pass#9203'},
]

created, failed = [], []
for emp in employees:
    r = requests.post(
        f'{BASE_URL}/employees',
        headers=HEADERS,
        json={
            'number':        emp['number'],
            'user_username': emp['name'],
            'sip_secret':    emp['secret'],
        }
    )
    result = r.json()
    if result.get('result'):
        created.append(emp['number'])
        print(f" {emp['number']} {emp['name']}")
    else:
        failed.append(emp['number'])
        print(f" {emp['number']}: {result.get('messages', {}).get('error', [])}")
    time.sleep(0.2)  # небольшая пауза между запросами

print(f'Создано: {len(created)}, Ошибок: {len(failed)}')
```

В случае успешного выполнения запроса, Вы увидите следующий вывод в консоль:

{% code overflow="wrap" %}
```python
 251 Иванов Иван
 252 Петрова Анна
 253 Сидоров Пётр
Создано: 3, Ошибок: 0

Process finished with exit code 0
```
{% endcode %}

На станции будет создано 3 сотрудника:

<figure><img src="../../.gitbook/assets/created3ExtensionsWithAPI.png" alt=""><figcaption><p>Созданные сотрудники с помощью REST API</p></figcaption></figure>

### Работа с SIP-провайдерами

**Эндпоинт:** `POST /pbxcore/api/v3/sip-providers`

Ниже приведена таблица с параметрами (полями) для такого запроса, которые Вы можете использовать.

| Поле                | Обяз. | Тип     | Описание                                                   |
| ------------------- | ----- | ------- | ---------------------------------------------------------- |
| `description`       | ✅     | string  | Название провайдера                                        |
| `host`              | ✅     | string  | Адрес SIP-сервера провайдера                               |
| `username`          | —     | string  | Логин на сервере провайдера                                |
| `secret`            | —     | string  | Пароль                                                     |
| `registration_type` | —     | string  | `inbound` / `outbound` / `none`                            |
| `qualify`           | —     | boolean | Мониторинг доступности (по умолч.: `true`)                 |
| transport           | —     | string  | `udp` / `tcp` / `tls` / `udp,tcp` (по умолч.: `udp,tcp`)   |
| dtmfmode            | —     | string  | `auto` / `rfc4733` / `inband` / `info` (по умолч.: `auto`) |
| port                | —     | integer | Порт подключения (по умолч.: `5060`)                       |
| disabled            | —     | boolean | Отключить провайдера (по умолч.: `false`)                  |

#### Создание провайдера

```python
def create_sip_provider(
    description: str,
    host: str,
    username: str = '',
    password: str = '',
    registration_type: str = 'outbound',
    qualify: bool = True,
) -> dict:
    payload = {
        'description': description,
        'host':        host,
    }
    if username:          payload['username']          = username
    if password:          payload['secret']            = password
    if registration_type: payload['registration_type'] = registration_type
    if not qualify:       payload['qualify']           = qualify

    r = requests.post(f'{BASE_URL}/sip-providers', headers=HEADERS, json=payload)
    result = r.json()
    if result.get('result'):
        print(f" Провайдер создан: {description}")
    else:
        print(f" Ошибка: {result.get('messages', {}).get('error', [])}")
    return result


create_sip_provider(
    description='Zadarma',
    host='sip.zadarma.com',
    username='316811',
    password='mysecretpass',
)
```

В случае успешного выполнения запроса Вы увидите следующий вывод в консоль:

{% code overflow="wrap" %}
```python
 Провайдер создан: Zadarma

Process finished with exit code 0
```
{% endcode %}

На станции будет создан провайдер:

<figure><img src="../../.gitbook/assets/createdProviderWithAPI.png" alt=""><figcaption><p>Созданный провайдер с помощью REST API</p></figcaption></figure>

#### **Список всех провайдеров**

```python
def list_providers() -> list:
    r = requests.get(f'{BASE_URL}/providers', headers=HEADERS)
    return r.json().get('data', [])

for prov in list_providers():
    print(f"  {prov.get('id'):<20} {prov.get('description', '')}  [{prov.get('type', '')}]")
```

***

#### История звонков (CDR)

**Эндпоинт:** `GET /pbxcore/api/v3/cdr` — только чтение.

**Параметры запроса**

| Параметр      | Тип     | Описание                                     |
| ------------- | ------- | -------------------------------------------- |
| `offset`      | integer | Смещение для пагинации (по умолч.: 0)        |
| `limit`       | integer | Кол-во записей, макс. 1000                   |
| `date_from`   | string  | Начало периода: `YYYY-MM-DD HH:MM:SS`        |
| `date_to`     | string  | Конец периода: `YYYY-MM-DD HH:MM:SS`         |
| `src_num`     | string  | Фильтр по номеру звонящего                   |
| `dst_num`     | string  | Фильтр по номеру назначения                  |
| `disposition` | string  | `ANSWERED` / `NO ANSWER` / `BUSY` / `FAILED` |

```python
from datetime import datetime, timedelta

def get_cdr(
    offset: int = 0,
    limit: int = 100,
    date_from: str = None,
    date_to: str = None,
    src_num: str = None,
    dst_num: str = None,
    disposition: str = None,
) -> list:
    params = {'offset': offset, 'limit': min(limit, 1000)}
    if date_from:   params['date_from']   = date_from
    if date_to:     params['date_to']     = date_to
    if src_num:     params['src_num']     = src_num
    if dst_num:     params['dst_num']     = dst_num
    if disposition: params['disposition'] = disposition

    r = requests.get(f'{BASE_URL}/cdr', headers=HEADERS, params=params)
    return r.json().get('data', [])


# Последние 20 звонков
for row in get_cdr(limit=20):
    print(
        str(row.get('start', ''))[:16],
        row.get('src_num', ''), '→', row.get('dst_num', ''),
        row.get('disposition', ''), row.get('billsec', 0), 'с'
    )
```

**Статистика за период**

```python
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

**Поля CDR-записи**

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

\{% hint style="info" %\} Стриминг записи разговора: `GET /cdr/{id}/recording` — поддерживает HTTP Range для постепенной загрузки. \{% endhint %\}

***

#### Мониторинг: статусы SIP и активные звонки

**Статусы регистрации сотрудников и провайдеров**

**Эндпоинт:** `GET /pbxcore/api/v3/sip` — только мониторинг.

```python
def get_sip_status() -> dict:
    r = requests.get(f'{BASE_URL}/sip', headers=HEADERS)
    return r.json()

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

**Значения поля `state`**

| Значение      | Описание                                               |
| ------------- | ------------------------------------------------------ |
| `OK`          | Устройство зарегистрировано и доступно                 |
| `UNKNOWN`     | Не зарегистрировано (оффлайн)                          |
| `Unreachable` | Было зарегистрировано, но не отвечает на QUALIFY-пинги |
| `OFF`         | Регистрация отключена в настройках провайдера          |

**Активные звонки в реальном времени**

**Эндпоинт:** `GET /pbxcore/api/v3/pbx-status`

```python
def get_active_calls() -> dict:
    r = requests.get(f'{BASE_URL}/pbx-status', headers=HEADERS)
    return r.json()

status   = get_active_calls()
calls    = status.get('calls', [])
channels = status.get('channels', [])

print(f'Активных звонков: {len(calls)}')
print(f'Активных каналов: {len(channels)}')

for call in calls:
    print(f"  📞 {call.get('src_num', '?')} → {call.get('dst_num', '?')}  ({call.get('duration', 0)}с)")
```

***

#### Полный скрипт мониторинга

```python
"""
mikopbx_monitor.py — мониторинг MikoPBX REST API v3
Запуск: python mikopbx_monitor.py
"""
import requests
from datetime import datetime, timedelta

BASE_URL = 'https://your-mikopbx.com/pbxcore/api/v3'
API_KEY  = 'ваш_api_ключ'

HEADERS = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type':  'application/json',
}


def section(title: str, width: int = 48):
    pad = '─' * (width - len(title) - 2)
    print(f'\n── {title} {pad}')


def show_sip():
    section('SIP-устройства и провайдеры')
    data     = requests.get(f'{BASE_URL}/sip', headers=HEADERS).json()
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
    status = requests.get(f'{BASE_URL}/pbx-status', headers=HEADERS).json()
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
    data = requests.get(
        f'{BASE_URL}/cdr',
        headers=HEADERS,
        params={
            'date_from': then.strftime('%Y-%m-%d %H:%M:%S'),
            'date_to':   now.strftime('%Y-%m-%d %H:%M:%S'),
            'limit':     1000
        }
    ).json()
    records  = data.get('data', [])
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

#### Справочник эндпоинтов

Базовый путь: `https://YOUR_HOST/pbxcore/api/v3/`

\{% hint style="warning" %\} «Чтение» = GET. «Чтение и запись» = GET + POST + PUT + DELETE. Права настраиваются отдельно для каждого ресурса в разделе «API ключи». \{% endhint %\}

**Телефония и маршрутизация**

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

**Мониторинг и статистика**

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

**Системные настройки**

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

Интерактивная Swagger-документация вашей станции: `https://your-mikopbx.com/pbxcore/api/v3/openapi`&#x20;
