---
description: Установка MikoPBX с помощью сервиса Alibaba cloud
---

# Alibaba cloud

{% hint style="danger" %}
Инструкция актуальна для версии 2024.2.131 и новее!
{% endhint %}

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Alibaba cloud.

&#x20;Перед началом Вам необходимо скопировать ссылку на актуальный образ MikoPBX с расширением **.raw**. Сделать это можно на [github MikoPBX](https://github.com/mikopbx/core/releases).

## Загрузка образа в Alibaba cloud

### Создание Bucket

Для начала необходимо создать bucket для хранения образа. Для этого необходимо перейти в "**OSS Management Console**" ([ссылка](https://oss.console.aliyun.com/overview)).

<figure><img src="../../.gitbook/assets/OSSConsole.jpg" alt=""><figcaption><p>OSS Консоль</p></figcaption></figure>

1. Перейдите в раздел "**Buckets**".

<figure><img src="../../.gitbook/assets/bucketsSection.jpg" alt=""><figcaption><p>Раздел "Buckets"</p></figcaption></figure>

2. Нажмите "**Create Bucket**" для создания нового **Bucket'а**:

<figure><img src="../../.gitbook/assets/createBucketButton.jpg" alt=""><figcaption><p>Элемент "Create bucket"</p></figcaption></figure>

3. Заполните следующие данные:

* "**Bucket name**" - произвольное название для хранилища.
* "**Region**" - выберите регион, где будет храниться ваш образ

{% hint style="danger" %}
Регион у хранилища для образа, регион виртуальной машины должны совпадать!
{% endhint %}

Нажмите "**OK**".

<figure><img src="../../.gitbook/assets/bucketParameters.jpg" alt=""><figcaption><p>Параметры Bucket'а</p></figcaption></figure>

4. Перейдите в созданный bucket, нажав на его название в разделе "**Buckets**":

