---
description: Установка MikoPBX в Yandex Cloud с помощью утилиты yc
---

# Коммандная строка YC

Утилита [Yandex Cloud](https://yandex.cloud/ru/docs/cli/cli-ref/managed-services/compute/instance/) позволяет автоматизировать создание инстансов. Ее можно использовать в скриптах.&#x20;

1. На [странице продукта](https://cloud.yandex.ru/marketplace/products/miko/mikopbx#product-ids) скопируйте значение **image\_id**. В текущем примере _**fd83g1qbk6m3jnl5cvah**_ (для версии 2022.3.15). Идентификатор образа следует укзаать в параметре **create-boot-disk image-id**
2. Получим список каталогов

```bash
yc resource-manager folder list
```

Выбираем каталог и указываем его имя в параметре **folder-name**

3. Список зон&#x20;

```bash
yc compute zone list 
```

Описываем зону в параметре **zone**

4. Запускаем инстанс&#x20;

```bash
yc compute instance create \
	--folder-name apor-test-catalog \
	--name test\
	--zone ru-central1-c \
	--cores 2 \
	--memory 2gb \
	--create-boot-disk image-id=fd83g1qbk6m3jnl5cvah \
	--ssh-key ~/.ssh/id_rsa.pub \
	--public-ip \
	--create-disk name=storage-test-mikopbx,size=20,auto-delete=1
```

5. Список инстансов обновится&#x20;

```bash
yc compute instance list                 
+----------------------+-----------------+---------------+---------+-----------------+-------------+
|          ID          |      NAME       |    ZONE ID    | STATUS  |   EXTERNAL IP   | INTERNAL IP |
+----------------------+-----------------+---------------+---------+-----------------+-------------+
| ef38gedvuug8qvv4l74p | test            | ru-central1-c | RUNNING | 51.250.39.55    | 10.130.0.6  |
+----------------------+-----------------+---------------+---------+-----------------+-------------+
```

Используйте **EXTERNAL IP** для входа и **ID** в качестве пароля для пользователя **admin** web интерфейса
