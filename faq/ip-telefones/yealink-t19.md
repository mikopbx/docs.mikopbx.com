---
description: Connecting and configuring the Yealink T19 phone
---

# Yealink T19

The **Yealink SIP-T19** is a budget model geared toward small and medium businesses. This guide demonstrates how to connect and configure the phone with MikoPBX.

{% hint style="info" %}
Make sure that **G.711 A-law** is enabled in **System** → **General Settings** → **Audio/Video Codecs** in MikoPBX.
{% endhint %}

## Account Settings in the “Extensions” Section

1. In the MikoPBX web interface, go to **Telephony** → **Extensions**:

<figure><img src="../../.gitbook/assets/extensionsSectionMikoPBX.png" alt=""><figcaption><p>"Extensions" section MikoPBX</p></figcaption></figure>

2. For the employee account you want to connect, copy the SIP password:

<figure><img src="../../.gitbook/assets/copyPasswordForSIP.jpg" alt=""><figcaption><p>Copying password for SIP</p></figcaption></figure>

## Configuring the Yealink T19

Connect the phone to the Ethernet network using the port labeled **Internet.**

{% hint style="info" %}
If your network has a DHCP server, the phone will automatically obtain an IP address.
{% endhint %}

Press the **OK** button on the phone to display its IP address on the screen. Then, in your web browser, enter <mark style="color:blue;">**http://phone\_ip\_address**</mark> to access the phone’s web interface.

{% hint style="success" %}
**Default credentials**:

* **Username**: admin
* **Password**: admin
{% endhint %}

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption><p>Authorization window</p></figcaption></figure>

Now configure the phone’s SIP account parameters:

* **Register Name**, **User Name** – the employee’s extension number
* **Password** – the SIP password you copied earlier
* **Server Host** – the IP address of your MikoPBX server

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption><p>SIP account parameters</p></figcaption></figure>

In **Features** → **Intercom** → **Intercom Barge**, set the relevant parameter:

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption><p>Intercom Barge parameter</p></figcaption></figure>

After successfully registering the phone, a green status indicator will appear next to the employee in MikoPBX:

<figure><img src="../../.gitbook/assets/successfulRegistraion.jpg" alt=""><figcaption><p>Successfull registration</p></figcaption></figure>
