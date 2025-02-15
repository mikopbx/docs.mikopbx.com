---
description: Инструкции по подключению софтфона 3CX Softphone
---

# 3CX Softphone

**3CX** — это SIP софтфон, предназначенный для бизнеса, который обеспечивает голосовые и видеозвонки, а также поддержку многоканальных звонков и конференц-связи.&#x20;

<figure><img src="../../.gitbook/assets/3cxSoftPhone.jpg" alt=""><figcaption><p>Интерфейс софтфона 3CX</p></figcaption></figure>

1. Скачайте и установите софтфон с оффициального сайта ([ссылка](https://www.3cx.com/voip/softphone/)).

<figure><img src="../../.gitbook/assets/mainpage.jpg" alt=""><figcaption><p>Стартовое окно</p></figcaption></figure>

2. Добавьте новый SIP-аккаунт; для этого нажмите "**New**":

<figure><img src="../../.gitbook/assets/newConnection.jpg" alt=""><figcaption><p>Добавление нового SIP-аккаунта</p></figcaption></figure>

3. Далее укажите все необходимые данные для SIP-подключения:

* **Account name** - произвольное
* **CallerID** - имя сотрудника
* **Extension** - внутренний номер сотрудника (Из карточки сотрудника)
* **ID** - внутренний номер сотрудника (Из карточки сотрудника)
* **Password** - пароль для SIP-подключения (Из карточки сотрудника)

<figure><img src="../../.gitbook/assets/credentials.jpg" alt=""><figcaption><p>Данные для подключения</p></figcaption></figure>

4. В разделе "**My location**", укажите IP-адрес Вашей MikoPBX. Если она расположена в одной локальной сети с софтфоном - выберите вариант "**I am in the office - local IP**", если вы хотите подключиться с помощью внешнего IP-адреса, выберите "**I am out of the office - external IP**"

После заполнения этих данных, нажмите "**OK**".&#x20;

<figure><img src="../../.gitbook/assets/myLocation.jpg" alt=""><figcaption><p>Раздел "My location"</p></figcaption></figure>

5. Выберите добавленую учетную запись и нажмите "**OK**":

<figure><img src="../../.gitbook/assets/connection.jpg" alt=""><figcaption><p>Выбор учетной записи</p></figcaption></figure>

Произойдет соединение. Вы можете проверить его успешность по индикатору подключения сотрудника:

<figure><img src="../../.gitbook/assets/successfulConnection (2).jpg" alt=""><figcaption><p>Успешное подключение!</p></figcaption></figure>
