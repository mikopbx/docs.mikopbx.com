# Звонки на экстренные номера

{% hint style="success" %}
Необходимо иметь возможность звонить на экстренные номера 101, 102, 103, 112 и т.д.
{% endhint %}

1. Переходим в раздел **Маршрутизация** → **Исходящие маршруты**.&#x20;

<figure><img src="../../.gitbook/assets/1 (1).png" alt=""><figcaption><p>Раздел "<a href="../../manual/routing/outbound-routes.md">Исходящие маршруты</a>"</p></figcaption></figure>

2. Добавляем новое правило

<figure><img src="../../.gitbook/assets/2 (1).png" alt=""><figcaption><p>Добавление нового правила </p></figcaption></figure>

3. &#x20;Заполняем параметры как на скриншоте ниже:

<figure><img src="../../.gitbook/assets/3.png" alt=""><figcaption><p>Параметры для правила "Звонки на экстренные номера"</p></figcaption></figure>

4. Звонки на экстренные номера направляете через произвольного провайдера. Провайдер **должен поддерживать** набор экстренных номеров. Приоритет данного правила должен быть наивысшим, т.е. в списке созданных исходящих маршрутов он должен быть описан первым.

<figure><img src="../../.gitbook/assets/4.png" alt=""><figcaption><p>Указание приоритета для звонков на экстренные номера </p></figcaption></figure>

5. &#x20;Перейдите в раздел **Система** → **Кастомизация системных файлов**.

<figure><img src="../../.gitbook/assets/5.png" alt=""><figcaption><p>Раздел "Кастомизация системных файлов"</p></figcaption></figure>

6. Откройте для редактирования конфигурационный файл **extensions.conf**.

<figure><img src="../../.gitbook/assets/6.png" alt=""><figcaption><p>Редактирование файла "extensions.conf"</p></figcaption></figure>

7. Установите режим «**Добавлять в конец файла**». В черное окно добавьте следующий фрагмент кода:

```php
[internal](+)
exten => _1XX,1,Goto(outgoing,${EXTEN},1)	
```

<figure><img src="../../.gitbook/assets/newForm.png" alt=""><figcaption><p>Добавление кода в конец файла</p></figcaption></figure>

В выше приведенном фрагменте кода мы описали правила для всех трехзначных номеров, начинающихся с 1. Если нужно указать конкретные экстренные номера, то вместо выше представленного кода нужно вставить следующие строки:

```php
[internal](+)
exten => 112,1,Goto(outgoing,${EXTEN},1)
exten => 101,1,Goto(outgoing,${EXTEN},1)
exten => 102,1,Goto(outgoing,${EXTEN},1)
exten => 103,1,Goto(outgoing,${EXTEN},1)
```
