# Network stops working in MikoPBX

{% hint style="danger" %}
The issue applies to release 2024.1.114
{% endhint %}

If the **Update external IP address on every reboot** checkbox is enabled in MikoPBX settings, the system automatically requests the current External IP address.

Recently, the service changed the response format, and now it returns an HTML page instead of an IP address.

This causes the External IP address in the system to be updated incorrectly, breaks the network connection, and may cause problems with Marketplace registration and with getting a new license key/registering an existing key.

To resolve the issue, follow these steps:

1. Go to MikoPBX network settings: **Network and Firewall** / **Network interfaces**.
2. Disable the **Update external IP address on every reboot** checkbox.
3. Clear the **External IP address of your router** field of incorrect data and manually enter the correct IP address.
4. Save the changes and reboot the PBX.
