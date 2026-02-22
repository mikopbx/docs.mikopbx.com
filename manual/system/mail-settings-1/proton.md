---
description: Настройка почты для сервиса proton.me
---

# Настройка Proton (Логин, Пароль)

### Генерация SMTP токена

1. Для начала, перейдите в настройки своего аккаунта Proton ([ссылка](https://account.proton.me/u/1/mail/dashboard)).

<figure><img src="../../../.gitbook/assets/ProtonDashboard.png" alt=""><figcaption><p>Настройки учетной записи Proton</p></figcaption></figure>

2. Далее перейдите в раздел "**Proton Mail**" -> "**IMAP/SMTP**".&#x20;

<figure><img src="../../../.gitbook/assets/ProtonImapSmtpSection.png" alt=""><figcaption><p>Раздел "IMAP/SMTP"</p></figcaption></figure>

3. Далее пролистайте до секции "**SMTP submission**". Нажмите "**Generate token**".

<figure><img src="../../../.gitbook/assets/ProtonGenerateTokenBtn.png" alt=""><figcaption><p>Кнопка для создания нового токена "Generate token"</p></figcaption></figure>

4. Введите произовольное название в поле "Token name" - MikoPBX в нашем случае, так же выберите Email address для которого Вы создаете токен.

<figure><img src="../../../.gitbook/assets/SMTPTokenParameters.png" alt=""><figcaption><p>Создание нового SMTP токена</p></figcaption></figure>

Будет создан токен. **Его параметры будут показаны один раз и когда Вы закроете окно, станут недоступны. Сохраните их, мы будем использовать их для дальнейшей настройки.**

<figure><img src="../../../.gitbook/assets/ProtonCreatedToken.png" alt=""><figcaption><p>Параметры созданного токена</p></figcaption></figure>

### Подключение в MikoPBX

1. Перейдите в раздел "**Система**" -> "**Почта и уведомления**".

<figure><img src="../../../.gitbook/assets/emailSection.png" alt=""><figcaption><p>Раздел "Система" -> "Почта и уведомления".</p></figcaption></figure>

2. Перейдите в "**Настройки SMTP**". Заполните все необходимые параметры:

* Адрес отправителя - Ваш адрес электронной почты, под которым Вы генерировали токен.
* Имя отправителя - имя от которого отправляется почта.
* Тип аутунтификации - "Логин и пароль".
* SMTP логин - SMTP Username из окна с данными токена.
* SMTP пароль - SMTP token из окна с данными токена.
* **SMTP хост** - **`smtp.protonmail.ch`**
* **SMTP порт** - 587.
* Тип шифрования - STARTLS (порт 587).

Нажмите "**Сохранить**".

<figure><img src="../../../.gitbook/assets/SMTPSettingsMikoPBXProton.png" alt=""><figcaption><p>Параметры почты в MikoPBX</p></figcaption></figure>

Нажмите "**Проверить подключение**". Вы увидите следующее окно, подтверждающее правильность введенных данных:

<figure><img src="../../../.gitbook/assets/successfulConnectionProton.png" alt=""><figcaption><p>Успешное подключение</p></figcaption></figure>
