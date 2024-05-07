# В Docker контейнере

## Docker

### Установка Docker и Docker Compose на Ubuntu 22.04

{% hint style="danger" %}
«**Host система**» должна быть запущена на базе Linux 5+. Тестировалось на Debian 11 и Ubuntu-21.04, Ubuntu Server 22.04 LTS
{% endhint %}

Перед началом работы с Docker, необходимо установить сам Docker и Docker Compose. Вот как это можно сделать:

{% code fullWidth="true" %}
```bash
# Обновление списка пакетов и установка необходимых зависимостей
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Добавление ключа GPG официального репозитория Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Добавление репозитория Docker в список источников APT
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Установка Docker CE
sudo apt update
sudo apt install docker-ce

# Установка Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Проверка версии Docker Compose
docker-compose --version

```
{% endcode %}

### Создание пользователя и каталогов на хост-системе

Перед созданием контейнера на хостовой машине необходимо создать пользователя и группу с ограничеными правами, а также папку для хранения настроек конфигурации и записей разговоров.

```bash
# Создание нового пользователя (например, www-user) без прав суперпользователя
sudo adduser --disabled-password --gecos "" www-user

# Создание каталогов для хранения данных
sudo mkdir -p /var/spool/mikopbx/cf
sudo mkdir -p /var/spool/mikopbx/storage

# Назначение прав созданному пользователю на каталоги
sudo chown -R www-user:www-user /var/spool/mikopbx/

```

### Запуск контейнера Docker

Для запуска контейнера с вашим приложением воспользуйтесь следующими командами:

{% code overflow="wrap" %}
```bash
# Получение образа контейнера
docker pull ghcr.io/mikopbx/mikopbx-x86-64

# Запуск контейнера в не привилегированном режиме
docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-user)" \
           -e ID_WWW_GROUP="$(id -g www-user)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64
```
{% endcode %}

### Проверка работы

Чтобы убедиться, что ваше приложение MikoPBX запостилось и работает в Docker-контейнере, можно выполнить следующие шаги после его запуска. Эти шаги помогут проверить состояние контейнера и просмотреть его логи.

#### Шаг 1: Проверка статуса контейнера

Сначала нужно удостовериться, что контейнер успешно запущен и работает. Для этого используем команду `docker ps`, которая покажет список запущенных контейнеров и их статус.

```bash
docker ps
```

Эта команда выведет информацию о всех активных контейнерах. Убедитесь, что контейнер `mikopbx` присутствует в списке и его статус указывает на то, что он запущен и работает (например, статус **up**).

#### Шаг 2: Просмотр логов контейнера

После подтверждения того, что контейнер запущен, следующим шагом будет просмотр логов для проверки, что приложение загрузилось без ошибок и функционирует нормально. Команда `docker logs` позволит вам увидеть вывод, который генерирует ваше приложение.

```bash
docker logs mikopbx
```

Просмотрите вывод команды на наличие сообщения, подобного указанному ниже. Это сообщение свидетельствует о том, что MikoPBX успешно загружена и готова к использованию:

