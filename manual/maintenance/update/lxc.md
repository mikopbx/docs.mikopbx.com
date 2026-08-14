---
description: Updating MikoPBX deployed in a Proxmox VE LXC container
---

# Updating an LXC container

{% hint style="warning" %}
We recommend updating sequentially, without skipping releases and versions.
{% endhint %}

This guide is intended for a system-level MikoPBX LXC container in Proxmox VE.

LXC is different from both a full-fledged virtual machine and Docker. The MikoPBX root file system is deployed from the `lxc.tar.gz` template, while the configuration and recordings are attached to the container as separate `/cf` and `/storage` mount points.

{% hint style="danger" %}
Do not run an LXC update via `.img` in the **PBX update** section and do not attach an ISO. The MikoPBX IMG updater is designed for a full system disk with a boot partition and partition UUIDs. An LXC container does not have such a disk layout.
{% endhint %}

## Safe approach

For LXC, it is safer not to modify the root file system of the running container manually, but to deploy a new container from the up-to-date `lxc.tar.gz`, migrate the settings and data, verify the operation, and only then delete the old container.

Advantages of this approach:

* the original container remains available for a quick rollback;
* there is no need to write a disk `.img` inside the LXC;
* the new version gets an up-to-date root file system and entrypoint;
* the switch is performed only after the new PBX has been verified.

## Preparing the old container

1. Create a backup of the settings with the MikoPBX tools and download it to the administrator's computer.
2. Separately save the call recordings and other data from `/storage` if they must be migrated to the new container.
3. In Proxmox VE, open the old CT → **Backup** → **Backup now** and create a full backup of the container.
4. Write down the parameters of the old CT:
   * hostname;
   * CPU and RAM amounts;
   * IPv4/IPv6, bridge, VLAN, and MAC address;
   * DNS;
   * the `/cf` and `/storage` mount point parameters;
   * additional capabilities, including `CAP_NET_ADMIN` if it was used;
   * password and SSH keys.
5. Do not delete or modify the old container until the new version has been fully verified.

{% hint style="warning" %}
A snapshot does not replace a backup. A snapshot depends on the original Proxmox storage, while a backup can be restored independently.
{% endhint %}

## Downloading the new template

1. Open the [MikoPBX releases](https://github.com/mikopbx/Core/releases) page.
2. Copy the link to the file of the required version ending with `lxc.tar.gz`.
3. In Proxmox VE, select the **local** storage → **CT Templates**.
4. Click **Download from URL**.
5. Paste the link, click **Query URL**, then **Download**.
6. Wait for the **TASK OK** message.

## Creating the new container

Create a new CT following the [Proxmox LXC installation](../../../setup/hypervisor/proxmox/lxc.md) guide, but keep the following rules in mind:

1. Use a **new CT ID** — the old container must be preserved.
2. Select the downloaded `lxc.tar.gz` template of the new version.
3. Create separate mount points:
   * `/storage` — for recordings and user data;
   * `/cf` — for the MikoPBX configuration.
4. Assign a different IP address for the verification period so that the new CT does not conflict with the old one.
5. Repeat the CPU, RAM, bridge, VLAN, DNS, and required capabilities of the old container.
6. Start the new CT and wait for the IP address to appear in the console.

{% hint style="info" %}
Do not attach the same writable `/cf` or `/storage` mount point to two running containers at the same time. This can lead to data corruption.
{% endhint %}

## Migrating settings and data

The recommended option is a new `/cf` configuration area with a restore from the MikoPBX backup.

1. Open the web interface of the new container via its temporary IP address.
2. Install the backup module you use if it is not installed.
3. Upload the backup of the old PBX and perform the restore.
4. Migrate the required data from the old `/storage` to the new `/storage` or restore it from a separate backup.
5. Reboot the new container and make sure the settings have been preserved.

If the volume of recordings is large, migrate the Storage using Proxmox or file storage tools with the old container stopped. The method depends on the type of Proxmox storage and therefore must not be replaced with a generic copy command.

## Switching to the new container

1. End active calls on the old PBX.
2. Stop the old CT.
3. Assign the former IP address to the new CT or switch DNS/NAT to the new address.
4. If the former MAC address was used, make sure the old CT is stopped before starting the new one.
5. Start or reboot the new CT.
6. Check access to the web interface from the working networks.

## Verification after the update

1. Check the MikoPBX version in the web interface.
2. Make sure `/cf` and `/storage` are mounted and writable.
3. Check the employees, providers, routes, modules, and firewall rules.
4. Make sure the phones and SIP providers have registered.
5. Make a test inbound and outbound call.
6. Check that a new call recording is created.
7. Reboot the CT from Proxmox and verify the network and web interface availability again.

{% hint style="warning" %}
The operation of the firewall inside an LXC container depends on the presence of `CAP_NET_ADMIN`. If the capability is missing, network restrictions must be provided by the Proxmox firewall and other external means.
{% endhint %}

## Rollback

If the new PBX does not work correctly:

1. Stop the new CT.
2. Restore the former IP, DNS, or NAT settings of the old CT.
3. Start the old container.
4. If the old container was modified, restore it from the Proxmox backup.

Delete the old CT and its backup only after several days of stable operation of the new version and after verifying the backup process.
