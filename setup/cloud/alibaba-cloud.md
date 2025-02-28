---
description: Installing MikoPBX using the Alibaba Cloud platform
---

# Alibaba Cloud

{% hint style="danger" %}
This guide applies to **MikoPBX version 2024.2.135** and later!
{% endhint %}

This step-by-step guide will walk you through installing MikoPBX on the Alibaba Cloud platform.

Before starting, download the latest **.raw** MikoPBX image from [MikoPBX’s GitHub releases](https://github.com/mikopbx/core/releases).

## Uploading the Image to Alibaba Cloud

### Creating a Bucket

First, create a bucket for storing your image. Go to the **OSS Management Console** ([link](https://oss.console.aliyun.com/overview)).

1. Go to **Buckets**.

<figure><img src="../../.gitbook/assets/bucketsSection.jpg" alt=""><figcaption><p>Buckets section</p></figcaption></figure>

2. Click **Create Bucket**:

<figure><img src="../../.gitbook/assets/createBucketButton.jpg" alt=""><figcaption><p>"Create Bucket" button</p></figcaption></figure>

3. Specify the following:

* **Bucket name** – a custom name for your storage.
* **Region** – pick the region where your image will be stored.

{% hint style="danger" %}
The bucket region for your image must match the region of your future virtual machine!
{% endhint %}

Click **OK**.

<figure><img src="../../.gitbook/assets/bucketParameters.jpg" alt=""><figcaption><p>Bucket parameters</p></figcaption></figure>

4. Go to your newly created bucket by clicking its name in the **Buckets** section:

<figure><img src="../../.gitbook/assets/bucketName.jpg" alt=""><figcaption><p>Bucket name</p></figcaption></figure>

5. Click **Upload object** and upload the previously downloaded **.raw** disk image file (leave other parameters at default).

<figure><img src="../../.gitbook/assets/uploadObject.jpg" alt=""><figcaption><p>"Upload Object" button</p></figcaption></figure>

6. Once the disk image file is uploaded, copy its link. Click **View Details** to the right of the file name; in the opened menu, copy the **URL** field.

<figure><img src="../../.gitbook/assets/URL.jpg" alt=""><figcaption><p>URL of the uploaded file</p></figcaption></figure>

### Creating the Image

1. Return to the **ECS Console** ([link](https://ecs.console.aliyun.com/home)) and go to **Images**.

<figure><img src="../../.gitbook/assets/imagesSection.jpg" alt=""><figcaption><p>"Images" section</p></figcaption></figure>

2. Click **Import Image**:

<figure><img src="../../.gitbook/assets/importImageButton.jpg" alt=""><figcaption><p>"Import Image" Button</p></figcaption></figure>

3. Select **Linux Operating System** and click **Next**.
4. Enter/select the following image parameters:
   * **Image File URL** – Paste the link to the disk image file you uploaded.
   * **Image Name** – A custom, **unique** name for your image.
   * **OS Type** – Linux
   * **OS Version** – Others Linux
   * **Architecture** – 64-bit OS
   * Uncheck **"Check After Import"**

Click **OK** to create the image. Wait until the process finishes (the Status will show <mark style="color:green;">**Available**</mark>).

<figure><img src="../../.gitbook/assets/imageParametersMikoPBX.jpg" alt=""><figcaption><p>Image Parameters</p></figcaption></figure>

## Creating an SSH Key Pair

Next, create and add an SSH key pair in Alibaba Cloud.

1. In the ECS Console, go to **Network Security** → **Key Pairs**:

<figure><img src="../../.gitbook/assets/keyPairsSection.jpg" alt=""><figcaption><p>"Key Pairs" section</p></figcaption></figure>

2. Click **Create SSH Key Pair**:

<figure><img src="../../.gitbook/assets/createSSHKeyPair.jpg" alt=""><figcaption><p>"Create SSH Key Pair" button</p></figcaption></figure>

3. Generate an SSH key pair. For details on how to generate a key pair, see [this guide](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/). Fill in the required parameters:

* **Name** – A custom name for your key pair.
* **Creation Mode** – Import
* **Public Key** – Paste your **public** key, generated earlier.
* **Resource Group** – Choose your resource group.

Click **OK** to create the key pair in the cloud.

<figure><img src="../../.gitbook/assets/sshKeyPairParameters.jpg" alt=""><figcaption><p>SSH Key Pair Parameters</p></figcaption></figure>

## Creating a Security Group

Before creating the virtual machine, you must set up a security group (firewall).

1. Go to **Network & Security** → **Security Groups**:

<figure><img src="../../.gitbook/assets/securityGroupsSection.jpg" alt=""><figcaption><p>"Security Groups" Section</p></figcaption></figure>

2. Click **Create Security Group**:

<figure><img src="../../.gitbook/assets/createSecurityGroup.jpg" alt=""><figcaption><p>"Create Security Group" button</p></figcaption></figure>

3. Specify the following security group parameters:

* **Security Group** – A custom name for your security group.
* **Network** – Your selected network. If it doesn’t exist yet, click **"Create VPC"** to the right.
* **Security Group** – Basic Security Group.
* **Resource Group** – Your resource group.
* Allow all inbound connections (see example below). Outbound is allowed by default.

{% hint style="info" %}
Be sure to configure the firewall within MikoPBX itself as soon as possible after creating the virtual machine. Read more about that [here](../../manual/connectivity/firewall.md).
{% endhint %}

Click **Create Security Group**.

<figure><img src="../../.gitbook/assets/ParametersOfTheSecurity.jpg" alt=""><figcaption><p>Security Group parameters</p></figcaption></figure>

## Creating the Virtual Machine

1. Go to **Instances & Images** → **Instances**:
2. Click **Create Instance** to create a new virtual machine:
3. Select your VM parameters:
   * **Billing Method** – Choose how you’ll pay for the VM.
   * **Region**, **Network, and Zone** – Select the region and zone to match your needs.
   * **Instance** – Pick a configuration for your VM.
4.  Configure additional VM parameters:

    * **Image** – **Custom Images** → Choose the previously imported image.
    * **Storage** – Select the type and size of the **System Disk** (20 GB is the minimum for Alibaba Cloud).
    * **Add a second disk** by clicking **Add Data Disk**, specifying its type and size.

    \{% hint style="info" %\} We recommend a minimum of **50GB** for call recordings. This guide uses 30GB as an example. \{% endhint %\}
5. Choose the network parameters for your VM. The security group created earlier will be assigned automatically:
6. Click **Create Order**.

### Connecting to the MikoPBX Console

In the **Instances** section, open the newly created VM by clicking its name.

#### Connecting via Built-in Cloud Console

1. Click **Connect**.
2. Select **VNC**. A new tab will open in your browser with console access.

#### Connecting via SSH

\{% hint style="info" %\} For more information on SSH connections, refer to [this set of articles](https://chatgpt.com/faq/troubleshooting/connecting-to-a-pbx-using-ssh/). In this guide, we demonstrate SSH access via PowerShell. \{% endhint %\}

Enter the following command to connect via SSH:

```powershell
ssh -i C:\Users\username\.ssh\id_ed25519 root@ip-adress
```

Replace:

* `C:\Users\username\.ssh\id_ed25519` with the path to your SSH key,
* `root` if you changed the default user when creating the VM,
* `ip-adress` with the external IP address of your MikoPBX instance.

You will then connect via SSH:

### First Login to the Web Interface

On the VM’s main page, you’ll see important parameters for logging into the MikoPBX web interface.

Paste the IP address into your browser’s address bar to access the MikoPBX web interface login page.

\{% hint style="info" %\} **Login credentials**:

* **Username**: admin
* **Password**: Your Virtual Machine’s ID \{% endhint %\}