```
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|               All services are fully loaded welcome                |
|                       MikoPBX 2024.1.60.                           |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|                        Web Interface Access                        |
|                                                                    |
| Local Network Address:                                             |
| https://10.0.0.4                                                   |
|                                                                    |
| Web credentials:                                                   |
|    Login: admin                                                    |
|    Password: admin                                                 |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
| SSH access disabled!                                               |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

Если отображается процесс запуска MikoPBX то необходимо подождать и повторить команду `docker logs mikopbx`

#### Шаг 3: Проверка доступа к веб-интерфейсу

При старте контенер не имеет информации об адресе хостовой системы, потому необходимо открыть внешний адрес хостовой системы, в данном случае Ubuntu в браузере.\
https://\<IP адрес хост системы>

Войдите в веб-интерфейс, используя логин `admin` и пароль `admin`, чтобы убедиться, что веб-интерфейс доступен и функционирует корректно.

<figure><img src="../.gitbook/assets/FirstStart.png" alt=""><figcaption></figcaption></figure>

### Особенности работы контейнизированной MikoPBX

* Флаг **NET\_ADMIN** необходим для возможности работы системы проактивной защиты **fail2ban** и фаервола **iptables** внутри контейнера. При срабатывании блокировки доступа, например при вводе неверного пароля, доступ с IP адреса злоумышленника будет заблокирован для всего хоста, а не только для контейнера.
* Если необходимо использовать «[Модуль резервного копирования](../manual/maintenance/backup.md)», то контейнер следует запускать с флагом **–privileged**. Когда MikoPBX запускается в контейнере, резервное копирование можно также выполнять архивированием каталогов **cf** и **storage** вручную . В этом случае привелегированный режим можно не использовать, но в момент копирования контенер должен быть остановлен.
* Флаг **–net=host** указывает на то, что NAT между хостов и контенером не будет использоваться. MikoPBX будет подключена напрямую к сети хостовой машины. Все порты, которые должен занять контейнер будут заняты и на хост машине. Если на хост машине, какой-то из портов недоступн, то при загрузке MikoPBX возникнут ошибки. Подробнее в [документации к Docker...](https://docs.docker.com/network/host/)&#x20;
* При необходимости можно скорректировать стандартный набор портов, которые использует MikoPBX. Это можно сделать объявляя переменные окружения при запуске контейнера.

### Переменные окружения для конфигурирования MikoPBX

Ниже перечислены некоторые переменные окружения, которые позволят скорректировать используемые MikoPBX порты и настройки.

* **SSH\_PORT** - порт для SSH (**22**)
* **WEB\_PORT** - порт для работы web интерфейса по протоколу HTTP (**80**)
* **WEB\_HTTPS\_PORT** - порт для работы web интерфейса по протоколу HTTPS (**443**)
* **SIP\_PORT** - порт для подключения SIP клиента (**5060**)
* **TLS\_PORT** - порт для подключения SIP клиента с шифрованием (**5061**)
* **RTP\_PORT\_FROM** - начало диапазона RTP портов, передача голоса (**10000**)
* **RTP\_PORT\_TO** - конец диапазона RTP портов, передача голоса (**10800**)
* **IAX\_PORT** - порт для подключения IAX клиентов (**4569**)
* **AMI\_PORT** - порт AMI (**5038**)
* **AJAM\_PORT** - порт AJAM используется для подключения панели телефонии для 1С (**8088**)
* **AJAM\_PORT\_TLS** - порт AJAM используется для подключения панели телефонии для 1С (**8089**)
* **BEANSTALK\_PORT** - порт для сервера очередей **Beanstalkd** (**4229**)
* **REDIS\_PORT** - порт для сервера **Redis** (**6379**)
* **GNATS\_PORT** - порт для сервера **gnatsd** (**4223**)
* **ID\_WWW\_USER** - идентификатор пользователя www-user (можно задать выражением \
  `$(id -u www-user)`, где **www-user** имя **НЕ root** пользователя)
* **ID\_WWW\_GROUP** - идентификатор группы www-user (можно задать выражением \
  `$(id -g www-user)`, где **www-user** имя **НЕ root** группы)
* **WEB\_ADMIN\_LOGIN** - логин для доступа в Web интерфейс
* **WEB\_ADMIN\_PASSWORD** - пароль для доступа в Web интерфейс

Полный список всех возможных параметров настроек доступен в исходном коде [по ссылке](https://github.com/mikopbx/Core/blob/develop/src/Common/Models/PbxSettingsConstants.php).

## Вариант запуска с помощью docker compose

Вот пример файла `docker-compose.yml`, который может быть использован для управления вашим контейнером MikoPBX через Docker Compose:

{% code title="docker-compose.yml" overflow="wrap" %}
```yaml
services:
  mikopbx:
    container_name: "mikopbx"
    image: "ghcr.io/mikopbx/mikopbx-x86-64"
    network_mode: "host"
    cap_add:
      - NET_ADMIN
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

Сохраните содержимое в файл docker-compose.yml, выполните необходимые корректировки и запускайте MikoPBX коммандой:

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)
docker compose -f docker-compose.yml up
```

### Запуск нескольких MikoPBX на одном хосте

#### Режим без изоляции сети хоста от контейнеров (**–net=host**)

Также можно оганизовать запуск нескольких контенеров MikoPBX на одном хосте, но здесь надо учитывать особенности работы Docker с портами, если не использовать режим **–net=host** то это приведет к высокой нагрузке на процессор хостовой системы, т.к. Docker создает для каждого выделенного порта отдельное правило в Iptables.&#x20;

С включенным режимом **–net=host** вам необходимо вручную следить за распределением доступных портов между запускаемыми контейнерами. Например, для запуска двух контейнеров с MikoPBX на одном хосте, можно использовать следующий конфигурационный файл.

{% code title="docker-compose.yml" %}
```yaml
services:
  mikopbx-first:
    container_name: "mikopbx-first"
    image: "ghcr.io/mikopbx/mikopbx-x86-64"
    network_mode: "host"
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-docker-first"
    volumes:
      - /var/spool/mikopbx/first/cf:/cf
      - /var/spool/mikopbx/first/storage:/storage
    tty: true
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      - PBX_NAME=MikoPBXFirst
      - PBX_FIREWALL_ENABLED=0
      - PBX_FAIL2BAN_ENABLED=0
      - SSH_PORT=123
      - WEB_PORT=8080
      - WEB_HTTPS_PORT=8443
      - SIP_PORT=5060
      - TLS_PORT=5061
      - RTP_PORT_FROM=10000
      - RTP_PORT_TO=10800
      - IAX_PORT=4569
      - AMI_PORT=5038
      - AJAM_PORT=8088
      - AJAM_PORT_TLS=8089
      - BEANSTALK_PORT=4229
      - REDIS_PORT=6379
      - GNATS_PORT=4223
