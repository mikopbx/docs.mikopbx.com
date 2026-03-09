---
description: >-
  Инструкция по подключению AWS S3 в качестве облачного хранилища для
  автоматической выгрузки записей разговоров из MikoPBX
---

# Подключение S3 хранилища AWS

### Создание бакета

1. Перейдите в консоль AWS ([ссылка](https://console.aws.amazon.com/)). Перейдите в раздел "**Все сервисы**" -> "**Storage**" -> "**S3**".

<figure><img src="../../../.gitbook/assets/awsS3section-updated.png" alt=""><figcaption><p>Раздел "S3" в AWS</p></figcaption></figure>

2. Нажмите "**Create bucket**".

<figure><img src="../../../.gitbook/assets/awsS3createBucketBtn.png" alt=""><figcaption><p>Кнопка для создания бакета</p></figcaption></figure>

3. Укажите произвольное название для бакета (поле "**Bucket name**"). Все остальные параметры оставьте по умолчанию, нажмите "**Create bucket**".

<figure><img src="../../../.gitbook/assets/awsS3bucketParametersUpdated.png" alt=""><figcaption><p>Параметры создаваемого бакета</p></figcaption></figure>

### Создание IAM пользователя и ключей доступа

1. Перейдите в раздел **"Все сервисы"** -> "**Security, Identity, & Compliance**" -> "**IAM**".

<figure><img src="../../../.gitbook/assets/awsS3IAMSection.png" alt=""><figcaption><p>Раздел "IAM"</p></figcaption></figure>

2. Далее необходимо создать нового IAM пользователя. Для этого перейдите во вкладку "**Access Management**", далее "**Users**". Нажмите "**Create user**".

<figure><img src="../../../.gitbook/assets/awsS3CreateUserBtn.png" alt=""><figcaption><p>Создание нового IAM пользователя</p></figcaption></figure>

3. Укажите имя создаваемого IAM пользователя в поле "**User name**".

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/awsS3userDetails.png" alt=""><figcaption><p>Вкладка "Specify user details"</p></figcaption></figure>

4. Выберите "**Attach policies directly**" в качестве "**Permissions options**". Пролистайте страницу.

<figure><img src="../../../.gitbook/assets/awsS3AttachPoliciesDirectly.png" alt=""><figcaption><p>Выбор "Permissions options"</p></figcaption></figure>

5. В разделе "**Permissions policies**" нажмите "**Create policy**".&#x20;

<figure><img src="../../../.gitbook/assets/awsS3CreatePolicy.png" alt=""><figcaption><p>Кнопка "Create policy"</p></figcaption></figure>

6. В открывшейся вкладке, в окне "**Policy editor**", выберите "**JSON**" в качестве формата и вставьте следующий контекст в поле с параметрами:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::имя-вашего-бакета",
        "arn:aws:s3:::имя-вашего-бакета/*"
      ]
    }
  ]
}
```

{% hint style="warning" %}
Замените "**имя-вашего-бакета**" на название созданого ранее бакета (в этой инструкции - "**aws-s3-mikopbxstorage**").
{% endhint %}

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/awsS3CreatingNewPolicyS1.png" alt=""><figcaption><p>Создание новой политики. Шаг 1</p></figcaption></figure>

7. Далее укажите произвольное название для создаваемой политики.

Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/awsS3access-mikopbx.png" alt=""><figcaption><p>Создание новой политики. Шаг 2</p></figcaption></figure>

8. Вернитесь на вкладку создания пользователя, обновите список политик и выберите ранее созданную policy (в этой инструкции - "**access-mikopbx**").

Нажмите "Next".

<figure><img src="../../../.gitbook/assets/awsS3ChoosingPolicy.png" alt=""><figcaption><p>Выбор ранее созданной политики</p></figcaption></figure>

9. Подтвердите создание пользователя: нажмите "**Create user**".

<figure><img src="../../../.gitbook/assets/awsS3ConfirmationOfUserCreation.png" alt=""><figcaption><p>Подтверждение создания пользователя</p></figcaption></figure>

10. Откройте страницу созданного пользователя, нажав на его имя.

<figure><img src="../../../.gitbook/assets/awsS3Username.png" alt=""><figcaption><p>Переход на страницу созданного пользователя</p></figcaption></figure>

11. Перейдите в раздел "**Security credentials**". Нажмите "Create access key".

<figure><img src="../../../.gitbook/assets/awsS3createAccessKeyBtn.png" alt=""><figcaption><p>Создание access ключа</p></figcaption></figure>

12. Выберите "**Application running outside AWS**". Нажмите "**Next**".

<figure><img src="../../../.gitbook/assets/awsS3ApplicationRunningOutside.png" alt=""><figcaption><p>Выбор параметров при создании ключа</p></figcaption></figure>

13. Введите описание ключа для того, чтобы идентифицировать его в будущем. Нажмите "**Create access key**".

<figure><img src="../../../.gitbook/assets/awsS3Params.png" alt=""><figcaption><p>Описание ключа</p></figcaption></figure>

Будет отображены access key и secret access key ключи. Сохраните их, они понадобятся далее для настройки внутри MikoPBX.

{% hint style="warning" %}
Только сейчас можно просмотреть или загрузить секретный ключ доступа. Восстановить его позже будет невозможно.
{% endhint %}

<figure><img src="../../../.gitbook/assets/awsS3CreatedAccessKey.png" alt=""><figcaption><p>Access key и Secret access key</p></figcaption></figure>

### Подключение к MikoPBX

1. Перейдите во вкладку "**Обслуживание**" -> "**Хранилище**".

<figure><img src="../../../.gitbook/assets/MikoPBXstorageSection-cut.png" alt=""><figcaption><p>Раздел "Облсуживание" -> "Хранилище"</p></figcaption></figure>

2. Перейдите на вкладку "**Облачное хранилище S3**" и заполните следующие поля:

* **Автоматическая загрузка записей в облачное хранилище** — включите переключатель.
* **URL точки доступа S3** — введите адрес доступа к S3 AWS, в зависимости от региона Вашего бакета ([ссылка ](https://docs.aws.amazon.com/general/latest/gr/s3.html)на таблицу со всеми url). В этой инструкции - `https://s3.ap-southeast-1.amazonaws.com`
* **Регион S3** — укажите **регион** **Вашего бакета**, в этой инструкции - `ap-southeast-1`
* **Имя бакета S3** — укажите имя бакета, созданного в AWS (например, `aws-s3-mikopbxstorage` в этой инструкции)
* **Ключ доступа** и **Секретный ключ** — вставьте значения, полученные при создании access ключа сервисного аккаунта.&#x20;

Настройте ползунок **«Локальное хранение (режим S3)»** — выберите, как долго записи будут храниться локально до удаления после выгрузки в облако.

{% hint style="info" %}
Более короткое локальное хранение быстрее освобождает дисковое пространство.
{% endhint %}

Нажмите **«Сохранить»**.

<figure><img src="../../../.gitbook/assets/mikopbxStorageParamsAWSupd.png" alt=""><figcaption><p>Параметры подключения облачного хранилища S3 в MikoPBX</p></figcaption></figure>

После сохранения настроек нажмите "Проверить соединение". При успешном подключении появится сообщение «**Подключено к S3**» и начнется синхронизация записей телефонных разговоров.

<figure><img src="../../../.gitbook/assets/successfulConnectionS3AWS.png" alt=""><figcaption><p>Успешное подключение</p></figcaption></figure>
