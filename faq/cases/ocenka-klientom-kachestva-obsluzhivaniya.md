# Оценка клиентом качества обслуживания

Совершенствование качества сервиса организации - крайне важная составляющая успешной работы компании. Механизм оценки работы оператора будет полезен любому бизнесу.

В текущей статье предлагаем пример реализации механизма оценки качества клиентом для АТС MikoPBX.

### Доработка Dialplan <a href="#dorabotka_dialplan" id="dorabotka_dialplan"></a>

{% hint style="warning" %}
На АТС предварительно следует закачать файлы записей вопросов / приветствия. Формат записей WAV (Microsoft) signed 16-bit PCM.\
Файлы можно для удобства разместить в каталоге: **/storage/usbdisk1/quality/sounds**.\
В dialplan путь к файлу следует указывать **без** расширения.
{% endhint %}

Для начала дополним логику обработки входящих звонков:

1\. Перейдите в раздел **Маршрутизация** → **Провайдеры телефонии**. Откройте для редактирования учетную запись провайдера для редактирования. Скопируйте в адресной строке **ID провайдера**, через которого абоненты звонят Вам в компанию. Обращаем Ваше внимание, что в нашем примере используется **единственный** провайдер Zadarma. Если у Вас настроено подключение **нескольких** провайдеров, то ниже описанные действия необходимо выполнить **для каждого провайдера**.

В нашем примере ID провайдера принимает вид: **SIP-1687947415**

<figure><img src="../../.gitbook/assets/1 (22).png" alt=""><figcaption></figcaption></figure>

2. Перейдите в раздел **Система** → **Кастомизация системных файлов**.

<figure><img src="../../.gitbook/assets/2 (23).png" alt=""><figcaption></figcaption></figure>

3. Откройте для редактирования конфигурационный файл **extensions.conf**. &#x20;

<figure><img src="../../.gitbook/assets/3 (11).png" alt=""><figcaption></figcaption></figure>

4. Установите режим «**Добавлять в конец файла**».

<figure><img src="../../.gitbook/assets/4 (34).png" alt=""><figcaption></figcaption></figure>

5. В черное окно добавьте следующий фрагмент кода:

```
[SIP-PROVIDER-SIP-1687947415-after-dial-custom]
exten => _.!,1,Goto(quality-start,s,1)
	same => n,return

[SIP-PROVIDER-SIP-1687947415-outgoing-custom]
exten => _.!,1,Set(DOPTIONS=${DOPTIONS}F(quality-start,s,1))
	same => n,return

[quality-start]
exten => _.!,1,NoOp(--- Quality assessment ---)
	same => n,ExecIf($[${M_DIALSTATUS}!=ANSWER]?return)
	;Описываем путь к файлу завершения разговора. Благодарим за оценку качества. 
	same => n,Set(filename_bye=/storage/usbdisk1/quality/sounds/bye)
	; Описываем пути к файлам опроса. 
	same => n,Set(filename_1=/storage/usbdisk1/quality/sounds/miko_hello)
	same => n,Set(filename_2=/storage/usbdisk1/quality/sounds/q_1)
	same => n,Set(filename_3=/storage/usbdisk1/quality/sounds/q_2)
	same => n,Set(f_num=0);
	same => n,Goto(ivr-quality,s,1)

[ivr-quality]
exten => s,1,NoOP( start ivr quality )
	same => n,Set(f_num=$[${f_num} + 1])
 	same => n,Set(filename=${filename_${f_num}})
	same => n,GotoIf($["x${filename}" == "x"]?ivr-quality,bye,1);
	same => n,Background(${filename})
	same => n,WaitExten(5)

exten => _[1-5],1,NoOP( quality is ${EXTEN})
	; AGI скрипт, который сохранит результат оценки разговора клиентом.
	same => n,AGI(/storage/usbdisk1/quality/quality_agi.php)
	same => n,Goto(ivr-quality,s,1)

exten => _[06-9],Goto(ivr-quality,s,1)

exten => bye,1,ExecIf($["x${filename_bye}" != "x"]?Playback(${filename_bye}));
	same => n,Hangup()
```

В выше приведенном фрагменте кода Вам необходимо составить правильное наименование контекста.\
Формат создаваемого контекста:

```
[ID-ПРОВАЙДЕРА-outgoing-custom]
[ID-ПРОВАЙДЕРА-after-dial-custom]
```

* **ID-ПРОВАЙДЕРА** - значение, которое вы сохранили на первом шаге данной инструкции. В нашем примере это **SIP-PROVIDER-**1687947415.

### Скрипт обработки результата оценки <a href="#skript_obrabotki_rezultata_ocenki" id="skript_obrabotki_rezultata_ocenki"></a>

{% hint style="info" %}
Файл следует сохранить на АТС по пути **/storage/usbdisk1/quality/quality\_agi.php**\
Создать файл можно используя файловый менеджер [WinSCP](../troubleshooting/podklyuchenie-k-ats-s-pomoshyu-winscp.md)\
Файлу следует установить права доступа на выполнение. Подключитесь по [SSH](../troubleshooting/podklyuchenie-k-ats-s-pomoshyu-ssh-klienta.md) и выполните команду

```
chmod +x /storage/usbdisk1/quality/quality_agi.php
```
{% endhint %}

Пример реализации quality\_agi.php:

```php
#!/usr/bin/php
<?php
require_once 'phpagi.php';
require_once 'globals.php';

$agi 		= new AGI();
$linkedid   = $agi->get_variable('CDR(linkedid)', true);

$arr = [
    'quality'   => $agi->request['agi_extension'],
    'f_num'     => $agi->get_variable('f_num', true),
    'filename'  => $agi->get_variable('filename', true),
    // 'Оцененный оператор:' => $operator,
    'linkedid'  => $linkedid,
    'date'      => date('Y-m-d H:i:s'),
    'callerid'  => $agi->request['agi_callerid']
];

$file_log = '/storage/usbdisk1/quality/'.$linkedid.'.log';
Util::mw_mkdir(dirname($file_log));
file_put_contents($file_log, json_encode($arr)."\n", FILE_APPEND);
```

{% hint style="info" %}
Результат оценки будет сохранятся в файлы вида **/storage/usbdisk1/quality/\<ID>.log**
{% endhint %}

Пример файла:

{% code fullWidth="false" %}
```json
{"quality":"1","f_num":"1","filename":"\/offload\/asterisk\/sounds\/other\/miko_hello","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:13","callerid":"79265775289"}
{"quality":"2","f_num":"2","filename":"beep","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:15","callerid":"79265775289"}
{"quality":"3","f_num":"3","filename":"\/offload\/asterisk\/sounds\/other\/out_work_times","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:16","callerid":"79265775289"}
```
{% endcode %}