mikopbx-second:
    container_name: "mikopbx-second"
    image: "ghcr.io/mikopbx/mikopbx-x86-64"
    network_mode: "host"
    tty: true
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-docker-second"
    volumes:
      - /var/spool/mikopbx/second/cf:/cf
      - /var/spool/mikopbx/second/storage:/storage
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      - PBX_NAME=MikoPBXSecond
      - PBX_FIREWALL_ENABLED=0
      - PBX_FAIL2BAN_ENABLED=0
      - SSH_PORT=2223
      - WEB_PORT=8081
      - WEB_HTTPS_PORT=9443
      - SIP_PORT=6060
      - TLS_PORT=6061
      - RTP_PORT_FROM=20000
      - RTP_PORT_TO=20800
      - IAX_PORT=5569
      - AMI_PORT=6038
      - AJAM_PORT=9088
      - AJAM_PORT_TLS=9089
      - BEANSTALK_PORT=5229
      - REDIS_PORT=7379
      - GNATS_PORT=5223      
```
{% endcode %}

#### Режим сетевого моста  (**–net=bridge**)

Существует вариант запуска контейнеров с MikoPBX в режиме **–net=bridge**, но как описано выше для использования этого режима необходимо или существенно ограничить диапазон RTP портов, или открывать к ним доступ на хостовой машине не используя возможности Docker.

Для этого вам необходимо написать скрипт, для определения имени текущего мостового интерфейса и IP адреса каждого контейнера, и после запуска docker compose добавить правила iptables для диапазона RTP портов следующим образом.

```bash
COMPOSE_FILE="$1"

if [ -z "$COMPOSE_FILE" ]; then
    echo "Usage: $0 path/to/docker-compose.yaml"
    exit 1
fi

IPTABLES_COMMENT="mikopbx-custom-rule"

# Удалим все правила IPTABLES помеченные нами как созданные вручную
iptables -S | grep "$IPTABLES_COMMENT" | sed 's/-A /-D /' | while read rule; do
    iptables $rule
done

# Обойдем каждый сервис в файле docker-compose.yaml и получим значения
# RTP_PORT_FROM, RTP_PORT_TO, bridge_name и container_ip
# создадим кастомные правила IPTABLES для проброса портов
yq e '.services[] | select(.environment[] | contains("RTP_PORT_FROM")) | {container_name, environment, network: .networks[0]}' -o json "$COMPOSE_FILE" | jq -c '.[]' | while read -r service; do
    container_name=$(echo $service | jq -r '.container_name')
    network_name=$(echo $service | jq -r '.network')
    environment=$(echo $service | jq -r '.environment | from_entries')

    RTP_PORT_FROM=$(echo "$environment" | jq -r '."RTP_PORT_FROM"')
    RTP_PORT_TO=$(echo "$environment" | jq -r '."RTP_PORT_TO"')

    bridge_name=$(get_bridge_name "$network_name")
    container_ip=$(get_container_ip "$container_name")

    # Set iptables rules
    iptables -A DOCKER -t nat ! -i "$bridge_name" -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j DNAT --to-destination $container_ip:$RTP_PORT_FROM-$RTP_PORT_TO -m comment --comment "$IPTABLES_COMMENT"
    iptables -A DOCKER -d $container_ip/32 ! -i "$bridge_name" -o "$bridge_name" -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j ACCEPT -m comment --comment "$IPTABLES_COMMENT"
    iptables -A POSTROUTING -t nat -s $container_ip/32 -d $container_ip/32 -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j MASQUERADE -m comment --comment "$IPTABLES_COMMENT"
done

```

### Создание контейнера из tar архива

Помимо использования нашего официального реестра, вам может понадобиться вариант создания контейнера из образа, например для бета версии. В составе опубликованных релизов и предрелизов поставляется tar архив, который мы используем для создания контейнера.

Вот пример кода, для его использования:

```bash
// Создаем контейнер из tar архива
docker import \
  --change 'ENTRYPOINT ["/bin/sh", "/sbin/docker-entrypoint"]' \
  mikopbx-2024.1.92-x86_64.tar \
  "mikopbx:2024.1.92"

// Запускаем созданный контейнер
docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
	 -v mikopbx_cf:/cf \
	 -v mikopbx_storage:/storage \
	 -e SSH_PORT=23 \
	 -e ID_WWW_USER="$(id -u www-user)" \
	 -e ID_WWW_GROUP="$(id -g www-user)" \
	 -it mikopbx:2024.1.92
```

## Полезные команды

Команда для подключения к консоли АТС:

```bash
docker exec -it mikopbx sh
```

Команда для подключения к консольному меню АТС:

```bash
docker exec -it mikopbx /etc/rc/console_menu
```

Подключение к shgrep для анализа SIP

```bash
docker exec -it mikopbx sngrep
```
