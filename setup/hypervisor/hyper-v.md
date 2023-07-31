# Hyper-V

## Создание виртуальной машины

1. В оснастке Hyper-V Manager выполните действие **«Действие» - «Создать» - «Виртуальная машина...«**. Будет открыт помощник создания виртуальной машины

<figure><img src="../../.gitbook/assets/1 (52).png" alt=""><figcaption></figcaption></figure>

2. На первом шаге введите имя виртуальной машины

<figure><img src="../../.gitbook/assets/2 (6).png" alt=""><figcaption></figcaption></figure>

3. Выберите вариант «**Поколение 1**»

<figure><img src="../../.gitbook/assets/3 (22).png" alt=""><figcaption></figcaption></figure>

4. Выделите необходимый размер оперативной памяти. Мы рекомендуем **не менее 2 Гб**

Уберите галочку с пункта **"Использовать для этой виртуальной машины динамическую память"**

<figure><img src="../../.gitbook/assets/4 (31).png" alt=""><figcaption></figcaption></figure>

5. Выберите подключение к сети

<figure><img src="../../.gitbook/assets/5 (23).png" alt=""><figcaption></figcaption></figure>

6. Для операционной системы выделите диск размером 1Гб

<figure><img src="../../.gitbook/assets/6 (7).png" alt=""><figcaption></figcaption></figure>

7. Выберите заранее скачанный файл образа диска с расширением **.iso**

<figure><img src="../../.gitbook/assets/7 (19).png" alt=""><figcaption></figcaption></figure>

8. &#x20;Нажмите **"Готово"**

<figure><img src="../../.gitbook/assets/8 (11).png" alt=""><figcaption></figcaption></figure>

## Создание диска для записи разговоров

1. Перейдите в параметры виртуальной машины

<figure><img src="../../.gitbook/assets/9 (15).png" alt=""><figcaption></figcaption></figure>

2. Перейдите во вкладку "**Контроллер 0 IDE**" и нажмите "**Добавить**"

<figure><img src="../../.gitbook/assets/10 (4).png" alt=""><figcaption></figcaption></figure>

3. Нажмите "**Создать**"

<figure><img src="../../.gitbook/assets/11 (16).png" alt=""><figcaption></figcaption></figure>

4. Выберите формат "**VHD"**

<figure><img src="../../.gitbook/assets/12 (7).png" alt=""><figcaption></figcaption></figure>

5. Выберите тип диска фиксированного размера

<figure><img src="../../.gitbook/assets/13 (3).png" alt=""><figcaption></figcaption></figure>

6. Укажите **Имя** и **Расположение** диска

<figure><img src="../../.gitbook/assets/14 (9).png" alt=""><figcaption></figcaption></figure>

7. Укажите **Размер** диска (Мы рекомендуем размер диска **не менее 50 Гб**)

<figure><img src="../../.gitbook/assets/15 (6).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**1 час** записи разговоров занимает примерно **14мб** на диске.
{% endhint %}

8. Нажмите "**Готово**"

<figure><img src="../../.gitbook/assets/16.png" alt=""><figcaption></figcaption></figure>

9. Удалите DVD-дисковод

<figure><img src="../../.gitbook/assets/21 (1).png" alt=""><figcaption></figcaption></figure>

## Установка MikoPBX

1. Перейдите в "**Действие**" -> "**Пуск**"

<figure><img src="../../.gitbook/assets/18.png" alt=""><figcaption></figcaption></figure>

2. Нажмите "**Подключить**"

<figure><img src="../../.gitbook/assets/19 (3).png" alt=""><figcaption></figcaption></figure>

3. Выберите "**\[8] Install**"

<figure><img src="../../.gitbook/assets/20 (1).png" alt=""><figcaption></figcaption></figure>

4. Введите имя диск, на который будет установлена система, в нашем случае - **sdb**

<figure><img src="../../.gitbook/assets/22 (2).png" alt=""><figcaption></figcaption></figure>

5. Подтвердите ваш выбор: введите **y**

<figure><img src="../../.gitbook/assets/23 (4).png" alt=""><figcaption></figcaption></figure>

6. Выберите диск для записи разговоров - в нашем случае **sdc**

<figure><img src="../../.gitbook/assets/24 (2).png" alt=""><figcaption></figcaption></figure>

7. Система перезагрузится и надпись "Recovery mode" исчезнет. MikoPBX готова к работе.

<figure><img src="../../.gitbook/assets/25 (2).png" alt=""><figcaption></figcaption></figure>

## Первое подключение к MikoPBX

1. IP адрес вашей станции вы можете найти в интерфейсе АТС

<figure><img src="../../.gitbook/assets/26 (1).png" alt=""><figcaption></figcaption></figure>

2. Введите его в строку браузера и у вас откроется Веб-интерфейс MikoPBX

Пароль и логин по умолчанию - **admin**

<figure><img src="../../.gitbook/assets/27.png" alt=""><figcaption></figcaption></figure>
