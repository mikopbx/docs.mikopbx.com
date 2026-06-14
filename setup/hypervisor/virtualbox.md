---
description: Installing MikoPBX as a guest machine in VirtualBOX
---

# VirtualBOX

{% hint style="warning" %}
Use versions of MikoPBX below 2024.1.114 for installation on VirtualBOX

Version 2024.1.114 temporarily does not support installation on VirtualBOX
{% endhint %}

{% embed url="https://youtu.be/aBSQr58lADA" %}

## Create a virtual machine

1. Download **Virtual Box** from the [link](https://www.virtualbox.org/wiki/Downloads) and install it.
2. Create a new virtual machine.

![Button for creating a new VM](<../../.gitbook/assets/1 (22).png>)

3. Specify the **Machine Name** and **Folder**.

**Type** - <mark style="color:green;">Linux.</mark>

**Version -** <mark style="color:green;">Other Linux (64Bit).</mark>

Click **Next**.

![Parameters of the new VM](<../../.gitbook/assets/2 (23).png>)

4. Specify the size of the base memory - **1024 MB**, as well as the number of processors - **2**

Press **Next**.

![Parameters of the new VM](<../../.gitbook/assets/3 (25).png>)

5. Select Create a new virtual hard disk. Enter a disk size of **700 MB**, and also check the box **"Pre-allocate Full Size"**

Click **Create**.

![Parameters of the new VM](<../../.gitbook/assets/4 (6).png>)

6. Confirm the creation of the virtual machine: click **Finish**.

<figure><img src="../../.gitbook/assets/5 (14).png" alt=""><figcaption><p>Summary configuration</p></figcaption></figure>

## Setting up a virtual machine

1. Go to the settings of the created virtual machine.

To do this, click "**Settings" in the upper menu**.

!["Settings" button](../../.gitbook/assets/6.png)

2. Click the "**Storage"** tab. Add a new hard drive to store call records.

![Creating a new hard disk](<../../.gitbook/assets/7 (4).png>)

3. In the window that appears, click **Create**.

![Creating a new hard disk](<../../.gitbook/assets/8 (3).png>)

4. Select the hard disk format - **VDI (VirtualBox Disk Image).**

Click **Next.**

![Creating a new hard disk](<../../.gitbook/assets/9 (1).png>)

5. The hard disk must be of a **fixed size**.

Check the box next to **"Pre-allocate Full Size"**

Click **Next.**

![Creating a new hard disk](<../../.gitbook/assets/10 (2).png>)

6. Specify the **Name** of the created disk.

Set the Disk Size to about **50 GB**.

Click **Finish.**

![Creating a new hard disk](<../../.gitbook/assets/11 (2).png>)

7. Choose the newly created drive and click **Select**.

![Selecting a new hard disk](<../../.gitbook/assets/12 (5).png>)

8. The created drive will appear in the media list.

![](<../../.gitbook/assets/16 (1).png>)

9. Please mount the MikoPBX image onto an optical disc. To do this, select the optical disc in the 'Media' section and click on the image file selection button in the 'Attributes' section.

![Mounting an image](<../../.gitbook/assets/13 (2).png>)

10. In the appeared menu, click on '**Choose a disk file..**.'

![Mounting an image](../../.gitbook/assets/14.png)

11. Select the downloaded **ISO disk image**.

<figure><img src="../../.gitbook/assets/new1 (3).png" alt=""><figcaption><p>List of MikoPBX images option. You need .iso image!</p></figcaption></figure>

12. "Go to the '**Network**' tab.

Set the Connection Type to '**Bridged Adapter**'. Click '**OK**' to save all the settings you have made.

!["Network" tab](../../.gitbook/assets/15.png)

## Installantion MikoPBX <a href="#ustanovka_mikopbx" id="ustanovka_mikopbx"></a>

1. Start the created virtual machine.

![Button for starting the created VM](../../.gitbook/assets/17.png)

2. The command interface of the PBX will open. The PBX will start booting.

At this stage, MikoPBX is booting from the optical disc containing the ISO image. This is indicated by the message: '<mark style="color:red;">The system is loaded in Recovery mode</mark>'.

![Installation process](../../.gitbook/assets/18.png)

{% hint style="info" %}
You can navigate through the menu items using the [arrow keys](https://en.wikipedia.org/wiki/Arrow_keys).

To select a menu item, press the Enter key.

Alternatively, you can select a menu item by pressing the corresponding [number on the alphanumeric keypad](https://en.wikipedia.org/wiki/Numeric_keypad)."
{% endhint %}

3. Install MikoPBX.

{% hint style="danger" %}
All data on the disk where MikoPBX is being installed will be lost
{% endhint %}

Click **Install.**

![Installation process](../../.gitbook/assets/19.png)

4. Information about all available disks will be displayed (in this example: sdb, sdc).

![Installation process](../../.gitbook/assets/20.png)

{% hint style="warning" %}
The disk where MikoPBX will be installed is referred to as the system disk (SYSTEM). You can choose a disk with a size larger than 500MB as the system disk.
{% endhint %}

5. Enter the name of the disk you referred to as the 'system disk' from the keyboard, in this case **sdb**, and press **Enter**. (If it is selected by default, you can simply press Enter).
6. The system will prompt for confirmation. Enter '**y**' and press **Enter**.

![Installation process](../../.gitbook/assets/21.png)

7. After completing the installation, you will be prompted to select a disk for storing call records.

{% hint style="info" %}
Approximately, **1 hour** of conversation takes up **14MB** of disk space.
{% endhint %}

Enter the disk name (in this example, the only available disk is 'cdc') and press Enter.

![Installation process](../../.gitbook/assets/22.png)

8. After the installation is complete, the system will reboot.

MikoPBX will now run from the sdb drive where you installed it.

We will see that the line "<mark style="color:red;">The system is loaded in Recovery mode</mark>" is missing.

![MikoPBX Console Page](../../.gitbook/assets/23.png)

## The first login to MikoPBX

To access the control panel, you need to enter the IP address of your virtual machine in the browser's address bar.

<figure><img src="../../.gitbook/assets/new2 (1).png" alt=""><figcaption><p>IP address of your MikoPBX</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/new3 (2).png" alt=""><figcaption><p>Web-interface of MikoPBX</p></figcaption></figure>

{% hint style="info" %}
Default creditionals for the first login are:

Username - admin

Password - admin

System will ask to change them after the first login. It is important for the security of your MikoPBX.
{% endhint %}

The installation of MikoPBX using VirtualBOX is now complete.
