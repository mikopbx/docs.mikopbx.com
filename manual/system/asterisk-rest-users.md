---
description: Short description of ARI (Asterisk REST Interface)
---

# ARI Access

ARI is a RESTful API with WebSocket support that gives full control over Asterisk channels, bridges, and media streams in real time. Unlike the MikoPBX REST API, ARI works directly with the Asterisk core and is designed for building custom telephony applications.

{% hint style="info" %}
Detailed ARI documentation is available on the official Asterisk website: [Asterisk REST Interface](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/)
{% endhint %}

### What is it used for?

ARI is used when the standard PBX features are not enough and custom call handling logic is required:

* **WebRTC applications and softphones** — web phones and mobile clients with direct media stream control
* **Interactive voice response (IVR)** — custom menu logic unavailable through the standard dialplan
* **Conference calls** — programmatic control of bridges and participants
* **Call recording and processing** — real-time audio interception for analytics or transcription
* **Voice bots and assistants** — integration with external AI services
* **Advanced queues** — custom call distribution logic

### How does it work?

ARI consists of three components:

1. **REST API** — management of Asterisk objects: channels, bridges, recordings
2. **WebSocket** (`/asterisk/ari/events`) — receiving asynchronous real-time events: incoming call, call end, DTMF keypress, etc.
3. **Stasis** — a dialplan application that passes a channel to your ARI application for control

Typical scenario: a call enters the dialplan → `Stasis()` passes the channel to your application → the application controls the call via REST API and receives events via WebSocket.

### Configuring an ARI User

1. Before starting, you need to enable the ARI interface (it is disabled by default). Go to **"System"** → **"General Settings"**.

<figure><img src="../../.gitbook/assets/generalSettingsMikoPBX.png" alt=""><figcaption><p>"General settings" section in MikoPBX</p></figcaption></figure>

2. Go to the **"AMI & ARI"** tab and toggle the **"Use ARI Interface"** switch. In the **"CORS allowed  origins"** field, specify the domains from which requests to ARI will be made. CORS is a browser security mechanism that restricts cross-domain API requests.

| Value                       | When to use                             |
| --------------------------- | --------------------------------------- |
| _(empty)_                   | Access from the same domain only        |
| `http://localhost:3000`     | Local development                       |
| `https://app.mycompany.com` | Production application                  |
| `*`                         | All origins — for testing purposes only |

{% hint style="warning" %}
Never use `*` in production. Only specify trusted domains over HTTPS.
{% endhint %}

<figure><img src="../../.gitbook/assets/AMI&#x26;ARIinGenSettings.png" alt=""><figcaption><p>Enabling ARI</p></figcaption></figure>

3. Go to **"System" → "ARI Access"**.

<figure><img src="../../.gitbook/assets/ARIAccessSection.png" alt=""><figcaption><p>"ARI Access" section in MikoPBX</p></figcaption></figure>

4. Click **"Add User"**.

<figure><img src="../../.gitbook/assets/ARIaddUserButton.png" alt=""><figcaption><p>"Add User" button</p></figcaption></figure>

5. Fill in the following parameters:

* **Username** — login for connection, e.g. `ari_user`.
* **Password** — password for connection.
* **Description** — description for the current user, e.g. "WebRTC Demo".
* **Applications** — specify the names of Stasis applications the user has access to. Leave the field empty for access to all applications.

{% hint style="info" %}
**Common applications**

* **ari-app:** Main ARI application
* **stasis:** Base Stasis application
* **external-media:** Working with external media streams
* **bridge-app:** Call bridge management
* **channel-spy:** Channel monitoring
{% endhint %}

Save the settings.

<figure><img src="../../.gitbook/assets/ARIUserParameters.png" alt=""><figcaption><p>Configuring user access to the ARI</p></figcaption></figure>

### Connection Parameters

#### **WebSocket**

| Type         | URL                                               |
| ------------ | ------------------------------------------------- |
| Regular      | `ws://your-mikopbx.com:8088/asterisk/ari/events`  |
| Secure (TLS) | `wss://your-mikopbx.com:8089/asterisk/ari/events` |

