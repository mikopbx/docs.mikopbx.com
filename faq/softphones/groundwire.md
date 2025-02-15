---
description: >-
  Instructions for configuring and connecting the Groundwire softphone to
  MikoPBX
---

# Groundwire

**Groundwire** is a softphone that offers low power consumption by avoiding constant SIP session maintenance with the server. When an incoming call arrives, a PUSH notification is first sent to the smartphone, which then launches the SIP client and establishes a connection to the server.

## Softphone Setup

1. Download and open the **Groundwire** app on your smartphone.

<figure><img src="../../.gitbook/assets/image (48).png" alt="" width="295"><figcaption><p>Main menu</p></figcaption></figure>

2. Go to **Groundwire** settings by tapping the corresponding icon at the top of the screen.

<figure><img src="../../.gitbook/assets/startPageGroundWire.jpg" alt="" width="295"><figcaption><p>Settings button</p></figcaption></figure>

3. Go to the "**Accounts**" section:

<figure><img src="../../.gitbook/assets/accountsSection.jpg" alt="" width="295"><figcaption><p>"Account" section</p></figcaption></figure>

4. Add a new account by tapping the "+" icon at the top of the screen.
5. Select "**Generic SIP Account**."
6. Fill in the required data:
   * **Title** – An arbitrary account name.
   * **Username** – The employee’s extension number (e.g., 201).
   * **Password** – The SIP password from the employee’s configuration.
   * **Domain** – The IP address of your MikoPBX server.

<figure><img src="../../.gitbook/assets/dataForAuth.jpg" alt="" width="295"><figcaption><p>Data for SIP connection</p></figcaption></figure>

7. Go to "**Advanced Settings**."
8. In "**Audio Codecs**" → "**Codecs for Wi-Fi**," choose the desired codecs for use.

<figure><img src="../../.gitbook/assets/AudioCodecs.jpg" alt="" width="295"><figcaption><p>Audio codecs</p></figcaption></figure>

9. Navigate to "**Caller Id Method**." Select "**P-Asserted-Identity**."

<figure><img src="../../.gitbook/assets/image (49).png" alt="" width="295"><figcaption><p>Section "Caller Id Method"</p></figcaption></figure>

Save the settings. A connection attempt will occur. In the MikoPBX "Employees" section, you can see a status indicator confirming a successful connection:

<figure><img src="../../.gitbook/assets/successfulConnection.jpg" alt=""><figcaption><p>Successful SIP-connection</p></figcaption></figure>

## Configuring Employee Status Monitoring

Groundwire can display the current status of employees (see example below):

<figure><img src="../../.gitbook/assets/extensionsStatuses.jpg" alt="" width="295"><figcaption><p>Employee statuses</p></figcaption></figure>

To enable this feature, populate the "**Quick Dial**" section with employees. Tap "**EDIT**" at the top of the screen. Then tap the "**+**" at the bottom of the screen to add entries:

* **Title** – The employee’s name to display in "**Quick Dial**."
* **Number or SIP Address** – The employee’s extension number.

Tap "**Save**."

<figure><img src="../../.gitbook/assets/newQuickDialContact.jpg" alt="" width="295"><figcaption><p>New employee in the "Quick Dial" section</p></figcaption></figure>

## Configuring GroundWire with Encryption

1. In the MikoPBX web interface, open the employee’s SIP account settings. Change the "**Transport Protocol**" to "**tls**."

<figure><img src="../../.gitbook/assets/tlsMode (1).jpg" alt=""><figcaption><p>Changing the transport protocol</p></figcaption></figure>

2. In **Groundwire** → **Employee’s SIP Account Settings**, change the **"Domain"** field to **"MikoPBXAdress:NumOfTLSPort"**.

{% hint style="info" %}
You can find or change the TLS port in MikoPBX’s "**General Settings**" → "**SIP**."&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/tlsDomain.jpg" alt="" width="295"><figcaption><p>Domain for the TLS connection</p></figcaption></figure>

3. Go to "**Advanced Settings**" → "**Transport Protocol**" and change "**udp**" to "**tls (sip)**."

<figure><img src="../../.gitbook/assets/tlsSipss.jpg" alt="" width="295"><figcaption><p>Changing the transport protoco</p></figcaption></figure>

4. In the "**Advanced Settings**" of your SIP account, go to "**Secure Calls**" and, under the "**SDES**" section, set "**Incoming Calls**" and "**Outgoing Calls**" to "**Required**":

<figure><img src="../../.gitbook/assets/SDES.jpg" alt="" width="295"><figcaption><p>SDES Parameters</p></figcaption></figure>
