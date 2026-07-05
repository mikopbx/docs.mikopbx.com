---
description: Вариант обновления для MikoPBX в Docker контейнере
---

# Обновление Docker

Для обновления контейнера MikoPBX до последней версии, вы можете выполнить следующие шаги в командной строке. Эти шаги включают остановку текущего контейнера, скачивание новой версии образа и запуск контейнера с использованием обновлённого образа.

### Обновление Docker контейнера

Для начала нужно корректно остановить работающий контейнер. После остановки контейнера его можно безопасно удалить

```bash
# Остановка текущего контейнера
sudo docker stop mikopbx

# Удаление текущего контейнера
sudo docker rm mikopbx
```

Для запуска нового контейнера с использованием последней версии образа и теми же настройками, что и ранее (включая монтирование томов и прочие параметры сети), воспользуйтесь следующими командами:

```bash
# Скачивание последней версии образа контейнера
sudo docker pull ghcr.io/mikopbx/mikopbx:latest

# Запуск контейнера в не привилегированном режиме
sudo docker run --net=host --name mikopbx --hostname mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-user)" \
           -e ID_WWW_GROUP="$(id -g www-user)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx:latest
```

### Обновление с помощью docker compose

Для начала нужно корректно остановить работающий контейнер. После остановки контейнера его можно безопасно удалить

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)

sudo docker compose -f docker-compose.yml pull
sudo docker compose -f docker-compose.yml up -d
```

Пример файла `docker-compose.yml`, который может быть использован для обновления вашего контейнера MikoPBX через Docker Compose:

{% code title="docker-compose.yml" overflow="wrap" %}
```yaml
services:
  mikopbx:
    container_name: "mikopbx"
    image: "ghcr.io/mikopbx/mikopbx:latest"
    network_mode: "host"
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-a-docker"
    volumes:
      - /var/spool/mikopbx/cf:/cf
      - /var/spool/mikopbx/storage:/storage
    tty: true
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      # Изменение имени станции через переменные окружения
      - PBX_NAME=MikoPBX-in-Docker
      # Изменение стандартного порта SSH на 23
      - SSH_PORT=23
      # Изменение стандартного порта WEB на 8080
      - WEB_PORT=8080
      # Изменение стандартного порта WEB HTTPS на 8443
      - WEB_HTTPS_PORT=8443
```
{% endcode %}

Сохраните содержимое в файл `docker-compose.yml`, выполните необходимые корректировки и запускайте командой:

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)

sudo docker compose -f docker-compose.yml pull
sudo docker compose -f docker-compose.yml up -d
```

### Примечания

* **Данные**: Поскольку данные сохраняются в Docker Volume, они остаются нетронутыми при обновлении, что позволяет сохранить настройки и пользовательские данные.
* **Переменные окружения**: Убедитесь, что все необходимые переменные окружения передаются корректно.
* **Безопасность**: Перед обновлением всегда рекомендуется создать резервные копии ваших данных.

Эти шаги помогут обеспечить гладкое и безопасное обновление вашего контейнера MikoPBX.
