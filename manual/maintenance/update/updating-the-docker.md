# Updating the docker

To update the MikoPBX container to the latest version, you can follow these steps in the command line. These steps include stopping the current container, downloading the new version of the image, and running the container with the updated image.&#x20;

### Updating the docker

First, you need to properly stop the running container. After stopping the container, you can safely remove it

```bash
# Stop the current container
sudo docker stop mikopbx

# Remove the current container
sudo docker rm mikopbx
```

To launch a new container using the latest image version with the same settings as before, use the following commands:

```bash
# Downloading the latest container image version
sudo docker pull ghcr.io/mikopbx/mikopbx-x86-64:latest

# Starting the container in unprivileged mode
sudo docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
           -v data_volume:/cf \
           -v data_volume:/storage \
           -e SSH_PORT=23 \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64:latest
```

### Updating using Docker compose

First, you need to properly stop the running container. After stopping the container, you can safely remove it

```bash
# Stop the current container
sudo docker stop mikopbx

# Remove the current container
sudo docker rm mikopbx
```

The next step is to download the latest MikoPBX image:

```bash
# Скачивание последней версии образа контейнера
sudo docker pull ghcr.io/mikopbx/mikopbx-x86-64:latest
```

An example of the `docker-compose.yml` file that can be used to update your MikoPBX container through Docker Compose:

{% code title="docker-compose.yml" overflow="wrap" %}
```yaml
services:
  mikopbx:
    container_name: "mikopbx"
    image: "ghcr.io/mikopbx/mikopbx-x86-64:latest"
    network_mode: "host"
    cap_add:
      - NET_ADMIN
    entrypoint: "/sbin/docker-entrypoint"
    hostname:  "mikopbx-in-a-docker"
    volumes:
      - data_volume:/cf
      - data_volume:/storage
    tty: true
    environment:
      # Change the station name through environment variables
      - PBX_NAME=MikoPBX-in-Docker
      # Change the default SSH port to 23
      - SSH_PORT=23
      # Change the default WEB port to 8080
      - WEB_PORT=8080
      # Change the default WEB HTTPS port to 8443
      - WEB_HTTPS_PORT=8443
      
volumes:
  data_volume:
```
{% endcode %}

Save the contents to a file named `docker-compose.yml`, make the necessary adjustments, and run the command:

```bash
sudo docker compose -f docker-compose.yml up
```

### Notes

* **Data**: Since data is stored in Docker volumes (`mikopbx_cf` and `mikopbx_storage`), it remains untouched during the update, preserving settings and user data.
* **Environment Variables**: Ensure that all necessary environment variables are correctly passed.
* **Safety**: Always create backups of your data before updating.

These steps will help ensure a smooth and safe update of your MikoPBX container.
