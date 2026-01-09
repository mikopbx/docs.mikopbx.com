---
description: Настройка почты для сервиса gmail
---

# Настройка Gmail (oAuth2)

{% hint style="info" %}
Для настройки OAuth 2.0 в Google требуется использовать **URL-адрес станции**.\
Самый простой способ — создать DNS-запись на локальном сервере **или** добавить соответствие IP-адреса и доменного имени в файл `hosts` на устройстве, с которого выполняется настройка.
{% endhint %}

### Настройки аккаунта Google

1. Перед началом настройки, необходимо поменять некоторые параметры аккаунта Google. Для этого перейдите на страницу управления ([ссылка](https://myaccount.google.com/)).

<figure><img src="../../../.gitbook/assets/myaccountgooglecom.png" alt=""><figcaption><p>Страница упрвления аккаунтом Google</p></figcaption></figure>

2. Перейдите в раздел "Безопасность и вход в аккаунт". Убедитесь, что у Вас настроена двухэтапная аутентификация.&#x20;

<figure><img src="../../../.gitbook/assets/2-stepverif_google.png" alt=""><figcaption><p>Настройка двухэтапной аутентификации</p></figcaption></figure>

3. Перейдите в консоль Google Cloud, в раздел "**APIs & Services**" ([ссылка](https://console.cloud.google.com/apis/dashboard)). Создайте проект под текущую задачу.

<figure><img src="../../../.gitbook/assets/googleCloudAPIs.png" alt=""><figcaption><p>Раздел "APIs &#x26; Services" в Google Cloud</p></figcaption></figure>

4. Перейдите в библиотеку APIs (раздел "**Library**").

<figure><img src="../../../.gitbook/assets/googleCloudAPIsLibrary.png" alt=""><figcaption><p>Раздел "Library" в APIs &#x26; services</p></figcaption></figure>

5. Введите в поиске: "gmail api". Перейдите в карточку Gmail API.

<figure><img src="../../../.gitbook/assets/googleCloudGmailAPI.png" alt=""><figcaption><p>Gmail API в библиотеке Google Cloud</p></figcaption></figure>

6. Нажмите "**Enable**" для подключения.

<figure><img src="../../../.gitbook/assets/googleCloudEnableGmailAPI.png" alt=""><figcaption><p>Подключение API</p></figcaption></figure>

7. Перейдите на главную страницу **APIs & Services**. Далее "**OAuth consent screen**".

<figure><img src="../../../.gitbook/assets/googleCloudOAuthConsentScreen.png" alt=""><figcaption><p>Раздел "OAuth consent screen" в APIs &#x26; Services</p></figcaption></figure>

8. Создайте проект (нажмите "**Get started**"). Заполните произвольное название и Вашу почту. В качестве Audience выберите "**Internal**". Нажмите "**Create**" для завершения.

<figure><img src="../../../.gitbook/assets/googleCloudAudienceInternal.png" alt=""><figcaption><p>Параметр "Audience" в создании проекта</p></figcaption></figure>

9. Вернитесь  на главную страницу **APIs & Services.** Далее в раздел "**Credentials**". Нажмите "**Create credentials**". Выберите "OAuth client ID" для создания.

<figure><img src="../../../.gitbook/assets/googleCloudnewOAuthCredentials.png" alt=""><figcaption><p>Создание нового OAuth client ID</p></figcaption></figure>

10. В качестве Application type, выберите "**Web application**". Далее введите произвольное название. Нажмите "**Create**".

<figure><img src="../../../.gitbook/assets/creatingWebApp.png" alt=""><figcaption><p>Создание нового OAuth client ID</p></figcaption></figure>

11. Добавьте новый "Authorized redirect URI".

{% hint style="info" %}
Формат:

<mark style="color:blue;">`https://mikopbx.station.com/pbxcore/api/v3/mail-settings/oauth2-callback`</mark>&#x20;

Замените "mikopbx.station.com" на URl Вашей станции.
{% endhint %}

<figure><img src="../../../.gitbook/assets/addingNewRedirectURl.png" alt=""><figcaption><p>Добавление нового URl для перевода</p></figcaption></figure>

12. Будет создан OAuth client. Сохраните ClientID и Client secret себе в заметки. В будущем эти данные понадобятся для подключения.

<figure><img src="../../../.gitbook/assets/OAuthclientCreated.png" alt=""><figcaption><p>Успешно созданный клиент</p></figcaption></figure>

### Настройки в MikoPBX

1. Перейдите в раздел "**Система**" -> "**Почта и уведомления**":

<figure><img src="../../../.gitbook/assets/emailSection.png" alt=""><figcaption><p>Раздел "Почта и уведомления" в MikoPBX</p></figcaption></figure>

2. Далее, "Настройки SMTP". Заполните следующие параметры:

* **Адрес отправителя, Имя отправителя** - Ваша почта и от какого имени будут отправляться письма.
* **Тип аутентификации** - OAuth2.
* **SMTP логин** - Ваша почта.
* **Провайдер OAuth2** - Google/Gmail.
* **Идентификатор приложения (Client ID), Секретный ключ (Client Secret)** - данные, которые сохранены из Google Cloud (12 пункт из прошлого раздела в этой инструкции).

Все остальные настройки оставьте по умолчанию. Более подробное описание Вы можете найти в главное статье о параметрах почты ([ссылка](./)).

После этого нажмите "**Сохранить**"!

<figure><img src="../../../.gitbook/assets/SMTPParametersGmailOAuth2.png" alt=""><figcaption><p>Параметры почты для подключения Gmail</p></figcaption></figure>

3. Нажмите на синюю кнопку "**Подключить через OAuth2**". Далее выберите Ваш аккаунт Gmail.

<figure><img src="../../../.gitbook/assets/GoogleOauthStep1.png" alt="" width="375"><figcaption><p>Выбор аккаунта Google</p></figcaption></figure>

4. Подтвердите вход: нажмите "**Continue**".

<figure><img src="../../../.gitbook/assets/GoogleOauthStep2.png" alt="" width="375"><figcaption><p>Продолжения авторизации</p></figcaption></figure>

5. Подтвердите выдачу необходимых разрешений. (Нажмите "**Allow**").

<figure><img src="../../../.gitbook/assets/GoogleOauthStep3.png" alt="" width="375"><figcaption><p>Выдача разрешений</p></figcaption></figure>

При успешной авторизации, вы увидите следующее окно.

<figure><img src="../../../.gitbook/assets/GoogleOauthSuccessful.png" alt="" width="375"><figcaption><p>Успешная авторизация</p></figcaption></figure>

### Решение возможных проблем

**Access blocked: Authorization Error (**&#x45;rror 400: invalid\_request)

<figure><img src="../../../.gitbook/assets/GoogleInvalidRequest.png" alt="" width="375"><figcaption><p>Ошибка 400: invalid_request</p></figcaption></figure>

Решение: впишите URl адрес станции в Веб-интерфейсе MikoPBX: "**Сеть и Firewall**" -> "**Сетевые интерфейсы**". Перейдите в раздел "Топология сети" и впишите имя хоста в поле "**Внешнее имя хоста вашего маршрутизатора**". (Включите "**Эта станция расположена за NAT маршрутизатором**")

<figure><img src="../../../.gitbook/assets/GoogleInvalidRequestSolution.png" alt=""><figcaption><p>Решение проблемы</p></figcaption></figure>
