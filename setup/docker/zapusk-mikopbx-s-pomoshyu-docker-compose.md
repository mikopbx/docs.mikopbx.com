# Запуск MikoPBX с помощью docker compose

Для работы с MikoPBX в контейнере необходимо установить docker и docker compose по инструкции

{% content-ref url="ustanovka-dokera-i-cozdanie-polzovatelya-i-katalogov.md" %}
[ustanovka-dokera-i-cozdanie-polzovatelya-i-katalogov.md](ustanovka-dokera-i-cozdanie-polzovatelya-i-katalogov.md)
{% endcontent-ref %}

### Запуск с помощью docker compose

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
sudo docker compose -f docker-compose.yml up
```

### Запуск нескольких MikoPBX на одном хосте

#### Режим без изоляции сети хоста от контейнеров (**–net=host**)

Также можно оганизовать запуск нескольких контенеров MikoPBX на одном хосте, но здесь надо учитывать особенности работы Docker с портами, если не использовать режим **–net=host** то это приведет к высокой нагрузке на процессор хостовой системы, т.к. Docker создает для каждого выделенного порта отдельное правило в Iptables.&#x20;

С включенным режимом **–net=host** вам необходимо вручную следить за распределением доступных портов между запускаемыми контейнерами и встроенными приложениями.\
Например, для запуска двух контейнеров с MikoPBX на одном хосте, можно использовать следующий конфигурационный файл:

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

Сохраните содержимое в файл docker-compose.yml, выполните необходимые корректировки и запускайте MikoPBX коммандой:

```bash

export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)
sudo docker compose -f docker-compose.yml up
```

#### Режим сетевого моста  (**–net=bridge**)

Существует вариант запуска контейнеров с MikoPBX в режиме **–net=bridge**, но как описано выше для использования этого режима необходимо или существенно ограничить диапазон RTP портов, или открывать к ним доступ на хостовой машине не используя возможности Docker.

Для этого вам необходимо написать небольшой скрипт, для определения имени текущего мостового интерфейса и IP адреса каждого контейнера, и после запуска docker compose добавить необходимые правила iptables для диапазона RTP портов следующим образом:

{% code title="start-multiple-mikopbx.sh" fullWidth="false" %}
```bash
#!/bin/bash

COMPOSE_FILE="$1"

if [ -z "$COMPOSE_FILE" ]; then
    echo "Usage: $0 path/to/docker-compose.yaml"
    exit 1
fi

# Получим идентификатор пользователя для запуска контейнера
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)

# Оставновим текущие контейнеры, если они запущены
docker compose -f "$COMPOSE_FILE" down

# Удалим их
docker compose -f "$COMPOSE_FILE" rm

# Запускаем контейнеры в фоне
docker compose -f "$COMPOSE_FILE" up -d
sleep 60

# Создадим метку для правил IPTABLES
IPTABLES_COMMENT="mikopbx-custom-rule"

# Определим идентификатор проекта, он используется при создании сетевого моста
project_prefix=$(cat "$COMPOSE_FILE" | yq e '.x-project-name' -)

# Если префикс не задан, устанавливаем значение по умолчанию
if [ -z "$project_prefix" ]; then
    project_prefix="default_prefix"
fi

# Функция для получения IP адреса контейнера
function get_container_ip() {
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' "$1"
}

# Функция для получения имени мостового интерфейса
function get_bridge_name() {
    local network_name="$1"
    local prefix="$2"
    local network_id=$(docker network inspect "${prefix}_${network_name}" -f '{{.Id}}')

    if [ -z "$network_id" ]; then
        echo "Error: Network ${prefix}_${network_name} not found."
        return 1
    fi

    local bridge_name=$(ip link show type bridge | grep -o "br-${network_id:0:12}\b")
    echo $bridge_name
}

echo "Delete tagged iptables rules"
# Удаляем все правила iptables промаркированные нашим комментарием
iptables -S | grep "$IPTABLES_COMMENT" | sed 's/-A /-D /' | while read rule; do
        echo "Delete rule $rule"
        iptables $rule
