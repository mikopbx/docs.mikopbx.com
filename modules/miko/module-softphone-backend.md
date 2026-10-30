---
description: >-
  The module provides the API required to connect the MikoTalk softphone to
  MikoPBX: push notifications, call history, contacts and supervisor tools.
---

# MikoTalk Softphone

The **MikoTalk Push Softphone API** module turns an employee's iPhone into a full-featured PBX extension: calls over Wi-Fi and mobile data, incoming calls delivered via push notifications — just like a regular call, even when the app is closed.

Setting up the softphone is as simple as scanning a QR code or opening an auto-configuration link.

## Features <a href="#features" id="features"></a>

| Feature             | Description                                                                                                                                                                                               |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Push calls          | Incoming call alerts are delivered even when the app is closed or the device is asleep                                                                                                                    |
| Call history        | MikoTalk users see their full call history, subject to access permissions ([instructions on configuring permissions](module-softphone-backend.md#access)). Call recordings can be played back right from the history                 |
| Contacts            | A caller's name can be resolved from carrier data (if the carrier provides it) or from 1C data — connect the ["Phone Panel 4.0"](https://docs.telefon1c.ru/v4/get-started/mikopbx/) module for the latter |
| Employee statuses   | Who is available and who is on a call. Call queues are visible to everyone                                                                                                                                |
| Transfer and pickup | Blind and attended transfer, picking up a colleague's incoming call                                                                                                                                       |
| Supervisor tools    | Listen, whisper, barge-in on a call, group monitoring                                                                                                                                                     |
| Missed calls        | Unanswered calls are marked with a dedicated counter in the app                                                                                                                                           |

## Installation <a href="#install" id="install"></a>

Connecting the softphone takes four steps: prepare the PBX, issue an SSL certificate, install the module and connect the employee's iPhone.

### Step 1. Install MikoPBX <a href="#install-pbx" id="install-pbx"></a>

Install MikoPBX **2026.1.223** or later — on a standalone computer, in a virtual machine or in the cloud. See [Quick start](../../master/quick-start.md) for the procedure. If the PBX is already running, check its version and update if necessary.

For employees to call over mobile data, the PBX must be reachable from the internet: a public static IP address or port forwarding (1:1 NAT). The port list and built-in firewall setup are described in [System requirements](module-softphone-backend.md#system-requirements).

### Step 2. Obtain an SSL certificate <a href="#install-ssl" id="install-ssl"></a>

The app connects to the PBX over a secure connection only (HTTPS/WSS), so a trusted certificate is required — a self-signed one will not work.

1. Install the ["Let's Encrypt SSL certificate generator"](module-get-ssl-lets-encrypt.md) module from the marketplace.
2. In the module settings, enter the PBX's public IP address — a domain name is not required, the certificate can be issued for an IP address.
3. Select the **HTTP-01** verification method: during verification the PBX must be reachable from the internet on port **80**, and **"Redirect to HTTPS"** must be disabled in **"System" → "General Settings" → "HTTP/HTTPS"**.
4. Enable **"Renew certificate automatically"** and click **"Get/Renew SSL Certificate"**.

{% hint style="warning" %}
A Let's Encrypt certificate for an IP address is short-lived: it is valid for only a few days. The module automatically tries to renew it every 5th day if **"Renew certificate automatically"** is enabled — do not disable it: once the certificate expires, the app can no longer connect to the PBX. See the [Let's Encrypt module instructions](module-get-ssl-lets-encrypt.md) for module setup and checking the issued certificate.
{% endhint %}

### Step 3. Install the MikoTalk module <a href="#install-module" id="install-module"></a>

1. In **"Modules" → "Modules Marketplace"** ([more about the marketplace](../../manual/modules/pbx-extension-modules.md#marketplace)), install the **MikoTalk Push Softphone API** module (formerly "WebRTC Softphone Backend").
2. Enable the module on the **"Installed Modules"** tab. A trial license is activated automatically on first installation — see [Requirements and licensing](module-softphone-backend.md#requirements).
3. Open the module settings: **"Modules" → "MikoTalk Push Softphone API"**.
4. Wait for the [status panel](module-softphone-backend.md#health) indicators to turn green — this usually takes up to a minute after enabling the module:
   * **nginx** — the app API is published;
   * **push-server** — push notification delivery is working;
   * **STUN** — the built-in coturn is running and the PBX's public address is determined;
   * **Built-in status service** — employee statuses and active calls;
   * **SSL certificate** — the certificate is trusted and issued for the address in the [external server](module-softphone-backend.md#external-server) settings.

{% hint style="info" %}
If an indicator stays red or yellow, hover over it to see the reason. The most common causes are a missing or undetected public PBX address or a self-signed certificate: check the **Host** and **Connection** fields in the [external server settings](module-softphone-backend.md#external-server).
{% endhint %}

### Step 4. Connect the iPhone softphone <a href="#ios" id="ios"></a>

1. Install the [MikoTalk app from the App Store](https://apps.apple.com/si/app/mikotalk/id6759347414) on the employee's iPhone.
2. In the module settings, **"Device setup"** tab, create an [auto-configuration link](module-softphone-backend.md#provision) for the employee's extension.
3. Scan the QR code with the iPhone camera or open the email with the link on the iPhone, then confirm sign-in in the app.
4. Allow notifications — without them, incoming calls will not be delivered.

After connecting, place a test call to the employee's extension with the app closed: it should arrive as a regular incoming call.

## Requirements and licensing <a href="#requirements" id="requirements"></a>

* MikoPBX **2026.1.223** or later.
* Licenses are issued **per employee**: all devices of the same employee consume a single license.

{% hint style="info" %}
The module is installed from the marketplace. A **trial license** is activated automatically on first installation, so you can try the softphone before purchasing. See [Registration in the modules marketplace](../../manual/modules/licensing.md) for details on registering and renewing licenses.
{% endhint %}

### System requirements <a href="#system-requirements" id="system-requirements"></a>

For calls over mobile data, the PBX must be reachable from the internet: a **public static IP address** or port forwarding from an external IP/domain to the PBX (1:1 NAT). The following ports must be open and forwarded:

| Port                                         | Protocol    | Purpose                                                                                                                                                               |
| -------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| External API port (default **8988**)         | TCP         | App connectivity: authentication, auto-configuration link, history, statuses. The port is configured in the module settings — see [External server](module-softphone-backend.md#external-server) |
| **3478** (STUN/TURN, built-in coturn)        | UDP and TCP | NAT traversal: helps the app discover its public address and build a media route to the PBX                                                                           |
| PBX RTP port range (default **10000–12000**) | UDP         | Call audio traffic. The range is configured in the MikoPBX network settings                                                                                           |

Additionally, in the PBX's own firewall settings (**"System" → "Network & Firewall" → "Firewall"**), enable the **Softphone RTP** service for the **"Internet"** profile — or at least for **"Trusted networks"** if employees only place calls from the office network. Otherwise, audio traffic is blocked by the built-in firewall.

<figure><img src="../../.gitbook/assets/softphone-firewall.png" alt=""><figcaption><p>The Softphone RTP service in the PBX firewall settings</p></figcaption></figure>

For secure connections (HTTPS/WSS), use the [Let's Encrypt](module-get-ssl-lets-encrypt.md) module — it issues a trusted SSL certificate. A certificate can also be obtained for an IP address; a domain name is not required.

## Connecting clients <a href="#clients" id="clients"></a>

### Auto-configuration link <a href="#provision" id="provision"></a>

The app is connected via an auto-configuration link. Open the module settings, **"Device setup"** tab:

1. Select the employee's extension number.
2. Set the link lifetime: 10 minutes, 1 hour, 5 hours or 1 day.
3. Optionally enable **"One-time link"** — the link stops working after the first use.
4. Click **"Create"** — the module shows a QR code and the link itself.

The link can be shared by any convenient means; the QR code can be scanned with the iPhone camera. The app downloads the settings and asks to confirm sign-in.

<figure><img src="../../.gitbook/assets/softphone-provision.png" alt=""><figcaption><p>The "Device setup" tab</p></figcaption></figure>

The same tab contains the employee table: search by name or number, emailing a QR code to one employee or to selected ones (**"Send to selected"**, up to 100 at a time). The email goes to the address specified in the employee's card.

{% hint style="info" %}
To email connection parameters, first configure the PBX mail server: **"System" → "Mail and Notifications"**. See the [Mail and Notifications](../../manual/system/mail-settings/README.md) section for instructions.
{% endhint %}

### Employee sessions <a href="#sessions" id="sessions"></a>

The module settings show each employee's active sessions and last activity time. The **"Terminate all employee sessions"** button forcibly signs out all of their devices and releases the license — useful, for example, when a phone is lost.

## Module settings <a href="#settings" id="settings"></a>

<figure><img src="../../.gitbook/assets/softphone-settings.png" alt=""><figcaption><p>The "Settings" tab</p></figcaption></figure>

### External server <a href="#external-server" id="external-server"></a>

The parameters the app uses to reach the PBX from the internet:

| Field          | Purpose                                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Host**       | Domain name or public IP address of the PBX. Determined automatically by default                                                                  |
| **Port**       | External web interface port, if the PBX is not published on the standard port                                                                     |
| **Connection** | Plain or secure (HTTPS). Values are used for QR code generation immediately, even before saving                                                   |
| **URI**        | The API address with a secret URL prefix. The refresh button generates a new prefix — all previously issued auto-configuration links stop working |

{% hint style="warning" %}
After changing the URL prefix or external server parameters, employees need to reconnect using a new auto-configuration link.
{% endhint %}

### Data sources <a href="#data-sources" id="data-sources"></a>

Call history and the user list always come from this PBX. The only external source is contacts:

| Source       | Options                                                                                              |
| ------------ | ---------------------------------------------------------------------------------------------------- |
| **Contacts** | The ["Phone Panel 4.0 for 1C"](https://docs.telefon1c.ru/v4/get-started/mikopbx/) module or this PBX |

The contacts source is available only if "Phone Panel 4.0" is installed and enabled. An availability indicator is shown next to it; if the module is unavailable, contact data is taken from this PBX.

### WebRTC networks (ICE candidates) <a href="#ice" id="ice"></a>

This section determines the network route the call audio takes. When a call is set up, the PBX sends the app a list of its addresses (ICE candidates) — one per network interface — and the client picks a reachable one. If the list contains an unreachable address (for example, a VPN tunnel address), the client may end up choosing it: the call "connects" but there is no audio.

The table shows the PBX interfaces: name, network and type — **LAN** (local network), **Public** (external address) or **VPN**. A checked box allows offering the network to clients; an unchecked one hides it from the list.

{% hint style="info" %}
Keep VPN tunnels disabled by default: their addresses are unreachable for employees on the internet and interfere with choosing the correct route. If audio disappeared after enabling the VPN module, the first thing to do is uncheck the VPN interface here.
{% endhint %}

### Status panel <a href="#health" id="health"></a>

At the top of the settings page are component indicators: **nginx**, **push-server** (push notification delivery), **STUN** (local coturn and the external STUN from the PBX settings), the **built-in status service**, and the SSL certificate. If calls fail or pushes don't arrive, start troubleshooting from this panel: it shows which component is unavailable and why.

## Access rights and supervisors <a href="#access" id="access"></a>

<figure><img src="../../.gitbook/assets/softphone-access.png" alt=""><figcaption><p>The "Access rights" tab</p></figcaption></figure>

### Basic rights <a href="#access-basic" id="access-basic"></a>

Without configured groups, each employee sees only their own call history and status. Other employees are not shown in the list; call queues are visible to everyone.

### Access groups <a href="#access-groups" id="access-groups"></a>

An access group combines employees and defines their rights within the group:

| Right                                 | Description                                                    |
| ------------------------------------- | -------------------------------------------------------------- |
| View statuses of group members        | Members see each other in the list along with current statuses |
| View the group's call history         | Calls involving at least one member of the group are available |
| Pick up calls within the group        | One can answer an incoming call ringing for a colleague        |
| View the group's active conversations | Active calls of group members and the call parties are visible |

<figure><img src="../../.gitbook/assets/softphone-access-edit.png" alt=""><figcaption><p>Editing an access group</p></figcaption></figure>

The selected rights are automatically granted to all group members.

### Supervisors <a href="#supervisors" id="supervisors"></a>

A supervisor is an employee with extended rights over their own and **controlled groups** (the list is defined during setup). Besides monitoring, the supervisor can act on calls:

<figure><img src="../../.gitbook/assets/softphone-supervisor.png" alt=""><figcaption><p>The "Supervisor" tab — extended rights over controlled groups</p></figcaption></figure>

| Right                    | Description                                                                               |
| ------------------------ | ----------------------------------------------------------------------------------------- |
| Call pickup              | Answer an incoming call for a group employee                                              |
| Listen to a conversation | Silently monitor a call without the parties knowing                                       |
| Whisper to an employee   | Talk to the employee during a call; their conversation partner cannot hear the supervisor |
| Join a conversation      | Join the call; both parties hear the supervisor                                           |
| Play call recordings     | Access recordings of group employees                                                      |
| View active call parties | Who is talking to whom across the groups                                                  |

Supervisor rights are configured on the **"Access rights"** tab: assign a supervisor, select the controlled groups and the required permissions.
