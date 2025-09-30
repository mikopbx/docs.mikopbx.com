---
description: Подключение к MikoPBX по SSH через Google Secure Shell
---

# Подключение с помощью Google Secure Shell

{% hint style="info" %}
Данный способ подходит, если Вы используете браузеры Google Chrome или Yandex Browser для подключения к Web-интерфейсу MikoPBX.
{% endhint %}

### Подключение доступа по паролю&#x20;

1. Перейдите в раздел "Система" -> "**Общие настройки**".

<figure><img src="../../../.gitbook/assets/generalSettingsSectionMikoPBXWEB.png" alt=""><figcaption><p>Раздел "Система" - "Общие настройки"</p></figcaption></figure>

2. Перейдите в раздел "**SSH**". Переключите тумблер "**Отключить авторизацию по паролю**" (По умолчанию в активном положении). Задайте Ваш пароль для SSH-подключения. Сохраните настройки.

{% hint style="danger" %}
**Не используйте простые пароли!** После окончания сессии всегда отключайте возможность SSH-подключения с использованием авторизации по паролю!
{% endhint %}

<figure><img src="../../../.gitbook/assets/SSHPasswordAuth.png" alt=""><figcaption><p>Подключения доступа по паролю</p></figcaption></figure>

### Установка расширения

1. В Web-интерфейсе MikoPBX перейдите в раздел **"Обслуживание"** -> **"SSH консоль"**.

<figure><img src="../../../.gitbook/assets/SSHConsoleSectionMikoPBXWEB.png" alt=""><figcaption><p>Раздел "Обслуживание" - "SSH консоль"</p></figcaption></figure>

2. В случае, если у Вас не установлено расширение "Secure Shell" для браузера, Вас переадресует в магазин расширений. Нажмите "**Установить**". После окончания установки, вернитесь в Web-интерфейс и повторите **первый** шаг.

<figure><img src="../../../.gitbook/assets/gshExtension.png" alt=""><figcaption><p>Установка расширения</p></figcaption></figure>

3. После переадрисации на вкладку Google Secure Shell, введите "yes" в диалоговом окне для нового подключения. Нажмите "**Enter**".

<figure><img src="../../../.gitbook/assets/newFingerprintSSHGoogle.png" alt=""><figcaption><p>Диалоговое окно #1</p></figcaption></figure>

4. В новом диалоговом окне введите ранее заданный пароль для SSH. Нажмите "Enter".&#x20;

<figure><img src="../../../.gitbook/assets/sshPasswordInput.png" alt=""><figcaption><p>Диалоговое окно #2</p></figcaption></figure>

Произойдет SSH-подключение. :tada:

<figure><img src="../../../.gitbook/assets/successfulSSHConnectionGSS.png" alt=""><figcaption><p>Успешное SSH-подключения через Google Secure Shell.</p></figcaption></figure>
