# Модуль Автоинформатор

Предоставляет REST API для формирования кампаний по автоматическому набору внешних номеров

* Если есть свободный сотрудник, то модуль набирает номер телефона из пула кампании
* Когда клиент отвечает, выполняется соединение с внутренним номером очереди или сотрудника
* Если внутренний номер занят, то вызов автоматически завершается

### Авторизация в REST API

Для всех запросов, кроме локальных, требуется авторизация.&#x20;

По умолчанию для авторизации можно использовать административный пароль web интерфейса.&#x20;

Если необходимо ограничить права к настройкам АТС - то можно задействовать модуль «**Управление доступом в систему**».&#x20;

Запрос

```
curl 'http://172.16.156.223/admin-cabinet/session/start' \
-X 'POST' --cookie-jar auth-cookies.txt \
-H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' \
-H 'X-Requested-With: XMLHttpRequest' \
--data 'login=admin&password=adminpassword'
```

* «`login`» - admin, имя пользователя web интерфейса
* «`password`» - adminpassword, пароль пользователя web интерфейса
* «`172.16.156.223`» - адрес сервера MikoP

Результат:

```
{"success":true,"reload":false,"message":[]}
```

Пример выполнения запросов:

```
curl -b auth-cookies.txt \
'http://172.16.156.223pbxcore/api/sip/getPeersStatuses'
```

### Опросы

Позволяют организовать автоматический опрос клиента. Средствами API возможно описать перечень вопросов, связи между ними и варианты ответов. &#x20;

Возможно использовать механизм генерации речи или готовые mp3 / wav файлы.&#x20;

#### Добавление опроса

Пример  cURL:

<pre><code><strong>JSON='{
</strong>  "name": "New polling",
  "questions": [
    {
      "questionId": "1",
      "questionText": "Добрый день, у нас есть новый товар, для заказа товара нажмите 1, не интересует - нажмите 2, для отказа от подписке нажмите 3",
      "press": [
        {
          "key": "1",
          "action": "playback",
          "value": "\"Спасибо за выбор этого товара\"",
          "valueOptions": "text"
        },
        {
          "key": "3",
          "action": "playback",
          "value": "\"Спасибо за ваш ответ\"",
          "valueOptions": "text"
        }
      ],
      "nextQuestion": ""
    }
  ]
}';
<strong>curl -X POST -d "$JSON" http://127.0.0.1/pbxcore/api/module-dialer/v1/polling
</strong></code></pre>

* "**`crmId`**" - идентификатор опроса, если указан, то будет создан новый опрос с этим ID либо отредактирован существующий, если пуст, то параметр будет назначен автоматически
* "**`name`**" - наименование опроса
* "**`questions`**" - массив вопросов
* "**`question`**`['`**`questionId`**`']`" - порядковый номер (идентификатор) вопроса
* "**`question['nextQuestion']`**" - идентификатор следующего вопроса
* "**`question['questionText']`**" - текст вопроса, для возможности генерации **Yandex SpeechKit**
* "**`question[`**`'`**`lang`**`']`" -  код языка для **Yandex SpeechKit**  к примеру "ru-RU"
* "**`question['questionFile']`**" - полный путь к файлу wav, если указан, то "questionText" не используется
* "**`press['key']`**" - добавочный номер, который должен нажать клиент
* "**`press['action']`**" - действие, которое должно быть выполнено
  * action "**`playback`**" - воспроизвести медиафайл, сохранить набранный номер
  * action "**`dial`**" - набрать добавочный номер, завершить опрос
  * action "**`answer`**" - сохранить набранный клиентом номер
* "**`press['`**`valueOptions']`" - используется при "action "**playback**""
  * option "**`file`**" - следует указать полный путь к wav файлу
  * option "**`text`**" - текст для возможности генерации Yandex SpeechKit

Пример ответа на запрос:

```
{
    "result": true,
    "data": {
        "id": "7",
        "crmId": 7,
        "name": "New polling"
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 13416
}
```

#### Получение списка опросов

Пример запроса cURL

```
curl 'http://127.0.0.1/pbxcore/api/module-dialer/v1/polling'
```

Пример ответа

