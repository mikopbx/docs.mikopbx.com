---
description: Инструкция по подключению Яндекс Cloud Object Storage в качестве S3-хранилища
---

# Подключение S3 хранилища Yandex Cloud

### Создание бакета

1. Перейдите в консоль Yandex Cloud ([ссылка](https://console.yandex.cloud/)).
2. Перейдите в раздел "**Все сервисы**" -> "**Object Storage**".

<figure><img src="../../../.gitbook/assets/YandexObjectStorageSection.png" alt=""><figcaption><p>Раздел "Object Storage"</p></figcaption></figure>

3. Нажмите "**Создать бакет**".

<figure><img src="../../../.gitbook/assets/YandexS3createBucket.png" alt=""><figcaption><p>Кнопка "Создать бакет"</p></figcaption></figure>

4. Заполните следующие параметры для создаваемого бакета:

* **Имя** — укажите название бакета (в нашем примере - `mikopbx-s3-storage`). Имя должно быть уникальным в рамках всего Yandex Cloud.
* **Макс. размер** — задайте максимальный объём бакета. Рекомендуется установить значение, соответствующее вашим потребностям (не менее 50 ГБ для рабочей станции), чтобы контролировать расход облачного пространства. Если ограничение не нужно — отметьте **«Без ограничения»**.
* **Доступ** — для всех трёх параметров (Чтение объектов, Чтение списка объектов, Чтение настроек) выберите значение **«С авторизацией»**.&#x20;

После заполнения всех параметров нажмите кнопку **«Создать бакет»**.

<figure><img src="../../../.gitbook/assets/YandexS3bucketParams.png" alt=""><figcaption><p>Параметры создаваемого бакета</p></figcaption></figure>

### Создание сервисного аккаунта

1. Перейдите в раздел "**Все сервисы**" -> "**Identity and Access Management**".

<figure><img src="../../../.gitbook/assets/YandexS3IdentityAccessManagement.png" alt=""><figcaption><p>Раздел "<strong>Identity and Access Management</strong>"</p></figcaption></figure>

2. Нажмите "**Создать сервисный аккаунт**".

<figure><img src="../../../.gitbook/assets/YandexS3CreateServiceAccount.png" alt=""><figcaption><p>Кнопка "<strong>Создать сервисный аккаунт</strong>"</p></figcaption></figure>

3. Укажите следующие параметры:

* **Имя** — введите название сервисного аккаунта (например, `mikopbx-s3-access`).
* **Роли в каталоге** — нажмите **«Добавить роль»**, в строке поиска введите `storage` и выберите роль **`storage.editor`**. Эта роль даёт необходимые права.

После заполнения параметров нажмите **«Создать»**.

<figure><img src="../../../.gitbook/assets/YandexS3-ServiceAccParams.png" alt=""><figcaption><p>Параметры создаваемого сервисного аккаунта</p></figcaption></figure>

4. Перейдите в дашбоард созданного сервисного аккаунта, нажав на его название.

<figure><img src="../../../.gitbook/assets/YandexS3CreatedServiceAcc.png" alt=""><figcaption><p>Созданный сервисный аккаунт</p></figcaption></figure>

5. Нажмите "**Создать новый ключ**" -> "**Создать статический ключ доступа**".

<figure><img src="../../../.gitbook/assets/YandexS3CreateNewKey.png" alt=""><figcaption><p>Создание нового ключа для сервисного аккаунта</p></figcaption></figure>

6\. Введите описание для создаваемого ключа и нажмите "Создать".

<figure><img src="../../../.gitbook/assets/YandexS3CreateNewKey-Description.png" alt=""><figcaption><p>Описание создаваемого ключа</p></figcaption></figure>

Будут отображены идентифкатор ключа и секретный ключ. Сохраните эти значения, они понадябтся позже для подключения хранилища к MikoPBX.

{% hint style="warning" %}
После закрытия диалога значение ключа будет недоступно.
{% endhint %}

<figure><img src="../../../.gitbook/assets/YandexS3-Keys.png" alt=""><figcaption><p>Созданный идентификатор и ключ</p></figcaption></figure>

### Подключение к MikoPBX

1. Перейдите во вкладку "**Обслуживание**" -> "**Хранилище**".

<figure><img src="../../../.gitbook/assets/MikoPBXstorageSection-cut.png" alt=""><figcaption><p>Раздел "<strong>Хранилище</strong>"</p></figcaption></figure>

2. Перейдите на вкладку "**Облачное хранилище S3**" и заполните следующие поля:

* **Автоматическая загрузка записей в облачное хранилище** — включите переключатель.
* **URL точки доступа S3** — введите `https://storage.yandexcloud.net`
* **Регион S3** — укажите регион Вашего аккаунта в Yandex Cloud, в этой инструкции - `ru-central1`
* **Имя бакета S3** — укажите имя бакета, созданного в Яндекс Cloud (например, `mikopbx-s3-storage` в этой инструкции)
* **Ключ доступа** и **Секретный ключ** — вставьте значения, полученные при создании статического ключа сервисного аккаунта.

Настройте ползунок **«Локальное хранение (режим S3)»** — выберите, как долго записи будут храниться локально до удаления после выгрузки в облако.

{% hint style="info" %}
Более короткое локальное хранение быстрее освобождает дисковое пространство.
{% endhint %}

Нажмите **«Сохранить»**.

<figure><img src="../../../.gitbook/assets/cloudStorageS3Section.png" alt=""><figcaption><p>Параметры для подключения S3 Yandex Cloud</p></figcaption></figure>

После сохранения настроек нажмите "Проверить соединение". При успешном подключении появится сообщение «**Подключено к S3**» и начнется синхронизация записей телефонных разговоров.

<figure><img src="../../../.gitbook/assets/cloudStorageSuccessful.png" alt=""><figcaption><p>Успешное подключение</p></figcaption></figure>
