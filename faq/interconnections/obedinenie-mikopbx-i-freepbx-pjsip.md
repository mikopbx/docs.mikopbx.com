---
description: Инструкция по объединению нескольких АТС
---

# Объединение MikoPBX и FreePBX (PJSIP)

## Создание провайдера MikoPBX

1. В MikoPBX перейдите во вкладку "**Маршрутизация**" -> "**Провайдеры телефонии**":

<figure><img src="../../.gitbook/assets/providersMikoPBX.jpg" alt=""><figcaption><p>Раздел "<strong>Провайдеры телефонии</strong>"</p></figcaption></figure>

2. Создайте нового SIP-провайдера. Для этого нажмите "**Подключить SIP**":

<figure><img src="../../.gitbook/assets/connectSIPMikoPBX.jpg" alt=""><figcaption><p>Элемент "<strong>Подключить SIP</strong>"</p></figcaption></figure>

3. Заполните следующие параметры:

* "**Название провайдера**" - произвольное
* "**Тип учетной записи**" - Входящая регистрация

Скопируйте **логин** и **пароль**, они понадобятся позже.

<figure><img src="../../.gitbook/assets/providerParametersMikoPBX.jpg" alt=""><figcaption><p>Параметры провайдера MikoPBX</p></figcaption></figure>

## Создание транка FreePBX

1. В интерфейсе FreePBX перейдите в раздел "**Connectivity**" -> "**Trunks**":

<figure><img src="../../.gitbook/assets/TrunksFreePBX.jpg" alt=""><figcaption><p>Раздел "Trunks" FreePBX</p></figcaption></figure>

2. Добавьте новый транк, типа "**chan\_pjsip**".

<figure><img src="../../.gitbook/assets/newTrunkFreePBX.jpg" alt=""><figcaption><p>Новый транк в FreePBX</p></figcaption></figure>

3. Вставьте логин провайдера из MikoPBX в поле "**Trunk Name**":

<figure><img src="../../.gitbook/assets/trunkNameFreePBX.jpg" alt=""><figcaption><p>"Trunk Name" FreePBX</p></figcaption></figure>

4. Перейдите во вкладку "**pjsip Settings**" -> "**Advanced**":

* В поле «**From User**» вставьте значение «**Логин провайдера MikoPBX**»
* Установите «**Trust RPID/PAI**» в значение "**yes"**
* Установите «**Send RPID/PAI**» в значение «**Send Remote-Party-ID header**»

<figure><img src="../../.gitbook/assets/additionalParametersTrunkFreePBX.jpg" alt=""><figcaption><p>Параметры транка FreePBX</p></figcaption></figure>

5. Опишите шаблоны номеров на вкладке «**Dialed Number Manipulation Rules**»:

<figure><img src="../../.gitbook/assets/numberTe,plates.jpg" alt=""><figcaption><p>Настройка шаблонов номеров FreePBX</p></figcaption></figure>

Сохраните изменения.

## Варианты регистрации

Далее Вам необходимо выбрать один из двух вариантов регистрации:

#### Регистрация FreePBX на MikoPBX

<figure><img src="../../.gitbook/assets/FreePBXMikoPBXReg.jpg" alt=""><figcaption><p>Вариант регистрации FreePBX на MikoPBX</p></figcaption></figure>

#### Регистрация MikoPBX на FreePBX

<figure><img src="../../.gitbook/assets/MikoPBXFreePBXReg.jpg" alt=""><figcaption><p>Вариант регистрации MikoPBX на FreePBX</p></figcaption></figure>

Устанавите пароль (**сложный**, произвольный). Он должен быть одинаковый как на MikoPBX, так на FreePBX.

В «расширенных настройках» MikoPBX, в «Дополнительных параметрах» укажите следующие опции:

```
[endpoint]
trust_id_inbound=yes
send_rpid=yes
```

Сохраните и примените изменения.

<figure><img src="../../.gitbook/assets/advancedOprionsMikoPBXProvider.jpg" alt=""><figcaption><p>Дополнительные параметры провайдера в MikoPBX</p></figcaption></figure>

Настройка маршрутизации
