---
description: Installing MikoPBX using VMware Fusion.
---

# VMware Fusion

## Creating a virtual machine

1. Creating a new virtual machine.

<figure><img src="../../.gitbook/assets/1 (6).png" alt=""><figcaption><p>Button for creating a new VM</p></figcaption></figure>

2. After downloading the latest version of the image ([link](https://www.mikopbx.ru/download/)), specify the ISO file with the installation distribution.

Click "**Continue**"

<figure><img src="../../.gitbook/assets/2 (10).png" alt=""><figcaption><p>Choosing image page</p></figcaption></figure>

3. Select the type of operating system **Other Linux 5.x and later kernel 64-bit**

Click "**Continue**"

<figure><img src="../../.gitbook/assets/22 (1).png" alt=""><figcaption><p>Choosing verison of OS for VM</p></figcaption></figure>

4. Choosing the **Legacy** bios type

Click "**Continue**"

<figure><img src="../../.gitbook/assets/4 (17).png" alt=""><figcaption><p>Choosing the boot firmware for VM</p></figcaption></figure>

5. Click "**Finish**"

<figure><img src="../../.gitbook/assets/5 (20).png" alt=""><figcaption><p>Summary config</p></figcaption></figure>

## Connecting a new disk

1. After creating a virtual machine, wait for it to load

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption><p>MikoPBX Console</p></figcaption></figure>

2. Go to the section "**\[3] Reboot the system**"

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>"[3] Reboot the system" element</p></figcaption></figure>

3. Choose "**\[2]** **Shutdown**"

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption><p>Shutting down the system</p></figcaption></figure>

4. After shutting down the virtual machine, go to Settings

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption><p>Settings button</p></figcaption></figure>

5. Select "**Add device**"

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p>Add device button</p></figcaption></figure>

6. Select "**New Hard Disk**"

&#x20;Click "**Add...**"

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>New Hard Disk</p></figcaption></figure>

7. Choose the size of the hard drive (we recommend **at least 50 GB**)

&#x20;Click "**Apply**"

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption><p>Parameters for new hard drive</p></figcaption></figure>

{% hint style="info" %}
**1 hour** of recording conversations takes approximately **14mb** on disk.
{% endhint %}

## Installing MikoPBX

1. Start the virtual machine

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption><p>Starting button</p></figcaption></figure>

2. Select "**\[8] Install**"

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>Installation process</p></figcaption></figure>

3. Enter the name of the disk on which MikoPBX will be installed

In our case - _sdb_, enter its name and press **Enter**

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption><p>Installation process</p></figcaption></figure>

4. Confirm the disk selection: enter **y**

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption><p>Installation process</p></figcaption></figure>

5. Select a disk for recording conversations

In our case - _sdc_, enter its name and press **Enter**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Installation process</p></figcaption></figure>

6. The system will reboot and MikoPBX will be ready for use.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>MikoPBX Console</p></figcaption></figure>

## First connection to MikoPBX

1. The PBX displays the IP address of the station by which you can connect to it

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption><p>IP address of MikoPBX</p></figcaption></figure>

2. Enter the IP address of the station in the browser bar and the MIkoPBX login menu will open:

{% hint style="info" %}
Default creditionals for the first login are:

Username - admin

Password - admin

System will ask to change them after the first login. It is important for the security of your MikoPBX.
{% endhint %}

<figure><img src="../../.gitbook/assets/21 (1).png" alt=""><figcaption></figcaption></figure>
