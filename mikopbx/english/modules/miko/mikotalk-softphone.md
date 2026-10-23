---
description: >-
  The module provides the API required to connect the MikoTalk softphone to
  MikoPBX: push notifications, call history, contacts and supervisor tools.
---

# MikoTalk Softphone

The **Softphone Backend** module turns an employee's iPhone into a full-featured PBX extension: calls over Wi-Fi and mobile data, incoming calls delivered via push notifications — just like a regular call, even when the app is closed.

Setting up the softphone is as simple as scanning a QR code or opening an auto-configuration link.

## Features <a href="#features" id="features"></a>

| Feature             | Description                                                                                                                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Push calls          | Incoming call alerts are delivered even when the app is closed or the device is asleep                                                                                                                         |
| Call history        | MikoTalk users see their full call history, subject to access permissions ([instructions on configuring permissions](mikotalk-softphone.md#access)). Call recordings can be played back right from the history |
| Contacts            | A caller's name can be resolved from carrier data (if the carrier provides it) or from 1C data — connect the ["Phone Panel 4.0"](https://docs.telefon1c.ru/v4/get-started/mikopbx/) module for the latter      |
| Employee statuses   | Who is available and who is on a call. Call queues are visible to everyone                                                                                                                                     |
| Transfer and pickup | Blind and attended transfer, picking up a colleague's incoming call                                                                                                                                            |
| Supervisor tools    | Listen, whisper, barge-in on a call, group monitoring                                                                                                                                                          |
| Missed calls        | Unanswered calls are marked with a dedicated counter in the app                                                                                                                                                |

## Installation <a href="#install" id="install"></a>

Install the **WebRTC Softphone Backend** module from the MikoPBX [modules marketplace](../../manual/modules/pbx-extension-modules.md#marketplace).

Once installed, open the module settings: **"Modules" → "WebRTC Softphone Backend"**. A license is required — see [Requirements and licensing](mikotalk-softphone.md#requirements).

## Requirements and licensing <a href="#requirements" id="requirements"></a>

* MikoPBX **2026.1.223** or later.
* Licenses are issued **per employee**: all devices of the same employee consume a single license.

{% hint style="info" %}
The module is installed from the marketplace. A **trial license** is activated automatically on first installation, so you can try the softphone before purchasing. See [Registration in the modules marketplace](../../manual/modules/licensing.md) for details on registering and renewing licenses.
{% endhint %}

### System requirements <a href="#system-requirements" id="system-requirements"></a>

For calls over mobile data, the PBX must be reachable from the internet: a **public static IP address** or port forwarding from an external IP/domain to the PBX (1:1 NAT). The following ports must be open and forwarded:

| Port                                         | Protocol    | Purpose                                                                                                                                                                                    |
| -------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| External API port (default **8988**)         | TCP         | App connectivity: authentication, auto-configuration link, history, statuses. The port is configured in the module settings — see [External server](mikotalk-softphone.md#external-server) |
| **3478** (STUN/TURN, built-in coturn)        | UDP and TCP | NAT traversal: helps the app discover its public address and build a media route to the PBX                                                                                                |
| PBX RTP port range (default **10000–12000**) | UDP         | Call audio traffic. The range is configured in the MikoPBX network settings                                                                                                                |

Additionally, in the PBX's own firewall settings (**"System" → "Network & Firewall" → "Firewall"**), enable the **Softphone RTP** service for the **"Internet"** profile — or at least for **"Trusted networks"** if employees only place calls from the office network. Otherwise, audio traffic is blocked by the built-in firewall.

<figure><img src="../../.gitbook/assets/softphone-firewall (1).png" alt=""><figcaption><p>The Softphone RTP service in the PBX firewall settings</p></figcaption></figure>

For secure connections (HTTPS/WSS), use the [Let's Encrypt](module-get-ssl-lets-encrypt.md) module — it issues a trusted SSL certificate. A certificate can also be obtained for an IP address; a domain name is not required.

## Connecting clients <a href="#clients" id="clients"></a>

### Auto-configuration link <a href="#provision" id="provision"></a>

The app is connected via an auto-configuration link. Open the module settings, **"Device setup"** tab:

1. Select the employee's extension number.
2. Set the link lifetime: 10 minutes, 1 hour, 5 hours or 1 day.
3. Optionally enable **"One-time link"** — the link stops working after the first use.
4. Click **"Create"** — the module shows a QR code and the link itself.

The link can be shared by any convenient means; the QR code can be scanned with the iPhone camera. The app downloads the settings and asks to confirm sign-in.

<figure><img src="../../.gitbook/assets/softphone-provision (1).png" alt=""><figcaption><p>The "Device setup" tab</p></figcaption></figure>

The same tab contains the employee table: search by name or number, emailing a QR code to one employee or to selected ones (**"Send to selected"**, up to 100 at a time). The email goes to the address specified in the employee's card.

{% hint style="info" %}
To email connection parameters, first configure the PBX mail server: **"System" → "Mail and Notifications"**. See the [Mail and Notifications](../../manual/system/mail-settings/) section for instructions.
{% endhint %}

### Installing MikoTalk <a href="#ios" id="ios"></a>

1. Install the [MikoTalk app from the App Store](https://apps.apple.com/si/app/mikotalk/id6759347414).
2. Open the auto-configuration link email on the iPhone, or scan the QR code from the module page.
3. Allow notifications — without them, incoming calls will not be delivered.

### Employee sessions <a href="#sessions" id="sessions"></a>

The module settings show each employee's active sessions and last activity time. The **"Terminate all employee sessions"** button forcibly signs out all of their devices and releases the license — useful, for example, when a phone is lost.

## Module settings <a href="#settings" id="settings"></a>

<figure><img src="../../.gitbook/assets/softphone-settings (1).png" alt=""><figcaption><p>The "Settings" tab</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/softphone-access (1).png" alt=""><figcaption><p>The "Access rights" tab</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/softphone-access-edit (1).png" alt=""><figcaption><p>Editing an access group</p></figcaption></figure>

The selected rights are automatically granted to all group members.

### Supervisors <a href="#supervisors" id="supervisors"></a>

A supervisor is an employee with extended rights over their own and **controlled groups** (the list is defined during setup). Besides monitoring, the supervisor can act on calls:

<figure><img src="../../.gitbook/assets/softphone-supervisor (1).png" alt=""><figcaption><p>The "Supervisor" tab — extended rights over controlled groups</p></figcaption></figure>

| Right                    | Description                                                                               |
| ------------------------ | ----------------------------------------------------------------------------------------- |
| Call pickup              | Answer an incoming call for a group employee                                              |
| Listen to a conversation | Silently monitor a call without the parties knowing                                       |
| Whisper to an employee   | Talk to the employee during a call; their conversation partner cannot hear the supervisor |
| Join a conversation      | Join the call; both parties hear the supervisor                                           |
| Play call recordings     | Access recordings of group employees                                                      |
| View active call parties | Who is talking to whom across the groups                                                  |

Supervisor rights are configured on the **"Access rights"** tab: assign a supervisor, select the controlled groups and the required permissions.
