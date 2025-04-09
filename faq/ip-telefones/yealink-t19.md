---
description: Подключение и настройка телефона Yealink T19
---

# Yealink T19

Yealink SIP-T19 - бюджетная модель для малого и среднего бизнеса. В данной инструкции будет изображён процесс подключения и настройки данного телефона к MikoPBX.

{% hint style="info" %}
Убедитесь, что в разделе "**Система**" -> "**Общие настройки**" -> "**Аудио/Видео кодеки**", кодек **G.711 A-law** включен.
{% endhint %}

### Настройки внутри раздела "Сотрудники" <a href="#uchetnaja_zapis_na_mikopbx" id="uchetnaja_zapis_na_mikopbx"></a>

1. Перейдите в web-интерфейс MikoPBX. В раздел **«Телефония» -> «Сотрудники»:**

<figure><img src="../../.gitbook/assets/extensionsSectionMikoPBX.png" alt=""><figcaption><p>Раздел "Сотрудники" в MikoPBX</p></figcaption></figure>

2. Напротив учетной записи сотрудника, которого Вы будете подключать к телефону, скопируйте пароль для SIP:

<figure><img src="../../.gitbook/assets/copiengSIPPassword.png" alt=""><figcaption><p>Элемент для копирования пароля для SIP</p></figcaption></figure>

### Настройка Yealink T19 <a href="#nastrojka_snom_d120" id="nastrojka_snom_d120"></a>

Подключите телефон к сети ethernet, используя порт с надписью **internet:**

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption><p>internet-порт</p></figcaption></figure>

{% hint style="success" %}
Если в вашей сети настроен DHCP сервер, то телефон получит IP адрес автоматически.
{% endhint %}

Нажмите на клавишу "OK", чтобы узнать IP-адрес телефона, по которому к нему будет произодиться подключение из браузера. Далее адрес будет отображен на экране телефона.

Перейдите по ссылке <mark style="color:blue;">http://ip\_адрес\_телефона</mark> в вашем браузере.

{% hint style="info" %}
Стандартные данные для первой авторизации:

Username: admin

Login: admin
{% endhint %}

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption><p>Окно авторизации</p></figcaption></figure>

Далее выполните настройку SIP-аккаунта в Вашем телефоне:

* «**Register Name**», «**User Name**» - внутренний номер подключаемой учетной записи сотрудника.&#x20;
* «**Password**» - пароль учетной записи Вашего сотрудника (Пароль для SIP)
* **«Server Host»** - IP-адрес MikoPBX

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption><p>Параметры учетной записи в web-интерфейсе телефона</p></figcaption></figure>

В разделе «**Features**» настройте параметр в «**Intercom**» -> "**Intercom Barge**":

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption><p>Настройка параметров</p></figcaption></figure>

После успешного подключения телефона, в интерфейсе MikoPBX индикатор состояния подключения загорится <mark style="color:green;">зелёным</mark>:

<figure><img src="../../.gitbook/assets/greenIndicator (1).png" alt=""><figcaption><p>Индикатор состояния в MikoPBX</p></figcaption></figure>
