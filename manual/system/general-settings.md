---
description: Description of the main system settings
---

# General settings

This section configures the core system parameters. It is recommended to complete these settings immediately after installing the PBX.

<figure><img src="../../.gitbook/assets/GeneralSettingsTabMikoPBX.png" alt=""><figcaption><p>"General Settings" section in the MikoPBX web interface</p></figcaption></figure>

### Main

* **PBX system name** - displayed on the MikoPBX main page.
* **Additional description** - visible to system administrators only.
* **Language of system audio messages** - language used for voice announcements.
* **Maximum length of internal numbers** - the maximum length of an employee's internal extension number.
* **Allow incoming calls from any servers** - allows accepting SIP calls from unauthorized devices and servers without registration.

{% hint style="info" %}
Enabling this option may pose a security risk. Make sure your network is properly protected and filtering rules are in place!
{% endhint %}

* **Restart PBX every night** — automatic restart of Asterisk at night (at 01:00 AM system time).
* **Send crash information to developers** — when an error occurs, its description is sent to developers (requires internet access).

Click **"Save"**.

<figure><img src="../../.gitbook/assets/MainTabGenSet.png" alt=""><figcaption><p>"General" tab in system settings</p></figcaption></figure>

### Call Recording

* **Call Recording** - enable or disable recording of all calls.
* **Recording** **internal conversations**- enable or disable recording of calls between employees.

Below, you can select audio files to be used as a recording notification (different audio files can be selected for incoming and outgoing calls).

<figure><img src="../../.gitbook/assets/CallRecordingTabGenSet.png" alt=""><figcaption><p>"Call Recording" tab in system settings</p></figcaption></figure>

Phone calls are saved in **WebM** format with the **Opus** codec. File size depends on call quality: if at least one participant uses a high-quality codec (e.g., G.722 or Opus), the recording is saved at a higher bitrate — this takes more disk space but improves speech recognition quality.

{% hint style="info" %}
Approximately **1 hour** of conversation takes **14–28 MB** of disk space depending on recording quality.
{% endhint %}

### Call Transfers

#### **Parking (Hold)**

**Call Parking** is a way to temporarily place a customer on hold while you look up information. The caller hears music while waiting.

MikoPBX supports two parking methods:

1. Dial **\*2** during a call — the call will be placed on hold and you will be told the parking slot number. Any employee can pick up the call by dialing that number.
2. In the settings, configure a **parking number** — when a call is transferred to this number, MikoPBX will place it on hold and announce the slot number. Any employee can retrieve the call.

The parking slot range and parking number can be configured in this section:

* **Call parking number** — the number to transfer a call to in order to place it on hold, default is **800**.
* **Parking slot range** — the range of parking slot numbers, default is **801–820**.

#### **Call Transfers**

MikoPBX supports two types of transfers:

* **Attended (Consultative) Transfer** — you can speak with a colleague before transferring the call to them. The caller is on hold during this time. The transfer completes when you hang up.
* **Blind (Unattended) Transfer** — the call is transferred immediately, without a prior conversation with the colleague. Useful when a second call comes in while you are already busy — the call can be instantly transferred to a free employee.

The key combinations for transfers can be changed in this section:

* **Combination for attended transfer** — default is **##**.
* **Combination for blind transfer** — default is **\*\***.

{% hint style="info" %}
Combinations are entered from the phone during an active call, followed by the internal extension number of the employee to transfer to.&#x20;
{% endhint %}

#### **Timeouts**

* **Call return time if no answer after attended transfer** — if no one answers after an attended transfer, the call returns. Set in seconds, default is **45 sec**.
* **Maximum timeout between digits when entering an extension number (in milliseconds)** — the wait time for the next digit when dialing an extension. Set in milliseconds, default is **2500 ms**.

#### **Call Pickup**

If a colleague's phone is ringing, you can pick up the call without leaving your desk:

* **\*8\<ColleagueNumber>** — pick up a specific employee's call.
* **\*8** — pick up any incoming call when the colleague's number is unknown.

The pickup combination can be changed in the **"Combination for intercepting incoming calls"** field, default is **\*8**.

<figure><img src="../../.gitbook/assets/CallTransfersGenSet.png" alt=""><figcaption><p>"Call Transfers" tab in system settings</p></figcaption></figure>

### SIP&#x20;

**Session Initiation Protocol (SIP)** is the signaling protocol used by most VoIP phones. You can change the SIP port (default **5060**) to improve security.

#### **SIP Signaling Port and RTP Range Settings**

**RTP (Real-time Transport Protocol)** defines the standard format for transmitting audio and video over IP networks. The default port range is **10000–10800**. Some routers and firewalls may require additional range configuration. Another reason to expand the range is a large number of concurrent calls: each active call uses two RTP ports, meaning 200 ports support no more than 100 simultaneous calls. If load is higher — expand the range.

