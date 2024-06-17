# Hyper-V

### **Creating a virtual machine**

1. Select **Action / New / Virtual Machine**
2. On the Specify Name and Location tab, enter the name of the virtual machine, for example, _mikopbx-vm_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_1.png" alt=""><figcaption></figcaption></figure>

3. Proceed to the Specify Generation tab, and select **Generation 1**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_2.png" alt=""><figcaption></figcaption></figure>

4. On the Assign Memory tab, allocate the required amount of RAM based on the expected load on the PBX. For a test machine, you can specify 2 GB

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_3.png" alt=""><figcaption></figcaption></figure>

5. Proceed to the Configure Networking tab, and select a pre-configured network connection

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_4.png" alt=""><figcaption></figcaption></figure>

6. On the Connect Virtual Hard Disk tab, adjust the system disk size to **1 GB**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_5.png" alt=""><figcaption></figcaption></figure>

7. On the Installation Options tab, check the **Install an operating system from a bootable CD/DVD-ROM** option
8. Select **Image file (.iso)** and provide the link to the MikoPBX distribution file with the **.iso** extension

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_6.png" alt=""><figcaption></figcaption></figure>

9. After entering all values, click the **Finish** button

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_7.png" alt=""><figcaption></figcaption></figure>

### **Data storage disk**

{% hint style="danger" %}
For deploying the PBX, use **two** disks:

* a **1 GB** disk for the main system
* a **50+ GB** disk for storing call recordings
{% endhint %}

1. Go to the settings of the created virtual machine
2. Select the IDE controller to which the system disk is connected
3. On the opened tab, select Hard Drive and click the **Add** button
4. Click the **New** button
5. On the Choose disk format tab, select **VHD**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_8.png" alt=""><figcaption></figcaption></figure>

6. On the Choose disk type tab, select **Fixed size**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_9.png" alt=""><figcaption></figcaption></figure>

7. On the Specify name and location tab, specify the name (e.g., _storage.vhd_) and the location of the disk

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_10.png" alt=""><figcaption></figcaption></figure>

8. On the Configure Disk tab, set the disk size for data storage to at least 50 GB

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_11.png" alt=""><figcaption></figcaption></figure>

9. Use the default values for other fields
10. Complete the setup by clicking the **Finish** button

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_12.png" alt=""><figcaption></figcaption></figure>

11. To start the virtual machine, click **Start**

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_13.png" alt=""><figcaption></figcaption></figure>

### **Installing MikoPBX**

1. Go to the Connect tab of the created virtual machine _mikopbx-vm_
2. If the boot is successful, a console menu will appear. Enter **8** from the keyboard to start the installation

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_14.png" alt=""><figcaption></figcaption></figure>

3. Select the system disk and enter the disk name from the keyboard, for example, _**sda**_. Confirm the selection by entering _**y**_ from the keyboard

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_15.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_16.png" alt=""><figcaption></figcaption></figure>

4. Connect the disk for storing call recordings, and enter the disk name for connection from the keyboard, for example, _**sdb**_

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_17.png" alt=""><figcaption></figcaption></figure>

### **Starting MikoPBX**

1. On the open Connect tab, copy the external address of the created virtual machine and enter it in the browser address bar

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_18.png" alt=""><figcaption></figcaption></figure>

2. To log in, use the username - admin and password - admin

<figure><img src="../../.gitbook/assets/MikoPBXHyperVInstallation_19.png" alt=""><figcaption></figcaption></figure>
