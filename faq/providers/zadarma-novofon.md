# Zadarma (Novofon)

## Настройка в личном кабинете Zadarma (Novofon) <a href="#nastrojka_v_lichnom_kabinete_zadarma_novofon" id="nastrojka_v_lichnom_kabinete_zadarma_novofon"></a>

1. Перейдите в личный кабинет Zadarma (Novofon)

<figure><img src="../../.gitbook/assets/1 (32).png" alt=""><figcaption></figcaption></figure>

2. Подключите виртуальный номер, нажав на соответствующую кнопку в меню управления

<figure><img src="../../.gitbook/assets/2 (37).png" alt=""><figcaption></figcaption></figure>

3. Перейдите в **Настройки** → **Подключение по SIP** настройте **CallerID** для исходящих

Параметры «**Логин**» / «**Пароль**» / «**Сервер**» понадобятся в дальнейшем для настройки подключения со стороны MikoPBX

<figure><img src="../../.gitbook/assets/4 (39).png" alt=""><figcaption></figcaption></figure>

4. Перейдите в настройки созданного номера

<figure><img src="../../.gitbook/assets/3 (27).png" alt=""><figcaption></figcaption></figure>

5. Настройте переадресацию входящих на sip аккаунт:

<figure><img src="../../.gitbook/assets/5 (23).png" alt=""><figcaption></figcaption></figure>

## Подключение провайдера в MikoPBX <a href="#podkljuchenie_provajdera_v_mikopbx" id="podkljuchenie_provajdera_v_mikopbx"></a>

1. Переходим в раздел **Маршрутизация** → **Провайдеры телефонии**, нажимаем на кнопку «**Подключить SIP**»

<figure><img src="../../.gitbook/assets/6 (5).png" alt=""><figcaption></figcaption></figure>

2. Указываем настройки подключения, как показано на скриншоте ниже:

* **Логин** - логин SIP-учетной записи пользователя Novofon
* **Пароль** - пароль SIP-учетной записи пользователя Novofon

<figure><img src="../../.gitbook/assets/11 (4).png" alt=""><figcaption></figcaption></figure>

3. Заполните все адреса, с которых могут поступать входящие звонки

<figure><img src="../../.gitbook/assets/8 (4).png" alt=""><figcaption></figcaption></figure>

4. Для этого перейдите в **Расширенные настройки**

<figure><img src="../../.gitbook/assets/12 (11).png" alt=""><figcaption></figcaption></figure>

5. Заполните **Дополнительные адреса провайдера**

Сохраните настройки

<figure><img src="../../.gitbook/assets/10 (12).png" alt=""><figcaption></figcaption></figure>

Результатом успешного подключения является зеленый индикатор.

<figure><img src="../../.gitbook/assets/13 (14).png" alt=""><figcaption></figcaption></figure>

## Настройка входящей маршрутизации <a href="#nastrojka_vxodjaschej_marshrutizacii" id="nastrojka_vxodjaschej_marshrutizacii"></a>

1. Чтобы можно было принять первый входящий звонок через провайдера Novofon, необходимо описать правила для входящего вызова. Для этого переходим в раздел **Маршрутизация** → **Входящие маршруты**. Нажимаем на кнопку «**Добавить новое правило**».

<figure><img src="../../.gitbook/assets/14 (9).png" alt=""><figcaption></figcaption></figure>

2. Выбираем провайдера Задарма(Novofon) и указываем на какой номер направляем все входящие звонки через этого провайдера. В нашем примере все вызова мы направили на сотрудника с внутренним номером 202, Вы можете выбрать очередь вызовов / IVR-меню. Подробнее о входящих маршрутах описано [здесь](../../manual/routing/incoming-routing.md).

<figure><img src="../../.gitbook/assets/15 (6).png" alt=""><figcaption></figcaption></figure>

## Исходящая маршрутизация <a href="#isxodjaschaja_marshrutizacija" id="isxodjaschaja_marshrutizacija"></a>

Чтобы внешние исходящие звонки осуществлялись через Novofon нужно настроить исходящую маршрутизацию.

1. Переходим **Маршрутизация** → **Исходящие маршруты**. Нажимаем на кнопку «**Добавить новое правило**».

<figure><img src="../../.gitbook/assets/16 (2).png" alt=""><figcaption></figcaption></figure>

2. Настройка как на скриншоте означает, что если номер начинается с **«7»** или **«8»** и остальная часть содержит 10 цифр, звонок будет осуществляться через провайдера Novofon.\
   Подробнее про настройку исходящих маршрутов можно прочитать [здесь.](../../manual/routing/outbound-routing.md)

<figure><img src="../../.gitbook/assets/17 (1).png" alt=""><figcaption></figcaption></figure>

На этом настройка провайдера Novofon завершена!
