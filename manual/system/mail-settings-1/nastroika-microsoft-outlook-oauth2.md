---
description: Настройка почты для сервиса Outlook (outlook.com; hotmail.com)
---

# Настройка Microsoft Outlook (oAuth2)

## Настройки внутри Microsoft Entra

### Регистрация приложения

1. Войдите в [центр администрирования Microsoft Entra.](https://entra.microsoft.com/)

<figure><img src="../../../.gitbook/assets/MicrosoftEntraDeshboard.png" alt=""><figcaption><p>Главная страница центра администрирования Microsoft Entra</p></figcaption></figure>

2. Перейдите в раздел "Entra ID" -> "App registrations". Далее нажмите "New registration" для регистрации нового приложения.

<figure><img src="../../../.gitbook/assets/MicrosoftEntraNewAppRegistration.png" alt=""><figcaption><p>Регистрация нового приложения</p></figcaption></figure>

3. Выберите следующие параметры для Вашего приложения:

* Name - укажите название для Вашего приложения.
* Supported account types - выберите параметр "**Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant**)".

Нажмите "**Register**".

<figure><img src="../../../.gitbook/assets/ApplicationParameters2(Version_Multitenant).png" alt=""><figcaption><p>Параметры приложения</p></figcaption></figure>

4. Будет созданно приложения. Сохраните client ID, в будущем он понадобится для настройки внутри веб-интерфейса MikoPBX.

<figure><img src="../../../.gitbook/assets/CreatedApplicationOverview.png" alt=""><figcaption><p>Главная страница созданного приложения</p></figcaption></figure>

### Выдача разрешений и создание secret ID

1. Из главной страницы приложения перейдите в "**Manage**" -> "**API permissions**".

<figure><img src="../../../.gitbook/assets/MicrosoftEntraAPIpermissions.png" alt=""><figcaption><p>Раздел "API permissions"</p></figcaption></figure>

2. Нажмите "**Add a permission**".

<figure><img src="../../../.gitbook/assets/MicrosoftEntraAddPermission.png" alt=""><figcaption><p>Добавление разрешения</p></figcaption></figure>

3. В разделе "**Microsoft Graph**" выберите "**Delegated Permissions**". В поиске введите "**SMTP**". Поставьте галочку напротив "**SMTP.Send**".

Нажмите "**Add permissions**".

<figure><img src="../../../.gitbook/assets/smtpsendPermission.png" alt=""><figcaption><p>Разрешение "SMTP.Send"</p></figcaption></figure>

3. Перейдите в раздел "**APIs my organization uses**". В поиске наберите "Office 365 Exchange Online", нажмите на соответствующее API.

<figure><img src="../../../.gitbook/assets/office365exchange.png" alt=""><figcaption><p>API "Office 365 Exchange Online"</p></figcaption></figure>

4. Выберите "**Delegated permissions**", в поиске наберите "**mail**".

<figure><img src="../../../.gitbook/assets/addingPermissions.png" alt=""><figcaption><p>Поиск разрешений</p></figcaption></figure>

5. Пролистайте вниз, в разделе "Mail" выберите:

* **Mail.Send**
* **Mail.Send.All**

Нажмите **"Add permissions".**

<figure><img src="../../../.gitbook/assets/addingPermissions2.png" alt=""><figcaption><p>Выдача разрешений</p></figcaption></figure>

6. Нажмите "Grant admin consent for ...".

<figure><img src="../../../.gitbook/assets/grantAdminConsent2.png" alt=""><figcaption><p>Выдача разрешений</p></figcaption></figure>

7. Далее перейдите в раздел "**Certificates & secrets**" -> "**Client secrets**". Нажмите "**New client secret**".

<figure><img src="../../../.gitbook/assets/creatingNewClientSecret.png" alt=""><figcaption><p>Создание нового Secret ID</p></figcaption></figure>

8. Задайте необходимые параметры:

* Description - произвольное описание.
* Expires - срок на который Вы выпускаете этот SecretID.&#x20;

Нажмите "**Add**".

<figure><img src="../../../.gitbook/assets/newClientSecret.png" alt=""><figcaption><p>Параметры для создания нового client secret</p></figcaption></figure>

9. Скопируйте Ваш secret ID. Он понадобится для настройки в веб-интерфейсе MikoPBX.

<figure><img src="../../../.gitbook/assets/copyingSecretID.png" alt=""><figcaption><p>Копирование Secret ID</p></figcaption></figure>
