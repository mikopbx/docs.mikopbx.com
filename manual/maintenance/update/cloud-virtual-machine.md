---
description: Updating MikoPBX deployed as a virtual machine in the cloud
---

# Updating a cloud virtual machine

{% hint style="warning" %}
We recommend updating sequentially, without skipping releases and versions.
{% endhint %}

This guide is intended for MikoPBX running as a full-fledged cloud virtual machine: from a Marketplace or from an imported RAW, VHD, or other disk image.

{% hint style="info" %}
The built-in update mechanism is designed for a standard disk installation of MikoPBX with a boot partition, the `/cf` configuration partition, and attached Storage. It is not intended for Docker and LXC.
{% endhint %}

The primary method for updating a cloud PBX is through the web interface using an `.img` image. The ability to boot from an ISO depends on the particular cloud provider and is not universal.

{% hint style="danger" %}
Before updating, create two independent recovery points: a backup of the MikoPBX settings and a snapshot of the system disk or the entire virtual machine in the cloud provider's control panel.
{% endhint %}

## Before you start

1. Make sure you have access to the cloud management panel and the MikoPBX web interface. Save a backup of the MikoPBX settings and call recordings outside the virtual machine being updated.
2. Create a snapshot of the system disk. If there is a separate disk with recordings, create a snapshot for it as well or use storage backup.
3. Write down the current external and internal IP addresses, Security Group rules, attached disks, and network settings.
4. Make sure at least **400 MB** is free on the system disk with MikoPBX.
5. Schedule a maintenance window: active calls will be dropped.

## Online update

1. Log in to the MikoPBX web interface.
2. Open **"Maintenance"** → **"PBX update"**.
3. Find the version you need in the **"Online updates available"** table. Review the list of changes and start the download.
4. Once the download is complete, enter the phrase **"Yes, I have a backup"**. Click **Update**.
5. Do not perform Restart or Stop from the cloud panel: MikoPBX will prepare the update and reboot on its own. During the reboot, the connection to the web interface will be interrupted. It is safer to monitor the process via the cloud provider's serial console or web console.

## Updating with a downloaded IMG file

Use a local file if the version is not in the list or the cloud VM does not have direct access to the update server.

1. Download the `.img` of the required version from the [MikoPBX releases](https://github.com/mikopbx/Core/releases) page.

{% hint style="info" %}
Pay attention to the architecture of your MikoPBX: if you have an ARM MikoPBX, use the update file with "arm64" in its name. If you have an x86 MikoPBX, use the update file with "x86\_64" in its name.
{% endhint %}

2. Verify that the image architecture matches the VM architecture.
3. Open **Maintenance** → **PBX update**.
4. Select the downloaded `.img` and click **Apply update**.
5. Enter the phrase **"Yes, I have a backup"** and confirm the operation.
6. Wait for the automatic reboot and the services to start again.

## Is it possible to update a cloud VM via ISO

Updating via ISO is possible only when the cloud platform simultaneously allows you to:

* upload or attach a custom ISO;
* change the VM boot order;
* open an interactive console;
* after the update, disconnect the ISO and boot from the system disk again.

{% hint style="danger" %}
If the provider does not support ISO, do not try to imitate this method by replacing the system disk. Use the web update or restore from a snapshot.
{% endhint %}

If all the conditions are met (for example, as [with Vultr](../../../setup/cloud/vultr.md)), the procedure is the same as for a regular virtual machine:

1. Download and attach the ISO image of the required version from the [MikoPBX releases](https://github.com/mikopbx/Core/releases) page.
2. Start the machine and open its console. Wait for the message about booting in recovery mode — **System booted in recovery mode (Live CD)**.
3. Press any key to open the console menu. Then open **"\[4] Install or recover"**.
4. Select **"Update to version..."**

{% hint style="danger" %}
Do not select the **Install** option. It starts a new installation and warns that data on the selected disk will be erased. To update while keeping the settings, use the **Update to version ...** option.
{% endhint %}

5. When finished, disconnect the ISO and boot from the system disk.
