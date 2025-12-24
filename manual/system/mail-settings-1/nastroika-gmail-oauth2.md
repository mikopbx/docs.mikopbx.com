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

7. Перейдите на главную страницу APIs & Services.&#x20;
