---
description: Подключение и настройка телефона Fanvil X3SP
---

# Fanvil X3SP

IP-телефон Fanvil X3SP сочетает современный дизайн, HD-качество звука и интуитивно понятный интерфейс, обеспечивая легкость в настройке и установке.&#x20;

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Телефон Fanvil X3SP</p></figcaption></figure>

{% hint style="info" %}
Убедитесь, что в разделе "**Система**" -> "**Общие настройки**" -> "**Аудио/Видео кодеки**", кодек **G.711 A-law** включен.
{% endhint %}

## Настройки внутри раздела "Сотрудники"

1. Перейдите в web-интерфейс MikoPBX. В раздел **«Телефония» -> «Сотрудники»:**

<figure><img src="../../.gitbook/assets/extensionsSectionMikoPBX.png" alt=""><figcaption><p>Раздел "Сотрудники"</p></figcaption></figure>

2. Напротив учетной записи сотрудника, которого Вы будете подключать к телефону, скопируйте пароль для SIP:

<figure><img src="../../.gitbook/assets/copiengSIPPassword.png" alt=""><figcaption><p>Копирования пароля для SIP</p></figcaption></figure>

## Настройка телефона <a href="#nastrojka_telefona" id="nastrojka_telefona"></a>

1. Подключите телефон к сети ethernet, используя порт с надписью "**internet".**

{% hint style="info" %}
Если в вашей сети настроен DHCP сервер, то телефон получит IP адрес автоматически.
{% endhint %}

2. Нажмите на клавишу "OK" на, чтобы узнать IP-адрес телефона, по которому к нему будет произодиться подключение из браузера. Далее адрес будет отображен на экране телефона.
3. Перейдите по ссылке <mark style="color:blue;">http://ip\_адрес\_телефона</mark> в вашем браузере.

{% hint style="info" %}
Стандартные данные для первой авторизации:

* Username: admin
* Login: admin
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Форма авторизации</p></figcaption></figure>

4. Перейдите в раздел "**Аккаунт**".

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Раздел "Аккаунт"</p></figcaption></figure>

5. Заполните следующие параметры:

* «**Имя пользователя**» - внутренний номер сотрудника.
* «**Имя регистрации**» - внутренний номер сотрудника.
* «**Пароль**» - пароль для SIP из карточки сотрудника в MikoPBX.
* «**Адрес SIP-сервера**» - адрес MikoPBX.

Перейдите в расширенные настройки и заполните следующие параметры:

* Укажите «**Caller ID Header**» в значение «**PAI-RPID-FROM**».
* «**Приоритет гарнитуры**» установите в значение «**Вкл.**».
* Включите «**Интерком**».

Сохраните настройки нажав кнопку «**Применить**». Статус телефона отобразиться как «**Зарегистрированно**».

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Параметры SIP аккаунта</p></figcaption></figure>

После успешного подключения телефона, в интерфейсе MikoPBX индикатор состояния подключения загорится зелёным:

<figure><img src="../../.gitbook/assets/successfulConnectionMikoPBX.jpg" alt=""><figcaption><p>Индикатор успешного подключения в интерфейсе MikoPBX</p></figcaption></figure>
