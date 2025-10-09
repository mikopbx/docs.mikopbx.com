---
description: Установка системы с помощью записью образа на USB-носитель
---

# Установка с записью образа на USB-носитель (Live USB)

## Запись образа на USB-носитель

Для записи образа будет использована утилита Rufus. Скачать ее можно [по ссылке](https://rufus.ie/en/).

{% hint style="danger" %}
Размер USB-носителя должен быть не менее 1 ГБ. **Все данные на USB-носителе будут удалены!**
{% endhint %}

<figure><img src="../../.gitbook/assets/rufusMainMenu.png" alt="" width="285"><figcaption><p>Утилита Rufus</p></figcaption></figure>

1. После установки утилиты, перейдите в ее интерфейс. В разделе "Device" выберите Ваш носитель, нажмите SELECT и выберите ранее загруженный [.iso образ](https://www.mikopbx.ru/download/). Начнется его проверка.

<figure><img src="../../.gitbook/assets/refusSecletedDevice&#x26;Image.png" alt="" width="285"><figcaption><p>Выбранный образ и носитель</p></figcaption></figure>

2. После окончания проверки оставьте все параметры по-умолчанию и нажмите "START".

<figure><img src="../../.gitbook/assets/rufusStartButton.png" alt="" width="285"><figcaption><p>Начало записи образа</p></figcaption></figure>

3. После этого, во всплывающем окне **выберите "Write in DD Image mode".** Нажмите "OK".

<figure><img src="../../.gitbook/assets/rufusWriteInDDmode.png" alt="" width="313"><figcaption><p>Параметр "Write in DD image mode"</p></figcaption></figure>

4. Во всплывающем предупреждении, о том, что все данные на диске будут удалены, нажмите "ОК".

<figure><img src="../../.gitbook/assets/rufusConfirmFormat.png" alt="" width="285"><figcaption><p>Подтверждение форматирования диска</p></figcaption></figure>

Дождитесь окончания записи образа. По его завершении, Вы увидите надпись "<mark style="color:$success;">READY</mark>".

<figure><img src="../../.gitbook/assets/rufusImageReady.png" alt="" width="285"><figcaption><p>Успешная запись образа на USB-носитель</p></figcaption></figure>