done

# Удаляем все NAT правила iptables промаркированные нашим комментарием
iptables -S -t nat | grep "$IPTABLES_COMMENT" | sed 's/-A /-D /' | while read rule; do
     echo "Delete rule $rule"
     iptables -t nat $rule
done

# Парсим docker-compose файл и получаем все необходимые параметры.
echo "Parsing docker-compose file and configuring iptables rules"
cat "$COMPOSE_FILE" | yq e '.services[] | select(.environment[] | test("RTP_PORT_FROM")) | {"container_name": .container_name, "environment": .environment, "network": .networks[0]}' -o=json | jq -c '.' | while read -r service; do
    container_name=$(echo $service | jq -r '.container_name')
    network_name=$(echo $service | jq -r '.network')
    bridge_name=$(get_bridge_name "$network_name" "$project_prefix")
    container_ip=$(get_container_ip "$container_name")

    RTP_PORT_FROM=$(echo $service | jq -r '.environment[] | select(contains("RTP_PORT_FROM")) | split("=")[1]')
    RTP_PORT_TO=$(echo $service | jq -r '.environment[] | select(contains("RTP_PORT_TO")) | split("=")[1]')

    echo "Configuring iptables for $container_name ($container_ip) on $bridge_name from port $RTP_PORT_FROM to $RTP_PORT_TO"

    iptables -A DOCKER -t nat ! -i "$bridge_name" -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j DNAT --to-destination $container_ip:$RTP_PORT_FROM-$RTP_PORT_TO -m comment --comment "$IPTABLES_COMMENT"
    iptables -A DOCKER -d $container_ip/32 ! -i "$bridge_name" -o "$bridge_name" -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j ACCEPT -m comment --comment "$IPTABLES_COMMENT"
    iptables -A POSTROUTING -t nat -s $container_ip/32 -d $container_ip/32 -p udp -m udp --dport $RTP_PORT_FROM:$RTP_PORT_TO -j MASQUERADE -m comment --comment "$IPTABLES_COMMENT"

    echo "Don't forget to open UDP ports $RTP_PORT_FROM to $RTP_PORT_TO on external firewall if it exists"
done

echo "iptables configuration completed successfully."
```
{% endcode %}

Опишем несколько контейнеров в файле docker-compose.yaml, определим разные порты для веб интерфейса, SIP портов и диапазоны RTP портов, чтобы они не пересекались.

{% code title="docker-compose.yaml" fullWidth="false" %}
```yaml

services:
  mikopbx-first:
    container_name: "mikopbx-first"
    image: "ghcr.io/mikopbx/mikopbx-x86-64"
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-docker-first"
    volumes:
      - /var/spool/mikopbx/first/cf:/cf
      - /var/spool/mikopbx/first/storage:/storage
    tty: true
    cap_add:
      - net_admin
    networks:
      - network-bridge1
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      - PBX_NAME=MikoPBXFirst
      - RTP_PORT_FROM=10000 # UPD дипазон 10000-10800 на хосте будет направлен в контейнер
      - RTP_PORT_TO=10800
      - WEB_ADMIN_PASSWORD=mikopbx-first-password
      - ENABLE_USE_NAT=1
      - PBX_FIREWALL_ENABLED=1
      - PBX_FAIL2BAN_ENABLED=1
    ports:
      - "8443:443"  # TCP порт 9443 на хосте направляется на порт 443 в контейнере
      - "5060:5060/udp"  # UDP порт 5060 на хосте направляется на порт 5060 в контейнере
  mikopbx-second:
    container_name: "mikopbx-second"
    image: "ghcr.io/mikopbx/mikopbx-x86-64"
    tty: true
    cap_add:
      - net_admin
    networks:
      - network-bridge2
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-docker-second"
    volumes:
      - /var/spool/mikopbx/second/cf:/cf
      - /var/spool/mikopbx/second/storage:/storage
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      - PBX_NAME=MikoPBXSecond
      - RTP_PORT_FROM=20000 # UPD дипазон 20000-20800 на хосте будет направлен в контейнер
      - RTP_PORT_TO=20800
      - EXTERNAL_SIP_PORT=6060 # Расскажем MikoPBX какой у нее внешнить SIP порт
      - WEB_ADMIN_PASSWORD=mikopbx-second-password
      - ENABLE_USE_NAT=1
      - PBX_FIREWALL_ENABLED=1
      - PBX_FAIL2BAN_ENABLED=1
    ports:
      - "9443:443"  # TCP порт 9443 на хосте направляется на порт 443 в контейнере
      - "6060:5060/udp"  # UDP порт 6060 на хосте направляется на порт 5060 в контейнере
