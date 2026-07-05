---
description: Подготовка к установке MikoPBX в Docker-контейнер
---

# Установка докера и cоздание пользователя и каталогов

### Установка Docker и Docker Compose на Ubuntu 22.04

Перед началом работы с Docker, необходимо установить сам Docker и Docker Compose. Вот как это можно сделать:

{% code fullWidth="false" %}
```bash
# Обновление списка пакетов и установка необходимых зависимостей
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Добавление ключа GPG официального репозитория Docker
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Добавление репозитория Docker в список источников APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker CE и Docker Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Проверка версии Docker Compose
sudo docker compose version

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
