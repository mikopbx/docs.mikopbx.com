---
description: Короткое описание ARI (Asterisk REST Interface)
---

# Доступ к ARI (IN DEV)

ARI — это RESTful API с поддержкой WebSocket, который даёт полный контроль над каналами, мостами и медиапотоками Asterisk в реальном времени. В отличие от [REST API MikoPBX](api-keys/), ARI работает напрямую с ядром Asterisk и предназначен для разработки собственных телефонных приложений.

**По умолчанию отключён** — включите при необходимости в разделе **«Система» → «Asterisk REST Interface (ARI)»**.

{% hint style="info" %}
Подробная документация по ARI доступна на официальном сайте Asterisk: [Asterisk REST Interface](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/)
{% endhint %}

### Для чего используется?

ARI применяется когда стандартных возможностей АТС недостаточно и нужна собственная логика обработки звонков:

* **WebRTC приложения и софтфоны** — веб-телефоны и мобильные клиенты с прямым управлением медиапотоками
* **Интерактивные голосовые меню (IVR)** — кастомная логика меню, недоступная через стандартный диалплан
* **Конференц-связь** — программное управление мостами и участниками
* **Запись и обработка звонков** — перехват аудио в реальном времени для аналитики или транскрибации
* **Голосовые боты и ассистенты** — интеграция с внешними AI-сервисами
* **Продвинутые очереди** — собственная логика распределения звонков

### Как это работает?

ARI состоит из трёх компонентов:

1. **REST API** — управление объектами Asterisk: каналами, мостами, записями
2. **WebSocket** (`/ari/events`) — получение асинхронных событий в реальном времени: входящий звонок, завершение звонка, нажатие DTMF и т.д.
3. **Stasis** — приложение диалплана, которое передаёт канал под управление вашего ARI приложения

Типичный сценарий: звонок попадает в диалплан → `Stasis()` передаёт канал вашему приложению → приложение управляет звонком через REST API и получает события через WebSocket.

### Настройка ARI пользователя

1. Перед началом, необходимо включить ARI интерфейс (по умолчанию он выключен). Для этого перейдите в раздел "**Система**" -> "**Общие настройки**".

<figure><img src="../../.gitbook/assets/generalSettingsMikoPBX.png" alt=""><figcaption><p>Раздел "Система" -> "Общие настройки" в MikoPBX</p></figcaption></figure>

2. Далее во вкладку "**AMI\&ARI**", переключите тумблер "**Использовать ARI интерфейс**".  В поле **«Разрешённые источники CORS»** укажите домены, с которых будут выполняться запросы к ARI. CORS — механизм безопасности браузера, который ограничивает кросс-доменные запросы к API.

| Значение                    | Когда использовать                |
| --------------------------- | --------------------------------- |
| _(пусто)_                   | Доступ только с того же домена    |
| `http://localhost:3000`     | Локальная разработка              |
| `https://app.mycompany.com` | Продакшн приложение               |
| `*`                         | Все источники — только для тестов |

{% hint style="warning" %}
Никогда не используйте `*` в продакшне. Указывайте только доверенные домены по HTTPS.
{% endhint %}

<figure><img src="../../.gitbook/assets/AMI&#x26;ARIinGenSettings.png" alt=""><figcaption><p>Подключение ARI</p></figcaption></figure>

3. Перейдите в раздел **«Система» → «Доступ к ARI»**.

<figure><img src="../../.gitbook/assets/ARIAccessSection.png" alt=""><figcaption><p>Раздел "Система" -> "Доступ к ARI" в MikoPBX</p></figcaption></figure>

4. Нажмите **«Добавить пользователя»**.

<figure><img src="../../.gitbook/assets/createAriUserBtn.png" alt=""><figcaption><p>Кнопка "Добавить пользователя" в разделе "Доступ к ARI"</p></figcaption></figure>

5. Заполните следующие параметры:

* **Имя пользователя** - логин для подключения, например `ari_user`.
* **Пароль** - пароль для подключения.
* **Описание** - описание для текущего пользователя, например "WebRTC Demo".
* **Приложения** - укажите имена Stasis приложений, к которым имеет доступ пользователь. Оставьте поле пустым для доступа ко всем приложениям.&#x20;

{% hint style="info" %}
**Распространенные приложения**

* **ari-app:** Основное приложение для ARI
* **stasis:** Базовое Stasis приложение
* **external-media:** Работа с внешними медиа-потоками
* **bridge-app:** Управление мостами вызовов
* **channel-spy:**&#x41F;рослушивание каналов
{% endhint %}

Сохраните настройки.

<figure><img src="../../.gitbook/assets/ARIUserParameters.png" alt=""><figcaption><p>Параметры создаваемого ARI-пользователя</p></figcaption></figure>

### Параметры подключения

#### WebSocket

| Тип              | URL                                                                      |
| ---------------- | ------------------------------------------------------------------------ |
| Обычный          | `ws://your-mikopbx.com:8088/ari/events?app=[application]&subscribe=all`  |
| Защищённый (TLS) | `wss://your-mikopbx.com:8089/ari/events?app=[application]&subscribe=all` |

Замените `[application]` на имя вашего Stasis приложения.

#### REST API

| Тип   | URL                                  |
| ----- | ------------------------------------ |
| HTTP  | `http://your-mikopbx.com:8088/ari/`  |
| HTTPS | `https://your-mikopbx.com:8089/ari/` |