* **SIP port for registering phones on this station** — the port for phone registration on the station, default **5060**. Changing the port can improve system security.
* **SIP TLS port (encrypted calls)** — the port for encrypted calls, default **5061**.
* **RTP port range** — the port range for audio transmission, default **10000–10800**.

#### **Additional Parameters**

* **STUN server address** — helps when the PBX is behind NAT, including when using WebRTC.
* **Auth Username prefix for authorization** — by default, the username for SIP account authorization matches the employee's internal extension (e.g., `101`). When this setting is filled in, the specified prefix will be appended to the auth username: `username` remains `101`, but `AuthUsername` becomes `101MIKO`. This approach significantly complicates password brute-forcing for SIP accounts.
* **Use WebRTC** — additional settings will be applied for WebRTC connections. For example, for internal extension 201, an additional endpoint will be created, accessible via WebRTC using the URL `sip:201-WS@IP_PBX`.

#### **Registration Duration Settings**

Some firewalls close ports after a period of inactivity — in such cases, it is advisable to reduce the registration timeout. Different SIP providers may also require different timeout values.

* **Default time in seconds to send Keep-alive** — the interval for sending keep-alive packets in seconds, default **120 seconds**.
* **Minimum Registration Time (SIPMiniExpiry)** — default **60 seconds**.
* **Maximum Registration Time (SIPMaxExpiry)** — default **3600 seconds**.

<figure><img src="../../.gitbook/assets/SIPTabGenSet.png" alt=""><figcaption><p>"SIP" tab in system settings</p></figcaption></figure>

### Audio/Video Codecs&#x20;

This section configures the allowed audio and video codecs for the entire PBX.

<figure><img src="../../.gitbook/assets/AVCodecsGenSet.png" alt=""><figcaption><p>"Audio/Video Codecs" section in system settings</p></figcaption></figure>

### AMI\&ARI

**Asterisk Manager Interface (AMI)** is a powerful and convenient software interface (API) for Asterisk that allows external programs to manage the system. Through AMI, external programs can connect to Asterisk via TCP, initiate command execution, read results, and receive real-time event notifications. AMI is often used for integration with business processes and CRM (Customer Relationship Management) systems.

**Asynchronous Javascript Asterisk Manager (AJAM)** is a technology that allows web browsers or other HTTP-capable applications to directly access the Asterisk Manager Interface (AMI) via HTTP/HTTPS.

**Asterisk REST Interface (ARI)** is a RESTful API with WebSocket support that provides full control over Asterisk channels, bridges, and media streams in real time. Designed for developing custom telephony applications.

#### **AMI Settings**

* **Use AMI Interface** — enable or disable AMI.
* **AMI Port** — the port for connecting external programs to AMI, default **5038**. A client application connects to AMI through this port and authenticates, after which Asterisk responds to requests and sends notifications about state changes in specified subsystems.

#### **HTTP Server Settings**

* **HTTP Port (AJAM and ARI)** — the port for HTTP connections, default **8088**.
* **HTTPS Port (AJAM and ARI)** — the port for HTTPS connections, default **8089**.

#### **AJAM Settings**

* **Use AJAM Interface** — enable or disable AJAM.

#### **ARI Settings**

* **Use ARI Interface** — enable or disable ARI. Disabled by default.
* **CORS allowed origins** — domains from which requests to ARI are permitted. CORS is a browser security mechanism that restricts cross-domain API requests.

{% hint style="danger" %}
Never use `*` in production. Only specify trusted domains over HTTPS.
{% endhint %}

<figure><img src="../../.gitbook/assets/AMI&#x26;ARIGenSet.png" alt=""><figcaption><p>"AMI&#x26;ARI" tab in system settings</p></figcaption></figure>

### SSH

**SSH (Secure Shell)** is an encrypted protocol commonly used for interacting with and remotely managing servers. An SSH server can authenticate users using various algorithms. The most popular is password authentication. It is fairly simple but not very secure: passwords are transmitted over a secure channel, but are not complex enough to withstand brute-force attempts. The computational power of modern systems combined with specialized scripts makes brute-forcing very easy.

A more secure authentication method is **SSH keys**. Each pair consists of a public and private key: the private key is stored on the client, and the public key is uploaded to the server in the `~/.ssh/authorized_keys` file. When connecting, the server sends a message encrypted with the public key — if the client decrypts it with the private key and returns the correct response, authentication is considered successful.

{% hint style="info" %}
In MikoPBX, password authentication is **disabled** by default — SSH keys must be used to connect. A key can be added in this section or when creating a virtual machine in the cloud (it will be automatically applied during MikoPBX installation).

