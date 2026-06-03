---
description: Instructions for connecting multiple PBX systems
---

# MikoPBX and FreePBX (IAX)

## MikoPBX Configuration

1. In MikoPBX, navigate to **"Routing" → "Telephony Providers"**:

"Telephony providers" section

1. Create a new **IAX** provider:

"Connect IAX" button

1. Fill in the parameters:

- **Provider Name** – any name
- **Host or IP Address** – the IP address of FreePBX
- **Username** – “tmp”
- **Password** – any (secure) password

Save the parameters.

Provider Parameters 1

1. After saving, you’ll see the **provider ID** in the browser’s address bar. Copy it into the **Username** field:

Provider Parameters 2

## FreePBX Configuration

1. In FreePBX, go to **"Connectivity" → "Trunks"** and add a new **IAX2** trunk:

New IAX2 Trunk

1. Under the **"General"** tab, set **Trunk Name** to the login used in MikoPBX (seen in the browser address bar, e.g., “**IAX-TRUNK-1E8B1CFE**”):

"Trunk Name" field

1. Under **"Dialed Number Manipulation Rules,"** define a pattern for outgoing calls:

Dialed Number Manipulation Rules

1. Go to **"pjsip Settings"** → **"iax2 Settings."** Under **Trunk Name**, use the same login from MikoPBX (e.g., “IAX-TRUNK-1E8B1CFE”):

"Trunk Name" field (iax2 Settings)

Fill in **PEER Details**:

```
type=friend
auth=plaintext
language=ru-ru
qualify=2000
transfer=mediaonly
disallow=all
;username=mikopbx
host=dynamic
trunk=yes
secret=123
allow=alaw&ulaw
```

"PEER Details" field

1. In the **"Incoming"** tab, fill in the **Register String** field in the format **“LOGIN:PASSWORD@IP\_MIKO\_PBX”**:

"Register String" field

## Routing Setup

### MikoPBX

1. Define an **incoming route** ([see “Incoming Routes” guide](../../manual/routing/incoming-routing.md)). In this example, all calls are routed to extension **202**:

Inbound routing rule MikoPBX

If needed, define a separate route for each DID with its own destination:

Inbound routing rule MikoPBX (Individual DID)

1. Define an **outgoing route** ([see “Outbound Routes” guide](https://chatgpt.com/manual/routing/outbound-routing.md)):

Outbound routing MikoPBX

### FreePBX

1. Go to **"Connectivity" → "Inbound Routes"** and define an inbound route:

Inbound routing FreePBX

1. Go to **"Connectivity" → "Outbound Routes"** and define an outbound route:

Outbound routing FreePBX