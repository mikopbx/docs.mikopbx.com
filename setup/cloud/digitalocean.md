---
description: Installing MikoPBX using the DigitalOcean Cloud Platform
---

# Digital Ocean

{% hint style="danger" %}
This guide applies to **MikoPBX version 2024.2.111** and newer!
{% endhint %}

{% embed url="https://youtu.be/2fL9pdPNcgE" %}

In this guide, we will perform a step-by-step installation of MikoPBX using the DigitalOcean cloud platform.

Before beginning, you need to copy the download link for the latest **.raw** MikoPBX image. You can find these on [MikoPBX's GitHub releases](https://github.com/mikopbx/core/releases).

## Uploading the Image to DigitalOcean

1. Go to "**Manage**" → "**Backups & Snapshots**":

<figure><img src="../../.gitbook/assets/backupsAndSnapshots.png" alt=""><figcaption><p>Section "Backups &#x26; Snapshots"</p></figcaption></figure>

2. Go to "**Custom Images**" → "**Import via URL**":

<figure><img src="../../.gitbook/assets/customImagesImportViaURL.png" alt=""><figcaption><p>"Import via URL"</p></figcaption></figure>

3. Paste the link to the **.raw** disk image file you copied earlier.
4.  Enter a name for the image, select the region where it will be uploaded (this should match the region of your future virtual machine), and choose "**Unknown**" as the operating system type.

    Click "**Upload image**".

<figure><img src="../../.gitbook/assets/imageParameters.png" alt=""><figcaption><p>Image parameters</p></figcaption></figure>

Wait for the image upload to complete.

## Creating a Virtual Machine in the Cloud

1. Go to DigitalOcean’s main page:

<figure><img src="../../.gitbook/assets/mainPageDO.png" alt=""><figcaption><p>DigitalOcean’s main page</p></figcaption></figure>

2. To create a new virtual machine (Droplet), go to "**Create**" → "**Droplets**":

<figure><img src="../../.gitbook/assets/buttonForCreatingANewDroplet.png" alt=""><figcaption><p>Creating a droplet</p></figcaption></figure>

3. Select a region and datacenter for your virtual machine:

<figure><img src="../../.gitbook/assets/regionAndDatacenter.png" alt=""><figcaption><p>VM Parameters #1</p></figcaption></figure>

4. Next, choose the previously uploaded image and configuration for your virtual machine:

<figure><img src="../../.gitbook/assets/ImageAndConfig.png" alt=""><figcaption><p>VM Parameters #2</p></figcaption></figure>

5. Go to the "**Additional Storage**" tab. Here, you can add a second disk that will be used for call recordings. To do this, click "**Add volume**" and specify the parameters for the new disk.

{% hint style="info" %}
We recommend a minimum size of **50GB** for the call recordings disk.
{% endhint %}

<figure><img src="../../.gitbook/assets/additionalStorage.png" alt=""><figcaption><p>"Additional Storage" section</p></figcaption></figure>

6. Go to "**Choose authentication method**." Here, you need to select "**SSH Key**" and add the key pair for SSH authentication. For more information on generating SSH keys, see:

* [Windows](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/powershell.md)
* [MacOS/Linux](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/terminal.md)

<figure><img src="../../.gitbook/assets/sshKey.png" alt=""><figcaption><p>Authentication Methods</p></figcaption></figure>

7. Click "**Create Droplet**."

## Connecting to the Console and First Login to the Web Interface

### Connecting via DigitalOcean Console

1. Go to the page of the newly created Droplet. Wait for it to start. Then connect via the built-in DigitalOcean console (shown in the screenshot).

<figure><img src="../../.gitbook/assets/console.png" alt=""><figcaption><p>Console in the Digital Ocean interface</p></figcaption></figure>

2. After the system boots, open the web interface using the external IP address shown in the console (**external**).

<figure><img src="../../.gitbook/assets/externalIPAddress.png" alt=""><figcaption><p>MikoPBX IP-adress</p></figcaption></figure>

3. Paste the machine’s IP address into your browser’s address bar. When you reach the MikoPBX login page, use the following credentials:

* **Username**: admin
* **Password**: The **Droplet ID**, which you can find in the browser’s address bar:

<figure><img src="../../.gitbook/assets/MachineID.png" alt=""><figcaption><p>Droplet ID</p></figcaption></figure>

### Connecting via SSH

1. To connect via SSH, follow [these instructions](https://chatgpt.com/faq/troubleshooting/connecting-to-a-pbx-using-ssh/). This example uses **PowerShell** (Windows).

{% hint style="warning" %}
The default login for SSH on a DigitalOcean VM is **do-user**.
{% endhint %}

2. Open PowerShell and run the following command:

```powershell
ssh -i C:\Users\<Username>\.ssh\id_ed25519 do-user@mikopbxipadress
```

{% hint style="success" %}
Adjust:

1. `C:\Users\<Username>\.ssh\id_ed25519` to the path of your local SSH key.
2. `do-user` if you changed the root user during VM creation.
3. `mikopbxipadress` to the IP address of your station (IPv4 in the VM management interface).&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/sshCode.jpg" alt=""><figcaption><p>Command for SSH connection</p></figcaption></figure>

After pressing **Enter**, you will be authenticated via SSH and arrive at the MikoPBX console menu.
