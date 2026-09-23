---
description: >-
  Инструкция по подключению DigitalOcean Spaces Object Storage в качестве
  S3-хранилища
---

# Подключение S3 хранилища DigitalOcean

## Создание бакета и ключей

1. Перейдите в консоль DigitalOcean ([ссылка](https://cloud.digitalocean.com/)).
2. Перейдите в раздел "**Manage**" -> "**Spaces Object Storage**". Нажмите "**Create a Spaces Bucket**" для создания нового бакета.

<figure><img src="../../../.gitbook/assets/S3DOSpacesObjectStorage.png" alt=""><figcaption><p>Раздел "Spaces Object Storage"</p></figcaption></figure>

3. На странице создания бакета, в разделе "**Choose a datacenter region**", выберите ближайший к серверу MikoPBX регион. Выберите "**Standart Storage**".

{% hint style="info" %}
Запомните название Вашего региона (**sgp1** на скриншоте ниже), оно понадобится в будущем для настройки внутри MikoPBX.
{% endhint %}

<figure><img src="../../../.gitbook/assets/S3DOBucketParameters1.png" alt=""><figcaption><p>Параметры создаваемого бакета #1</p></figcaption></figure>

4. В поле "**Choose a unique Spaces Bucket name**" укажите произвольное название для бакета.

Нажмите "**Subscribe & Create Bucket**".

<figure><img src="../../../.gitbook/assets/S3DOBucketParameters2.png" alt=""><figcaption><p>Параметры создаваемого бакета #2</p></figcaption></figure>

5. Перейдите на страницу созданного бакета (нажмите на его название в разделе "**Buckets**").

<figure><img src="../../../.gitbook/assets/S3DOcreatedBucket.png" alt=""><figcaption><p>Созданный бакет в разделе "Buckets".</p></figcaption></figure>

6. Перейдите на вкладку "**Settings**".

<figure><img src="../../../.gitbook/assets/S3DOCreatedBucketSettings.png" alt=""><figcaption><p>Вкладка "Settings" на странице созданного бакета</p></figcaption></figure>

7. Пролистайте до раздела "**Access Keys**". Нажмите "**Create Access Key**" для создания новой связки ключей.

<figure><img src="../../../.gitbook/assets/S3DOCreateAccessKeyBtn.png" alt=""><figcaption><p>Раздел "Access Keys"</p></figcaption></figure>

8. Заполните необходимые параметры для создаваемого ключа:

* **Select access scope** - Limited Access.
* **Buckets** - выберите ранее созданный бакет.
* **Permissions** - Read/Write/Delete.
* **Give this access key a name** - укажите произвольное название для идентификации связки ключей.

Нажмите "**Create Access Key**".

<figure><img src="../../../.gitbook/assets/S3DOAccessKeysParameters.png" alt=""><figcaption><p>Параметры создаваемой связки ключей</p></figcaption></figure>

Будет отображены значения связки ключей (Access Key ID и Secret Key). Сохраните эти значения, они понадобятся в будущем при настройке на стороне MikoPBX.

<figure><img src="../../../.gitbook/assets/S3DOCreatedAccessKeys&#x27;.png" alt=""><figcaption><p>Связка access ключей</p></figcaption></figure>

#### Подключение к MikoPBX <a href="#podklyuchenie-k-mikopbx" id="podklyuchenie-k-mikopbx"></a>

1. Перейдите во вкладку "**Обслуживание**" -> "**Хранилище**".

<figure><img src="../../../.gitbook/assets/MikoPBXstorageSection-cut.png" alt=""><figcaption><p>Раздел "Хранилище"</p></figcaption></figure>

2. Перейдите на вкладку "**Облачное хранилище S3**" и заполните следующие поля:

* **Автоматическая загрузка записей в облачное хранилище** — включите переключатель.
* **URL точки доступа S3** — введите `https://sgp1.digitaloceanspaces.com` - замените sgp1 на Ваш регион.
* **Регион S3** — укажите регион Вашего бакета в DigitalOcean, в этой инструкции - `sgp1`.
* **Имя бакета S3** — укажите имя бакета, созданного в DigitalOcean (например, `mikopbx-s3-storage` в этой инструкции)
* **Ключ доступа** и **Секретный ключ** — вставьте значения, полученные в первой части этой инструкции (связка Access ключей).

Настройте ползунок **«Локальное хранение (режим S3)»** — выберите, как долго записи будут храниться локально до удаления после выгрузки в облако.

{% hint style="info" %}
Более короткое локальное хранение быстрее освобождает дисковое пространство.
{% endhint %}

Нажмите **«Сохранить»**.

<figure><img src="../../../.gitbook/assets/S3DOMikoPBXCredentials-RU.png" alt=""><figcaption><p>Параметры для подключения S3 DigitalOcean</p></figcaption></figure>

После сохранения настроек нажмите "**Проверить соединение**". При успешном подключении появится сообщение «**Соединение с S3 успешно**» и начнется синхронизация записей телефонных разговоров.

<figure><img src="../../../.gitbook/assets/S3DOSuccesfulConnection-RU.png" alt=""><figcaption><p>Успешное подключение</p></figcaption></figure>