Replace `[application]` with the name of your Stasis application.

#### **REST API**

| Type  | URL                                           |
| ----- | --------------------------------------------- |
| HTTP  | `http://your-mikopbx.com:8088/asterisk/ari`   |
| HTTPS | `https://your-mikopbx.com:8089/asterisk/ari/` |

**Authentication:** HTTP Basic Auth — ARI user login and password.

```bash
# REST API request via curl
curl -u username:password https://your-mikopbx.com:8089/asterisk/ari/asterisk/info
```

{% hint style="info" %}
It is recommended to use secure connections (wss:// and https://) with a valid SSL certificate. Regular ws:// and http:// are acceptable only in an isolated test environment.
{% endhint %}

### Example: Hello World

This is a minimal ARI example — a channel enters a Stasis application, the application plays a sound file and ends the call.

The example is taken from the official Asterisk documentation: [Getting Started with ARI](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/Getting-Started-with-ARI/)

#### **Step 1 — Connect to WebSocket**

```bash
wscat -c "wss://username:password@your-mikopbx.com:8089/asterisk/ari/events?app=hello-world"
```

#### **Step 2 — Configure the Incoming Route**

In MikoPBX, go to **"Routing" → "Dialplan Applications"**, create an application with the type **"Asterisk Dialplan"** and the following code:

```php
1,Answer()
n,Stasis(hello-world)
n,Hangup()
```

Assign the application to the required incoming route.

#### **Step 3 — Make a Call**

When an incoming call arrives, the WebSocket will receive a `StasisStart` event:

```json
{
  "type": "StasisStart",
  "timestamp": "2026-03-25T07:18:27.423+0300",
  "args": [],
  "channel": {
    "id": "mikopbx-1774412307.28",
    "name": "Local/10003258@internal-incoming-0000000a;2",
    "state": "Up",
    "protocol_id": "",
    "caller": {
      "name": "79257184275",
      "number": "79257184275"
    },
    "connected": {
      "name": "",
      "number": "252"
    },
    "accountcode": "",
    "dialplan": {
      "context": "internal",
      "exten": "10003258",
      "priority": 2,
      "app_name": "Stasis",
      "app_data": "hello-world"
    },
    "creationtime": "2026-03-25T07:18:27.371+0300",
    "language": "en-en"
  },
  "asterisk_id": "82:6a:9e:68:10:11",
  "application": "hello-world"
}
```

#### **Step 4 — Play Sound via REST API**

Open a new terminal window and run the following command:

{% hint style="info" %}
Use the channel `id` from the `StasisStart` event!
{% endhint %}

```awk
curl -u username:password -X POST \
  "https://your-mikopbx.com:8089/asterisk/ari/channels/mikopbx-1774412307.28/play?media=sound:/storage/usbdisk1/mikopbx/media/custom/miko_hello"
```

On successful playback, you will see the following output in the terminal:

```json
{
  "id": "a0ee0d43-2af5-4250-a303-43825507a06c",
  "media_uri": "sound:/storage/usbdisk1/mikopbx/media/custom/miko_hello",
  "target_uri": "channel:mikopbx-1774412307.28",
  "language": "en-en",
  "state": "playing"
}
```

#### **Step 5 — End the Call**

After the call ends, the WebSocket will send a `StasisEnd` event:

```json
{
  "type": "StasisEnd",
  "timestamp": "2026-03-25T07:19:44.982+0300",
  "channel": {
    "id": "mikopbx-1774412307.28",
    "name": "Local/10003258@internal-incoming-0000000a;2",
    "state": "Up",
    "protocol_id": "",
    "caller": {
      "name": "79257184275",
      "number": "79257184275"
    },
    "connected": {
      "name": "",
      "number": "252"
    },
    "accountcode": "",
    "dialplan": {
      "context": "internal",
      "exten": "10003258",
      "priority": 2,
      "app_name": "Stasis",
      "app_data": "hello-world"
    },
    "creationtime": "2026-03-25T07:18:27.371+0300",
    "language": "en-en"
  },
  "asterisk_id": "82:6a:9e:68:10:11",
  "application": "hello-world"
}
```

### Example: Presence Monitor

A live employee status table in the terminal — no incoming route or Stasis application configuration required. Works by subscribing to all station events.

Install dependencies:

```bash
pip install requests websockets
```

```python
import asyncio
import websockets
import json
import os
from datetime import datetime

ARI_HOST = 'your-mikopbx.com'
ARI_USER = 'ari_user'
ARI_PASS = 'your-ari-password'

peers = {}

STATES = {
    'NOT_INUSE':   ('🟢', 'Available'),
    'BUSY':        ('🔴', 'Busy'),
    'UNAVAILABLE': ('⚫', 'Unavailable'),
}

def draw():
    print('\033[2J\033[H', end='')
    now = datetime.now().strftime('%H:%M:%S')
    print(f'MikoPBX — Presence Monitor [{now}]')
    print('─' * 50)
    print(f'  {"Number":<10} {"Name":<20} {"Status":<15} {"Updated"}')
    print('─' * 50)
    for number, info in sorted(peers.items()):
        icon, label = STATES.get(info['state'], ('❓', info['state']))
        print(f'  {number:<10} {info["name"]:<20} {icon} {label:<12} {info["updated"]}')
    print('─' * 50)
    print(f'  Employees: {len(peers)}')

async def run():
    uri = (
        f"wss://{ARI_USER}:{ARI_PASS}@{ARI_HOST}:8089/asterisk/ari/events"
        f"?app=auto-receptionist&subscribeAll=true"
    )
    async with websockets.connect(uri) as ws:
        draw()
        async for message in ws:
            event = json.loads(message)
            etype = event.get('type')

            if etype == 'DeviceStateChanged':
                ds     = event.get('device_state', {})
                name   = ds.get('name', '')
                state  = ds.get('state', '')

                if not name.startswith('PJSIP/'):
                    continue

                number = name.replace('PJSIP/', '')

                if number not in peers:
                    peers[number] = {'name': number, 'state': state, 'updated': '—'}

                peers[number]['state']   = state
                peers[number]['updated'] = datetime.now().strftime('%H:%M:%S')
                draw()

            elif etype == 'PeerStatusChange':
                ep     = event.get('endpoint', {})
                number = ep.get('resource', '')
                state  = ep.get('state', '')

                if not number:
                    continue

                if number not in peers:
                    peers[number] = {'name': number, 'state': 'unknown', 'updated': '—'}

                if state == 'online':
                    peers[number]['state'] = 'NOT_INUSE'
                elif state == 'offline':
                    peers[number]['state'] = 'UNAVAILABLE'

                peers[number]['updated'] = datetime.now().strftime('%H:%M:%S')
                draw()

            elif etype == 'ContactStatusChange':
                ep     = event.get('endpoint', {})
                number = ep.get('resource', '')
                ci     = event.get('contact_info', {})
                status = ci.get('contact_status', '')

                if not number:
                    continue

                if number not in peers:
                    peers[number] = {'name': number, 'state': 'unknown', 'updated': '—'}

                if status == 'Reachable':
                    peers[number]['state'] = 'NOT_INUSE'
                elif status in ('Unreachable', 'NonQualified'):
                    peers[number]['state'] = 'UNAVAILABLE'

                peers[number]['updated'] = datetime.now().strftime('%H:%M:%S')
                draw()

asyncio.run(run())
```

As calls are made, the table will update in real time:

{% code overflow="wrap" %}
```python
MikoPBX — Presence Monitor [11:34:43]
──────────────────────────────────────────────────
  Number     Name                 Status          Updated
──────────────────────────────────────────────────
  202        202                  🟢 Available    11:34:25
  243        243                  ⚫ Unavailable  11:34:43
  252        252                  🔴 Busy         11:34:41
──────────────────────────────────────────────────
  Employees: 3
```
{% endcode %}

***

Full ARI documentation is available on the Asterisk website: [docs.asterisk.org](https://docs.asterisk.org/Configuration/Interfaces/Asterisk-REST-Interface-ARI/)
