---
description: Installing MikoPBX in an LXC container
---

# Proxmox LXC Container

**Proxmox LXC** is a lightweight container solution within the Proxmox VE virtualization platform, based on LXC (Linux Containers) technology. They allow running isolated Linux systems with minimal resource consumption compared to full virtual machines.

### Downloading the Container Template

1. Go to the "**local**" storage, then "**CT Templates**". Click "**Download from URL**" to open the template download dialog from a URL.



Downloading a template from a link

2. Go to [MikoPBX GitHub](https://github.com/mikopbx/core/releases) with releases and copy the download link for the template file with the "**lxc.tar.gz**" extension.



Copying the template link

3. Paste the link into the "**URL**" field and click "**Query URL**". If you copied the correct link, the "**File name**" field will be populated with the filename having the "**lxc.tar.gz**" extension.

Click "**Download**" to start the download.



Downloading a template from URL

After the download is complete, you will see the "**TASK OK**" message.



Successful template download

### Creating an LXC Container

1. Click "**Create CT**" in the upper right part of the interface to create a new container.

\<figure>\<img src="../../../.gitbook/assets/createCTbutton.png" alt="">\<figcaption>\<p>"Create CT" button for creating a new container\</p>\</figcaption>\</figure>

2. Fill in all the basic container parameters:

* **Hostname** — enter a name for the service.
* **Password** — enter the password for logging into the MikoPBX web interface.
* **SSH public keys** — generate and paste your SSH key. You will then be able to use it to connect to the station via SSH. More details about key generation and SSH connection can be found [here](../../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/).

Click "**Next**".



Basic container parameters

3. Select the previously downloaded template in the "**Template**" section.

Click "**Next**".



Selecting a template for the container being created

4. Next, specify the system disk size. The recommended value is 1 GB.

Click "**Add**" to add a new disk.



System disk parameters

5. Specify the size of the second disk — this is where call recordings will be stored. The recommended size is at least 50 GB. Also specify the disk path — "**/storage**".

Click "**Add**" to add a new disk.



Parameters for the second disk

6. Specify the size of the third disk for storing configuration. The recommended size is 0.5 GB. Also specify the disk path — "**/cf**".

Click "**Next**".



Parameters for the third disk

7. On the next tab, specify the number of CPU cores to be used. For a small company, 1–2 cores is sufficient (see [this article](../../../master/system-requirements.md) for more details).

Click "**Next**".



Container parameters (CPU)

8. Next, specify the amount of RAM and Swap memory for the container.

{% hint style="info" %}
Swap is a disk area that the system uses as additional memory when RAM runs out. It operates significantly slower than RAM and serves as a reserve to prevent the system from terminating processes when memory is insufficient.
{% endhint %}

Click "**Next**".



Container parameters (Memory)

9. In the next section, configure your network settings. In our case, DHCP is used to obtain an IPv4 address. The Firewall does not need to be enabled here, but it must be configured later in MikoPBX (see [this article](../../../manual/connectivity/firewall.md) for more details).

Click "**Next**".



Container parameters (Network)

10. In the DNS settings section, click "**Next**".



Container parameters (DNS)

You will see the final container configuration. Click "**Finish**".



Final container configuration

### First Launch

1. Go to the management window of the created container by clicking on its name. Click the "**Start**" button to launch it.



Container startup process

2. Then go to the "**Console**" tab. Wait for the system to load and find the web interface IP address.



Web interface IP address

Enter it in your browser's address bar. Then perform the first login to MikoPBX.

{% hint style="info" %}
Login credentials:

**Login:** admin

**Password:** the password you set during the initial container creation.
{% endhint %}



MikoPBX web interface
