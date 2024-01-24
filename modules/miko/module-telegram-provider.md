# Настройка звонков через Telegram

{% hint style="danger" %}
**Обратите внимание**: Все данные авторизации / секретные коды / данные переписки Telegram используются исключительно ЛОКАЛЬНО на вашей АТС и НЕ передаются на сервера МИКО.
{% endhint %}

Модуль состоит из нескольких компонент:

* **Шлюз SIP ↔ Telegram** - позволяет **принимать звонки** от telegram и **совершать исходящие** звонки через telegram. Шлюз не зависит от прочих компонент, они лишь его дополняют
* **Телеграм клиент** - работает совместно с ботом, во время входящего звонка звонящему отправляется клавиатура для ввода добавочного номера.
* **Телеграм бот** - служит для формирования клавиатуры и отправки тональных команд в текущий канал. Для работы необходимо предварительно получить токен через интерфейс бота «**@BotFather**»

При звонке в компанию клиенту будет отправлена с сообщением **клавиатура для ввода добавочного номера**:

<figure><img src="../../.gitbook/assets/tgScreenshot1.png" alt=""><figcaption><p>Клавиатура для ввода добавочного номера</p></figcaption></figure>

После завершения звонка клавиатура будет скрыта.

После неудачного звонка из компании (клиент не ответил). Клиенту будет отправлена **визитка**:

<figure><img src="../../.gitbook/assets/tgScreenshot2.png" alt=""><figcaption><p>Визитка</p></figcaption></figure>

Текст визитки и клавиатуры ввода добавочного можно исправить в настройках **модуля**:

<mark style="color:red;">**Screenshot here!**</mark>

## Настройка Telegram <a href="#nastrojka_telegram" id="nastrojka_telegram"></a>

### Аккаунт <a href="#nastrojka_telegram" id="nastrojka_telegram"></a>

Необходимо настроить видимость номера телефона в разделе "**Конфиденциальность**" , а так же разрешить Всем звонить на номер телефона в разделе "**Звонки**":

<figure><img src="../../.gitbook/assets/PrivacyTelegram.png" alt=""><figcaption><p>Настройки приватности номера телефона</p></figcaption></figure>

### Telegram Bot

**Создайте бота**

1. Откройте чат с **@BotFather** в любом удобном клиенте Телеграм

<figure><img src="../../.gitbook/assets/botFather.png" alt=""><figcaption><p>@BotFather</p></figcaption></figure>

2. Отправьте команду

```
/newbot
```

3. Введите «**name**» нового бота, к примеру **new\_keyboard\_bot**, имя должно быть коротким и понятным, **имя увидит клиент при получении клавиатуры**
4. Введите «**username**» для нового бота, к примеру **newest\_keyboard\_bot**

<figure><img src="../../.gitbook/assets/botFatherScreen2.png" alt=""><figcaption><p>@BotFather</p></figcaption></figure>

#### **Включите Inline Mode**

1.  Отправьте боту команду

    ```
    /setinline
    ```
2. Выберите созданного ранее бота

<figure><img src="../../.gitbook/assets/botFatherScreen3.png" alt=""><figcaption><p>Inline Mode in @BotFather</p></figcaption></figure>

#### **Получите Token**

1.  Отправьте команду

    ```
    /mybots
    ```
2. Выберите созданного ранее бота
3. Кликните по кнопке «**API Token**»
4. Скопируйте значение «**token**»

#### **Начните диалог с ботом**

1.  Отправьте из аккаунта телеграмм вашему боту команду

    ```
    /start
    ```
2. Напишите боту произвольное сообщение, к примеру «**Привет**!»

<figure><img src="../../.gitbook/assets/HelloWorld)).png" alt=""><figcaption><p>Hello world to your bot</p></figcaption></figure>

{% hint style="warning" %}
С ботом должен быть открытый диалог, если не отправить ему команду «**/start**», то часть функционала работать не будет.
{% endhint %}

## Получение "App api\_id" и "App api\_hash" <a href="#poluchenie_app_api_id_i_app_api_hash" id="poluchenie_app_api_id_i_app_api_hash"></a>

{% hint style="danger" %}
«**App api\_id**» и «**App api\_hash**» - это секретные параметры, которые привязываются к вашему _telegram_ аккаунту. **Никому не сообщайте эти значения!** Они необходимы для подключения к API Telegram. Подробнее можно почитать в [документации](https://core.telegram.org/api/obtaining\_api\_id).
{% endhint %}

1. Перейдите по ссылке [https://my.telegram.org/auth](https://my.telegram.org/auth)

<figure><img src="../../.gitbook/assets/linktoTelegram.png" alt=""><figcaption><p>Telegram page</p></figcaption></figure>

2. Укажите номер телефона, привязанный к telegram аккаунту
3. Перейдите в раздел «**API development tools**»

<figure><img src="../../.gitbook/assets/API.png" alt=""><figcaption><p>API development tools</p></figcaption></figure>

4. Опишите параметры нового приложения:

* «**App title**» - заголовок приложения
* «**Short name**» - уникальное короткое имя приложения
* «**URL**»- произвольный адрес сайта
* «**Platform**» - Other
* «**Description**» - произвольное описание

Выполните действие «**Create application**»

<figure><img src="../../.gitbook/assets/settingsOfNewApplicationInTG.png" alt=""><figcaption><p>AppParameters</p></figcaption></figure>

5. Скопируйте значения «**App api\_id**» и «**App api\_hash**»

<figure><img src="../../.gitbook/assets/yourApi.png" alt=""><figcaption><p>«<strong>App api_id</strong>» и «<strong>App api_hash</strong>»</p></figcaption></figure>

## Настройки модуля MikoPBX <a href="#nastrojki_modulja_mikopbx" id="nastrojki_modulja_mikopbx"></a>

{% hint style="warning" %}
При восстановлении из бекапа может потребоваться повторная авторизация в telegram.
{% endhint %}

1. Установите дополнительный модуль «**Провайдер Telegram**», (см. инструкции по установке модулей в разделе [Управление модулями](../../manual/modules/pbx-extension-modules/))
2. Ведите значения «**App api\_id**» и «**App api\_hash**», полученные на предыдущем этапе, а так же **Token** бота, который вы получили при создании бота.

Сохраните изменения

<figure><img src="../../.gitbook/assets/RUMikoPBXProvider.png" alt=""><figcaption></figcaption></figure>

3. Добавьте новый номер по кнопке «**Добавить**»
4. Запустите процесс авторизации для **Шлюза**, **Клиента Telegram**, для **Бота**

<figure><img src="../../.gitbook/assets/numberMikoPBX.png" alt=""><figcaption></figcaption></figure>

5. Авторизуйте все линии. После подключения отобразится "Линии подключены"
