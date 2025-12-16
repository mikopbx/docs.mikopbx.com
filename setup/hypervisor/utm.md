---
description: Installing MikoPBX in UTM
---

# UTM

In this manual, the installation will be performed on UTM. Before it starts, download the disk image file with the ".iso" extension. You can do this by [following this link](https://github.com/mikopbx/core/releases).&#x20;

{% hint style="info" %}
This instruction has been relevant since the first release, published in 2026. Tested on Apple Silicon processors.
{% endhint %}

## Creating a virtual machine

1. Go to UTM. Click "Create a New Virtual Machine" to create a new virtual machine.

<figure><img src="../../.gitbook/assets/UTMDashboard.png" alt=""><figcaption><p>The main page of UTM. Creating a new VM.</p></figcaption></figure>

2. Select "Virtualize" as the VM type.

<figure><img src="../../.gitbook/assets/chooseTypeOfGuestM.png" alt=""><figcaption><p>Selecting the type of virtual machine</p></figcaption></figure>

3. Select "Preconfigured" - "Linux" as the operating system type.

<figure><img src="../../.gitbook/assets/chooseOS.png" alt=""><figcaption><p>Choosing the type of operating system</p></figcaption></figure>

4. Select the previously downloaded disk image file in the "Boot ISO Image" section. To do this, click on "Browse...".

<figure><img src="../../.gitbook/assets/chooseAnImage.png" alt=""><figcaption><p>Selecting a disk image file for a VM</p></figcaption></figure>

5. Next, specify the characteristics of your virtual machine. In our case, 2 GB of RAM and 2 processor cores will be used.

<figure><img src="../../.gitbook/assets/setHardware.png" alt=""><figcaption><p>VM Configuration</p></figcaption></figure>

6. Next, specify the size for the system disk. In our case, 1 GB.

{% hint style="info" %}
MikoPBX uses two disks:

1. The system disk. The system is installed on it, the recommended size is 1 GB.
2. A disk for storing recordings of conversations. The recommended size is from 50 GB.
{% endhint %}

<figure><img src="../../.gitbook/assets/setStorage.png" alt=""><figcaption><p>Specifying the size of the system disk</p></figcaption></figure>

7. Click Continue.

<figure><img src="../../.gitbook/assets/setSharedDirectory.png" alt=""><figcaption><p>The "Shared Directory" section</p></figcaption></figure>

8. The final configuration of the VM will be displayed. Give it the desired name (the "Name" field). And click "Save".

<figure><img src="../../.gitbook/assets/guestMachineSummary.png" alt=""><figcaption><p>The final configuration</p></figcaption></figure>

## Connecting a disk for data storage

1. Go to the VM settings. To do this, right-click on its name, then "Edit".

<figure><img src="../../.gitbook/assets/VMGoToSettings.png" alt=""><figcaption><p>VM Settings</p></figcaption></figure>

2. Go to "Drives". Click "New..."

<figure><img src="../../.gitbook/assets/addingNewDisk(Ch1).png" alt=""><figcaption><p>"Drives" section</p></figcaption></figure>

3. Create a new disk with the following parameters:

* **Interface** - VirtlO
* **Size** - at least 50 GB (in this documentation, 10 GB will be used for the test machine)

Click "Create".

<figure><img src="../../.gitbook/assets/addingNewDisk(Ch2).png" alt=""><figcaption><p>Creating a second disk</p></figcaption></figure>

## System installation

1. Start the VM.

<figure><img src="../../.gitbook/assets/UTMDashboardStartVM.png" alt=""><figcaption><p>Launching a VM</p></figcaption></figure>

2. After loading, you will see the message <mark style="color:red;">PBX is running in Live or Recovery mode</mark>. This means that the system is loaded from the disk image in Live mode. It is necessary to install the system. To do this, go to the "\[8] Install on Hard Drive" section.

<figure><img src="../../.gitbook/assets/PBBRunningLiveCD.png" alt=""><figcaption><p>MikoPBX in LiveCD mode</p></figcaption></figure>

3. Select the disk to install the system. In our case, vda and vdb disks are available, and we select the vda disk for installation.

<figure><img src="../../.gitbook/assets/ConnectingSystemDisk(Ch1).png" alt=""><figcaption><p>Selecting a disk to install the system on</p></figcaption></figure>

4. Confirm the selection: enter "**y**" from the keyboard and press Enter.

<figure><img src="../../.gitbook/assets/ConnectingSystemDisk(Ch2).png" alt=""><figcaption><p>Confirming the disk selection</p></figcaption></figure>

5. Next, select a disk for storing conversation recordings. In our case, the only remaining one is 10 GB size disk.

<figure><img src="../../.gitbook/assets/ConnectingStorageDisk.png" alt=""><figcaption><p>Selecting a disk for storing conversation recordings</p></figcaption></figure>

After that, the system will reboot and be available in normal mode (the label "<mark style="color:red;">PBX is running in Live or Recovery mode</mark>" will disappear).

<figure><img src="../../.gitbook/assets/PBXReady.png" alt=""><figcaption><p>MikoPBX IP-address</p></figcaption></figure>

Enter this IP address in the browser bar to access the Web interface.

<figure><img src="../../.gitbook/assets/WEBLoginForm.png" alt=""><figcaption><p>MikoPBX Web-interface</p></figcaption></figure>

{% hint style="info" %}
Standard login information:

* Login: admin
* Password: admin
{% endhint %}
