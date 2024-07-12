# Обновление Docker

Для обновления контейнера MikoPBX до последней версии, вы можете выполнить следующие шаги в командной строке. Эти шаги включают остановку текущего контейнера, скачивание новой версии образа и запуск контейнера с использованием обновлённого образа.

### Обновление Docker контейнера

Для начала нужно корректно остановить работающий контейнер. После остановки контейнера его можно безопасно удалить

```bash
# Остановка текущего контейнера
sudo docker stop mikopbx

# Удаление текущего контейнера
sudo docker rm mikopbx
```

Для запуска нового контейнера с использованием последней версии образа и теми же настройками, что и ранее (включая монтирование томов и прочие параметры сети), воспользуйтесь следующими командами:

```bash
# Скачивание последней версии образа контейнера
sudo docker pull ghcr.io/mikopbx/mikopbx-x86-64:latest

# Запуск контейнера в не привилегированном режиме
sudo docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
           -v data_volume:/cf \
           -v data_volume:/storage \
           -e SSH_PORT=23 \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64:latest
```

### Обновление с помощью docker compose

Для начала нужно корректно остановить работающий контейнер. После остановки контейнера его можно безопасно удалить

```bash
# Остановка текущего контейнера
sudo docker stop mikopbx

# Удаление текущего контейнера
sudo docker rm mikopbx
```

Следующий шаг — это скачивание последней версии образа MikoPBX:

```bash
# Скачивание последней версии образа контейнера
sudo docker pull ghcr.io/mikopbx/mikopbx-x86-64:latest
```

Пример файла `docker-compose.yml`, который может быть использован для обновления вашего контейнера MikoPBX через Docker Compose:

{% code title="docker-compose.yml" overflow="wrap" %}
```yaml
services:
  mikopbx:
    container_name: "mikopbx"
    image: "ghcr.io/mikopbx/mikopbx-x86-64:latest"
    network_mode: "host"
    cap_add:
      - NET_ADMIN
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-a-docker"
    volumes:
      - data_volume:/cf
      - data_volume:/storage
    tty: true
    environment:
      # Изменение имени станции через переменные окружения
      - PBX_NAME=MikoPBX-in-Docker
      # Изменение стандартного порта SSH на 23
      - SSH_PORT=23
      # Изменение стандартного порта WEB на 8080
      - WEB_PORT=8080
      # Изменение стандартного порта WEB HTTPS на 8443
      - WEB_HTTPS_PORT=8443
      
volumes:
  data_volume:
```
{% endcode %}

Сохраните содержимое в файл `docker-compose.yml`, выполните необходимые корректировки и запускайте командой:

```bash
sudo docker compose -f docker-compose.yml up
```

### Примечания

* **Данные**: Поскольку данные сохраняются в Docker Volume, они остаются нетронутыми при обновлении, что позволяет сохранить настройки и пользовательские данные.
* **Переменные окружения**: Убедитесь, что все необходимые переменные окружения передаются корректно.
* **Безопасность**: Перед обновлением всегда рекомендуется создать резервные копии ваших данных.

Эти шаги помогут обеспечить гладкое и безопасное обновление вашего контейнера MikoPBX.