**Аутентификация:** HTTP Basic Auth — логин и пароль ARI пользователя.

```bash
# REST API запрос через curl
curl -u username:password http://your-mikopbx.com:8088/ari/channels
```

{% hint style="info" %}
Рекомендуется использовать защищённые подключения (wss:// и https://) с валидным SSL сертификатом. Обычные ws:// и http:// допустимы только в изолированной тестовой среде.
{% endhint %}

### Пример: Hello World

Это минимальный пример работы с ARI — канал входит в Stasis приложение, приложение воспроизводит звуковой файл и завершает звонок.

Пример взят из официальной документации Asterisk: [Getting Started with ARI](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/Getting-Started-with-ARI/)

#### Шаг 1 — подключитесь к WebSocket

```bash
wscat -c "ws://username:password@your-mikopbx.com:8088/ari/events?app=hello-world"
```

#### Шаг 2 — настройте входящий маршрут

Чтобы звонки попадали в ARI приложение, добавьте в диалплан Asterisk вызов `Stasis()` с именем вашего приложения:

```
exten => 1000,1,NoOp()
 same => n,Answer()
 same => n,Stasis(hello-world)
 same => n,Hangup()
```

{% hint style="info" %}
В MikoPBX это делается через раздел **«Маршрутизация» → «Приложения диалплана»** — создайте приложение с кодом `Stasis(hello-world)` и назначьте его на нужный входящий маршрут.
{% endhint %}

#### Шаг 3 — позвоните на номер

При входящем звонке WebSocket получит событие `StasisStart`:

```json
{
  "type": "StasisStart",
  "application": "hello-world",
  "channel": {
    "id": "1400609726.3",
    "name": "PJSIP/202-00000001",
    "state": "Up"
  }
}
```

#### Шаг 4 — воспроизведите звук через REST API

Используйте `id` канала из события `StasisStart`:

```bash
curl -u username:password -X POST \
  "http://your-mikopbx.com:8088/ari/channels/1400609726.3/play?media=sound:hello-world"
```

WebSocket последовательно пришлёт два события:

```json
{"type": "PlaybackStarted", "playback": {"state": "playing", "media_uri": "sound:hello-world"}}
{"type": "PlaybackFinished", "playback": {"state": "done", "media_uri": "sound:hello-world"}}
```

#### Шаг 5 — завершите звонок

После завершения звонка WebSocket пришлёт `StasisEnd`:

```json
{
  "type": "StasisEnd",
  "channel": {
    "id": "1400609726.3",
    "name": "PJSIP/202-00000001"
  }
}
```

### Пример: автосекретарь с DTMF меню

Более практичный пример — автосекретарь на Python, который принимает звонок, воспроизводит приветствие и переводит на нужный номер по нажатию клавиши.

Установите зависимости:

```bash
pip install requests websockets
```

```python
import asyncio
import websockets
import json
import requests
from requests.auth import HTTPBasicAuth

ARI_URL  = 'https://your-mikopbx.com:8089/ari'
ARI_USER = 'ari-login'
ARI_PASS = 'ari-password'
APP_NAME = 'auto-receptionist'

auth = HTTPBasicAuth(ARI_USER, ARI_PASS)

MENU = {
    '1': '201',  # Отдел продаж
    '2': '202',  # Поддержка
    '0': '200',  # Оператор
}

def answer_channel(channel_id: str):
    requests.post(f'{ARI_URL}/channels/{channel_id}/answer', auth=auth)

def play_audio(channel_id: str, media: str):
    requests.post(
        f'{ARI_URL}/channels/{channel_id}/play',
        auth=auth,
        json={'media': media}
    )

def transfer_channel(channel_id: str, extension: str):
    requests.post(
        f'{ARI_URL}/channels/{channel_id}/redirect',
        auth=auth,
        json={'endpoint': f'PJSIP/{extension}'}
    )

async def run():
    uri = (
        f"wss://your-mikopbx.com:8089/ari/events"
        f"?app={APP_NAME}&subscribe=all"
    )
    async with websockets.connect(
        uri,
        extra_headers={'Authorization': 'Basic dXNlcjpwYXNz'}
    ) as ws:
        print(f'Автосекретарь запущен [{APP_NAME}]')
        async for message in ws:
            event = json.loads(message)
            etype = event.get('type')
            ch    = event.get('channel', {})
            ch_id = ch.get('id', '')

            if etype == 'StasisStart':
                print(f'Входящий звонок: {ch.get("name")}')
                answer_channel(ch_id)
                play_audio(ch_id, 'sound:welcome')

            elif etype == 'ChannelDtmfReceived':
                digit = event.get('digit', '')
                if digit in MENU:
                    transfer_channel(ch_id, MENU[digit])
                else:
                    play_audio(ch_id, 'sound:invalid-selection')

            elif etype == 'StasisEnd':
                print(f'Звонок завершён: {ch.get("name")}')

asyncio.run(run())
```

{% hint style="info" %}
Имя приложения `APP_NAME` должно совпадать с полем **«Приложения»** в настройках ARI пользователя и с именем в вызове `Stasis(auto-receptionist)` в диалплане.
{% endhint %}

***

> Полная документация по ARI — на сайте Asterisk: [docs.asterisk.org](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/)
>
> Полная REST API документация MikoPBX — в разделе Интерактивная документация и список эндпоинтов.
