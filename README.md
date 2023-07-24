# Руководство по MikoPBX

## Предисловие <a href="#predislovie" id="predislovie"></a>

Благодарим Вас за выбор **MikoPBX**!\
**MikoPBX -** это сервер телефонии с операционной системой и удобным веб интерфейсом. Она работает практически с любой телефонной технологией в мире. MikoPBX основана на **Asterisk** и имеет очень низкие требования к аппаратному обеспечению ПК.

{% hint style="info" %}
**MikoPBX** основана на:

* **Asterisk 16.5.1**
* **Linux 4.10.13**

Более подробная информация о пакетах Linux и их версий представлена [здесь](prochee/versii-i-pakety-linux.md).
{% endhint %}

{% hint style="success" %}
Не знаете с чего начать? Следуйте [данной инструкции](master/quick-start.md), которая поможет Вам запустить MikoPBX в работу как можно быстрее.
{% endhint %}

## Структура данного руководства <a href="#struktura_dannogo_rukovodstva" id="struktura_dannogo_rukovodstva"></a>

### Глава 1. Установка <a href="#glava_1_ustanovka" id="glava_1_ustanovka"></a>

В данной главе приведена пошаговая инструкция для выполнения установки MikoPBX на различных платформах.

* [Отдельный компьютер ](setup/bare-metal.md)
* [Виртуальная машина](setup/hypervisor/)
* [Облако](setup/cloud/)
* [Докер контейнер](setup/docker.md)

### Глава 2. Знакомство с MikoPBX <a href="#glava_2_znakomstvo_s_mikopbx" id="glava_2_znakomstvo_s_mikopbx"></a>

Первые шаги с MikoPBX. Как узнать IP адрес вашей системы, войти в веб-интерфейс и выполнить базовые настройки. [читать далее...](master/znakomstvo-s-mikopbx.md)

### Глава 3. Настройка лицензирования в MikoPBX <a href="#glava_3_nastrojka_licenzirovanija_v_mikopbx" id="glava_3_nastrojka_licenzirovanija_v_mikopbx"></a>

После установки MikoPBX необходимо произвести настройку программной лицензии MIKO SaaS License, без неё невозможна работа с дополнительными модулями. [читать далее...](manual/modules/upravlenie-modulyami/licensing.md)

### Глава 4. Телефония <a href="#glava_4_telefonija" id="glava_4_telefonija"></a>

&#x20;В разделе **Телефония** Вы можете настроить список сотрудников, настроить для каждого сотрудника его внутренний номер, настроить маршрутизацию звонков и проанализировать историю вызовов.

* [Сотрудники](manual/telefoniya/extensions.md)&#x20;
* [Очереди вызовов](manual/telefoniya/call-queues.md)
* [IVR меню](manual/telefoniya/ivr-menu.md)
* [Конференции](manual/telefoniya/conference-rooms.md)
* [Звуковые файлы](manual/telefoniya/sound-files.md)
* [История вызовов](manual/telefoniya/call-detail-records.md)

### Глава 5. Маршрутизация <a href="#glava_5_marshrutizacija" id="glava_5_marshrutizacija"></a>

&#x20;MikoPBX управляет учетными записями телефонов, поставщиков услуг. Вы узнаете, как настраивать учетные записи в этой главе.

* [Провайдеры телефонии](manual/routing/providers.md)
* [Входящие маршруты](manual/routing/incoming-routes.md)
* [Нерабочее время](manual/routing/out-off-work-time.md)
* [Исходящие маршруты](manual/routing/outbound-routes.md)

### Глава 6. Модули

* [Регистрация в marketplace](manual/modules/upravlenie-modulyami/licensing.md)
* [Управление модулями ](manual/modules/upravlenie-modulyami/)
* [Модули МИКО](modules/miko/)
* [Приложения диалпланов](manual/modules/dialplan-applications.md)

### Глава 7. Обслуживание <a href="#glava_7_obsluzhivanie" id="glava_7_obsluzhivanie"></a>

* [Обновление системы](manual/maintenance/update.md)
* [Диагностика системы](manual/maintenance/system-diagnostic.md)
* [Перезагрузка и выключение системы](manual/maintenance/restart.md)
* [Модуль резервного копирования ](manual/maintenance/modul-rezervnogo-kopirovaniya.md)

### Глава 8. Сеть и Firewall <a href="#glava_8_set_i_firewall" id="glava_8_set_i_firewall"></a>

* [Сетевые интерфейсы](manual/connectivity/network.md)
* [Сетевой экран](manual/connectivity/firewall.md)
* [Защита от взлома](manual/connectivity/fail2-ban.md)

### Глава 9. Система <a href="#glava_9_sistema" id="glava_9_sistema"></a>

* [Системные настройки](manual/system/general-settings.md)
* [Дата и время](manual/system/time-settings.md)
* [Почта и уведомления ](manual/system/mail-settings/)
* [Доступ к AMI](manual/system/asterisk-managers.md)
* [Кастомизация системных файлов](manual/system/custom-files.md)

## FAQ (Часто задаваемые вопросы) <a href="#faq" id="faq"></a>

В разделе представлено решение задач телефонии, с которыми чаще всего сталкиваются администраторы телефонных станций. Перейдя по ссылкам снизу, Вы сможете ознакомиться с практическими примерами и найти ответы на часто задаваемые вопросы о MikoPBX.

* [Диагностика проблем](faq/troubleshooting/)
* [Интеграция с 1С](broken-reference)
* [Сценарии и кейсы](faq/cases/)
* [Настройка провайдеров](faq/providers/)
* [Настройка софтфонов](faq/softphones/)
* [Voip шлюзы](faq/voip-gateways/)
* [IP-Телефоны](faq/ip-telefones/)
