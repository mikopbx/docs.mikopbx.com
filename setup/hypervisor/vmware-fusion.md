# VMware Fusion

## Creating a virtual machine

1. Creating a new virtual machine.

<figure><img src="../../.gitbook/assets/1 (6).png" alt=""><figcaption></figcaption></figure>

2. After downloading the latest version of the image ([link](https://www.mikopbx.ru/download/)), specify the ISO file with the installation distribution.

Click "**Continue**"

<figure><img src="../../.gitbook/assets/2 (10).png" alt=""><figcaption></figcaption></figure>

3. Select the type of operating system **Other Linux 5.x and later kernel 64-bit**

Click "**Continue**"

<figure><img src="../../.gitbook/assets/22 (1).png" alt=""><figcaption></figcaption></figure>

4. Choosing the **Legacy** bios type

Click "**Continue**"

<figure><img src="../../.gitbook/assets/4 (17).png" alt=""><figcaption></figcaption></figure>

5. Click "**Finish**"

<figure><img src="../../.gitbook/assets/5 (20).png" alt=""><figcaption></figcaption></figure>

## Connecting a new disk

1. After creating a virtual machine, wait for it to load

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

2. Go to the section "**\[3] Reboot the system**"

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

3. Choose "**\[2]** **Shutdown**"

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

4. After shutting down the virtual machine, go to Settings

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

5. Select "**Add device**"

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

6. Select "**New Hard Disk**"

&#x20;Click "**Add...**"

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

7. Choose the size of the hard drive (we recommend **at least 50 GB**)

&#x20;Click "**Apply**"

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**1 hour** of recording conversations takes approximately **14mb** on disk.
{% endhint %}

## Installing MikoPBX

1. Start the virtual machine

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

2. Select "**\[8] Install**"

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

3. Enter the name of the disk on which MikoPBX will be installed

In our case - _sdb_, enter its name and press **Enter**

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

4. Confirm the disk selection: enter **y**

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

5. Select a disk for recording conversations

In our case - _sdc_, enter its name and press **Enter**

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

6. The system will reboot and MikoPBX will be ready for use.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## First connection to MikoPBX

1. The PBX displays the IP address of the station by which you can connect to it

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

2. Enter the IP address of the station in the browser bar and the MIkoPBX login menu will open

&#x20;The default username and password is "**admin**"

<figure><img src="../../.gitbook/assets/21 (1).png" alt=""><figcaption></figcaption></figure>
