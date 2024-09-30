# VMware Fusion

## Создание виртуальной машины

1. Создаем новую виртуальную машину.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

2. После скачивания последней версии образа ([ссылка](https://www.askozia.ru/download/)), указываем ISO файл с установочным дистрибутивом.

Нажимаем "**Continue**"

<figure><img src="../../.gitbook/assets/2 (44).png" alt=""><figcaption></figcaption></figure>

3. Выбираем тип операционной системы **Other Linux 5.x and later kernel 64-bit**

Нажимаем "**Continue**"

<figure><img src="../../.gitbook/assets/22 (1).png" alt=""><figcaption></figcaption></figure>

4. Выбираем тип биоса **Legacy**

Нажимаем "**Continue**"

<figure><img src="../../.gitbook/assets/4 (4).png" alt=""><figcaption></figcaption></figure>

5. Нажимаем "**Finish**"

<figure><img src="../../.gitbook/assets/5 (15).png" alt=""><figcaption></figcaption></figure>

## Подключение нового диска

1. После создания виртуальной машины, дождитесь ее загрузки

<figure><img src="../../.gitbook/assets/6 (22).png" alt=""><figcaption></figcaption></figure>

2. Перейдите в раздел "**\[3] Reboot the system**"

<figure><img src="../../.gitbook/assets/7 (3).png" alt=""><figcaption></figcaption></figure>

3. Выберите "**\[2]** **Shutdown**"

<figure><img src="../../.gitbook/assets/8 (10).png" alt=""><figcaption></figcaption></figure>

4. После выключения виртуальной машины, перейдите в настройки

<figure><img src="../../.gitbook/assets/9 (4).png" alt=""><figcaption></figcaption></figure>

5. Выберите "**Add device**"

<figure><img src="../../.gitbook/assets/10 (7).png" alt=""><figcaption></figcaption></figure>

6. Выберите "**New Hard Disk**"

Нажмите "**Add...**"

<figure><img src="../../.gitbook/assets/11 (14).png" alt=""><figcaption></figcaption></figure>

7. Выберите размер жесткого диска(мы рекомендуем **не менее 50ГБ**)

Нажмите "**Apply**"

<figure><img src="../../.gitbook/assets/12 (4).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**1 час** записи разговоров занимает примерно **14мб** на диске.
{% endhint %}

## Установка MikoPBX&#x20;

1. Запустите виртуальную машину

<figure><img src="../../.gitbook/assets/13 (7).png" alt=""><figcaption></figcaption></figure>

2. Выберите пункт "**\[8] Install**"

<figure><img src="../../.gitbook/assets/14 (3).png" alt=""><figcaption></figcaption></figure>

3. Введите имя диска, на который будет установлена MikoPBX&#x20;

В нашем случае - _sdb_, введите его название и нажмите **Enter**

<figure><img src="../../.gitbook/assets/15 (5).png" alt=""><figcaption></figcaption></figure>

4. Подтвердите выбор диска: введите **y**

<figure><img src="../../.gitbook/assets/16 (6).png" alt=""><figcaption></figcaption></figure>

5. Выберите диск для записи разговоров&#x20;

В нашем случае - _sdc_, введите его название и нажмите **Enter**

<figure><img src="../../.gitbook/assets/17 (6).png" alt=""><figcaption></figcaption></figure>

6. Система перезагрузится и MikoPBX будет готова к использованию.

<figure><img src="../../.gitbook/assets/18 (2).png" alt=""><figcaption></figcaption></figure>

## Первое подключение к MikoPBX

1. В АТС отображается **IP адрес** станции, по которому к ней можно подключится&#x20;

<figure><img src="../../.gitbook/assets/19 (1).png" alt=""><figcaption></figcaption></figure>

2. Введите IP адрес станции в строку браузера и у вас откроется меню входа в MIkoPBX&#x20;

Логин и пароль по умолчанию - "**admin**"

<figure><img src="../../.gitbook/assets/20 (3).png" alt=""><figcaption></figcaption></figure>
