---
description: Инструкции по подключению софтфона Groundwire
---

# Groundwire

**Groundwire** обладает низким энергопотреблением за счет того, что не поддерживает постоянно с сервером SIP сессию. При поступлении входящего звонка сперва на смартфон поступей PUSH уведомление, а затем запускается SIP клиент и происходит соединение с сервером.

## Подключение софтфона

1. Скачайте и откройте приложение **Groundwire** на смартфоне.

<figure><img src="../../.gitbook/assets/startPageGroundWire.jpg" alt="" width="295"><figcaption><p>Главное меню Groundwire</p></figcaption></figure>

2. Перейдите в настройки **Groundwire**, используя соответствующий элемент в верхней части экрана.

<figure><img src="../../.gitbook/assets/settingsButton.jpg" alt="" width="295"><figcaption><p>Переход в настройки</p></figcaption></figure>

3. Перейдите в раздел "**Accounts**":

<figure><img src="../../.gitbook/assets/accountsSection.jpg" alt="" width="295"><figcaption><p>Переход в раздел с аккаунтами</p></figcaption></figure>

4. Добавьте новый аккаунт, нажав на "+" в верхней части экрана.
5. Выберите тип "**Generic SIP Account**".
6. Заполните необходимые данные:

* "**Title**" - Название аккаунта. (произвольное)
* "**Username**" - Внутренний номер сотрудника. (например, 201)
* "**Password**" - Пароль для SIP из карточки сотрудника.
* "**Domain**" - IP-адрес Вашей станции MikoPBX.

<figure><img src="../../.gitbook/assets/dataForAuth.jpg" alt="" width="295"><figcaption><p>Данные для SIP-подключения</p></figcaption></figure>

7. Перейдите в раздел "**Advanced Settings**".
8. В разделе "**Audio Codecs**" -> "**Codecs for Wi-Fi**" укажите необходимые Вам кодеки для использования.&#x20;

<figure><img src="../../.gitbook/assets/AudioCodecs.jpg" alt="" width="295"><figcaption><p>Аудио кодеки</p></figcaption></figure>

9. Перейдите в раздел "**Caller Id Method**". Укажите "**P-Asserted-Identify".**

<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="295"><figcaption><p>Раздел "Caller Id Method"</p></figcaption></figure>

Сохраните параметры. Произойдет соединение. По индикатору в разделе "Сотрудники" в MikoPBX, Вы можете убедиться, что оно было успешным.

<figure><img src="../../.gitbook/assets/successfulConnection (1).jpg" alt=""><figcaption><p>Успешное SIP-подключение</p></figcaption></figure>

## Настройка отслеживания статусов сотрудников

Существует возможность вывода информации о текущем статусе сотрудника (пример - изображение далее)

<figure><img src="../../.gitbook/assets/extensionsStatuses.jpg" alt="" width="295"><figcaption><p>Статусы сотрудников</p></figcaption></figure>

Для реализации, наполните раздел "Quick Dial" сотрудниками. Для этого нажмите "EDIT" в верхей части экрана, наполните список, используя элемент "+", в нижней части экрана:

* **"Title"** - имя сотрудника, которое будет отображено в разделе "**Quick Dial**".
* "**Number or SIP Address**" - внутренний номер сотрудника.

Нажмите "**Save**"

<figure><img src="../../.gitbook/assets/newQuickDialContact.jpg" alt="" width="295"><figcaption><p>Новый сотрудник в разделе "Quick Dial"</p></figcaption></figure>

## Настройка GroundWire Softphone с поддержкой шифрования&#x20;

1. Перейдите в раздел настроек учетной записи сотрудника в интерфейсе MikoPBX. Измените "**Транспортный протокол**" на "tls".

<figure><img src="../../.gitbook/assets/tlsMode.jpg" alt=""><figcaption><p>Настройка транспортного протокола</p></figcaption></figure>

2. Перейдите в интерфейс **Groundwire ->** Настройка SIP-аккаунта сотрудника. Измените **"Domain"** на **"АдресMikoPBX>:НомерПортаДляTLS**".

{% hint style="info" %}
Номер порта для TLS Вы можете найти/изменить в web-интерфейсе MikoPBX: "**Общие настройки**" -> "**SIP**".
{% endhint %}

<figure><img src="../../.gitbook/assets/tlsDomain.jpg" alt="" width="295"><figcaption><p>Поле "Domain"</p></figcaption></figure>

3. Перейдите в раздел "**Advanced Settings**" -> "**Transport Protocol**". Измените "**udp**" на "**tls (sip)**".

<figure><img src="../../.gitbook/assets/tlsSipss (1).jpg" alt="" width="295"><figcaption><p>Изменение транспортного протокола</p></figcaption></figure>

4. Перейдите в раздел "**Advanced Settings**" Вашего SIP-аккаунта -> "**Secure Calls**".

Измените параметры "**Incoming Calls**", "**Outgoing Calls**" в разделе "**SDES**" на "**Required**":

<figure><img src="../../.gitbook/assets/SDES.jpg" alt="" width="295"><figcaption><p>SDES параметры</p></figcaption></figure>
