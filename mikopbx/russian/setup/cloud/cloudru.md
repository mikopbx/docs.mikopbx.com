---
description: Установка MikoPBX на платформу Cloud.ru
---

# Cloud.ru

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Cloud.ru.

Перед началом вам необходимо скачать актуальный образ MikoPBX с расширением .raw. Сделать это можно по [ссылке](https://github.com/mikopbx/Core/releases).

## Загрузка образа в cloud.ru

1. Перейдите в раздел "**Сервисы**" -> "**Образы**".

<figure><img src="../../.gitbook/assets/CloudRUAllServicesImages.png" alt=""><figcaption><p>Раздел "Образы" в сервисах</p></figcaption></figure>

2. Нажмите "Создать образ".

<figure><img src="../../.gitbook/assets/CloudRUCreateNewImageBtn.png" alt=""><figcaption><p>Создание нового образа</p></figcaption></figure>

3. Нажмите "Создать шаблон cloud\_init"

<figure><img src="../../.gitbook/assets/CloudRUCreatecloudinitBtn.png" alt=""><figcaption><p>Создание шаблона cloud_init</p></figcaption></figure>

4. Вставьте следующий контекст:

{% code overflow="wrap" %}
```shellscript
#cloud-config
{% if public_key %}
mikopbx:
  ssh_authorized_keys:
    - >-
      {{ public_key }}
{% endif %}
```
{% endcode %}

Нажмите "**Сохранить**".

<figure><img src="../../.gitbook/assets/CloudRUcloudinit.png" alt=""><figcaption><p>Создание шаблона cloud_init</p></figcaption></figure>

5. Задайте название для создаваемого образа (например mikopbx-in-cloud). Нажмите "**Сохранить**".

<figure><img src="../../.gitbook/assets/CloudRUImageParams.png" alt=""><figcaption><p>Имя для создаваемого образа</p></figcaption></figure>

6. Нажмите "**Загрузить образ**". Выберите ранее скаченный файл .raw. Дождитесь окончания загрузки.

<figure><img src="../../.gitbook/assets/CloudRUUploadImage.png" alt=""><figcaption><p>Загрузка образа на платформу</p></figcaption></figure>

## Создание виртуальной машины

1. Перейдите в раздел "**Сервисы**" -> "**Виртуальные машины**".

<figure><img src="../../.gitbook/assets/CloudRUVMsInServices.png" alt=""><figcaption><p>Раздел "Виртуальные машины" в сервисах</p></figcaption></figure>

2. Нажмите "**Создать виртуальную машину**".

<figure><img src="../../.gitbook/assets/CloudRUCreateVMBtn.png" alt=""><figcaption><p>Кнопка для создания новой виртуальной машины</p></figcaption></figure>

3. Укажите название для виртуальной машины и задайте ей описание, выберите зону доступности.

<figure><img src="../../.gitbook/assets/CloudRUVMParamsP1.png" alt=""><figcaption><p>Параметры создаваемой виртуальной машины</p></figcaption></figure>

4. Далее выберите ранее загруженный образ в разделе "**Пользовательские**". Далее в разделе "**Диски**", нажмите "**Добавить диск**", укажите **50ГБ** в качестве его размера и тип - **HDD**. Этот диск будет использован для хранения записей разговоров. Для первого диска укажите минимально возможный размер на платформе - **10ГБ.**

<figure><img src="../../.gitbook/assets/CloudRUVMParamsP2.png" alt=""><figcaption><p>Параметры создаваемой виртуальной машины</p></figcaption></figure>

5. Далее выберите конфигурацию вычислительного ресурса. Подойдут типы "**Dedicated**" и "**Balanced**", выберите 1-2 vCPU и 2ГБ RAM (подбирайте конфигурацию под ваши задачи).

Нажмите "**Продолжить**".

<figure><img src="../../.gitbook/assets/CloudRUVMParamsP3.png" alt=""><figcaption><p>Параметры создаваемой виртуальной машины</p></figcaption></figure>

6. В разделе с настройками сети выберите "**Подсеть с публичным IP**", создайте подсеть если этого не было сделано ранее.

<figure><img src="../../.gitbook/assets/CloudRUVMNetwork.png" alt=""><figcaption><p>Параметры сети</p></figcaption></figure>

7. Далее укажите Ваш публичный ключ для SSH подключения если это требуется. Нажмите "**Создать**" для создания ВМ.

<figure><img src="../../.gitbook/assets/CloudRUVMSSH.png" alt=""><figcaption><p>Добавление SSH ключа к создаваемой машине</p></figcaption></figure>

## Настройка сетевого экрана в облаке

1. После создания, нажмите на название Вашей виртуальной машины.

<figure><img src="../../.gitbook/assets/CloudRUVMs.png" alt=""><figcaption><p>Переход в меню управления ВМ</p></figcaption></figure>

2. Перейдите в раздел "**Сетевые параметры**". Нажмите на название Вашей группы безопасности.

<figure><img src="../../.gitbook/assets/CloudVMNetworkParamenets.png" alt=""><figcaption><p>Группа безопасности в сетевых параметрах</p></figcaption></figure>

3. Разрешите входящие подключения для следующих протоколов и портов:

* TCP - 443 (https)
* UDP, TCP - 5060 (SIP)
* TCP - 5061 (SIP TLS)
* UDP - 10000-10200 (RTP)

<figure><img src="../../.gitbook/assets/CloudRUSecurityRule.png" alt=""><figcaption><p>Параметры группы безопасности</p></figcaption></figure>

## Первый вход в MikoPBX

1. Вернитесь к созданной виртуальной машине. Найдите ее публичный IP адрес в разделе "**Сетевые параметры**".

<figure><img src="../../.gitbook/assets/CloudRUVMsAddress.png" alt=""><figcaption><p>Публичный IP-адрес виртуальной машины</p></figcaption></figure>

2. Скопируйте и откройте этот адрес в браузере.

<figure><img src="../../.gitbook/assets/CloudRUMikoPBXWeb.png" alt=""><figcaption><p>Веб-интерфейс MikoPBX</p></figcaption></figure>

Данные для первой авторизации в MikoPBX:

* Логин - **admin**
* Пароль - **идентификатор виртуальной машины.**

<figure><img src="../../.gitbook/assets/CloudVMID.png" alt=""><figcaption><p>Идентификатор виртуальной машины (используется в качестве пароля)</p></figcaption></figure>
