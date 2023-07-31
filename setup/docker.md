# Docker container

### Installation

{% hint style="danger" %}
The "**host system**" must be running on Linux 5+. Tested on Debian 11 and Ubuntu-21.04. Plans to add ARM support.
{% endhint %}

```
# Creating directories on the host system for storing MikoPBX data
# To store settings:
mkdir -p /var/spool/mikopbx/cf 
# For storing conversation recordings and backups:
mkdir -p /var/spool/mikopbx/storage

# Getting the container:
docker pull ghcr.io/mikopbx/mikopbx-x86-64

# MikoPBX launch option
# Unprivileged mode. The user and the "www-data" group must exist in the system:
docker run --cap-add=NET_ADMIN --net=host --name mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-data)" \
           -e ID_WWW_GROUP="$(id -g www-data)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64
```

{% hint style="warning" %}
The **NET\_ADMIN** flag is required for **fail2ban** and **iptables** to work inside the container.
{% endhint %}

{% hint style="danger" %}
If it is necessary to use the "[Backup Module](../manual/maintenance/modul-rezervnogo-kopirovaniya.md)", then the container should be started with the **–privileged** flag. Backup can be performed manually archiving the **cf** and **storage** directories (when the container is stopped).
{% endhint %}

{% hint style="info" %}
The **--net=host** flag indicates that NAT will not be used for the new container. All ports that the container needs will be occupied on the host machine. Please refer to the [documentation ](https://docs.docker.com/network/host/)for more details. If any of the ports are already occupied on the host machine, errors may occur during the MikoPBX loading process.

Below are the environment variables that allow you to adjust the ports used by MikoPBX:

* **SSH\_PORT** - Port for SSH (**22**)&#x20;
* **WEB\_PORT** - Port for the web interface using HTTP protocol (**80**)&#x20;
* **WEB\_HTTPS\_PORT** - Port for the web interface using HTTPS protocol (**443**)&#x20;
* **SIP\_PORT** - Port for SIP client connections (**5060**)&#x20;
* **RTP\_FROM** - Starting point of the RTP port range for voice transmission (**10000**)&#x20;
* **RTP\_TO** - Ending point of the RTP port range for voice transmission (**10200**)&#x20;
* **IAX\_PORT** - Port for IAX client connections (**4569**)&#x20;
* **AMI\_PORT** - AMI port (**5038**)&#x20;
* **AJAM\_PORT** - AJAM port used for connecting the telephony panel for 1C (**8088**)&#x20;
* **AJAM\_PORT\_TLS** - AJAM port used for connecting the telephony panel for 1C (**8089**)&#x20;
* **BEANSTALK\_PORT** - Port for **Beanstalkd** queue server (**4229**)&#x20;
* **REDIS\_PORT** - Port for **Redis** server (**6379**)&#x20;
* **GNATS\_PORT** - Port for **gnatsd** server (**4223**)&#x20;
* **ID\_WWW\_USER** - Identifier of the "www" user (can be set with the expression "**$(id -u www-data)**", where www-data is the name of the **NON-root** user)&#x20;
* **ID\_WWW\_GROUP** - Identifier of the "www" group (can be set with the expression "**$(id -g www-data)**", where www-data is the name of the **NON-root** user)
{% endhint %}

### Example docker-compose.yml:

<pre class="language-docker"><code class="lang-docker">version: "3.9"
services:
<strong>    mikopbx:
</strong>        container_name: "mikopbx"
        image: ""
        network_mode: "host"
        command: '-d'
        cap_add:
        - NET_ADMIN
        volumes:
        - /var/spool/mikopbx/cf:/cf
        - /var/spool/mikopbx/storage:/storage
        # environment:
        ## Change the default SSH port to 23
        # - SSH_PORT=23
        ## Change the default HTTP port to 81
        # - WEB_PORT=81
        # DAHDI is not a mandatory requirement. It is necessary for MeetMe functionality in the telephony panel.
        # devices:
        # - "/dev/dahdi/transcode:/dev/dahdi/transcode"
        # - "/dev/dahdi/channel:/dev/dahdi/channel"
        # - "/dev/dahdi/ctl:/dev/dahdi/ctl"
        # - "/dev/dahdi/pseudo:/dev/dahdi/pseudo"
        # - "/dev/dahdi/timer:/dev/dahdi/timer"
</code></pre>

{% hint style="warning" %}
There are no modules compatible with the DAHDI host system kernel inside the container. For this reason, if the functionality of MeetMe conferences is needed, then DAHDI will need to be assembled on the host system manually.
{% endhint %}

Command to connect to the PBX console:

```
docker exec -it mikopbx sh
```

Command to connect to the PBX console menu:

```
docker exec -it mikopbx /etc/rc/console_menu
```
