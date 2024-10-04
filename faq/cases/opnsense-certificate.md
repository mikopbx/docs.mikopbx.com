# SSL Certificate for MikoPBX Web Interface from OPNSense

**OPNsense** is a FreeBSD-based operating system designed for creating firewalls and routers. It provides powerful network management tools, including VPN, traffic filtering, monitoring, and load balancing.

OPNSense includes a certification authority (CA) and can issue SSL certificates for the MikoPBX web interface.

First, ensure that the OPNSense certification authority is configured and that the root certificate is installed on the user's workstation.

### Creating a Certificate in OPNSense

1. **Generate a certificate:** Navigate to your OPNSense server and open the certificate manager:

<figure><img src="../../.gitbook/assets/sections.jpg" alt=""><figcaption><p>Certificate Manager</p></figcaption></figure>

2. **Issue an internal certificate:** Assign a clear **name**, choose **server** as the **certificate type**, specify your internal certification authority as the **issuer**, and set the certificate validity period in days.

<figure><img src="../../.gitbook/assets/Issue an internal certificate.jpg" alt=""><figcaption><p>Certificate Parameters</p></figcaption></figure>

3. **Specify DNS:** Enter the DNS configured for MikoPBX.

<figure><img src="../../.gitbook/assets/domainName.jpg" alt=""><figcaption><p>DNS Domain Names</p></figcaption></figure>

Save the certificate in OPNSense.

4. **Save the public and private keys:** Locate your PBX certificate in the list and click "Download."

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXf6_IguGhBqXvi4gvQlSYg8xPtMnBnXMFkvLChzkblpckXz7G9EoAnhc9hueKFCFH-GM6aRdTsrRkBPRqxh7Q9cQ_LJAaAsbTqOn4ObE0x-BnqOTUT32nFUfGqzY3She3B9HCebCu_X4tRr3aGZl01t_iYu_GN7TXZTFOOMuw?key=mKA6FU2fXdcsY4hVdKBAGA" alt=""><figcaption><p>Download Button</p></figcaption></figure>

5. **Download the public key (1) and private key (2).**

<figure><img src="../../.gitbook/assets/certificatesForDownload.png" alt=""><figcaption><p>Downloading Public and Private Keys</p></figcaption></figure>

### Installing the Certificate in MikoPBX

1. Navigate to **General Settings -> WEB-Interface**.

<figure><img src="../../.gitbook/assets/webInterfaceSection.png" alt=""><figcaption><p>WEB-interface section</p></figcaption></figure>

2. Open the previously downloaded files using a text editor. Paste the contents into the Web interface fields.

* **Public Key**: Enter the content starting with "BEGIN CERTIFICATE."
* **Private Key**: Enter the content starting with "BEGIN PRIVATE KEY."

3. **Save** the settings.
4. Open the MikoPBX web interface in your browser's incognito mode using an HTTPS connection. Your connection is now secure.
