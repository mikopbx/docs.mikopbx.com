---
description: Connecting and configuring the Snom D120 phone
---

# Snom D120

**Snom D120** is an entry-level desk phone by **Snom**. This guide explains how to connect and configure the Snom D120 with MikoPBX.

{% hint style="info" %}
Verify that **G.711 A-law** is enabled under **System** → **General Settings** → **Audio/Video Codecs** in MikoPBX.
{% endhint %}

## MikoPBX Extensions Settings

1. Open the MikoPBX web interface and go to **Telephony** → **Extensions**:

<figure><img src="../../.gitbook/assets/extensionsSectionMikoPBX.png" alt=""><figcaption><p>"Extensions" section</p></figcaption></figure>

2. Next to the employee account you wish to connect, copy the SIP password:

<figure><img src="../../.gitbook/assets/copyPasswordForSIP.jpg" alt=""><figcaption><p>Copying password for SIP</p></figcaption></figure>

## Configuring the Snom D120

Connect the D120 to your local network using the LAN port.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Scheme of Snom D120</p></figcaption></figure>

{% hint style="info" %}
If DHCP is enabled in your network, the phone will automatically receive an IP address. The IP address will be shown on the phone’s screen at startup.
{% endhint %}

1. Enter the phone’s IP address into your browser as **http://phone\_ip\_address** and log into its web interface.
2. To configure the SIP account, go to **Setup** → **Identity 1**:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>"Identity 1" section</p></figcaption></figure>

* **Identity active** – set to “on”
* **Displayname** – any name (preferably in Latin characters)
* **Account** – the extension number of the MikoPBX employee
* **Password** – the SIP password you copied previously
* **Registrar** – your MikoPBX address

Click **Apply**, then **Re-Register**.

To verify, go to **Status** → **System Information**. If registration is successful, the phone will show something like:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Succesfull connection</p></figcaption></figure>

In the MikoPBX web interface the status indicator will turn green:

<figure><img src="../../.gitbook/assets/successfulRegistraion.jpg" alt=""><figcaption><p>Status Indicator in MikoPBX interface</p></figcaption></figure>

## Additional Settings

1. We recommend enabling **PnP Config** to use the [**Automatic Phone Setup**](../../modules/miko/module-autoprovision.md) module features.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Configuring "PnP Config"</p></figcaption></figure>

2. In some cases, if DHCP is used, **option 132** might be set for auto-provisioning terminals. You may need to disable this option for proper phone operation:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Option 132</p></figcaption></figure>
