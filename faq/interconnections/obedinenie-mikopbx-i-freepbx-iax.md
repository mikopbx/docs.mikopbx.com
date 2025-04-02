# Объединение MikoPBX и FreePBX (IAX)

## Настройка MikoPBX

1. Перейдите в раздел "**Маршрутизация**" -> "**Провайдеры телефонии**":

<figure><img src="../../.gitbook/assets/providersMikoPBX.jpg" alt=""><figcaption><p>Раздел "Провайдеры телефонии"</p></figcaption></figure>

2. Создайте нового IAX провайдера:

<figure><img src="../../.gitbook/assets/newIAXProvider.jpg" alt=""><figcaption><p>Новый IAX провайдер</p></figcaption></figure>

3. Заполните параметры:

* **"Название провайдера**" - произвольное.
* "**Хост или IP адрес"** - IP адрес FreePBX.
* **"Логин**" - "tmp".
* "**Пароль**" - произвольный, сложный пароль.

Сохраните параметры.

<figure><img src="../../.gitbook/assets/IAXparameters.jpg" alt=""><figcaption><p>Параметры для IAX провайдера</p></figcaption></figure>

4. После сохранения параметров, в адресной строке появится идентификатор провайдера. Скопируйте его в раздел "**Логин**":

<figure><img src="../../.gitbook/assets/IAXparameters2.jpg" alt=""><figcaption><p>Логин</p></figcaption></figure>

## Настройки FreePBX
