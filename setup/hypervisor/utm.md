---
description: Установка MikoPBX в UTM
---

# UTM

{% embed url="https://vkvideo.ru/video-100268702_456239065" %}

В данной иснтрукции будет произведена установка на UTM. Перед ее началом скачайте файл образа диска с расширением ".iso". Сделать это можно [по ссылке](https://github.com/mikopbx/core/releases).

{% hint style="info" %}
Данная инструкция актуальна с первого релиза, опубликованного в 2026 году. Протестированно на процессорах Apple Silicon.
{% endhint %}

## Создание виртуальной машины

1. Перейдите в UTM. Нажмите "Create a New Virtual Machine" для создания новой виртуальной машины.

<figure><img src="../../.gitbook/assets/UTMDashboard.png" alt=""><figcaption><p>Главная страница UTM. Создание новой виртуальной машины.</p></figcaption></figure>

2. В качестве типа виртуальной машины выберите "Virtualize".

<figure><img src="../../.gitbook/assets/chooseTypeOfGuestM.png" alt=""><figcaption><p>Выбор типа виртуальной машины</p></figcaption></figure>

3. В качестве типа операционной системы выберите "Preconfigured" - "Linux".

<figure><img src="../../.gitbook/assets/chooseOS.png" alt=""><figcaption><p>Выбор типа операционной системы</p></figcaption></figure>

4. Выберите ранее загруженный файл образа диска в разделе "Boot ISO Image". Для этого нажмите на "Browse...".

<figure><img src="../../.gitbook/assets/chooseAnImage.png" alt=""><figcaption><p>Выбор файла образа диска для виртуальной машины</p></figcaption></figure>

5. Далее укажите характеристики Вашей виртуальной машины. В нашем случае будут использованы 2 ГБ ОЗУ и 2 ядра процессора.

<figure><img src="../../.gitbook/assets/setHardware.png" alt=""><figcaption><p>Конфигурация ВМ</p></figcaption></figure>

6. Далее укажите размер для системного диска. В нашем случае - 1 Гб.

{% hint style="info" %}
В MikoPBX используются два диска:

1. Системный диск. На него устанавливается система, рекомендуемый размер - 1 Гб.
2. Диск для хранения записей разговоров. Рекомендуемый размер - от 50 Гб.
{% endhint %}

<figure><img src="../../.gitbook/assets/setStorage.png" alt=""><figcaption><p>Указание размера системного диска</p></figcaption></figure>

7. Нажмите Continue.

<figure><img src="../../.gitbook/assets/setSharedDirectory.png" alt=""><figcaption><p>Раздел "Shared Directory"</p></figcaption></figure>

8. Будет отображена итоговая конфигурация виртуальной машины. Задайте ей желаемое имя (поле "Name"). И нажмите "Save".

<figure><img src="../../.gitbook/assets/guestMachineSummary.png" alt=""><figcaption><p>Итоговая конфигурация</p></figcaption></figure>

## Подключение диска для хранения данных

1. Перейдите в настройки ВМ. Для этого нажмите правой кнопкой мыши по её названию, далее "Edit".

<figure><img src="../../.gitbook/assets/VMGoToSettings.png" alt=""><figcaption><p>Настройки ВМ</p></figcaption></figure>

2. Перейдите в "Drives". Нажмите "New..."

<figure><img src="../../.gitbook/assets/addingNewDisk(Ch1).png" alt=""><figcaption><p>Раздел "Drives"</p></figcaption></figure>

3. Создайте новый диск со следующими параметрами:

* **Interface** - VirtlO
* **Size** - не менее 50Гб (в этой документации для тестовой машины будет использовано 10Гб)

Нажмите "Create".

<figure><img src="../../.gitbook/assets/addingNewDisk(Ch2).png" alt=""><figcaption><p>Создание второго диска</p></figcaption></figure>

## Установка системы

1. Запустите виртуальную машину.

<figure><img src="../../.gitbook/assets/UTMDashboardStartVM.png" alt=""><figcaption><p>Запуск ВМ</p></figcaption></figure>

2. После загрузки Вы увидите надпись <mark style="color:red;">PBX is running in Live or Recovery mode</mark>. Это означает, что система загружена из образа диска в Live режиме. Необходимо произвести установку системы. Для этого перейдите к разделу "\[8] Install on Hard Drive".

<figure><img src="../../.gitbook/assets/PBBRunningLiveCD.png" alt=""><figcaption><p>MikoPBX в режиме LiveCD</p></figcaption></figure>

3. Выберите диск для установки системы. В нашем случае доступны диски vda и vdb, для установки выбираем диск vda.

<figure><img src="../../.gitbook/assets/ConnectingSystemDisk(Ch1).png" alt=""><figcaption><p>Выбор диска для установки системы</p></figcaption></figure>

4. Подтвердите выбор: введите "y" с клавиатуры и нажмите Enter.

<figure><img src="../../.gitbook/assets/ConnectingSystemDisk(Ch2).png" alt=""><figcaption><p>Подтверждение выбора диска</p></figcaption></figure>

5. Далее выберите диск для хранения записей разговоров. В нашем случае единственный оставшийся размером 10 Гб.

<figure><img src="../../.gitbook/assets/ConnectingStorageDisk.png" alt=""><figcaption><p>Выбор диска для хранения записей разговоров</p></figcaption></figure>

После этого система будет перезагружена и доступна в обычном режиме (надпись "<mark style="color:red;">PBX is running in Live or Recovery mode</mark>" пропадет).

<figure><img src="../../.gitbook/assets/PBXReady.png" alt=""><figcaption><p>IP-адрес MikoPBX</p></figcaption></figure>

Введите этот IP-адрес в строку браузера для перехода в Веб-интерфейс.

<figure><img src="../../.gitbook/assets/WEBLoginForm.png" alt=""><figcaption><p>Web-интерфейс MikoPBX</p></figcaption></figure>

{% hint style="info" %}
Стандартные данные для входа:

* Login: admin
* Password: admin
{% endhint %}
