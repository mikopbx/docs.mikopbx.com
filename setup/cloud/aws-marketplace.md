# AWS Marketplace

Sign in to the service Amazon Web Services [https://aws.amazon.com](https://aws.amazon.com/)

Let's get started with the setup

{% hint style="info" %}
For quick and convenient navigation within the Amazon service, use the search panel
{% endhint %}

### Creating a virtual machine <a href="#creating-a-virtual-machine" id="creating-a-virtual-machine"></a>

1. Open Services / Compute / **EC2** and navigate to Images / AMI Catalog
2. In the open tab enter **MikoPBX** in the search bar
3. In the AWS Marketplace AMIs section select the MikoPBX image by clicking the **Select** button
4. Click the **Launch an instance from AMI** button to create a virtual machine

<figure><img src="../../.gitbook/assets/MikoPBXAmazonInstallation_s_1.png" alt=""><figcaption></figcaption></figure>

5. Enter the virtual machine name, for example _mikopbx-vm_

<figure><img src="../../.gitbook/assets/MikoPBXAmazonInstallation_s_2.png" alt=""><figcaption></figcaption></figure>

If you have an SSH key

6. Specify the SSH key in the Key pair field

If you don't have an SSH key

6. Select **Create new key pair** and specify the key pair name, for example _mikopbx\_key_

<figure><img src="../../.gitbook/assets/MikoPBXAmazonInstallation_s_3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/MikoPBXAmazonInstallation_s_4.png" alt=""><figcaption></figcaption></figure>

Follow the instructions further

{% hint style="danger" %}
To deploy the PBX use **two** disks:

* A **1 Gb** disk for the main system
* A **50+ Gb** disk for storing call recordings
{% endhint %}

7. If necessary, change the size of the storage disk in Configure storage, default size is 50Gb

<figure><img src="https://docs.mikopbx.com/~gitbook/image?url=https:%2F%2F835495363-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FsZ8acWnNlSalIHQjMFu1%252Fuploads%252FcyfxOWyWKGZElAxGJZZN%252FMikoPBXAmazonInstallation_10.png%3Falt=media%26token=f1f66c5e-816e-4e89-9a4c-9dd2db32e6a0&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=49d6dfe1d2e1e20a1a76dadae6eb29355b62b3453f33e19dfe7eb44d5a77ec5a" alt=""><figcaption></figcaption></figure>

8. For other fields use default values
9. Click **Launch instance**

<figure><img src="https://docs.mikopbx.com/~gitbook/image?url=https:%2F%2F835495363-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FsZ8acWnNlSalIHQjMFu1%252Fuploads%252FoNKPr4S7jhjpbh2x8AJ4%252FMikoPBXAmazonInstallation_11.png%3Falt=media%26token=bff66ca5-dcf6-4993-952c-4be3c28d8bd8&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=73e39307d9c1c1cf3605822eb7b966b2c16e576896da76f2473f0becc3ed1346" alt=""><figcaption></figcaption></figure>

### Starting MikoPBX <a href="#starting-mikopbx" id="starting-mikopbx"></a>

1. Go to the created virtual machine _mikopbx-vm_
2. On the opened tab, select Connect / EC2 serial console, wait for the system to fully load until the authentication parameters are displayed

<figure><img src="https://docs.mikopbx.com/~gitbook/image?url=https:%2F%2F835495363-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FsZ8acWnNlSalIHQjMFu1%252Fuploads%252FbgfXCeyrsZyGL86xyM3H%252FMikoPBXAmazonInstallation_12.png%3Falt=media%26token=61a70166-bac5-48c2-9b98-7771ea71bef2&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=9eddddce7a5f7c2930f45ad832772eedd4a23a19304a2de50a51fc64149a5df2" alt=""><figcaption></figcaption></figure>

3. Copy the external address of the created virtual machine and enter it in the browser's address bar
4. Use the login and password provided in EC2 serial console for login

<figure><img src="https://docs.mikopbx.com/~gitbook/image?url=https:%2F%2F835495363-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FsZ8acWnNlSalIHQjMFu1%252Fuploads%252Fr3139qeBOQRGeq2q2bP3%252FMikoPBXAmazonInstallation_15.png%3Falt=media%26token=eeb9e2d7-af16-41db-af7a-1d61d4629d81&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=25713c16dbbe91a7326d156ffc9ed3fdf53715caf7f083ae7f85d8374ccf2b9a" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
Make sure to configure the Firewall on the MikoPBX
{% endhint %}