```
{
    "result": true,
    "data": {
        "results": [
            {
                "id": "8",
                "crmId": "8",
                "name": "New polling"
            },
            {
                "id": "9",
                "crmId": "9",
                "name": "New polling"
            },
            {
                "id": "10",
                "crmId": "10",
                "name": "\u041e\u043f\u0440\u043e\u0441 \u043e \u043d\u043e\u0432\u043e\u043c \u0442\u043e\u0432\u0430\u0440\u0435"
            }
        ]
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 28941
}
```

#### Получение данных опроса по ID

Пример запроса cURL

```
curl 'http://127.0.0.1/pbxcore/api/module-dialer/v1/polling/7'
```

* где "`7`" - идентификатор опроса.&#x20;

Пример ответа:

```
{
    "result": true,
    "data": {
        "id": "7",
        "crmId": "7",
        "name": "New polling",
        "questions": [
            {
                "id": "79",
                "questionText": "\u0414\u043e\u0431\u0440\u044b\u0439 \u0434\u0435\u043d\u044c, \u0443 \u043d\u0430\u0441 \u0435\u0441\u0442\u044c \u043d\u043e\u0432\u044b\u0439 \u0442\u043e\u0432\u0430\u0440, \u0434\u043b\u044f \u0437\u0430\u043a\u0430\u0437\u0430 \u0442\u043e\u0432\u0430\u0440\u0430 \u043d\u0430\u0436\u043c\u0438\u0442\u0435 1, \u043d\u0435 \u0438\u043d\u0442\u0435\u0440\u0435\u0441\u0443\u0435\u0442 2, \u0434\u043b\u044f \u043e\u0442\u043a\u0430\u0437\u0430 \u043e\u0442 \u043f\u043e\u0434\u043f\u0438\u0441\u043a\u0435 \u043d\u0430\u0436\u043c\u0438\u0442\u0435 3",
                "questionFile": null,
                "lang": "ru-RU",
                "press": [
                    {
                        "key": "1",
                        "action": "playback",
                        "value": "\"\u0421\u043f\u0430\u0441\u0438\u0431\u043e \u0437\u0430 \u0432\u044b\u0431\u043e\u0440 \u044d\u0442\u043e\u0433\u043e \u0442\u043e\u0432\u0430\u0440\u0430\"",
                        "valueOptions": null,
                        "nextQuestion": null
                    },
                    {
                        "key": "2",
                        "action": "playback",
                        "value": "\"\u0421\u043f\u0430\u0441\u0438\u0431\u043e \u0437\u0430 \u0432\u0430\u0448 \u043e\u0442\u0432\u0435\u0442\"",
                        "valueOptions": null,
                        "nextQuestion": null
                    }
                ]
            }
        ]
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 16875
}
```

### Задачи для опроса

Задача позволяет описать все номера телефонов, которые следует оповестить, связать их с  опросом или направить вызов на сотрудника / группу сотрудников.&#x20;

#### Создать опрос

```
JSON="{
  "name": "New pollingtask",
  "state": 0,
  "innerNum": "9",
  "innerNumType": "polling",
  "maxCountChannels": 1,
  "dialPrefix": "999",
  "numbers": [
    {
      "number": "79522233446"
    },
    {
      "number": "79522233416"
    }
  ]
}";
curl -X POST -d "$JSON" http://127.0.0.1/pbxcore/api/module-dialer/v1/task
```

* «`name`» - наименование задачи, можно оставить пустым&#x20;
* «`state`» - состояние / статус
  * `0` - открыт / разрешен
  * `1` - закрыт / завершен
  * `2` - пауза
* «`id`» и «`crmId`» - идентификатор обзвона, если пустой, то будет создан новый
* «`innerNumType`» - если «polling» - направить обзвон на «опрос»
* «`innerNum`» - идентификатор опроса или внутренний номер очереди / сотрудника
* «`maxCountChannels`» - максимальное количество допустимых одновременных звонков в рамках задачи
* «`dialPrefix`» - перед набором, к номеру телефона будет добавлен этот префикс. Опция позволяет выбрать нужный исходящий маршрут звонка
* «`numbers`» - массив номеров телефонов, которые следует опросить

Пример ответа:

```
{
    "result": true,
    "data": {
        "id": "41",
        "crmId": "41",
        "name": "New pollingtask",
        "innerNum": "9",
        "innerNumType": "polling",
        "maxCountChannels": 1,
        "state": 0,
        "dialPrefix": "999"
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 2593
}
```

