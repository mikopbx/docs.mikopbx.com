# Callback: перезваниваем клиенту и заводим в IVR

### Введение <a href="#vvedenie" id="vvedenie"></a>

Сценарий «callback» (обратный звонок) полезен, когда входящая связь для абонента дорогая (международные или межгород звонки, роуминг) либо когда нужно, чтобы за соединение платила компания, а не клиент. Как только клиент дозванивается, MikoPBX **сбрасывает входящий вызов и сама перезванивает** абоненту, а после ответа заводит его в IVR (голосовое меню) — дальше звонок идёт обычным внутренним маршрутом.

{% hint style="info" %}
Это не то же самое, что модуль [«Автообработка пропущенных»](../../modules/miko/module-callback-queues.md): тот фиксирует _пропущенный_ звонок и заставляет _сотрудника_ перезвонить клиенту. Здесь же перезвон выполняется **сразу и автоматически**, без участия оператора, с заводом абонента в IVR.
{% endhint %}

Реализуется связкой из двух частей: кастомный контекст dialplan в `extensions.conf` (обёртка с паузой перед набором) и приложение dialplan на PHP-AGI, которое ставит обратный звонок.

### 1. Добавьте кастомный контекст в extensions.conf <a href="#custom_context" id="custom_context"></a>

Перейдите в раздел «[**Кастомизация системных файлов**](../../manual/system/custom-files.md)», откройте на редактирование файл **extensions.conf** и добавьте в конец контекст-обёртку:

```php
[callback-originate]
exten => _[+0-9].,1,Wait(4)
 same => n,Goto(outgoing,${EXTEN},1)
```

Контекст выжидает 4 секунды (чтобы входящая нога успела корректно завершиться), а затем отправляет вызов в контекст `outgoing` — то есть набирает номер абонента по вашим **исходящим маршрутам**.

### 2. Создайте приложение dialplan <a href="#dialplan_app" id="dialplan_app"></a>

Добавьте новое приложение dialplan (см. [**Приложения диалпланов**](../../manual/modules/dialplan-applications.md)), тип кода — «**PHP-AGI скрипт**», и вставьте код:

```php
<?php
require_once 'Globals.php';

use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;
use MikoPBX\Core\System\SystemMessages;

const CALLBACK_WHITELIST    = [];                   // пусто = для всех
const CALLBACK_IVR_EXTEN    = '2003';               // номер IVR в MikoPBX
const CALLBACK_IVR_CONTEXT  = 'internal';
const CALLBACK_ORIG_CONTEXT = 'callback-originate'; // контекст-обёртка с Wait
const CALLBACK_TIMEOUT      = 60000;                // мс: Wait(4) + дозвон

$agi    = new AGI();
$raw    = (string)$agi->get_variable('CALLERID(num)', true);
$caller = preg_replace('/\D/', '', $raw);

if ($caller === '') {
    SystemMessages::sysLogMsg('Callback', "Empty CID ($raw)", LOG_WARNING);
    return;
}
if (CALLBACK_WHITELIST !== [] && !in_array($caller, CALLBACK_WHITELIST, true)) {
    return; // не наш CID — звонок пойдёт обычным маршрутом
}

// 1. Ставим обратный звонок (AMI-экшен живёт в ядре, независимо от AGI).
try {
    $am = Util::getAstManager('off');
    $am->Originate(
        'Local/' . $caller . '@' . CALLBACK_ORIG_CONTEXT . '/n',
        CALLBACK_IVR_EXTEN,
        CALLBACK_IVR_CONTEXT,
        1,
        null, null,
        CALLBACK_TIMEOUT,
        $caller,
        '__ISCALLBACK=1',
        null,
        true
    );
    SystemMessages::sysLogMsg('Callback', "Scheduled callback $caller -> IVR " . CALLBACK_IVR_EXTEN, LOG_NOTICE);
} catch (\Throwable $e) {
    SystemMessages::sysLogMsg('Callback', 'Originate failed: ' . $e->getMessage(), LOG_ERR);
    return; // не роняем входящий, если originate не ушёл
}

// 2. Завершаем входящий штатным путём MikoPBX (сам выставит M_DIALSTATUS и Hangup).
$agi->exec_goto('internal', 'hangup', '1');
```

### 3. Настройте константы <a href="#nastrojka" id="nastrojka"></a>

| Константа | Назначение |
| --- | --- |
| `CALLBACK_WHITELIST` | Белый список входящих CID, кому доступен callback. Пустой массив — перезваниваем **всем** дозвонившимся |
| `CALLBACK_IVR_EXTEN` | Номер IVR (или другого назначения), куда попадёт абонент после ответа на обратный звонок |
| `CALLBACK_IVR_CONTEXT` | Контекст назначения. Для внутренних номеров и IVR — `internal` |
| `CALLBACK_ORIG_CONTEXT` | Имя кастомного контекста-обёртки из шага 1 |
| `CALLBACK_TIMEOUT` | Таймаут (мс) на дозвон до абонента, с запасом на `Wait(4)` |

### 4. Направьте входящий вызов на приложение <a href="#vhodyashhij_marshrut" id="vhodyashhij_marshrut"></a>

В разделе «[**Входящие маршруты**](../../manual/routing/incoming-routing.md)» укажите в качестве назначения (для нужного провайдера или DID) созданное приложение dialplan. После этого каждый такой входящий будет сбрасываться и перезваниваться автоматически.

### Важные моменты <a href="#vazhnye_momenty" id="vazhnye_momenty"></a>

1. Обратный звонок уходит по вашим исходящим маршрутам — **за него платит компания**. Ограничьте доступ через `CALLBACK_WHITELIST`, иначе callback можно использовать для бесплатных звонков за ваш счёт.
2. Номер абонента берётся из `CALLERID(num)` и чистится от нецифровых символов. Он должен быть в формате, который принимают ваши исходящие маршруты и [шаблоны номеров](../outbound-routing/number-templates/normalization.md); при необходимости нормализуйте номер в контексте `[callback-originate]`.
3. IVR с номером `CALLBACK_IVR_EXTEN` (в примере `2003`) должен существовать в MikoPBX (см. [Базовый пример IVR](basic-ivr-example.md)).
4. Защита от зацикливания: обратный вызов помечается переменной `__ISCALLBACK=1` — используйте её в кастомном dialplan, если нужно исключить повторный заход в скрипт при обратном звонке.
5. Все события пишутся через `SystemMessages::sysLogMsg('Callback', ...)` — ищите по тегу `Callback` в системном журнале.
6. **Скрипт не является завершённым продуктом**, но открыт для кастомизации.
