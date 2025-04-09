---
description: Подключение и настройка телефона Snom D120
---

# Snom D120

**Snom D120** - телефон начального уровня, базовая модель от **Snom.** В данной инструкции будет изображён процесс подключения и настройки данного телефона к MikoPBX.

{% hint style="info" %}
Убедитесь, что в разделе "**Система**" -> "**Общие настройки**" -> "**Аудио/Видео кодеки**", кодек **G.711 A-law** включен.
{% endhint %}

## Настройки внутри раздела "Сотрудники" <a href="#uchetnaja_zapis_na_mikopbx" id="uchetnaja_zapis_na_mikopbx"></a>

1. Перейдите в web-интерфейс MikoPBX. В раздел **«Телефония» -> «Сотрудники»:**

<figure><img src="../../.gitbook/assets/extensionsSectionMikoPBX.png" alt=""><figcaption><p>Раздел "Сотрудники" в MikoPBX</p></figcaption></figure>

2. Напротив учетной записи сотрудника, которого Вы будете подключать к телефону, скопируйте пароль для SIP:

<figure><img src="../../.gitbook/assets/copiengSIPPassword.png" alt=""><figcaption><p>Элемент для копирования пароля для SIP</p></figcaption></figure>

## Настройка Snom D120 <a href="#nastrojka_snom_d120" id="nastrojka_snom_d120"></a>

Подключите телефон к сети ethernet через порт с надписью LAN.

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption><p>Схема портов Snom D120</p></figcaption></figure>

{% hint style="info" %}
Если в вашей сети настроен DHCP сервер, то телефон получит IP адрес автоматически. При включении адрес будет отображен на аппарате.
{% endhint %}

1. Перейдите в браузере по ссылке [http://ip\_адрес\_телефона](http://xn--ip__-73db0aiba9dzamwqr1b/). Авторизуйтесь в web-интерфейсе.
2. Для настройки учетной записи SIP перейдите в меню «**Setup**» -> «**Identity 1**»:

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption><p>Раздел «<strong>Setup</strong>» -> «<strong>Identity 1</strong>»</p></figcaption></figure>

* «**Identity active**» - установите в значение «**on**».
* «**Displayname**» - произвольное имя, лучше указать латиницей.
* «**Account**» - идентификатор аккаунта - в MikoPBX совпадает с внутренним номером сотрудника.
* «**Password**» - пароль от учетной записи сотрудника (раннее скопированый пароль для SIP)
* «**Registrar**» - адрес вашей MikoPBX.

Нажмите кнопку «**Apply**». Выполните действе «**Re-Register**».

Для проверки перейдите в раздел «**Status**» - «**System Information**». В случае успешной регистрации отобразится:

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption><p>Успешная регистрация</p></figcaption></figure>

В Web интерфейсе MikoPBX, в списке сотрудников статус-иконка изменит цвет на <mark style="color:green;">**зеленый**</mark>**:**

<figure><img src="../../.gitbook/assets/greenIndicator (1).png" alt=""><figcaption><p>Индикатор подключения </p></figcaption></figure>

## Дополнительные настройки <a href="#dopolnitelnye_nastrojki" id="dopolnitelnye_nastrojki"></a>

1. Рекомендуем включить настройку **PnP Config** для использования фунций модуля **"**[**Автоматическая настройка телефонов**](../../modules/miko/module-autoprovision.md)**".**

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption><p>Настройка PnP Config</p></figcaption></figure>

2. В некоторых случаях, при использовании DHCP может использоваться опция 132 для автоматической настройки терминалов. Возможно потребуется отключить использование этой опции для корректной работы телефонного аппарата:

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption><p>Опция 132</p></figcaption></figure>