#### Получение данных задачи и результатов

```
curl -X GET http://127.0.0.1/pbxcore/api/module-dialer/v1/task/36
```

где "`36`" - идентификатор задачи

пример ответа:

```
{
    "result": true,
    "data": {
        "id": "41",
        "crmId": "41",
        "name": "New pollingtask",
        "innerNum": "9",
        "innerNumType": "polling",
        "maxCountChannels": "1",
        "state": "1",
        "dialPrefix": "999",
        "results": [
            {
                "id": "132",
                "taskId": "41",
                "phoneId": "9522233446",
                "phone": "79522233446",
                "linkedId": "mikopbx-1724235065.54",
                "callFile": "dialer-41-79522233446-9.call",
                "result": "SUCCESS_POLLING",
                "outDialState": null,
                "inDialState": null,
                "state": "EVENT_POLLING_END",
                "verboseCallId": "[C-0000001b]",
                "cause": null,
                "params": "s:0:\"\";",
                "changeTime": "1724235073",
                "countTry": null,
                "timeCallAllow": "0",
                "closeTime": "1724235073"
            }
        ],
        "resultsPoling": [
            {
                "id": "45",
                "taskId": "41",
                "questionCrmId": "0",
                "pollingId": "9",
                "phoneId": "9522233446",
                "phone": "79522233446",
                "result": "1",
                "exten": "1",
                "changeTime": "1724235071.0536"
            }
        ]
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 2514
}
```

* "`results`"  - результаты и статус звонка
  * "`phoneId`" - последние 10 цифр номера клиента, префиксы удалены
  * "`linkedId`" - идентификатор звонка, по нему можно отфильтровать историю звонков&#x20;
  * "`verboseCallId`" - идентификатор verbose лога&#x20;
  * "`changeTime`" - дата модификации записи
  * "`result`" - результат звонка&#x20;
* "`resultsPoling`" - результат опроса
  * "`questionCrmId`" - идентификатор вопроса
  * "`pollingId`" - идентификатор опроса
  * "`phoneId`" - последние 10 цифр номера клиента, префиксы удалены
  * "`result`"  - результат ввода
  * "`exten`" - введенный номер в тональном режиме
  * "`changeTime`" - дата модификации записи

Расшифровка статусов (`result`) звонка

* «`FAIL_USER_NO_ANSWER`» - До клиента дозвонились, пользователь не ответил, к примеру завершил вызов
* «`FAIL_USER_BUSY`» - До клиента дозвонились, пользователи заняты или не доступны
* «`FAIL_CLIENT_H_BEFORE_ANSWER`» - До клиента дозвонились, клиент положил трубку до ответа сотрудника (не дождался)
* «`FAIL_ROUTE`» - До клиента не дозвонились, нет маршрута для звонка по номеру клиента, нет провайдера на MIkoPBX
* «`FAIL_PROVIDER`» - До клиента не дозвонились, провайдер не доступен, не активен, ошибка регистрации
* «`SUCCESS_USER_H`» -  Успешный звонок, трубку положил сотрудник
* «`SUCCESS_CLIENT_H`» -  Успешный звонок, трубку положил клиент
* «`SUCCESS_POLLING`» - Опрос пройден успешно.
* «`FAIL_POLLING`» - Опрос не пройден
* «`SUCCESS`» - Успешный звонок
* «`FAIL`» - Безуспешный вызов, проблема не классифицирована

### Получение статусов

#### Получение статуса звонков

```
curl -X GET http://127.0.0.1/pbxcore/api/module-dialer/v1/results/{changeTime}
```

где "`changeTime`" - timestamp, возвращает данные начиная с указанной даты

