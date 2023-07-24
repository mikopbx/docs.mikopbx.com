# Динамические агенты очередей

Периодически возникает необходимость добавлять и удалять сотрудников из очереди. До сих пор это было возможно сделать только через web интерфейс телефонной станции.

В текущей статье я расскажу как реализовать возможность подключиться и отключиться из очереди средствами набора служебного внутреннего номера.

1. Создайте новую очередь с четырехзначным внутренним номером. К примеру **2001**. (см. документацию «[Очереди вызовов](../../manual/telefoniya/call-queues.md)»)
2. Опишите новое «**Приложение диалпланов**»

<figure><img src="../../.gitbook/assets/6 (18).png" alt=""><figcaption></figcaption></figure>

3. Назначьте «**Номер для вызова приложения**» - **999XXXXX** первые три цифры можете переопределить своей комбинацией

&#x20;      Тип кода - «**PHP AGI скрипт**»

<figure><img src="../../.gitbook/assets/7 (16).png" alt=""><figcaption></figcaption></figure>

4. На вкладке «**Программный код**» вставьте следующее содержимое:

```php
<?php

use MikoPBX\Common\Models\CallDetailRecordsTmp;
use MikoPBX\Common\Models\CallQueueMembers;
use MikoPBX\Common\Models\CallQueues;
use MikoPBX\Common\Models\Extensions;
use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\Asterisk\Configs\QueueConf;

require_once 'Globals.php';

$agi = new AGI();
$agi->answer();

$extension = $agi->get_variable("CHANNEL(peername)", true);
if(empty($extension)){
    $extension = $agi->get_variable("CALLERID(num)", true);
}
$q_exten   = substr($agi->request['agi_extension'], 4);
$add_agent = substr($agi->request['agi_extension'], 3, 1);

$agi->noop("$extension $q_exten $add_agent");

$res = Extensions::findFirst("number='{$extension}'");
if(!$res){
    // Ошибка, такого внутреннего номера не существует.
    $agi->hangup();
}
$res = CallQueues::findFirst("extension='{$q_exten}'");
if(!$res){
    // Ошибка, такой очереди не существует.
    $agi->hangup();
}
$a_count = CallQueueMembers::maximum([ "queue='{$res->uniqid}'", 'column' => 'priority'] ) + 1;
$member  = CallQueueMembers::findFirst("queue='{$res->uniqid}' AND extension='{$extension}'");

if($add_agent === '1' && !$member){
    $member = new CallQueueMembers();
    $member->extension  = $extension;
    $member->queue      = $res->uniqid;
    $member->priority   = $a_count;
    $member->save();
    QueueConf::queueReload();
}

if($add_agent == false && $member){
    $member->delete();
    QueueConf::queueReload();
}

$linkedid = $agi->get_variable("CDR(linkedid)", true);
$res = CallDetailRecordsTmp::find("linkedid='{$linkedid}'");
foreach ($res as $data){
    $data->delete();
}
sleep(2);
$agi->hangup();
```

Нажмите "**Сохранить**"

<figure><img src="../../.gitbook/assets/8 (17).png" alt=""><figcaption></figcaption></figure>

Приложение готово!&#x20;

{% hint style="info" %}
1. Наберите комбинацию 9991**2001** чтобы присоединиться к очереди 2001
2. Наберите комбинацию 9990**2001** чтобы отключиться от очереди 2001
{% endhint %}
