---
description: Действия для сброса данных из консоли MikoPBX
---

# Сброс пароля администратора

Может возникнуть ситуация, когда Вы забыли логин или пароль от web-интерфейса MikoPBX. В данной инструкции вы найдете порядок действий для их сброса.

<figure><img src="../../.gitbook/assets/LoginError(MikoPBXEntryWEB).png" alt=""><figcaption><p>Ошибка авторизации в WEB-интерфейс</p></figcaption></figure>

## Сброс из консоли MikoPBX

1. Перейдите в консоль MikoPBX.

{% hint style="info" %}
В зависимости от способа установки, консоль может находиться:

* При установке на физический сервер - на мониторе, подключенному к серверу.
* При установке в виртуальную машину - в консоли управления виртуальной машиной.
* При установке в облако - в серийной консоли облака (так же в консоли управления виртуальной машиной).
* Способ сброса при установке в Docker-контейнер описан далее в текущей документации.

Помимо этого, в консоль можно попасть, используя авторизацию по SSH. Подробнее про SSH-подключение, Вы можете прочитать [здесь](../troubleshooting/connecting-to-a-pbx-using-ssh/).

Для знакомства с системой MikoPBX, используйте следующую [документацию](../../readme/getting-to-know-mikopbx.md).
{% endhint %}

<figure><img src="../../.gitbook/assets/mikopbxconsole.png" alt=""><figcaption><p>Консоль MikoPBX</p></figcaption></figure>

2. Перейдите в раздел "**\[7] Reset password for the web interface**".
3. Введите "**y**" для подтверждения сброса логина и пароля.

<figure><img src="../../.gitbook/assets/passwordReset.png" alt=""><figcaption><p>Подтверждение сброса пароля</p></figcaption></figure>

4. Авторизуйтесь в web-интерфейс по стандартным данным:

{% hint style="info" %}
Стандартные данные для входа в web-интерфейс:

Логин: admin

Пароль: admin
{% endhint %}

Измените данные для входа после первой авторизации.

<figure><img src="../../.gitbook/assets/changeLoginAndPassword.png" alt=""><figcaption><p>Изменение данных для входа</p></figcaption></figure>

## Сброс пароля в Docker-контейнере

1. Перейдите в **container-shell**.

```
docker exec -it mikopbxcontaierNameOrID sh
```

{% hint style="info" %}
Замените _mikopbxcontaierNameOrID_ на название или ID вашего контейнера.
{% endhint %}

2. Запустите меню, используя следующую команду:

```
/etc/rc/console_menu
```

3. Перейдите в раздел "**\[7] Reset password for the web interface**".
4. Введите "**y**" для подтверждения сброса логина и пароля.
5. Авторизуйтесь в web-интерфейс по стандартным данным:

{% hint style="info" %}
Стандартные данные для входа в web-интерфейс:

Логин: admin

Пароль: admin
{% endhint %}

Измените данные для входа после первой авторизации.

<figure><img src="../../.gitbook/assets/changeLoginAndPassword.png" alt=""><figcaption><p>Изменение данных для авторизации</p></figcaption></figure>

