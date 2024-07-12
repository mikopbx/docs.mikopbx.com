# Updating the docker container

## Updating the docker container <a href="#docker-upgrade" id="docker-upgrade"></a>

To update the MikoPBX container to the latest version, you can follow these steps in the command line. These steps include stopping the current container, downloading the new version of the image, and running the container with the updated image. Here is an example of the process:

#### Step 1: Stop the current container

First, you need to properly stop the running container. This prevents data loss and ensures that all processes within the container are correctly terminated:

```bash
docker stop mikopbx
```

#### Step 2: Remove the current container

After stopping the container, you can safely remove it. Removing the container allows you to start a new instance with new settings and the updated image:

```bash
docker rm mikopbx
```

#### Step 3: Download the latest version of the image

The next step is to download the latest version of the MikoPBX image. Using the `latest` tag ensures you get the most recent version:

```bash
docker pull ghcr.io/mikopbx/mikopbx-x86-64:latest
```

#### Step 4: Run the new container with the updated image

Finally, run the new container using the latest version of the image and the same settings as before (including volume mounts and other network parameters):

```bash
docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
  -v mikopbx_cf:/cf \
  -v mikopbx_storage:/storage \ 
  -e SSH_PORT=23 \ 
  -e ID_WWW_USER="$(id -u www-user)" \ 
  -e ID_WWW_GROUP="$(id -g www-user)" \ 
  -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64:latest
```

### Notes

* **Data**: Since data is stored in Docker volumes (`mikopbx_cf` and `mikopbx_storage`), it remains untouched during the update, preserving settings and user data.
* **Environment Variables**: Ensure that all necessary environment variables are correctly passed.
* **Safety**: Always create backups of your data before updating.

These steps will help ensure a smooth and safe update of your MikoPBX container.