x-project-name: mikopbx # Этот параметр обязательно должен присуствовать
networks:
  network-bridge1:
    driver: bridge
  network-bridge2:
    driver: bridge
```
{% endcode %}

Создаем папку для скриптов

```bash
mkdir -p /usr/src/mikopbx
```

Сохраняем файлы **start-multiple-mikopbx.sh** и **docker-compose.yaml** в эту папку.

Устанавливаем необходимые зависимости для работы скрипта.

```bash
sudo apt-get update
sudo apt-get install jq
sudo snap install yq
```

Переходим в нашу папку, добавляем права на выполнение и запускаем наш скрипт.

```bash
cd /usr/src/mikopbx
sudo chmod +x start-multiple-mikopbx.sh
sudo ./start-multiple-mikopbx.sh docker-compose.yaml
```

Пока ожидаем запуск контейнеров, проверяем настройки брендмауера на хосте, при необходимости открваем те порты, которые указаны в нашем **docker-compose.yaml** файле, а именно:

* TCP/UDP порты **5060** и **6060** для SIP
* UDP диапазоны **10000**-**10800** и **20000**-**20800** для передачи звука по RTP
* TCP порты **8443** и **9443** для HTTPS протокола, для работы Web интерфейса.

Входим по очереди на каждую из станций по адресам:

* https://\<IP хостовой машины>:8443
* https://\<IP хостовой машины>:9443

Для входа в веб-интерфейс первой АТС MikoPBX используйте логин `admin` и  пароль `mikopbx-first-password`

Для входа в веб-интерфейс второй АТС MikoPBX используйте логин `admin` и  пароль `mikopbx-second-password`

В каждой машине должен быть включен режим NAT, указывая что контейнер находится за маршрутизатором в настройках сетевого интерфейса. Если станции будут использоваться внутри локальной сети, то в поле внешнего IP прописваем локальный IP адрес хостовой машины, в противном случе ее публичный IP адрес.

{% hint style="warning" %}
Важное замечание!\
Один из наших контейнеров использует проброс с SIP порта с изменением его значения 5060 -> 6060. \
В данном случае, для корректной работы системы, необходимо добавить внешнее значение SIP порта в настройках NAT в разделе сетевых интерфейсов MikoPBX. \
Эту настройку также можно сделать задав соответвующее значение переменной окружения EXTERNAL\_SIP\_PORT=6060 в файле docker-compose.
{% endhint %}

На этом настройка завершена, можно настраивать учетные записи и выполнять звонки.

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
* **ID\_WWW\_USER** - идентификатор пользователя www-user (можно задать выражением `$(id -u www-user)`, где **www-user** имя **НЕ root** пользователя)
* **ID\_WWW\_GROUP** - идентификатор группы www-user (можно задать выражением `$(id -g www-user)`, где **www-user** имя **НЕ root** группы)
* **WEB\_ADMIN\_LOGIN** - логин для доступа в Web интерфейс
* **WEB\_ADMIN\_PASSWORD** - пароль для доступа в Web интерфейс

Полный список всех возможных параметров настроек доступен в исходном коде [по ссылке](https://github.com/mikopbx/Core/blob/develop/src/Common/Models/PbxSettingsConstants.php).
