---
description: Установка системы с помощью записью образа на USB-носитель
---

# Установка с записью образа на USB-носитель (Live USB)

## Запись образа на USB-носитель&#x20;

### Windows

Перед началом процесса, отформатируйте Ваш носитель со следующими параметрами:

* **File system** - FAT32
* **Allocation unit size** - 8192 bytes

<figure><img src="../../.gitbook/assets/formatFlashUSB.png" alt="" width="221"><figcaption><p>Форматирование диска</p></figcaption></figure>

Для записи образа будет использована утилита Rufus. Скачать ее можно [по ссылке](https://rufus.ie/en/).

{% hint style="danger" %}
Размер USB-носителя должен быть не менее 1 ГБ. **Все данные на USB-носителе будут удалены!**
{% endhint %}

<figure><img src="../../.gitbook/assets/rufusMainMenu.png" alt="" width="285"><figcaption><p>Утилита Rufus</p></figcaption></figure>

1. После установки утилиты, перейдите в ее интерфейс. В разделе "Device" выберите Ваш носитель, нажмите SELECT и выберите ранее загруженный [.iso образ](https://www.mikopbx.ru/download/). Начнется его проверка.

<figure><img src="../../.gitbook/assets/refusSecletedDevice&#x26;Image (1).png" alt="" width="285"><figcaption><p>Выбранный образ и носитель</p></figcaption></figure>

2. После окончания проверки, установите следующие параметры и нажмите "START":

* **File system** - FAT32
* **Cluster size** - 8192 Bytes
* **Quick format -** отмечено
* **Create extended label and icon files -&#x20;**<mark style="color:$success;">**убрать галочку**</mark>

<figure><img src="../../.gitbook/assets/rufusStartButton (2).png" alt="" width="285"><figcaption><p>Начало записи образа</p></figcaption></figure>

3. После этого, во всплывающем окне **выберите "Write in DD Image mode".** Нажмите "OK".

<figure><img src="../../.gitbook/assets/rufusWriteInDDmodeNew.png" alt="" width="314"><figcaption><p>Параметр "Write in DD image mode"</p></figcaption></figure>

4. Во всплывающем предупреждении, о том, что все данные на диске будут удалены, нажмите "ОК".

<figure><img src="../../.gitbook/assets/rufusConfirmFormatNew.png" alt="" width="285"><figcaption><p>Подтверждение форматирования диска</p></figcaption></figure>

Дождитесь окончания записи образа. По его завершении, Вы увидите надпись "<mark style="color:$success;">READY</mark>".

<figure><img src="../../.gitbook/assets/rufusImageReadyNew.png" alt="" width="285"><figcaption><p>Успешная запись образа на USB-носитель</p></figcaption></figure>

### MacOS/Linux

...

## Установка системы

Запуститесь с USB-носителя. При возникновении ошибок (черный экран) - убидитесь, что:

* **Secure Boot** - Disabled
* **CSM (Compatibility Support Module)** - Enabled

