---
description: Обновление MikoPBX, запущенной в Docker или Docker Compose
---

# Обновление Docker-контейнера

{% hint style="warning" %}
Рекомендуется производить обновления последовательно, «не перепрыгивая» через релизы и версии.
{% endhint %}

Docker-контейнер MikoPBX не обновляется файлами `.img` или `.iso`. Для перехода на новую версию нужно скачать новый Docker image и пересоздать контейнер, сохранив каталоги `/cf`, `/storage`, сетевые параметры и переменные окружения.

## Подготовка

1. Создайте резервную копию настроек MikoPBX. Если модуль резервного копирования недоступен в непривилегированном контейнере, подготовьте резервное копирование каталогов или volumes на стороне Docker-хоста.
2. Запишите имя и версию текущего image:

```bash
sudo docker inspect mikopbx --format '{{.Config.Image}}'
```

3. Сохраните текущую конфигурацию контейнера:

```bash
sudo docker inspect mikopbx > mikopbx-container-before-update.json
```

4. Проверьте подключения `/cf` и `/storage`:

```bash
sudo docker inspect mikopbx --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
```

5. Запишите используемые порты, `network_mode`, hostname и переменные окружения. Если контейнер запускается через Compose, сохраните актуальный `docker-compose.yml` и файл `.env`.
6. Завершите активные вызовы и запланируйте техническое окно.

## Обновление контейнера, запущенного командой docker run

### Загрузка нового image

Для последней стабильной версии выполните:

```bash
sudo docker pull ghcr.io/mikopbx/mikopbx:latest
```

Если используется фиксированный тег, укажите тот же registry и тег нужной версии вместо `latest`.

### Остановка и замена контейнера

1. Остановите контейнер:

```bash
sudo docker stop mikopbx
```

2. Создайте консистентную резервную копию `/cf`, пока контейнер остановлен. Для bind mount из примера выше можно выполнить:

```bash
sudo tar -C /var/spool/mikopbx -czf mikopbx-cf-before-update.tar.gz cf
```

Помимо `/cf`, новую версию стоит подстраховать бэкапом базы CDR — она лежит в `/storage` и при первом старте тоже может быть мигрирована. В `/cf` находится только `mikopbx.db` (настройки), а история звонков — это отдельная `cdr.db` (и `recording_storage.db`) в `/storage/usbdisk1/mikopbx/astlogs/asterisk/`.

Бэкап делается **на остановленном контейнере** (как и для `/cf`), чтобы база была в согласованном состоянии:

```bash
sudo docker stop mikopbx# бэкап /cf (настройки)sudo tar -C /var/spool/mikopbx -czf mikopbx-cf-before-update.tar.gz cf# бэкап CDR + метаданных записейsudo tar -C /var/spool/mikopbx/storage/usbdisk1/mikopbx \  -czf mikopbx-cdr-before-update.tar.gz astlogs/asterisk
```

При откате восстанавливайте оба архива: сначала `/cf`, затем CDR - иначе история звонков и привязка записей к звонкам могут рассинхронизироваться с настройками.

3. Переименуйте старый контейнер вместо немедленного удаления:

```bash
sudo docker rename mikopbx mikopbx-before-update
```

4. Запустите новый контейнер с теми же параметрами. Пример для установки с host network и bind mounts:

```bash
sudo docker run --net=host --name mikopbx --hostname mikopbx \
  -v /var/spool/mikopbx/cf:/cf \
  -v /var/spool/mikopbx/storage:/storage \
  -e SSH_PORT=23 \
  -e ID_WWW_USER="$(id -u www-user)" \
  -e ID_WWW_GROUP="$(id -g www-user)" \
  -it -d --restart always ghcr.io/mikopbx/mikopbx:latest
```

{% hint style="warning" %}
Пример нельзя копировать без проверки. Повторите все параметры именно вашей прежней установки: тип сети, volumes, порты, hostname, capabilities и переменные окружения.
{% endhint %}

5. Следите за запуском:

```bash
sudo docker logs -f mikopbx
```

Для выхода из просмотра логов нажмите `Ctrl+C` — контейнер продолжит работу.

## Обновление с Docker Compose

1. Перейдите в каталог с `docker-compose.yml`.
2. Если идентификаторы пользователя передаются через окружение, задайте их так же, как при первом запуске:

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)
```

3. Загрузите новый image:

```bash
sudo docker compose pull
```

4. Пересоздайте контейнер:

```bash
sudo docker compose up -d
```

5. Проверьте состояние и логи:

```bash
sudo docker compose ps
sudo docker compose logs -f mikopbx
```

Docker Compose пересоздаст контейнер, но сохранит bind mounts и именованные volumes, указанные в `docker-compose.yml`.

## Проверка после обновления

1. Убедитесь, что контейнер работает:

```bash
sudo docker ps --filter name=mikopbx
```

2. Откройте web-интерфейс и проверьте версию MikoPBX.
3. Проверьте наличие настроек и записей разговоров.
4. Убедитесь, что телефоны и SIP-провайдеры зарегистрировались.
5. Выполните входящий и исходящий тестовые звонки.
6. Проверьте публикацию web-, SIP- и RTP-портов, если используется bridge network.

## Откат

Если новый контейнер не запускается:

1. Сохраните его логи:

```bash
sudo docker logs mikopbx > mikopbx-update-error.log 2>&1
```

2. Остановите и переименуйте новый контейнер:

```bash
sudo docker stop mikopbx
sudo docker rename mikopbx mikopbx-failed-update
```

3. Восстановите резервную копию `/cf`, созданную до обновления. Способ восстановления зависит от того, используется bind mount, именованный volume или snapshot системы хранения.
4. Верните прежнее имя старому контейнеру и запустите его:

```bash
sudo docker rename mikopbx-before-update mikopbx
sudo docker start mikopbx
```

{% hint style="warning" %}
После первого запуска новая версия могла обновить базу данных в `/cf`. Поэтому надёжный откат должен опираться не только на старый container image, но и на резервную копию `/cf`, созданную до обновления.
{% endhint %}

Для отката установки Docker Compose укажите в `docker-compose.yml` предыдущий тег image, восстановите предыдущее состояние `/cf` и снова выполните:

```bash
sudo docker compose up -d
```

После успешной проверки новый контейнер можно удалить старый:

```bash
sudo docker rm mikopbx-before-update
```
