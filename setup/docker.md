# Докер контейнер

### Установка

{% hint style="danger" %}
«**Host система**» должна быть запущена на базе Linux 5+. Тестировалось на Debian 11 и Ubuntu-21.04. В планах добавить поддержку ARM.
{% endhint %}

{% code overflow="wrap" %}
```docker
# Создание на хост системе каталогов для хранения данных MikoPBX
# Для хранения настроек:
mkdir -p /var/spool/mikopbx/cf 
# Для хранения записей разговоров и бекапов:
mkdir -p /var/spool/mikopbx/storage

# Получаем контейнер:
docker pull ghcr.io/mikopbx/mikopbx-x86-64

# Вариант запуска MikoPBX
# НЕ привилегированный режим. Пользователь и группа "www-data" дожены существовать в системе:
docker run --cap-add=NET_ADMIN --net=host --name mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-data)" \
           -e ID_WWW_GROUP="$(id -g www-data)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64
```
{% endcode %}

{% hint style="warning" %}
Флаг **NET\_ADMIN** необходим для возможности работы **fail2ban** и **iptables** внутри контейнера.
{% endhint %}

{% hint style="danger" %}
Если необходимо использовать «[Модуль резервного копирования](../manual/maintenance/backup.md)», то контейнер следует запускать с флагом **–privileged**. Резервное копирование можно выполнять архивированием каталогов **cf** и **storage** вручную (когда контейнер остановлен).
{% endhint %}

{% hint style="info" %}
Флаг **–net=host** указывает на то, что NAT не будет использоваться для нового контейнера. Все порты, что должен занять контейнер будет заняты и на хост машине. Подробнее в [документации...](https://docs.docker.com/network/host/) Если на хост машине, какой то из портов занят, то при загрузке MikoPBX могут возникнуть ошибки.\
\
Ниже описаны переменные окружение, которые позволят скорректировать используемые MikoPBX порты:

* **SSH\_PORT** - порт для SSH (**22**)
* **WEB\_PORT** - порт для работы web интерфейса по протоколу HTTP (**80**)
* **WEB\_HTTPS\_PORT** - порт для работы web интерфейса по протоколу HTTPS (**443**)
* **SIP\_PORT** - порт для подключения SIP клиента (**5060**)
* **RTP\_FROM** - начало диапазона RTP портов, передача голоса (**10000**)
* **RTP\_TO** - конец диапазона RTP портов, передача голоса (**10200**)
* **IAX\_PORT** - порт для подключения IAX клиентов (**4569**)
* **AMI\_PORT** - порт AMI (**5038**)
* **AJAM\_PORT** - порт AJAM используется для подключения панели телефонии для 1С (**8088**)
* **AJAM\_PORT\_TLS** - порт AJAM используется для подключения панели телефонии для 1С (**8089**)
* **BEANSTALK\_PORT** - порт для сервера очередей **Beanstalkd** (**4229**)
* **REDIS\_PORT** - порт для сервера **Redis** (**6379**)
* **GNATS\_PORT** - порт для сервера **gnatsd** (**4223**)
* **ID\_WWW\_USER** - идентификатор пользователя www (можно задать выражением **«$(id -u www-data)«**, где www-data имя **НЕ root** пользователя)
* **ID\_WWW\_GROUP** - идентификатор группы www (можно задать выражением **«$(id -g www-data)«**, где www-data имя **НЕ root** пользователя)
{% endhint %}

### Пример **docker-compose.yml**:

{% code overflow="wrap" %}
```docker
version: "3.9"
services:
  mikopbx:
    container_name: "mikopbx"
    image: ""
    network_mode: "host"
    command: '-d'
    cap_add:
      - NET_ADMIN
    volumes:
      - /var/spool/mikopbx/cf:/cf
      - /var/spool/mikopbx/storage:/storage
    # environment:
    ## Изменение стандартного порта SSH на 23 
    #  - SSH_PORT=23
    ## Изменение стандартного порта HTTP на 81
    #  - WEB_PORT=81
    # DAHDI не обязательное условие. Он необходим для работы MeetMe в панели телефонии.
    # devices:
    #  - "/dev/dahdi/transcode:/dev/dahdi/transcode"
    #  - "/dev/dahdi/channel:/dev/dahdi/channel"
    #  - "/dev/dahdi/ctl:/dev/dahdi/ctl"
    #  - "/dev/dahdi/pseudo:/dev/dahdi/pseudo"
    #  - "/dev/dahdi/timer:/dev/dahdi/timer
```
{% endcode %}

{% hint style="warning" %}
Внутри контейнера совместимых с ядром хост системы DAHDI модулей нет. По этой причине если необходим функционал MeetMe конференций, то DAHDI потребуется собрать на хост системе вручную.
{% endhint %}

Команда для подключения к консоли АТС:

```
docker exec -it mikopbx sh
```

Команда для подключения к консольному меню АТС:

```
docker exec -it mikopbx /etc/rc/console_menu
```
