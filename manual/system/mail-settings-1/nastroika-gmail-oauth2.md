---
description: Настройка почты для сервиса gmail
---

# Настройка Gmail (oAuth2)

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

8. Создайте проект (нажмите "**Get started**"). Заполните произвольное название и Вашу почту. В качестве Audience выберите "**External**". Нажмите "**Create**" для завершения.

<figure><img src="../../../.gitbook/assets/googleCloudAudience.png" alt=""><figcaption><p>Параметр "Audience" в создании проекта</p></figcaption></figure>

9. Далее перейдите в раздел "Audience". Под разделом "Test users" добавьте Ваш email.

<figure><img src="../../../.gitbook/assets/newTestUser.png" alt=""><figcaption><p>Добавление Test user</p></figcaption></figure>

10. Вернитесь  на главную страницу **APIs & Services.** Далее в раздел "**Credentials**". Нажмите "**Create credentials**". Выберите "OAuth client ID" для создания.

<figure><img src="../../../.gitbook/assets/googleCloudnewOAuthCredentials.png" alt=""><figcaption><p>Создание нового OAuth client ID</p></figcaption></figure>

11. В качестве Application type, выберите "**Desktop app**". Далее введите произвольное название. Нажмите "**Create**".

<figure><img src="../../../.gitbook/assets/googleCloudnewOAuthClientID.png" alt=""><figcaption><p>Создание нового OAuth client ID</p></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/SMTPParametersGmailOAuth2.png" alt=""><figcaption><p>Параметры почты для подключения Gmail</p></figcaption></figure>

3. Нажмите на синюю кнопку "**Подключить через OAuth2**". Далее выберите Ваш аккаунт Gmail.

<figure><img src="../../../.gitbook/assets/chooseAccountGoogle.png" alt="" width="375"><figcaption><p>Выбор аккаунта Google</p></figcaption></figure>

4. Далее нажмите "**Продолжить**".

<figure><img src="../../../.gitbook/assets/googleWindowContinue.png" alt="" width="375"><figcaption><p>Продолжение с предупреждением</p></figcaption></figure>

5. Снова нажмите продолжить

<figure><img src="../../../.gitbook/assets/LoginToTheMikoPBXEmailService.png" alt="" width="375"><figcaption><p>Продолжение настройки</p></figcaption></figure>

6. Выдайте разрешение&#x20;
