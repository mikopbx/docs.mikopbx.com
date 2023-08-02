# Добавить P-Preferred-Identity и Remote-Party-ID заголовок

Некоторые провайдеры при инициации исходящего звонка требуют отправки дополнительного SIP заголовка. В этой статье опишем способ его установить.

1. Для решения задачи следует описать дополнительные контексты через меню [Кастомизация системных файлов](../../manual/system/custom-files.md).&#x20;

<figure><img src="../../.gitbook/assets/1 (57).png" alt="Раздел &#x22;Кастомизация системных файлов&#x22;"><figcaption><p>Раздел "Кастомизация системных файлов"</p></figcaption></figure>

2. Править будем файл **extensions.conf**.&#x20;

<figure><img src="../../.gitbook/assets/2 (43).png" alt=""><figcaption><p>Файл "extensions.conf"</p></figcaption></figure>

3. Добавьте в конец файла следующий текст:

```php
[SIP-1611151795-outgoing-custom]
exten => _X!,1,Dial(PJSIP/${number}@SIP-${CUT(CONTEXT,-,2)},600,${DOPTIONS}TKU(dial_answer)b(dial_create_chan_${CUT(CONTEXT,-,2)}_custom,s,1))
    same => n,ExecIf($["${DIALSTATUS}" = "ANSWER"]?Hangup())
    same => n,ExecIf($["${DIALSTATUS}" = "BUSY"]?Busy(2))
    same => n,return
    
[dial_create_chan_1611151795_custom] 
exten => s,1,Gosub(lua_${ISTRANSFER}dial_create_chan,${EXTEN},1)
    same => n,Set(pt1c_is_dst=1) 
    same => n,Set(OUTGOING_CID=74952293042)
    same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,P-Preferred-Identit)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,Remote-Party-ID)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,Set(__PT1C_SIP_HEADER=${UNDEFINED}) 
    same => n,Set(CHANNEL(hangup_handler_wipe)=hangup_handler,s,1) 
    same => n,return
```

<figure><img src="../../.gitbook/assets/3 (38).png" alt=""><figcaption><p>Добавление кода в конец файла</p></figcaption></figure>

{% hint style="danger" %}
**Обратите внимание**:

1. Все вхождения строки «**SIP-1611151795**» следует заменить на ID провайдера «**найти**» можно в адресной строке браузера при редактировании учетной записи в web интерфейсе MIKOPBX
2. Вместо «**\<sip:${OUTGOING\_CID}@127.0.0.1>**» следует задать требуемое значение заголовков
{% endhint %}

#### При использовании модуля «Группы пользователей»:

```php
[SIP-1611151795-outgoing-ug-custom]
exten => _X!,1,Dial(PJSIP/${number}@SIP-${CUT(CONTEXT,-,2)},600,${DOPTIONS}TKU(dial_answer)b(dial_create_chan_custom,s,1))
	same => n,ExecIf($["${DIALSTATUS}" = "ANSWER"]?Hangup())
	same => n,ExecIf($["${DIALSTATUS}" = "BUSY"]?Busy(2))
    same => n,return
    
[dial_create_chan_custom] 
exten => s,1,Gosub(lua_${ISTRANSFER}dial_create_chan,${EXTEN},1)
    same => n,Set(pt1c_is_dst=1) 
    same => n,Set(GR_VARS=${DB(UsersGroups/${FROM_PEER})}) 
    same => n,Set(tmpName=${CUT(GR_VARS,\,,2)})
    same => n,Set(tmpName=${CUT(tmpName,\_,3)})
	same => n,ExecIf($["${GR_VARS}x" != "x"]?Exec(Set(${GR_VARS}))) 
	same => n,ExecIf($["${GR_PERM_ENABLE}" == "1" && "${GR_ID_${tmpName}}" != "1"]?return) 
	same => n,ExecIf($["${GR_PERM_ENABLE}" == "1" && "${GR_CID_${tmpName}}x" != "x"]?MSet(GR_OLD_CALLERID=${CALLERID(num)},OUTGOING_CID=${GR_CID_${tmpName}}))
	same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,P-Preferred-Identit)=<sip:${OUTGOING_CID}@127.0.0.1>))
	same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,Remote-Party-ID)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,Set(__PT1C_SIP_HEADER=${UNDEFINED}) 
    same => n,Set(CHANNEL(hangup_handler_wipe)=hangup_handler,s,1) 
    same => n,return
```

{% hint style="warning" %}
Особенность этого варианта в том, что значение «OUTGOING\_CID» будет браться из настроек групп пользователей. Для каждой группы можно назначить свое значение исходящего callerid. К примеру провайдер Westcall таким образом позволяет управлять значением callerid, который видит клиент.
{% endhint %}
