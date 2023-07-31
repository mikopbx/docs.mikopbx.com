# Let's Encrypt

Designed to simplify the process of obtaining an SSL certificate for HTTPS. System requirements:

1. The PBX must have a dedicated domain, for example, sip.test.ru.
2. The web interface of the PBX must be accessible on the internet (at the time of obtaining the SSL certificate).
3. The flag "This station is behind a **NAT router**" must be activated on the PBX.
4. The "**External Host Name of Your Router**" must be configured on the PBX. Refer to the [network interface settings.](../../manual/connectivity/network.md)
5. Disable the "**Redirect to HTTPS**" option in the general settings. The Lets Encrypt server verifies the resource by making an **HTTP** request to your PBX.

To obtain the certificate, follow these steps:

1. Go to the "**Modules**" -> **"Extensions and plugins"** tab.

Install and enable the "**Lets Encrypt Get SSL**" module.

<figure><img src="../../.gitbook/assets/1 (10).png" alt=""><figcaption></figcaption></figure>

2. Go to the module interface

<figure><img src="../../.gitbook/assets/2.png" alt=""><figcaption></figcaption></figure>

3. Perform the "**Get/update cert**" action:

<figure><img src="../../.gitbook/assets/3 (27).png" alt=""><figcaption></figcaption></figure>
