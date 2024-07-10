# Установка докера и базовые команды

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

### Полезные команды

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
