# MikoPBX 2023.1.223

### **PBX Update Mechanism**

We have fundamentally redesigned the approach to updating through the **img** file. This update method was triggered in the following cases:

* Online update
* Update using a local **img** file

Previously, the update was performed according to the following algorithm:

1. Uploading the img file to the PBX
2. Termination of active processes
3. **Unmounting the disks**
4. Writing the update file to the disk (dd)

Often, issues arose during the **3rd** step. The system refused to unmount the disk, even if it was no longer utilizing its resources. As a result, problems occurred, and data could be lost.

In **release 2022.3.15**, a mechanism for creating a preliminary "snapshot" of the system partitions was added, allowing the system to be restored from a backup.

In **release 2023.1.223**, we changed the algorithm as follows:

1. Uploading the img file to the PBX
2. Restarting the PBX
3. Snapshot of the system disk
4. Writing the update file to the disk (BEFORE connecting the disks)

In addition, we implemented a mechanism to save a backup copy of the PBX's settings database to the **storage** disk. When modifications to the settings are made, a backup of the database will be created in the following path: `/storage/usbdisk1/mikopbx/backup/db`

The directory will store the last 5 versions:

```
── db
    ├── 2023-04-13_h14_m30_s00_mikopbx.db.gz
    ├── 2023-04-13_h16_m15_s01_mikopbx.db.gz
    ├── 2023-04-17_h13_m15_s00_mikopbx.db.gz
    ├── 2023-04-17_h13_m30_s01_mikopbx.db.gz
    └── 2023-04-17_h15_m25_s00_mikopbx.db.gz
```

This should increase fault tolerance and expand the system recovery capabilities in case of failures.

### **Firewall**

We upgraded **fail2ban** to version **1.0.2**.

The mechanism for blocking IP addresses has been reworked.

In earlier releases, if there were password guessing attempts, access from the IP address was blocked to all PBX ports.

In **release 2023.1.223**, the mechanism was reworked. If password guessing occurs on port 5060, access will only be blocked to 5060+RTP range. Access to the web interface will remain open.

This can be useful when the PBX is in the cloud. We avoid blocking access to the station when an incorrect password is unintentionally entered. It will still be possible to access the web interface and remove the block.

The `iptables` rules have been reworked using `-m multiport`, which makes the output of `iptables -L -n` more understandable.

### **Customization**

We have expanded the possibilities of customizing queues. An example of a queue dialplan:

```clike
exten => 2002,1,NoOp(--- Start Queue ---) 
    same => n,Set(__QUEUE_SRC_CHAN=${CHANNEL})
    same => n,ExecIf($["${CHANNEL(channeltype)}" == "Local"]?Gosub(set_orign_chan,s,1))
    same => n,Set(CHANNEL(hangup_handler_wipe)=hangup_handler,s,1)
    same => n,GosubIf($["${DIALPLAN_EXISTS(queue-pre-dial-custom,${EXTEN},1)}" == "1"]?queue-pre-dial-custom,${EXTEN},1)
    same => n,Answer() 
    same => n,Gosub(queue_start,${EXTEN},1)
    same => n,Queue(QUEUE-D676A,kT${MQ_OPTIONS},,,30,,,queue_agent_answer) 
    same => n,Gosub(queue_end,${EXTEN},1)
```

Now it is possible to define a context through customization of system files:

```clike
[queue-pre-dial-custom]
exten => 2002,1,NoOp() 
same => n,retutn
```

In this context, you can perform arbitrary actions before the call is directed to the queue. Play a media file, set additional channel variables, send an email to the responsible person.

### Call Recording

Several bugs related to call recording and call resumption have been fixed.

Now, in the employee's profile, there is an option to disable call recording:

<figure><img src="https://809364261-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MPK4TuzRBnP7rt8htho-887967055%2Fuploads%2FLmjThVW6Hte7UMTQGMDy%2FChangeLog223CallRecording.png?alt=media&#x26;token=2784dd35-9cef-419e-bc9e-c4545a2159ce" alt=""><figcaption></figcaption></figure>

Все диалоги, в которых будут участвовать внутренний и мобильный номера сотрудника записаны НЕ будут. Это может быть полезно для Директора и прочих руководителей компании.

Появилась возможность отключить запись всех внутренних разговоров. Для этого следует отключить флаг в разделе "**Система**" - "**Общие настройки**" - "**Запись разговоров**" - "**Запись внутренних разговоров**"

### **Нерабочее время**

Новый интерфейс отображения списка нерабочего времени: Он стал более информативным и компактным.

Ранее, чтобы описать нерабочее время с 18:00 по 8:00 было необходимо создать два правила 00:00 - 08:00 и 18:00 - 23:59. Теперь возможно указать одно правило **18:00 - 08:00**:

Это позволит сократить количество правил и навести порядок.

Нас часто просили предоставить возможность настроить разное расписание работы для разных отделов компании. Теперь это стало возможно.

В карточке правила нерабочего времени появилась опция "**Применять только к определенным входящим маршрутам**". Теперь правило можно привязать к конкретным входящим маршрутам. Именно эта опция позволит создавать более гибкие правила, которые будут действовать на конкретных подразделения компании.

Пример:

В данном примере правило будет действовать только для одного номера компании.

### **Анализ проблем со звонками**

В журнал истории звонка добавлена кнопка перехода к логу звонка:

Такой лог можно передать в техническую поддержку для анализа поведения АТС.

Расширены возможности отбора в журнале лога:

Можно ввести несколько подстрок, разделенных символом &, в итоге будет выведен лог, содержащий все перечисленные подстроки.

В сборку asterisk добавлены модули для подключения к [Homer](https://github.com/sipcapture/homer). Модули можно настроить через кастомизацию системных файлов.

### **Docker**

Проведена большая работа над ошибками. Считаю эту версию стабильной. Проверено из под **Ubuntu, Debian**, тут основное требование - **Linux 5** версии.

**Новые инструкции**

* [Настройка Jitter](https://wiki.mikopbx.ru/faq)
* [Мониторинг провайдеров на MikoPBX](https://wiki.mikopbx.ru/faq:monitoring-trunks)
* [Объединение MIKOPBX и FreePBX (PJSIP)](https://wiki.mikopbx.ru/faq:mikopbx\_freepbx)
* [Настройка Grandstream HT503](https://wiki.mikopbx.ru/faq:grandstreamht503)
* [Настройка шлюза GOIP](https://wiki.mikopbx.ru/faq:goip)
* [Инструкции по подключению почтового клиента](https://wiki.mikopbx.ru/mail-settings)
* [Маршрутизация по DID номеру](https://wiki.mikopbx.ru/faq:did-routs)
* [Уведомление в телеграмм о пропущенных](https://wiki.mikopbx.ru/faq:simple\_tg\_notify)

### **Заключение**

Перечислены лишь наиболее важные изменения. Полный список можно найти в [описании релиза](https://github.com/mikopbx/Core/releases/tag/2023.1.223).

Мы стараемся сделать MikoPBX стабильной, простой, понятной в настройке и сопровождении. Надеемся, что она станет надежным инструментом в вашей компании.
