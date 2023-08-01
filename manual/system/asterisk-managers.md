# Доступ к AMI

**Asterisk Manager Interface (AMI)** - мощный и удобный программный интерфейс (API) Asterisk для управления системой из внешних программ. Благодаря AMI внешние программы могут осуществлять соединения с Asterisk посредством TCP протокола, инициировать выполнение команд, считывать результат их выполнения, а так же получать уведомления о происходящих событиях в реальном времени.\
\
AMI часто используют для интеграции с бизнес-процессами и системами, программным обеспечением CRM (Customer Relationship Managment — управление взаимодействия с клиентами). Управление Asterisk часто осуществляется из консоли CLI, но при использовании AMI не требуется прямой доступ к серверу, на котором запущен Asterisk. AMI — это наиболее простой инструмент, который в руках разработчика может оказаться очень мощным и гибким средством для интеграции с другими программными продуктами. Он дает возможность разработчикам использовать информацию, генерируемую Asterisk, в реальном масштабе времени.

Первое что необходимо сделать, это включить AMI и завести пользователя, с помощью которого клиентская программа будет аутентифицироваться. «**Система**» - «**Доступ к AMI**»

<figure><img src="../../.gitbook/assets/1 (53).png" alt=""><figcaption></figcaption></figure>

Для добавления новой учетной записи необходимо указать **Имя пользователя** и **Пароль**. Кроме того, необходимо задать **Сетевой фильтр**, т.е. из какой подсети разрешено подключение к пользователю AMI. Вы можете разрешить подключения с любых адресов, либо указать конкретную сеть, настройку который Вы произвели в разделе **Сеть и Firewall** → [Сетевой экран](../connectivity/firewall.md).

<figure><img src="../../.gitbook/assets/2 (18).png" alt=""><figcaption></figcaption></figure>

## Опции и права пользователя AMI <a href="#opcii_i_prava_polzovatelja_ami" id="opcii_i_prava_polzovatelja_ami"></a>

_Права пользователя AMI, устанавливаемые в секции \[user] конфигурационного файла /etc/asterisk/manager.conf_

<table><thead><tr><th width="215.33333333333331">Идентификатор прав</th><th width="193">Чтение</th><th>Запись</th></tr></thead><tbody><tr><td>System</td><td>Чтение общей информации о системе, например, уведомления о перезагрузке конфигурации</td><td>Позволяет пользователю выполнять Asterisk системы управления такими командами, как Restart, Reload, или Shutdown. Это разрешение также предоставляет пользователям возможность запускать системные команды за пределами Asterisk. Предоставление такого разрешения эквивалентно предоставлению доступа к командной оболочке, с правами пользователя / группы, под которыми запущен процесс Asterisk</td></tr><tr><td>Call</td><td>Чтение события о каналах в системе</td><td>Позволяет пользователю устанавливать информация на каналах</td></tr><tr><td>Log</td><td>Предоставляет пользователю доступ к чтению логов</td><td>Только чтение</td></tr><tr><td>Verbose</td><td>Предоставляет пользователю доступ к чтению подробных логов</td><td>Только чтение</td></tr><tr><td>Agent</td><td>Чтение событий статуса агентов из app_queue и chan_agent модулей</td><td>Позволяет пользователю выполнять действия для управления и получения состояния очередей и агентов</td></tr><tr><td>User</td><td>Доступа к пользовательским событиям, а также событиям Jabber / XMPP пользователей</td><td>Позволяет пользователю выполнять команду UserEvent, для создания пользовательских событий</td></tr><tr><td>Config</td><td>Только для записи</td><td>Позволяет пользователю получать, обновлять и перегружать файлы конфигурации</td></tr><tr><td>Command</td><td>Только для записи</td><td>Позволяет пользователю выполнять команды Asterisk CLI из AMI</td></tr><tr><td>Dtmf</td><td>Позволяет пользователю получать события DTMF</td><td>Только чтение</td></tr><tr><td>Reporting</td><td>Доступ к событиям качества звонка, таким как jitterbuffer или RTCP</td><td>Позволяет пользователю выполнять ряд действий для получения статистики и информации о состоянии всей системы</td></tr><tr><td>Cdr</td><td>Чтение событий записи данных в CDR</td><td>Только чтение</td></tr><tr><td>Dialplan</td><td>Чтение событий установки переменных диалплана, создания экстенов</td><td>Только чтение</td></tr><tr><td>Originate</td><td>Только для записи</td><td>Разрешение пользователю выполнять команду Origitate, которая отправляет запрос на создание нового звонка</td></tr></tbody></table>