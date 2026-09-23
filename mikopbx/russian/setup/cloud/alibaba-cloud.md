---
description: Установка MikoPBX с помощью сервиса Alibaba cloud
---

# Alibaba cloud

{% hint style="danger" %}
Инструкция актуальна для версии 2024.2.135 и новее!
{% endhint %}

{% embed url="https://vkvideo.ru/video-100268702_456239058" %}

В данной инструкции мы пошагово произведем установку MikoPBX с помощью облачной платформы Alibaba cloud.

Перед началом Вам необходимо скачать актуальный образ MikoPBX с расширением **.raw**. Сделать это можно на [github MikoPBX](https://github.com/mikopbx/core/releases).

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
Регион у хранилища для образа и регион виртуальной машины должны совпадать!
{% endhint %}

Нажмите "**OK**".

<figure><img src="../../.gitbook/assets/bucketParameters.jpg" alt=""><figcaption><p>Параметры Bucket'а</p></figcaption></figure>

4. Перейдите в созданный bucket, нажав на его название в разделе "**Buckets**":

<figure><img src="../../.gitbook/assets/bucketName.jpg" alt=""><figcaption><p>Созданный bucket</p></figcaption></figure>

5. Нажмите "**Upload object**" и загрузите ранее скаченный файл образа диска в формате **.raw** (все параметры оставьте по умолчанию).

<figure><img src="../../.gitbook/assets/uploadObject.jpg" alt=""><figcaption><p>Элемент "Upload object"</p></figcaption></figure>

6. После того, как файл образа диска будет загружен, Вам необходимо скопировать ссылку на него. Для этого нажмите "**View Details**" справа от названия файла, в открывшемся меню скопируйте ссылку из поля "**URL**".

<figure><img src="../../.gitbook/assets/URL.jpg" alt=""><figcaption><p>Ссылка на файл образа диска</p></figcaption></figure>

### Создание образа

1. Вернитесь в ECS Console ([ссылка](https://ecs.console.aliyun.com/home)). Перейдите в раздел "**Images**".

<figure><img src="../../.gitbook/assets/imagesSection.jpg" alt=""><figcaption><p>Раздел "Images"</p></figcaption></figure>

2. Нажмите "**Import Image**" для испорта образа из хранилища (**Bucket**):

<figure><img src="../../.gitbook/assets/importImageButton.jpg" alt=""><figcaption><p>Элемент "Import Image"</p></figcaption></figure>

3. В открывшейся вкладке выберите "**Linux Operating System**" и нажмите "**Next**".
4. Введите/выберите следующие параметры для образа:

* "**Image File URL**" - вставьте ранее скопированную ссылку на загруженный файл образа диска.
* "**Image Name**" - введите произвольное, **уникальное** название для Вашего образа.
* "**OS Type**" - linux
* "**OS Version**" - Others Linux
* "**Architecture**" - 64-bit OS
* Уберите галочку с чек-бокса "**Check After Import**".

Нажмите "**OK**" для создания образа. Дождитесь его создания (при завершении в поле Status вы увидите надпись <mark style="color:green;">Available</mark>)

<figure><img src="../../.gitbook/assets/imageParametersMikoPBX.jpg" alt=""><figcaption><p>Параметры импорта образа</p></figcaption></figure>

## Создание пары SSH-ключей

Далее необходимо создать и добавить связку SSH ключей в Alibaba Cloud.

1. В ESS Консоли перейдите в раздел "**Network Security**" -> "**Key Pairs**":

<figure><img src="../../.gitbook/assets/keyPairsSection.jpg" alt=""><figcaption><p>Раздел "Key Pairs"</p></figcaption></figure>

2. Нажмите "**Create SSH Key Pair**".

<figure><img src="../../.gitbook/assets/createSSHKeyPair.jpg" alt=""><figcaption><p>Элемент "Create SSH Key Pair"</p></figcaption></figure>

3. Далее необходимо сгенерировать пару SSH-ключей. Про то как это сделать - Вы можете прочитать [здесь](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/). Далее заполните все необходимые данные:

* "**Name**" - произвольное название для создаваемой связки ключей
* "**Creation Mode**" - Import
* "**Public Key**" - вставьте Ваш **публичный** ключ, сгенерированный ранее
* "**Resource Group**" - выберите Вашу ресурсную группу в облаке

Нажмите "**OK**" для создания связки ключей в облаке

<figure><img src="../../.gitbook/assets/sshKeyPairParameters.jpg" alt=""><figcaption><p>Параметры создаваемой связки ключей</p></figcaption></figure>

## Создание группы правил

Перед созданием виртуальной машины так же нужно создать и группу правил (firewall).

1. Перейдите в раздел "**Network & Security**" -> "**Security Groups**"

<figure><img src="../../.gitbook/assets/securityGroupsSection.jpg" alt=""><figcaption><p>Раздел "Security Groups"</p></figcaption></figure>

2. Нажмите "**Create Security Group**":

<figure><img src="../../.gitbook/assets/createSecurityGroup.jpg" alt=""><figcaption><p>Элемент для создания новой группы правил</p></figcaption></figure>

3. Укажите следующие параметры для группы правил:

* "**Security Group**" - введите произвольное название для группы правил.
* "**Network**" - выберите вашу сеть. Если она не созданна - нажмите "**Create VPC**" справа от поля.
* "**Security Group**" - Basic Security Group.
* "**Resource Group**" - выбрите Вашу группу ресурсов.
* Разрешите все входящие подключения (пример на скриншоте ниже). Все исходящие подключения разрешены по умолчанию.

{% hint style="info" %}
Обязательно настройте firewall в самой MikoPBX как можно раньше после создания виртуальной машины. Подробнее про то как это сделать, Вы можете прочитать [здесь](../../manual/connectivity/firewall.md).
{% endhint %}

Нажмите "**Create Security Group**".

<figure><img src="../../.gitbook/assets/ParametersOfTheSecurity (3).jpg" alt=""><figcaption><p>Параметры группы правил</p></figcaption></figure>

## Создание виртуальной машины

1. Перейдите в раздел "**Instances & Images**" -> "**Instances**":

<figure><img src="../../.gitbook/assets/instancesSection.jpg" alt=""><figcaption><p>Раздел "Instances"</p></figcaption></figure>

2. Нажмите "**Create Instance**" для создания новой виртуальной машины.

<figure><img src="../../.gitbook/assets/createInstanceButton.jpg" alt=""><figcaption><p>Элемент "Create Instance"</p></figcaption></figure>

3. Выберите параметры для вашей виртуальной машины:

* "**Billing Method**" - выберите вариант оплаты ВМ.
* "**Region**", "**Network and Zone**" - выберите параметры региона и зоны, подходящие Вам.
* "**Instance**" - выберите конфигурацию Вашей виртуальной машины.

<figure><img src="../../.gitbook/assets/VMParameters1.jpg" alt=""><figcaption><p>Параметры виртуальной машины №1</p></figcaption></figure>

4. Выберите параметры для вашей виртуальной машины:

* "**Image**" - выберите "**Custom Images**" -> Загруженный ранее образ
* "**Storage**" - выберите тип и размер "**System Disk**". 20 Гб - минимально возможный в Alibaba Cloud.
* Добавьте второй диск, нажав "**Add Data Disk**". Укажите его тип и размер.

{% hint style="info" %}
Рекомендуемый размер диска для хранения записей разговора - не менее 50ГБ. В данной инструкции, в качестве примера, используется диск размером 30ГБ.
{% endhint %}

<figure><img src="../../.gitbook/assets/VMParameters2.jpg" alt=""><figcaption><p>Параметры виртуальной машины №2</p></figcaption></figure>

5. Выберите параметры сети для Вашей ВМ. Группа правил будет назначена автоматически (ранее созданная):

<figure><img src="../../.gitbook/assets/bandwidthsSecurityGroupsSection.jpg" alt=""><figcaption><p>Параметры сети</p></figcaption></figure>

6. Нажмите "**Create Order**".

<figure><img src="../../.gitbook/assets/createOrder.jpg" alt=""><figcaption><p>Элемент "Create Order"</p></figcaption></figure>

## Подключение к консоли MikoPBX

В разделе "**Instances**" перейдите к созданной виртуальной машине, нажав на ее название.

<figure><img src="../../.gitbook/assets/goToVM.jpg" alt=""><figcaption><p>Переход к созданной виртуальной машине</p></figcaption></figure>

### Подключение из встроенной в облако консоли

1. Нажмите "**Connect**".

<figure><img src="../../.gitbook/assets/connectToTheConsole.jpg" alt=""><figcaption><p>Элемент "Connect"</p></figcaption></figure>

2. Выберите "**VNC**". Произойдет подключение в новой вкладке Вашего браузера.

<figure><img src="../../.gitbook/assets/VNCConsole.jpg" alt=""><figcaption><p>VNC консоль</p></figcaption></figure>

### Подключение по SSH

{% hint style="info" %}
Подробнее про SSH-подключения Вы можете узнать [в этом блоке статей](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/). В данной документации, в качестве примера будет продемонстрировано подключение по SSH через PowerShell.
{% endhint %}

Введите следующую команду для SSH-подключения:

```powershell
ssh -i C:\Users\username\.ssh\id_ed25519 root@ip-adress
```

Замените `C:\Users\username\.ssh\id_ed25519` на путь к ssh-ключам; `root`- на Ваше имя для ssh-авторизации (если оно было изменено при создании ВМ); `ip-adress` - на Внешний адрес MikoPBX.

<figure><img src="../../.gitbook/assets/sshCommandPowerShell.jpg" alt=""><figcaption><p>Команда для ssh-подключения</p></figcaption></figure>

Произойдет подключение по SSH:

<figure><img src="../../.gitbook/assets/SSHConnection.jpg" alt=""><figcaption><p>SSH-подключение</p></figcaption></figure>

## Первая авторизация в WEB-интерфейсе

На главной странице виртуальной машины находятся несколько важных параметров для авторизации в WEB-интерфейсе.

<figure><img src="../../.gitbook/assets/authorizationParametersRU.jpg" alt=""><figcaption><p>Важные параметры для авторизации в web-интерфейс</p></figcaption></figure>

Вставьте IP-адрес в адресную строку браузера - Вы попадете на страницу авторизации в web-интерфейс MikoPBX.

{% hint style="info" %}
Данные для входа:

* **Username** - admin
* **Password** - ID Вашей виртуальной машины
{% endhint %}

<figure><img src="../../.gitbook/assets/mikopbxWEB.jpg" alt=""><figcaption><p>WEB-интерфейс MikoPBX</p></figcaption></figure>
