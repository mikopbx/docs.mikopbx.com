---
description: Вариант переноса данных с использованием rsync (предпочтительный)
---

# Перенос с помощью rsync

В данной статье будет разобран вариант переноса данных на новый хост с помощью rsync. Данный вариант - перенос с использованием генерируемого ключа для ssh-авторизации (предпочтительный). Этот способ является самым надёжным из представленных в разделе, поэтому и является рекомендуемым к использованию.

Схематично процесс переноса можно изобразить следующим образом:

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption><p>Схема переноса данных</p></figcaption></figure>

## Создание файла для хранения скрипта и наполнение его содержимым <a href="#variant_3" id="variant_3"></a>

1. Для начала нам необходимо установить SSH соединение с **новой** MikoPBX. Прочитать как сделать это, можно в [этой статье](../../troubleshooting/connecting-to-a-pbx-using-ssh/).&#x20;

<figure><img src="../../../.gitbook/assets/sshConnection (2).png" alt=""><figcaption><p>Успешное SSH соединение с новой MikoPBX</p></figcaption></figure>

2. Далее переходим в консоль (**\[9] Console**). Первым делом необходимо создать директорию для хранения файла со скриптом. Используйте следующую команду:

```
mkdir -p /storage/usbdisk1/transfer
```

3. Перейдите в созданную директорию:

```
cd /storage/usbdisk1/transfer
```

4. Создадим файл "**transfer-rsync.sh**"  для хранения скрипта:

```
touch transfer-rsync.sh
```

<figure><img src="../../../.gitbook/assets/firstPartOfCommands (1).png" alt=""><figcaption><p>Выполнение команд для создания файла</p></figcaption></figure>

5. Далее необходимо заполнить файл содержимым (скриптом). Ознакомиться с ним Вы можете[ по ссылке](https://gist.github.com/excla1mmm/c9891306b459cac0c7ea3c785ab0936e).

```php
curl -o /storage/usbdisk1/transfer/transfer-rsync.sh https://gist.githubusercontent.com/excla1mmm/c9891306b459cac0c7ea3c785ab0936e/raw/ec57ab60ee48112b4a16635e7b47955e5a044513/transfer-rsync.sh
```

## Запуск и работа со скриптом

1. На данном этапе необходимо сделать файл исполняемым. Для этого используйте следующую команду:

```
chmod +x transfer-rsync.sh
```

2. Запустите скрипт, используя команду:

```
./transfer-rsync.sh
```

3. Для начала будет предложено ввести необходимые данные о вашей старой станции MikoPBX:

* IP-адрес вашей старой станции
* Имя для ssh-авторизации
* Порт для ssh-авторизации

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Ввод необходимых данных</p></figcaption></figure>

4. Далее будет предложено сгенерировать новый ключ. В случае, если ранее вы этого не делали, введите "y" для подтверждения. Если ранее вы уже генерировали ключ для доступа ко второй MikoPBX - введите "n":

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>Генерация нового ключа</p></figcaption></figure>

5. Будет создан новый ключ. Вам необходимо скопировать его и вставить в web-Интерфейсе старой MikoPBX. Сделать это нужно в разделе "**Общие настройки**" -> "**SSH**" -> Поле "**SSH Authorized keys**"

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>Сгенерированный ключ ssh</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/SshAuthorizedKeysField.png" alt=""><figcaption><p>Вставленный ключ</p></figcaption></figure>

6. После того, как вы сохранили ключ на старой MikoPBX, подождите несколько секунд и нажмите любую клавишу для продолжения выполнения скрипта.

Будет произведен перенос всех данных на новый хост. Это может занять некоторое время.

{% hint style="danger" %}
После переноса обязательно проверяйте целостность всех данных, перед тем, как сбрасывать старую MikoPBX!
{% endhint %}

<figure><img src="../../../.gitbook/assets/successfulTransfer.png" alt=""><figcaption><p>Успешный перенос</p></figcaption></figure>
