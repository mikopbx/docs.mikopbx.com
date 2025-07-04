---
description: Подготовка к установке MikoPBX в Docker-контейнер
---

# Установка докера и cоздание пользователя и каталогов

### Установка Docker и Docker Compose на Ubuntu 22.04

Перед началом работы с Docker, необходимо установить сам Docker и Docker Compose. Вот как это можно сделать:

{% code fullWidth="false" %}
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
sudo docker-compose --version

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

### Полезные команды

Команда для подключения к консоли АТС:

```bash
sudo docker exec -it mikopbx sh
```

Команда для подключения к консольному меню АТС:

```bash
sudo docker exec -it mikopbx /etc/rc/console_menu
```

Подключение к shgrep для анализа SIP

<pre class="language-bash"><code class="lang-bash"><strong>sudo docker exec -it mikopbx sngrep
</strong></code></pre>
