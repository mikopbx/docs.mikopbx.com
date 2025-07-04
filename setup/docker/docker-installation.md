---
description: Preparation guide for MikoPBX using Docker
---

# Docker installation and creating a user and directories

### Installing Docker and Docker Compose on Ubuntu 22.04

Before working with Docker, you need to install Docker and Docker Compose themselves. Here's how to do it:

{% code fullWidth="true" %}
```bash
# Update package list and install required dependencies
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Add the GPG key for Docker's official repository
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker's repository to the APT sources list
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker CE
sudo apt update
sudo apt install docker-ce

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify Docker Compose version
sudo docker-compose --version
```
{% endcode %}

### Creating a user and directories on the host system

Before creating the container on the host machine, it's necessary to create a user and group with limited permissions, as well as a folder for storing configuration settings and call recordings

```bash
# Creating a new user (e.g., www-user) without superuser rights
sudo adduser --disabled-password --gecos "" www-user

# Creating directories for data storage
sudo mkdir -p /var/spool/mikopbx/cf
sudo mkdir -p /var/spool/mikopbx/storage

# Granting the created user ownership of the directories
sudo chown -R www-user:www-user /var/spool/mikopbx/
```

### Useful commands

Command to connect to the PBX console:

```bash
sudo docker exec -it mikopbx sh
```

Command to connect to the PBX console menu:

```bash
sudo docker exec -it mikopbx /etc/rc/console_menu
```

Connecting to sngrep for SIP analysis

```bash
sudo docker exec -it mikopbx sngrep
```
