# Docker installation and basic commands

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
