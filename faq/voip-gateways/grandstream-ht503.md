---
description: Подключение Voip шлюза Grandstream HT503
---

# Grandstream HT503

**Grandstream HT503** - FXS-FXO шлюз, подходит для подключения как одной городской линии, так и одного телефона. В шлюзе предусмотрена поддержка протокола T.38. Шлюз может выступать в роли роутера. В примере рассмотрим, как подключить к MikoPBX городскую линию через FXO порт шлюза Grandstream HT503.

Подключите сетевой шнур в WAN порт, городскую линию в порт LINE, подайте питание на шлюз.

## Подключение к WEB-интерфейсу шлюза

По умолчанию подключение к Web-интерфейсу отключено. Для доступа к Web-интерфейсу необходимо сделать следующие шаги:

1. Подключите аналоговый телефон к шлюзу и наберите `***` . Вы попадете в голосовое меню шлюза.
2. Наберите 12, затем 9, таким образом, Вы включите доступ к Web интерфейсу через WAN порт
3. Наберите `***`.
4. Затем 99, затем 9 – устройство перезагрузится.
5. Чтобы узнать IP-адрес WAN порта, наберите на аналоговом телефоне `***`, затем 02. Или посмотрите IP адрес на вашем роутере.
6. В адресной строке вашего браузера введите полученный IP адрес. Для входа в Web-интерфейс введите пароль - `admin`.

## Настройка шлюза в MikoPBX <a href="#nastrojka_shljuza_v_mikopbx" id="nastrojka_shljuza_v_mikopbx"></a>

1. Перейдите в web-интерфейс MikoPBX на вкладку **"Маршрутизация" → "Провайдеры телефонии"**. Нажмите на кнопку **Подключить SIP** для добавления новой учетной записи для шлюза:

<figure><img src="../../.gitbook/assets/newSIPProviderMikoPBX.jpg" alt=""><figcaption><p>Новое SIP подключение в MikoPBX</p></figcaption></figure>

2. В web-интерфейсе шлюза перейдите на страницу **STATUS**. Здесь отображается информация о IP адресе шлюза. Скопируйте его.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>IP-адрес шлюза</p></figcaption></figure>

3. Заполните следующие параметры:

* **Название провайдера** - произвольное
* **Тип учетной записи** - Исходящая регистрация
* **Хост или IP адрес** - IP-адрес шлюза
* **Логин, пароль** - произвольные
* **Режим DTMF** - rfc4733

<figure><img src="../../.gitbook/assets/providerParameters.jpg" alt=""><figcaption><p>Параметры провайдера</p></figcaption></figure>

4. Перейдите в Расширенные настройки. Укажите порт соединения - **5062**

Сохраните настройки.

<figure><img src="../../.gitbook/assets/portSIP.jpg" alt=""><figcaption><p>Параметры порта соединения</p></figcaption></figure>

5. Перейдите в раздел "**Система**" -> "**Общие настройки**".

<figure><img src="../../.gitbook/assets/generalSettings.jpg" alt=""><figcaption><p>Раздел "<strong>Общие настройки</strong>"</p></figcaption></figure>

6. Перейдите в раздел "**Аудио/Видео кодеки**". Оставьте включенным только кодек "**G.711 A-law**":

<figure><img src="../../.gitbook/assets/g711AlawCodec.jpg" alt=""><figcaption><p>Раздел "<strong>Аудио/Видео кодеки</strong>"</p></figcaption></figure>

## Основные настройки шлюза <a href="#osnovnye_nastrojki_shljuza" id="osnovnye_nastrojki_shljuza"></a>

Снова переходим в web-интерфейс шлюза. Если необходимо внести сетевые настройки, то перейдем во вкладку **BASIC SETTINGS**. Мы можем выбрать тип подключения DHCP, PPPoE, статический IP адрес.

<figure><img src="../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

На вкладке **BASIC SETTINGS** листаем до низа страницы. В поле **Unconditional Call Forward to VOIP** необходимо указать ваш User ID (DID номер для MikoPBX), Sip Server и Sip Destination Port, относящиеся к настройкам FXO порта.

## Настройки FXO порта <a href="#nastrojki_fxo_porta" id="nastrojki_fxo_porta"></a>

Приступим к настройке FXO порта. Для этого переходим во вкладку **FXO PORT**. Здесь необходимо заполнить следующие поля:

* **Account Active** - Yes
* **Primary SIP Server** - Указываем IP адрес или доменное имя вашей АТС MikoPBX.
* **SIP User ID** - Имя пользователя
* **Authenticate ID** - Идентификационное имя, которое идет в соответствии с паролем
* **Authenticate Password** - Пароль для регистрации на АТС
* **Name** - Отображаемое имя, при звонках

Для корректного завершения вызов. Укажите следующие параметры:

* **Enable Current Disconnect** - Устанавливаем данный параметр в No
* **Enable Tone Disconnect** - Устанавливаем данный параметр в Yes
* **PSTN Ring Thru FXS** - Устанавливаем данный параметр в No, этот параметр отвечает за перевод вызовов с FXO порта на FXS порт
* **Wait for Dial-Tone** - Устанавливаем данный параметр в No
* **Stage Method (1/2)** - Устанавливаем данный параметр в 1 (При звонке на существующую городскую линию, шлюз будет обрабатывать вызов и автоматически переадресовывать на нужный SIP ID или номер

Настройки подробно приведены ниже на скриншотах:

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

Если все корректно настроено, то во вкладке **STATUS** мы увидим зарегистрированные порты:

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption><p>Статус портов</p></figcaption></figure>
