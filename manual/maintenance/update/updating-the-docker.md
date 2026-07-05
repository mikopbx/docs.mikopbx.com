---
description: Upgrade option for MikoPBX in Docker container
---

# Updating the docker

To update the MikoPBX container to the latest version, you can follow these steps in the command line. These steps include stopping the current container, downloading the new version of the image, and running the container with the updated image.&#x20;

### Updating the docker

First, you need to properly stop the running container. After stopping the container, you can safely remove it

```bash
# Stop the current container
sudo docker stop mikopbx

# Remove the current container
sudo docker rm mikopbx
```

To launch a new container using the latest image version with the same settings as before, use the following commands:

```bash
# Downloading the latest container image version
sudo docker pull ghcr.io/mikopbx/mikopbx:latest

# Starting the container in unprivileged mode
sudo docker run --net=host --name mikopbx --hostname mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-user)" \
           -e ID_WWW_GROUP="$(id -g www-user)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx:latest
```

### Updating using Docker compose

First, you need to properly stop the running container. After stopping the container, you can safely remove it

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)

sudo docker compose -f docker-compose.yml pull
sudo docker compose -f docker-compose.yml up -d
```

An example of the `docker-compose.yml` file that can be used to update your MikoPBX container through Docker Compose:

{% code title="docker-compose.yml" overflow="wrap" %}
```yaml
services:
  mikopbx:
    container_name: "mikopbx"
    image: "ghcr.io/mikopbx/mikopbx:latest"
    network_mode: "host"
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-a-docker"
    volumes:
      - /var/spool/mikopbx/cf:/cf
      - /var/spool/mikopbx/storage:/storage
    tty: true
    environment:
      - ID_WWW_USER=${ID_WWW_USER}
      - ID_WWW_GROUP=${ID_WWW_GROUP}
      # Change the station name through environment variables
      - PBX_NAME=MikoPBX-in-Docker
      # Change the default SSH port to 23
      - SSH_PORT=23
      # Change the default WEB port to 8080
      - WEB_PORT=8080
      # Change the default WEB HTTPS port to 8443
      - WEB_HTTPS_PORT=8443
```
{% endcode %}

Save the contents to a file named `docker-compose.yml`, make the necessary adjustments, and run the command:

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)

sudo docker compose -f docker-compose.yml pull
sudo docker compose -f docker-compose.yml up -d
```

### Notes

* **Data**: Since data is stored in Docker volumes (`mikopbx_cf` and `mikopbx_storage`), it remains untouched during the update, preserving settings and user data.
* **Environment Variables**: Ensure that all necessary environment variables are correctly passed.
* **Safety**: Always create backups of your data before updating.

These steps will help ensure a smooth and safe update of your MikoPBX container.
