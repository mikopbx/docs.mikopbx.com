---
description: MikoPBX Installation Guide using Microsoft Azure
---

# Microsoft Azure

First, log in to the Microsoft Azure portal [https://portal.azure.com/](https://portal.azure.com/)

Let's proceed with the setup.

{% hint style="info" %}
For quick and convenient searching on the Azure portal, use the search bar.
{% endhint %}

### Creating a resource group

1. Open Menu / All services / General / **Resource groups**
2. In the Resource groups tab, select **Create**
3. Enter the group name, for example _MikoPBX\_group_
4. Use default values for other fields
5. After entering the values, click the **Review + create** button, then the **Create** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_1.png" alt=""><figcaption></figcaption></figure>

### Creating a storage account

1. Open Menu / All services / Analyze and transform data / **Storage accounts**
2. In the Storage accounts tab, select **Create**
3. Specify the created resource group _MikoPBX\_group_
4. Enter the storage account name, for example _pbximgs_
5. Use default values for other fields
6. After entering the values, click the **Review + create** button, then the **Create** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_2.png" alt=""><figcaption></figcaption></figure>

### Configuring the created storage account

1. Go to the card of the created storage account _pbximgs_
2. In the opened tab, go to the Data storage / Containers
3. Add a new container
4. Enter the container name, for example _imgs_
5. Click the **Create** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_3.png" alt=""><figcaption></figcaption></figure>

6. Open the created container _imgs_
7. In the opened tab, select **Upload**
8. Select a file from the MikoPBX distribution with the **.vhd** extension
9. Click the **Upload** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_4.png" alt=""><figcaption></figcaption></figure>

### Creating an image

1. Open Menu / All services / Compute / **Images**
2. In the Images tab, select **Create**, let's create a new image based on the uploaded \*.**vhd** file
3. Specify the resource group _MikoPBX\_group_
4. Enter a unique name for the image, for example _MikoPBX\_Azure_

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_5.png" alt=""><figcaption></figcaption></figure>

5. Specify the OS type - **Linux**
6. Specify the generation of virtual machines - **Gen 1**
7. Select the blob storage object by clicking **Browse**, _Browse / pbximgs / imgs / \*.vhd_
8. Specify the account type - **Standard HDD**
9. Use default values for other fields
10. After entering the values, click the **Review + create** button, then the **Create** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_23.png" alt=""><figcaption></figcaption></figure>

### Creating a virtual machine

1. Open Menu / All services / Compute / **Virtual machines**
2. In the Virtual machines tab, select **Create / Azure virtual machine**
3. Specify the resource group _MikoPBX\_group_
4. Enter the virtual machine name, for example _MikoPBX-vm_

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_7.png" alt=""><figcaption></figcaption></figure>

5. Choose the previously created image, _See all images / Other items / My images / MikoPBX\_Azure_
6. Specify the machine size (combination of CPU / RAM to be at least 1GB / HDD parameters)

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_22.png" alt=""><figcaption></figcaption></figure>

7. Specify the username for the administrator account

If you have an SSH key, do the following

8. Select the source of the SSH public key - **Use existing public key**
9. Specify it in the SSH public key field

If you do not have an SSH key, do the following

8. Select the source of the SSH public key - **Generate new key pair**
9. Specify the key pair name, for example _mikopbx\_key_

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_9.png" alt=""><figcaption></figcaption></figure>

Continue with the following instructions:

10. In the license type field, specify **Other**
11. Use default values for the other fields

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_10.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
To deploy the PBX use **two** disks:

* A **1 Gb** disk for the main system
* A **50+ Gb** disk for storing call recordings
{% endhint %}

12. Go to the **Disks** tab
13. Specify the OS disk type
14. Check the **Delete with VM** checkbox

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_24.png" alt=""><figcaption></figcaption></figure>

15. Create a new data disk
16. Specify the disk size to be at least 50GB
17. Use default values for the other fields, click **OK**

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_12.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_25.png" alt=""><figcaption></figcaption></figure>

18. After entering the values, click the **Review + create** button, then click **Create**

### Configuring ports for incoming connections

1. Open the virtual machine you created and go to Networking / Network settings / Rules
2. In the tab, choose **Create inbound port rule**
3. Specify the destination port ranges - **0 - 65535**
4. Choose the protocol **TCP**

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_26.png" alt=""><figcaption></figcaption></figure>

5. Specify a name, for example _TCP_
6. Use default values for the other fields
7. After entering the values, click the **Add** button

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_27.png" alt=""><figcaption></figcaption></figure>

8. Similarly, create a rule for UDP. Specify the destination port ranges - **0 - 65535**, protocol **UDP**, and name

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_28.png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Make sure to configure the Firewall on the MikoPBX
{% endhint %}

### Starting MikoPBX

1. Open the virtual machine you created and go to the **Connect** section
2. In the drop-down menu under More ways to connect, select **Serial console**

<figure><img src="../../.gitbook/assets/MikoPBXAzureInstallation_eng_29.png" alt=""><figcaption></figcaption></figure>

3. Copy the external address of the created virtual machine and enter it in the browser's address bar
4. For login use the login and password provided in Serial console