You can read more about connecting to MikoPBX via SSH [here](../../faq/troubleshooting/connecting-to-a-pbx-using-ssh/).&#x20;
{% endhint %}

#### **Section Parameters**

* **SSH port** — the port for SSH connections, default **22**.
* **SSH console login** — the username for connecting.
* **Disable password authentication** — **enabled** by default in MikoPBX (password authentication is disabled).
* **SSH password** — the login password (available only if password authentication is **not disabled**).
* **Authorized SSH Keys** — add your public SSH key here using the **"+ Add Key"** button. If you have multiple keys, add each one separately.
* **System Public SSH Key** — the public SSH key of the current PBX. It can be copied into the **"Authorized SSH Keys"** field on another station — this allows connecting to the remote server without additional authentication.

<figure><img src="../../.gitbook/assets/SSHTabGenSet.png" alt=""><figcaption><p>"SSH" tab in system settings</p></figcaption></figure>

### HTTP/HTTPS&#x20;

To improve security, you can change the HTTP port (default **80**) or enable HTTPS mode. **HTTPS** encrypts traffic between the browser and the PBX using SSL/TLS protocols. The default TCP port is **443**.

* **HTTP port** — the port for accessing the web interface via HTTP, default **80**.
* **HTTPS port** — the port for accessing the web interface via HTTPS, default **443**.
* **Redirect to HTTPS** — when the web interface is opened via HTTP, the user will be automatically redirected to HTTPS.

#### **HTTPS Public Key (SSL/TLS Certificate)**

An SSL/TLS certificate is a digital document that verifies the server's identity and ensures encrypted communication between the browser and the PBX. In MikoPBX, the certificate is used for:

* HTTPS access to the web interface
* WebRTC connections (required for browser-based calls)
* Secure AJAM and ARI connections via HTTPS
* Secured REST API for integrations

The certificate must be in **PEM** format — beginning with `-----BEGIN CERTIFICATE-----` and ending with `-----END CERTIFICATE-----`. If you have intermediate certificates, add them after the main certificate in the same field.

Ways to obtain a certificate:

* **Let's Encrypt Module** — automatic issuance and renewal of free certificates. Recommended method.
* **Purchase from a Certificate Authority** (DigiCert, Comodo, GlobalSign, etc.)
* **Self-signed Certificate** — automatically generated on first PBX startup, but causes browser warnings.

{% hint style="info" %}
We recommend using the Let's Encrypt module for automatic certificate management. Learn more [in this article](../../modules/miko/module-get-ssl-lets-encrypt.md).
{% endhint %}

#### **HTTPS private key**

The secret key used to decrypt SSL/TLS connections. It must exactly match the public certificate — if they do not match, HTTPS will not work.

The key must be in **PEM** format — beginning with `-----BEGIN RSA PRIVATE KEY-----` or `-----BEGIN PRIVATE KEY-----`.

{% hint style="info" %}
Never share your private key with third parties. If the key is compromised, an attacker will be able to intercept encrypted traffic. In case of compromise — replace the key pair immediately.&#x20;
{% endhint %}

Security recommendations:

* Keep a backup copy of the key in a secure location.
* Use keys of at least **2048 bits** in length (**4096** recommended).
* Regularly renew certificates and keys.

<figure><img src="../../.gitbook/assets/HTTP-HTTPSGenSet.png" alt=""><figcaption><p>"HTTP/HTTPS" tab in system settings</p></figcaption></figure>

### WEB interface password

In this section, you can change the login and password for accessing the web interface, and configure login via Passkeys.

{% hint style="info" %}
Default MikoPBX credentials:

* Login: **admin**
* Password: **admin** — it is recommended to change this immediately.
{% endhint %}

* **Login** — the username for logging into the web interface.
* **Password** — the password for logging into the web interface.

#### **Passkeys (Biometric Authentication)**

**Passkeys** are a modern passwordless login method using biometrics or a hardware security key: Face ID, Touch ID, Windows Hello, or YubiKey. This is faster and more secure than traditional passwords.

To add a Passkey, click the **"+ Add Passkey"** button and follow the browser instructions.

{% hint style="info" %}
You can read more about this [here](../../faq/management/passkeys.md).&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/WEBInterfacePwdGenSet.png" alt=""><figcaption><p>"WEB Interface Password" tab in system settings</p></figcaption></figure>

### System settings deletion

This section allows you to fully reset the system to its factory state. The reset will permanently delete all settings, call history, call recording files, and installed extension modules.

{% hint style="info" %}
This action is irreversible. Before clearing the system, make sure you have a backup of all important data.
{% endhint %}

To confirm, type **delete everything** in the input field and click **"Save"**.

<figure><img src="../../.gitbook/assets/SystemSettingsDeletionGenSet.png" alt=""><figcaption><p>"System Settings Reset" section in system settings</p></figcaption></figure>
