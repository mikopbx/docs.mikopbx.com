---
description: Steps to reset the WEB interface credentials from the MikoPBX console
---

# Resetting WEB Interface Credentials

You may encounter a situation where you have forgotten the username or password for the MikoPBX web interface. This guide explains how to reset them.

<figure><img src="../../.gitbook/assets/LoginError(MikoPBXEntryWEB).png" alt=""><figcaption><p>Authorization failed</p></figcaption></figure>

## Solution

1. Go to the MikoPBX console.

{% hint style="info" %}
The location of the console depends on the installation method:

* If installed on a physical server - on the monitor connected to the server.
* If installed  in a virtual machine - in the virtual machine management console.
* If installed  in the cloud - in the cloud serial console (also in the virtual machine management console).
* The reset method when installing in a Docker container is described in more detail in the current documentation.

In addition, the console can be accessed using SSH authorization. You can read more about SSH connection  [here](../troubleshooting/connecting-to-a-pbx-using-ssh/).

To learn more about MikoPBX, please refer to the [following documentation](../../master/getting-to-know-mikopbx.md).
{% endhint %}

<figure><img src="../../.gitbook/assets/mikopbxconsole.png" alt=""><figcaption><p>MikoPBX Console</p></figcaption></figure>

2. Select the option "**\[7] Reset password for the web interface**".
3. Type **y** to confirm resetting the login and password.

<figure><img src="../../.gitbook/assets/passwordReset.png" alt=""><figcaption><p>Password reset confirmation</p></figcaption></figure>

4. Log in to the web interface using the default credentials:

{% hint style="info" %}
**Default web interface credentials:**

* **Username**: admin
* **Password**: admin
{% endhint %}

After the first login, you will be prompted to change the credentials.

<figure><img src="../../.gitbook/assets/changeLoginAndPassword.png" alt=""><figcaption><p>Changing your login information</p></figcaption></figure>

## Resetting the Password in a Docker Container

1. Access the **container shell**:

```bash
docker exec -it mikopbxContainerNameOrID sh
```

{% hint style="success" %}
Replace _mikopbxContainerNameOrID_ with the name or ID of your container.
{% endhint %}

2. Launch the menu using this command:

```bash
/etc/rc/console_menu
```

3. Navigate to **"\[7] Reset password for the web interface"**.
4. Enter **"y"** to confirm resetting the username and password.
5. Log in to the web interface using the default credentials:

{% hint style="info" %}
**Default web interface credentials**:

* **Username**: admin
* **Password**: admin
{% endhint %}

Change the login credentials after the first authorization:

<figure><img src="../../.gitbook/assets/changeLoginAndPassword.png" alt=""><figcaption><p>Changing your login information</p></figcaption></figure>