```
{
    "result": true,
    "data": {
        "results": [
            {
                "id": "134",
                "taskId": "1000000001",
                "phoneId": "9522233446",
                "phone": "79522233446",
                "linkedId": "mikopbx-1724323615.3",
                "callFile": "dialer-1000000001-79522233446-10.call",
                "result": "SUCCESS_POLLING",
                "outDialState": null,
                "inDialState": null,
                "state": "EVENT_POLLING_END",
                "verboseCallId": "[C-00000003]",
                "cause": null,
                "params": "s:0:\"\";",
                "changeTime": "1724323625",
                "countTry": null,
                "timeCallAllow": "0",
                "closeTime": "1724323625"
            },
            {
                "id": "135",
                "taskId": "1000000002",
                "phoneId": "9522233446",
                "phone": "79522233446",
                "linkedId": "mikopbx-1724324361.6",
                "callFile": "dialer-1000000002-79522233446-10.call",
                "result": "SUCCESS_POLLING",
                "outDialState": null,
                "inDialState": null,
                "state": "EVENT_POLLING_END",
                "verboseCallId": "[C-00000005]",
                "cause": null,
                "params": "s:0:\"\";",
                "changeTime": "1724324376",
                "countTry": null,
                "timeCallAllow": "0",
                "closeTime": "1724324376"
            }
        ]
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 28941
}
```

* "`results`"  - результаты и статус звонка
  * "`phoneId`" - последние 10 цифр номера клиента, префиксы удалены
  * "`linkedId`" - идентификатор звонка, по нему можно отфильтровать историю звонков&#x20;
  * "`verboseCallId`" - идентификатор verbose лога&#x20;
  * "`changeTime`" - дата модификации записи
  * "`result`" - результат звонка&#x20;

#### Получение статуса опросов

Пример запроса cURL

```
curl -X GET http://127.0.0.1/pbxcore/api/module-dialer/v1/polling-results/{changeTime}
```

где "`changeTime`" - timestamp, возвращает данные начиная с указанной даты

Пример ответа:

```
{
    "result": true,
    "data": {
        "results": [
            {
                "id": "48",
                "taskId": "1000000002",
                "questionCrmId": "0",
                "pollingId": "10",
                "phoneId": "9522233446",
                "phone": "79522233446",
                "result": "3",
                "exten": "3",
                "changeTime": "1724324374.2487"
            }
        ]
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 28941
}
```

* "`results`" - результат опроса
  * "`questionCrmId`" - идентификатор вопроса
  * "`pollingId`" - идентификатор опроса
  * "`phoneId`" - последние 10 цифр номера клиента, префиксы удалены
  * "`result`"  - результат ввода
  * "`exten`" - введенный номер в тональном режиме
  * "`changeTime`" - дата модификации записи

### Медиа файлы

#### Заранее подготовленные медиа файлы могут быть использованы в опросах для оповещения клиентов.&#x20;

#### Загрузка файла на АТС

Рекомендованные параметры аудио файла mp3

```
Channels       : 1
Sample Rate    : 8000
Precision      : 16-bit
Bit Rate       : 16.0k
Sample Encoding: MPEG audio (layer I, II or III)
```

Пример загрузки файла на АТС средствами cURL

```
curl -F "file=@1.mp3" 'http://127.0.0.1/pbxcore/api/module-dialer/v1/audio'
```

Имя файла "1.mp3" будет являться его идентификатором. При загрузке нового файла с прежним именем старый файл будет заменен на новый.&#x20;

Пример ответа:

```
{
    "result": true,
    "data": {
        "filename": "\/storage\/usbdisk1\/mikopbx\/custom_modules\/ModuleAutoDialer\/db\/audio\/474d57f8204699fbb2249f20dede5f8e.mp3"
    },
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 10884
}
```

* «`result`» - результат операции, булево
* «`filename`» - полный путь к загруженному файлу, его можно использовать при создании «опроса»&#x20;

#### Получение списка файлов

Пример загрузки файла на АТС средствами cURL

```
curl 'http://127.0.0.1/pbxcore/api/module-dialer/v1/audio'
```

Пример ответа:

```
{
    "result": true,
    "data": [
        {
            "id": "1",
            "name": "1.mp3",
            "path": "\/storage\/usbdisk1\/mikopbx\/custom_modules\/ModuleAutoDialer\/db\/audio\/474d57f8204699fbb2249f20dede5f8e.mp3"
        }
    ],
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 27195
}
```

#### Удаление аудиофайла

Пример загрузки файла на АТС средствами cURL

```
curl -X DELETE 'http://127.0.0.1/pbxcore/api/module-dialer/v1/audio/1.mp3'
```

Пример ответа

```
{
    "result": true,
    "data": [],
    "messages": [],
    "function": "",
    "processor": "",
    "pid": 22226
}
```
