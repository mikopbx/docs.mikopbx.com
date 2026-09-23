---
description: Updating MikoPBX running in Docker or Docker Compose
---

# Updating a Docker container

{% hint style="warning" %}
We recommend updating sequentially, without skipping releases and versions.
{% endhint %}

A MikoPBX Docker container is not updated with `.img` or `.iso` files. To move to a new version, you need to download the new Docker image and recreate the container, preserving the `/cf` and `/storage` directories, network settings, and environment variables.

## Preparation

1. Create a backup of the MikoPBX settings. If the backup module is unavailable in the unprivileged container, prepare backup of the directories or volumes on the Docker host side.
2. Write down the name and version of the current image:

```bash
sudo docker inspect mikopbx --format '{{.Config.Image}}'
```

3. Save the current container configuration:

```bash
sudo docker inspect mikopbx > mikopbx-container-before-update.json
```

4. Check the `/cf` and `/storage` mounts:

```bash
sudo docker inspect mikopbx --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
```

5. Write down the ports in use, `network_mode`, hostname, and environment variables. If the container is started via Compose, save the current `docker-compose.yml` and the `.env` file.
6. End active calls and schedule a maintenance window.

## Updating a container started with the docker run command

### Downloading the new image

For the latest stable version, run:

```bash
sudo docker pull ghcr.io/mikopbx/mikopbx:latest
```

If a fixed tag is used, specify the same registry and the tag of the required version instead of `latest`.

### Stopping and replacing the container

1. Stop the container:

```bash
sudo docker stop mikopbx
```

2. Create a consistent backup of `/cf` while the container is stopped. For the bind mount from the example above, you can run:

```bash
sudo tar -C /var/spool/mikopbx -czf mikopbx-cf-before-update.tar.gz cf
```

In addition to `/cf`, it is worth backing up the CDR database before the new version — it is located in `/storage` and may also be migrated on first start. `/cf` contains only `mikopbx.db` (the settings), while the call history is a separate `cdr.db` (and `recording_storage.db`) in `/storage/usbdisk1/mikopbx/astlogs/asterisk/`.

The backup must be made **with the container stopped** (as for `/cf`) so that the database is in a consistent state:

```bash
sudo docker stop mikopbx
# backup of /cf (settings)
sudo tar -C /var/spool/mikopbx -czf mikopbx-cf-before-update.tar.gz cf
# backup of the CDR + recording metadata
sudo tar -C /var/spool/mikopbx/storage/usbdisk1/mikopbx \
  -czf mikopbx-cdr-before-update.tar.gz astlogs/asterisk
```

When rolling back, restore both archives: first `/cf`, then the CDR — otherwise the call history and the link between recordings and calls may get out of sync with the settings.

3. Rename the old container instead of deleting it immediately:

```bash
sudo docker rename mikopbx mikopbx-before-update
```

4. Start a new container with the same parameters. Example for an installation with host network and bind mounts:

```bash
sudo docker run --net=host --name mikopbx --hostname mikopbx \
  -v /var/spool/mikopbx/cf:/cf \
  -v /var/spool/mikopbx/storage:/storage \
  -e SSH_PORT=23 \
  -e ID_WWW_USER="$(id -u www-user)" \
  -e ID_WWW_GROUP="$(id -g www-user)" \
  -it -d --restart always ghcr.io/mikopbx/mikopbx:latest
```

{% hint style="warning" %}
The example must not be copied without verification. Repeat all the parameters of your exact previous installation: network type, volumes, ports, hostname, capabilities, and environment variables.
{% endhint %}

5. Monitor the startup:

```bash
sudo docker logs -f mikopbx
```

To exit the log view, press `Ctrl+C` — the container will keep running.

## Updating with Docker Compose

1. Go to the directory containing `docker-compose.yml`.
2. If user IDs are passed through the environment, set them the same way as on first launch:

```bash
export ID_WWW_USER=$(id -u www-user)
export ID_WWW_GROUP=$(id -g www-user)
```

3. Download the new image:

```bash
sudo docker compose pull
```

4. Recreate the container:

```bash
sudo docker compose up -d
```

5. Check the status and logs:

```bash
sudo docker compose ps
sudo docker compose logs -f mikopbx
```

Docker Compose will recreate the container but keep the bind mounts and named volumes specified in `docker-compose.yml`.

## Verification after the update

1. Make sure the container is running:

```bash
sudo docker ps --filter name=mikopbx
```

2. Open the web interface and check the MikoPBX version.
3. Check that the settings and call recordings are in place.
4. Make sure the phones and SIP providers have registered.
5. Make a test inbound and outbound call.
6. Check the publishing of web, SIP, and RTP ports if a bridge network is used.

## Rollback

If the new container does not start:

1. Save its logs:

```bash
sudo docker logs mikopbx > mikopbx-update-error.log 2>&1
```

2. Stop and rename the new container:

```bash
sudo docker stop mikopbx
sudo docker rename mikopbx mikopbx-failed-update
```

3. Restore the `/cf` backup created before the update. The restoration method depends on whether a bind mount, a named volume, or a storage snapshot is used.
4. Give the old container its previous name back and start it:

```bash
sudo docker rename mikopbx-before-update mikopbx
sudo docker start mikopbx
```

{% hint style="warning" %}
On first start, the new version may have upgraded the database in `/cf`. Therefore, a reliable rollback must rely not only on the old container image but also on the `/cf` backup created before the update.
{% endhint %}

To roll back a Docker Compose installation, specify the previous image tag in `docker-compose.yml`, restore the previous state of `/cf`, and run again:

```bash
sudo docker compose up -d
```

After successful verification, you can delete the old container:

```bash
sudo docker rm mikopbx-before-update
```
