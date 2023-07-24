# Ростелеком

## Настройка в личном кабинете Ростелеком <a href="#nastrojka_v_lichnom_kabinete_rostelekom" id="nastrojka_v_lichnom_kabinete_rostelekom"></a>

1. Для начала работы вам необходимо авторизоваться в управление "**Виртуальной АТС**" Ростелекома

<figure><img src="../../.gitbook/assets/1 (6).png" alt=""><figcaption></figcaption></figure>

2. Перейдите во вкладку "**Пользователи**", добавьте нового пользователя

<figure><img src="../../.gitbook/assets/2 (4).png" alt=""><figcaption></figcaption></figure>

3. Заполните все необходимые данные для пользователя

<figure><img src="../../.gitbook/assets/3 (11).png" alt=""><figcaption></figcaption></figure>

4. Перейдите в настройки пользователя. Для этого нажмите на его имя в меню "**Пользователи**"

<figure><img src="../../.gitbook/assets/4 (18).png" alt=""><figcaption></figcaption></figure>

5. Выберите номер для исходящих вызовов

Нажмите "**Сохранить изменения**"

<figure><img src="../../.gitbook/assets/5 (24).png" alt=""><figcaption></figcaption></figure>

6. Перейдите во вкладку "**Номера и маршрутизация**"

Для подключенного номера направьте все входящие звонки на созданного вами пользователя.

<figure><img src="../../.gitbook/assets/7 (5).png" alt=""><figcaption></figcaption></figure>

## Подключение провайдера Ростелеком в MikoPBX <a href="#podkljuchenie_provajdera_rostelekom_v_mikopbx" id="podkljuchenie_provajdera_rostelekom_v_mikopbx"></a>

1. &#x20;Переходим в раздел **Маршрутизация** → **Провайдеры телефонии**, нажимаем на кнопку «**Подключить SIP**».

<figure><img src="../../.gitbook/assets/8 (9).png" alt=""><figcaption></figcaption></figure>

2. Указываем настройки подключения, как показано на скриншоте ниже:

* **Хост или IP адрес** - это имя домена
* **Логин** - логин SIP-учетной записи пользователя Ростелеком
* **Пароль** - пароль SIP-учетной записи пользователя Ростелеком

<figure><img src="../../.gitbook/assets/10 (12).png" alt=""><figcaption></figcaption></figure>

Остальные настройки оставьте по умолчанию.

Результатом успешного подключения является зеленый индикатор.

<figure><img src="../../.gitbook/assets/11 (7).png" alt=""><figcaption></figcaption></figure>

## Входящая маршрутизация в MikoPBX <a href="#vxodjaschaja_marshrutizacija_v_mikopbx" id="vxodjaschaja_marshrutizacija_v_mikopbx"></a>

1. Чтобы можно было принять первый входящий звонок через провайдера Ростелеком, необходимо описать правила для входящего вызова. Для этого переходим в раздел **Маршрутизация** → **Входящие маршруты**. Нажимаем на кнопку «**Добавить новое правило**».

<figure><img src="../../.gitbook/assets/12 (3).png" alt=""><figcaption></figcaption></figure>

2. Выбираем провайдера Ростелеком и указываем на какой номер направляем все входящие звонки через этого провайдера. В нашем примере все вызова мы направили на сотрудника с внутренним номером 202, Вы можете выбрать очередь вызовов / IVR-меню. Подробнее о входящих маршрутах описано [здесь](../../manual/routing/incoming-routes.md).

<figure><img src="../../.gitbook/assets/13 (2).png" alt=""><figcaption></figcaption></figure>

## Исходящая маршрутизация <a href="#isxodjaschaja_marshrutizacija" id="isxodjaschaja_marshrutizacija"></a>

Чтобы внешние исходящие звонки осуществлялись через Ростелеком нужно настроить исходящую маршрутизацию.

1. Переходим **Маршрутизация** → **Исходящие маршруты**. Нажимаем на кнопку «**Добавить новое правило**».

<figure><img src="../../.gitbook/assets/14 (9).png" alt=""><figcaption></figcaption></figure>

2. Настройка как на скриншоте означает, что если номер начинается с **«7»** или **«8»** и остальная часть содержит 10 цифр, звонок будет осуществляться через провайдера Ростелеком.\
   Подробнее про настройку исходящих маршрутов можно прочитать [здесь.](../../manual/routing/outbound-routes.md)

<figure><img src="../../.gitbook/assets/15 (2).png" alt=""><figcaption></figcaption></figure>

На этом настройка провайдера Ростелеком завершена!
