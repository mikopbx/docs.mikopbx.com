---
description: Установка MikoPBX в облако VK Cloud solution
---

# VK Cloud Solutions

## Добавление локальной подсети

1. Авторизуйтесь на сайте [mcs.mail.ru](https://mcs.mail.ru/).&#x20;

<figure><img src="../../.gitbook/assets/1 (12).png" alt=""><figcaption></figcaption></figure>

2. Откройте меню **Виртуальных сетей**

<figure><img src="../../.gitbook/assets/2 (11).png" alt=""><figcaption></figcaption></figure>

3. Перейдите во вкладку "**Сети**"

<figure><img src="../../.gitbook/assets/3 (16).png" alt=""><figcaption></figcaption></figure>

4.  Для начала следует убедиться, что в разделе «**Виртуальные сети**» - «**Сети**» доступа локальная подсеть.

    MikoPBX будем устанавливать за NAT.

    Если сеть не создана: нажмите **"Создать"**

<figure><img src="../../.gitbook/assets/4 (5).png" alt=""><figcaption></figcaption></figure>

&#x20;       Введите **название** Сети и нажмите "**Добавить Сеть**"

<figure><img src="../../.gitbook/assets/5 (8).png" alt=""><figcaption></figcaption></figure>

## Добавление ssh-rsa ключа.

1. Перейдите к настройке ключевых пар:

&#x20;       Для этого нажмите на свой e-mail, далее на надпись "**Ключевые пары**"

<figure><img src="../../.gitbook/assets/6 (13).png" alt=""><figcaption></figcaption></figure>

2. Создайте или импортируйте ключ.

&#x20;       Он будет использоваться при доступе к АТС по ssh.

## Загрузка образа MikoPBX <a href="#zagruzka_obraza_mikopbx" id="zagruzka_obraza_mikopbx"></a>

1. Перейдите во вкладку "**Облачные вычисления**"

<figure><img src="../../.gitbook/assets/7 (8).png" alt=""><figcaption></figcaption></figure>

2. Оттуда перейдите к вкладке "**Образы**":

<figure><img src="../../.gitbook/assets/8 (4).png" alt=""><figcaption></figcaption></figure>

3. Нажмите "**Создать образ**":

<figure><img src="../../.gitbook/assets/9 (5).png" alt=""><figcaption></figcaption></figure>

4. Скачайте образ диска формата raw.

<figure><img src="../../.gitbook/assets/extra3.png" alt=""><figcaption></figcaption></figure>

5. Выберите "**Диск**", выберите скачанный образ, задайте название образа.

&#x20;        Нажмите "**Создать образ**":

<figure><img src="../../.gitbook/assets/extra2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/extra1.png" alt=""><figcaption></figcaption></figure>

## Создадим инстанс <a href="#sozdadim_instans" id="sozdadim_instans"></a>

1. Перейдите в раздел «**Облачные вычисления**» - «**Виртуальные машины**»

Нажмите "**Создать инстанс**":

<figure><img src="../../.gitbook/assets/extra6.png" alt=""><figcaption></figcaption></figure>

2. &#x20;Введите:

* _Имя виртуальной машины_
* _Тип виртуальной машины_
* _Зону доступности_
* _Количество машин в конфигурации_
* _Размер Диска_
* _Тип диска_

<mark style="color:red;">**Укажите MikoPBX в качестве операционной системы**</mark>

Нажмите **"Следующий шаг"**

<figure><img src="../../.gitbook/assets/extra4.png" alt=""><figcaption></figcaption></figure>

3. Выберите **сеть**, **ключ виртуальной машины** и нажмите "**Следующий шаг**"

<figure><img src="../../.gitbook/assets/15 (8).png" alt=""><figcaption></figcaption></figure>

4. Нажмите "**Создать инстанс**"

<figure><img src="../../.gitbook/assets/16 (3).png" alt=""><figcaption></figcaption></figure>

5. Как только инстанс будет создан, сразу остановите его запуск:

<figure><img src="../../.gitbook/assets/17 (4).png" alt=""><figcaption></figcaption></figure>

Инстанс создан.

## Диск для хранения данных <a href="#disk_dlja_xranenija_dannyx" id="disk_dlja_xranenija_dannyx"></a>

1. Перейдите во вкладку **Диски**, нажмите "**Создать диск**"

<figure><img src="../../.gitbook/assets/18 (3).png" alt=""><figcaption></figcaption></figure>

2. Введите следующие параметры:

* Название диска
* Источник **(**_Пустой диск_**)**
* Тип диска
* Зону доступности**(**_Должна быть такая же, как и у инстанса_**)**
* Размер**(**_более 50 ГБ_**),**

Далее отметьте галочку "**подключить диск"** и укажите инстанс MikoPBX, который мы сейчас запускаем.

Нажмите "**Создать диск**"

<figure><img src="../../.gitbook/assets/19 (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/20 (2).png" alt=""><figcaption></figcaption></figure>

3. Запустите инстанс:

<figure><img src="../../.gitbook/assets/extra5.png" alt=""><figcaption></figcaption></figure>

4. Перейдите в консоль АТС:

Зайдите во вкладку "**Виртуальные машины**", выберите свою виртуальную машину

<figure><img src="../../.gitbook/assets/superExtra1.png" alt=""><figcaption></figcaption></figure>

Перейдите во вкладку "**Консоль**"

<figure><img src="../../.gitbook/assets/superExtra2.png" alt=""><figcaption></figcaption></figure>

5. Выберите пункт «**\[6] Data storage**»

<figure><img src="../../.gitbook/assets/22 (4).png" alt=""><figcaption></figcaption></figure>

6. Далее выберите пункт «**\[1] Mount drive as data storage**»&#x20;

<figure><img src="../../.gitbook/assets/23 (1).png" alt=""><figcaption></figcaption></figure>

7. На следующем шаге нажмите **ENTER**

<figure><img src="../../.gitbook/assets/24 (1).png" alt=""><figcaption></figcaption></figure>

Система перезагрузится, надпись **"**<mark style="color:red;">**Storage disk not connected**</mark>**"** пропадет

<figure><img src="../../.gitbook/assets/superExtra3.png" alt=""><figcaption></figcaption></figure>

Диск для хранения данных успешно подключен.

## Подключение <a href="#nastrojka_firewall" id="nastrojka_firewall"></a>

1. Определите шлюз, маску, внешний IP-адрес с помощью раздела **"Облачные вычисления" -> "Виртуальные машины" -> "Ваша виртуальная машина" -> "Сети"**

<figure><img src="../../.gitbook/assets/extra7 (2).png" alt=""><figcaption></figcaption></figure>

**Для данного примера:**

&#x20;  _IP-адрес: 185.130.115.36_

&#x20;  _Шлюз: 185.130.115.254_

&#x20;  _Маска: 22_

2. Перейдите в АТС:

Зайдите во вкладку "**Виртуальные машины**", выберите свою виртуальную машину

<figure><img src="../../.gitbook/assets/superExtra1.png" alt=""><figcaption></figcaption></figure>

Перейдите во вкладку "**Консоль**"

<figure><img src="../../.gitbook/assets/superExtra2.png" alt=""><figcaption></figcaption></figure>

3. Перейдите к разделу «**\[2] Configure LAN IP Address**»

<figure><img src="../../.gitbook/assets/26.png" alt=""><figcaption></figcaption></figure>

4. Выбираем «**\[2] Manual settings**»

<figure><img src="../../.gitbook/assets/27 (1).png" alt=""><figcaption></figcaption></figure>

5. Введите внешний IP-адрес.

Нажмите **Enter**

<figure><img src="../../.gitbook/assets/28.png" alt=""><figcaption></figcaption></figure>

6. Введите значение маски подсети

Нажмите **Enter**

<figure><img src="../../.gitbook/assets/29.png" alt=""><figcaption></figcaption></figure>

7. Введите значение шлюза

Нажмите **Enter:**

<figure><img src="../../.gitbook/assets/30.png" alt=""><figcaption></figcaption></figure>

8. Введите значение DNS

Рекомендуется использовать стандартное - **8.8.8.8**

Нажмите **Enter**

<figure><img src="../../.gitbook/assets/31.png" alt=""><figcaption></figcaption></figure>

## Firewall

Для того, чтобы получить доступ к панели управления, нужно изменить стандартные настройки firewall в VKCloud

Для этого: Перейдите в "**Виртуальные сети**" -> "**Настройки firewall**" -> "**Добавить**"

<figure><img src="../../.gitbook/assets/32 (1).png" alt=""><figcaption></figcaption></figure>

Введите **Имя группы правил, Описание**

нажмите "**Создать группу**"

<figure><img src="../../.gitbook/assets/33.png" alt=""><figcaption></figcaption></figure>

Нажмите "**Добавить правило**" в разделе "**Входящий трафик**"

<figure><img src="../../.gitbook/assets/34.png" alt=""><figcaption></figcaption></figure>

4. Выберите тип "**Все протоколы и все порты**", выберете Удаленный адрес "**Все IP-адреса**"

Нажмите "**Создать правило**"

<figure><img src="../../.gitbook/assets/35.png" alt=""><figcaption></figcaption></figure>

5. Нажмите "**Добавить виртуальную машину**"

<figure><img src="../../.gitbook/assets/extra9.png" alt=""><figcaption></figcaption></figure>

6. Выберете вашу виртуальную машину и нажмите "**Добавить группу правил**"

<figure><img src="../../.gitbook/assets/extra10.png" alt=""><figcaption></figcaption></figure>

_<mark style="color:red;">**Не забудьте настроить firewall внутри MikoPBX**</mark>_. Инструкцию можно прочитать [здесь.](../../manual/connectivity/firewall.md)

## Первый вход в MikoPBX

Для того, чтобы открыть панель управления вам надо вбить в строку браузера IP-адрес вашей виртуальной машины.

<figure><img src="../../.gitbook/assets/extra8.png" alt=""><figcaption></figcaption></figure>

Логин и пароль по умолчанию - **Admin**



На этом установка MikoPBX на VKCloud завершена.
