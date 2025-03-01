---
description: Установка MikoPBX с помощью сервиса Vultr
---

# Vultr (In dev stage)

{% hint style="danger" %}
Инструкция актуальна для версии 2024.2.138 и новее!
{% endhint %}

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Vultr.

Перед началом Вам необходимо скачать актуальный образ MikoPBX с расширением **.iso**. Сделать это можно на [github MikoPBX](https://github.com/mikopbx/core/releases).

## Загрузка образа в Vultr

### Загрузка файла в хранилище

Для начала необходимо загрузить образ в облачную платформу.

1. Перейдите в раздел "**Cloud Storage**" -> "**Object Storage**":

<figure><img src="../../.gitbook/assets/objectStorageSection.jpg" alt=""><figcaption><p>Раздел "Object Storage"</p></figcaption></figure>

2. Необходимо создать новое хранилище. Для этого нажмите "**Add Object Storage**":

<figure><img src="../../.gitbook/assets/addObjectStorageButton.jpg" alt=""><figcaption><p>Элемент "Add Object Storage"</p></figcaption></figure>

3. Выберите тип хранилища (рекомендуется использовать самый базовый, так как он нужен только для хранения файла образа диска). Так же укажите название.
4. Перейдите в созданное хранилище, нажав на его название:

<figure><img src="../../.gitbook/assets/storageName.jpg" alt=""><figcaption><p>Название хранилища</p></figcaption></figure>

5. Перейдите во вкладку "**Buckets**" и создайте новый Bucket с произвольным названием.

<figure><img src="../../.gitbook/assets/createBucket.jpg" alt=""><figcaption><p>Новый Bucket</p></figcaption></figure>

6. В информации о хранилище, будут указаны данные для S3 подключения.

<figure><img src="../../.gitbook/assets/s3Credetionals.jpg" alt=""><figcaption><p>Данные для S3-подключения</p></figcaption></figure>

7. Далее необходимо подключиться к хранилищу через WinSCP. Для этого, перейдем в его интерфейс. Выберите "**New Site**":

<figure><img src="../../.gitbook/assets/newSite.jpg" alt=""><figcaption><p>"New Site"</p></figcaption></figure>

8. Укажите следующие параметры:

* "**File protocol**" - Amazon S3.
* "**Encryption**" - TLS/SSL Implict encryption.
* "**Port number**" - 443.
* "**Host Name**", "**Access key ID**", "**Secret access key**" - параметры из информации о хранилище.

Нажмите "**Login**".

<figure><img src="../../.gitbook/assets/s3WinSCP.jpg" alt=""><figcaption><p>Параметры авторизации</p></figcaption></figure>

9. Загрузите файл образа диска в хранилище.

<figure><img src="../../.gitbook/assets/importingFileWinSCP.jpg" alt=""><figcaption><p>Загрузка файла в хранилище</p></figcaption></figure>

10. Вернитесь в интерфейс Vultr, перейдите в директорию Вашего Bucket'а.&#x20;

<figure><img src="../../.gitbook/assets/bucketMenu.jpg" alt=""><figcaption><p>Директория Bucket'а</p></figcaption></figure>

11. Нажмите на три точки справа от названия файла. Перейдите в раздел "**Change Access**". Разрешите доступ, переключив тумблер.

<figure><img src="../../.gitbook/assets/CurrentPermission.jpg" alt=""><figcaption><p>Разрешение доступа</p></figcaption></figure>

### Импорт образа

1. Нажмите на три точки справа от названия файла. Скопируйте URL.

<figure><img src="../../.gitbook/assets/CopyURL.jpg" alt=""><figcaption><p>Элемент "Copy URL"</p></figcaption></figure>

2. Перейдите в раздел "**Orchestration**" -> "**ISOs**":

<figure><img src="../../.gitbook/assets/ISOs Section.jpg" alt=""><figcaption><p>Раздел "ISOs"</p></figcaption></figure>

3. Нажмите "**Add ISO**":

<figure><img src="../../.gitbook/assets/AddISO.jpg" alt=""><figcaption><p>Элемент "Add ISO"</p></figcaption></figure>

4. Вставьте ссылку на ранее загруженный файл, нажмите "**Upload**".

## Добавление связки SSH-ключей

1. Перейдите в раздел "**Account**" -> "**SSH Keys**". Нажмите "**Add SSH Key**"

<figure><img src="../../.gitbook/assets/addSSHKey.jpg" alt=""><figcaption><p>Элемент "Add SSH Key"</p></figcaption></figure>

2. Сгенерируйте пару SSH ключей [по инструкции](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/).
3. В интерфейсе добавления пары SSH-ключей введите произвольное название, а так же вставьте сгенерированный ключ.

Нажмите "**Add SSH Key**".

<figure><img src="../../.gitbook/assets/SSHkeysParameters.jpg" alt=""><figcaption><p>Добавление связки</p></figcaption></figure>

## Создание виртуальной машины

1. Перейдите в раздел "**Products**" -> "**Compute**":

<figure><img src="../../.gitbook/assets/computeSection.jpg" alt=""><figcaption><p>Раздел "Compute"</p></figcaption></figure>

2. Нажмите "**Deploy Server**":

<figure><img src="../../.gitbook/assets/deployServer.jpg" alt=""><figcaption><p>Элемент "Deploy Server"</p></figcaption></figure>

3. В открывшемся разделе выберите регион и конфигурацию вашей виртуальной машины.

<figure><img src="../../.gitbook/assets/VMParameters1 (1).jpg" alt=""><figcaption><p>Параметры ВМ №1</p></figcaption></figure>

4. Перейдите далее.

* Выберите "**ISO/iPXE**" -> Ранее загрузочный образ.
* Так же выберите ранее созданную пару SSH-ключей.

Нажмите "**Deploy**".

<figure><img src="../../.gitbook/assets/VMParameters2 (1).jpg" alt=""><figcaption><p>Параметры ВМ №2</p></figcaption></figure>

## Создание второго диска

После создания сервера, остановите его запуск.&#x20;

1. Перейдите в раздел "**Cloud Storage**" -> "**Block Storage**":

<figure><img src="../../.gitbook/assets/blockStorageSection.jpg" alt=""><figcaption><p>Раздел "Block Storage"</p></figcaption></figure>

2. Нажмите "**Add Block Storage**":

<figure><img src="../../.gitbook/assets/addBlockStorage.jpg" alt=""><figcaption><p>Элемент "Add Block Storage"</p></figcaption></figure>

3. Выберите тип диска, регион (такой же как у ранее созданной виртуальной машины), размер, а так же укажите произвольное название.

{% hint style="success" %}
Рекомендуемый размер диска для хранения записей разговоров - не менее 50Гб.
{% endhint %}

4. Перейдите в раздел управления созданным диском. Прикрепите диск к созданной виртуальной машине используя пункт "**Attach to:**"

<figure><img src="../../.gitbook/assets/attachTo.jpg" alt=""><figcaption><p>Элемент "Attach to"</p></figcaption></figure>

## Установка системы

1. Перейдите в меню управления виртуальной машиной.

<figure><img src="../../.gitbook/assets/serverInformation.jpg" alt=""><figcaption><p>Меню управления виртуальной машиной</p></figcaption></figure>

2. Перейдите в консоль, нажав на соответствующий элемент.

<figure><img src="../../.gitbook/assets/ConsoleButton.jpg" alt=""><figcaption><p>Элемент для открытия консоли</p></figcaption></figure>

3. Вы попадете во встроенную консоль.

<figure><img src="../../.gitbook/assets/internalConsole.jpg" alt=""><figcaption><p>Встроенная консоль</p></figcaption></figure>

4. Перейдите в "**\[8] Install**".
5. Выберите диск, который будет использован в качестве системного. Подтвердите действия - введите "**y**" и нажмите "**Enter**":

<figure><img src="../../.gitbook/assets/mountingSystemDrive.jpg" alt=""><figcaption><p>Выбор системного диска</p></figcaption></figure>

6. Выберите диск для хранения записей разговоров. Система перезагрузится.
7. Перейдите в настройки виртуальной машины "**Settings**", далее в "**Custom ISO**". Нажмите "**Remove ISO**".

<figure><img src="../../.gitbook/assets/removeISO.jpg" alt=""><figcaption><p>Элемент "Remove ISO"</p></figcaption></figure>

На данном этапе система установлена и готова к работе!

## Подключение к WEB-интерфейсу

1. В адресную строку введите IP-адрес Вашей виртуальной машины. Найти его Вы можете в консоли MikoPBX.

<figure><img src="../../.gitbook/assets/MikoPBXIPadress.jpg" alt=""><figcaption><p>IP-адрес станции</p></figcaption></figure>

2.
