---
description: Preparation guide for MikoPBX using Docker
---

# Docker installation and creating a user and directories

### Installing Docker and Docker Compose on Ubuntu 22.04

Before working with Docker, you need to install Docker and Docker Compose themselves. Here's how to do it:

{% code fullWidth="true" %}
```bash
# Update package list and install required dependencies
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Add the GPG key for Docker's official repository
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker's repository to the APT sources list
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker CE and Docker Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker Compose version
sudo docker compose version
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
