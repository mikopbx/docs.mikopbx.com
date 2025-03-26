---
description: >-
  В данной статье будет рассмотрено два способа реализации уведомления о
  пропущеных вызовах в Telegram
---

# Уведомление в телеграмм о пропущенных

## Пример на базе Dialplan <a href="#primer_na_baze_dialplan" id="primer_na_baze_dialplan"></a>

{% hint style="success" %}
[Полезная статья](https://gist.github.com/dideler/85de4d64f66c1966788c1b2304b9caf1) по работе с Telegram-ботом средствами **curl.**
{% endhint %}

1. Перейдите в раздел "**Кастомизация системных файлов**":

<figure><img src="../../.gitbook/assets/systemFileCustomization.png" alt=""><figcaption><p>Раздел "<strong>Кастомизация системных файлов</strong>"</p></figcaption></figure>

2. Перейдите в раздел редактирования файла "**extensions.conf**". Установите режим "**Добавлять в конец файла**" и вставьте следующий контекст:

```php
[add-trim-prefix-clid-custom]
exten => _[0-9*#+a-zA-Z][0-9*#+a-zA-Z]!,1,NoOp(start check blacklist)
	same => n,Set(CHANNEL(hangup_handler_push)=hangup-ext-queues,h,1);
	same => n,Return()

[hangup-ext-queues]
exten => h,1,ExecIf($["${M_DIALSTATUS}" = "ANSWER"]?return)
    same => n,Set(TOKEN=5118292900:AAEWCOAXkay5fXb8AJptZmDyqkNk8QbP200)
    same => n,Set(CHAT_ID=939950800)
    same => n,Set(URL=https://api.telegram.org/bot${TOKEN}/sendMessage)
    same => n,Set(TEXT=MISSED CALL from: ${CALLERID(name)}, did: ${FROM_DID}, callid: ${CHANNEL(callid)})
    same => n,SHELL(curl -s -X POST '${URL}' -d chat_id='${CHAT_ID}' -d text='${TEXT}')
    same => n,Set(MISSED=${SHELL(curl -s -X POST '${URL}' -d chat_id='${CHAT_ID}' -d text='${TEXT}')})
    same => n,return
```

В данном конктексте замените:

* **TOKEN** - токен вашего бота в телеграмм.
* **CHAT\_ID** - идентификатор чата, куда отправлять текстовое сообщение.

Сохраните изменения.

<figure><img src="../../.gitbook/assets/extensionsFileCode.jpg" alt=""><figcaption><p>Редактирование файла "extensions.conf"</p></figcaption></figure>

3. Перейдите в раздел редактирования файла "**modules.conf**". Установите режим "**Добавлять в конец файла**" и вставьте следующий контекст:

```
load => func_shell.so
```

Сохраните изменения.

<figure><img src="../../.gitbook/assets/modulesFileCode.jpg" alt=""><figcaption><p>Редактирование файла "modules.conf"</p></figcaption></figure>

{% hint style="success" %}
Средставми **curl** можно выполнить запрос к любому сайту. К примеру можно отправить уведомление в **slack**:

{% code overflow="wrap" fullWidth="false" %}
```php
same => n,Set(MISSED=${SHELL(curl -X POST --data-urlencode "payload={\"channel\": \"#cannel_name\", \"username\": \"bot_name\", \"text\": \"Пропущенный вызов от ${CALLERID(name)} по внешней линии: ${FROM_DID} в ${STRFTIME(${EPOCH},,%H:%M:%S %d-%m-%Y)}\", \"icon_emoji\": \":sos:\"}" https://hooks.slack.com/services/T76G7L0/B01R/VMPQUeAN)}) 
```
{% endcode %}
{% endhint %}

## Пример на базе PHP-AGI <a href="#primer_na_baze_php-agi" id="primer_na_baze_php-agi"></a>

1. Перейдите в раздел "**Приложения диалпланов**". Создайте новое приложение, нажав на "**Добавить новое**":

<figure><img src="../../.gitbook/assets/addNewDialplanApp.jpg" alt=""><figcaption><p>Добавление нового приложения диалплана</p></figcaption></figure>

2. Укажите следующие параметры для диалплана:

* **Название** - произвольное
* **Номер для вызова приложения** - произвольный номер
* **Тип кода** - "_PHP-AGI скрипт_"

<figure><img src="../../.gitbook/assets/dialplanParameters.jpg" alt=""><figcaption><p>Параметры диалплана</p></figcaption></figure>

2. Перейдите во вкладку "**Программный код**". Вставьте следующий PHP-AGI скрипт:

{% code overflow="wrap" fullWidth="false" %}
```php
<?php
require_once 'Globals.php';
use \GuzzleHttp\Client;

const API_KEY = '';
const CHAT_ID = '';

$agi = new MikoPBX\Core\Asterisk\AGI();

$name = $agi->get_variable('CALLERID(name)', true);
$num  = $agi->get_variable('CALLERID(num)', true);
$did  = $agi->get_variable('$FROM_DID', true);
$id   = $agi->get_variable('CHANNEL(linkedid)', true);
$date = date('Y.d.m H:i:s', str_replace('mikopbx-', '', $id));

$TEXT = "Пропущенный вызов: $name, did: $did, callid: $num, id: $id, date: $date";
$apiURL = 'https://api.telegram.org/bot' . API_KEY . '/';
$client = new Client([
    'base_uri' => $apiURL,
    'timeout' => 1,
    'http_errors' => false,
]);
try {
    $client->post( 'sendMessage', ['query' => ['chat_id' => CHAT_ID, 'text' => $TEXT]] );
}catch (Throwable $e){
}
```
{% endcode %}

В данном конктексте замените:

* **API\_KEY** - токен вашего бота в телеграмм.
* **CHAT\_ID** - идентификатор чата, куда отправлять текстовое сообщение.

Текст уведомления можно исправить в переменной «$**TEXT**».

3. После сохранения скрипта из адресной строки браузера скопируйте идентификатор скрипта, который имеет вид: «**DIALPLAN-APP-1B2B846E**»:

<figure><img src="../../.gitbook/assets/dialplanID.jpg" alt=""><figcaption><p>Идентификатор приложения диалплана</p></figcaption></figure>

4. Перейдите в раздел "**Кастомизация системных файлов**":

<figure><img src="../../.gitbook/assets/systemFileCustomization.png" alt=""><figcaption><p>Раздел "<strong>Кастомизация системных файлов</strong>"</p></figcaption></figure>

5. Перейдите в раздел редактирования файла "**extensions.conf**". Установите режим "**Добавлять в конец файла**" и вставьте следующий контекст:

```php
[add-trim-prefix-clid-custom]
exten => _.X!,1,Set(CHANNEL(hangup_handler_push)=hangup-ext-queues,h,1);
	same => n,return
[hangup-ext-queues]
exten => h,1,ExecIf($["${M_DIALSTATUS}" = "ANSWER"]?return)
    same => n,AGI(DIALPLAN-APP-1B2B846E.php)
    same => n,return
```

{% hint style="info" %}
Замените "**DIALPLAN-APP-1B2B846E"** на Ваш идентификатор провайдера.
{% endhint %}

Сохраните изменения.

<figure><img src="../../.gitbook/assets/extensionsFileCode2.jpg" alt=""><figcaption><p>Редактирование файла "extensions.conf"</p></figcaption></figure>
