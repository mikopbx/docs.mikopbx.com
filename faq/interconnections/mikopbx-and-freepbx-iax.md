---
description: Instructions for connecting multiple PBX systems
---

# MikoPBX and FreePBX (IAX)

## MikoPBX Configuration

1. In MikoPBX, navigate to **"Routing" → "Telephony Providers"**:

<figure><img src="../../.gitbook/assets/TelephonyProvidersSection.jpg" alt=""><figcaption><p>"Telephony providers" section</p></figcaption></figure>

2. Create a new **IAX** provider:

<figure><img src="../../.gitbook/assets/connectIAXBtn.jpg" alt=""><figcaption><p>"Connect IAX" button</p></figcaption></figure>

3. Fill in the parameters:

* **Provider Name** – any name
* **Host or IP Address** – the IP address of FreePBX
* **Username** – “tmp”
* **Password** – any (secure) password

Save the parameters.

<figure><img src="../../.gitbook/assets/providerParameters (1).jpg" alt=""><figcaption><p>Provider Parameters 1</p></figcaption></figure>

4. After saving, you’ll see the **provider ID** in the browser’s address bar. Copy it into the **Username** field:

<figure><img src="../../.gitbook/assets/providerParameters2.jpg" alt=""><figcaption><p>Provider Parameters 2</p></figcaption></figure>

## FreePBX Configuration

1. In FreePBX, go to **"Connectivity" → "Trunks"** and add a new **IAX2** trunk:

<figure><img src="../../.gitbook/assets/newIAXTrunkFreePBX.jpg" alt=""><figcaption><p>New IAX2 Trunk</p></figcaption></figure>

2. Under the **"General"** tab, set **Trunk Name** to the login used in MikoPBX (seen in the browser address bar, e.g., “**IAX-TRUNK-1E8B1CFE**”):

<figure><img src="../../.gitbook/assets/TrunkNameFreePBX.jpg" alt=""><figcaption><p>"Trunk Name" field</p></figcaption></figure>

3. Under **"Dialed Number Manipulation Rules,"** define a pattern for outgoing calls:

<figure><img src="../../.gitbook/assets/DNMRFreePBX.jpg" alt=""><figcaption><p>Dialed Number Manipulation Rules</p></figcaption></figure>

4. Go to **"pjsip Settings"** → **"iax2 Settings."** Under **Trunk Name**, use the same login from MikoPBX (e.g., “IAX-TRUNK-1E8B1CFE”):

<figure><img src="../../.gitbook/assets/iax2TrunkName.jpg" alt=""><figcaption><p>"Trunk Name" field (iax2 Settings)</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/PEERDetailsParameters.jpg" alt=""><figcaption><p>"PEER Details" field</p></figcaption></figure>

5. In the **"Incoming"** tab, fill in the **Register String** field in the format **“LOGIN:PASSWORD@IP\_FREE\_PBX”**:

<figure><img src="../../.gitbook/assets/registerStringFieldFreePBX.jpg" alt=""><figcaption><p>"Register String" field</p></figcaption></figure>

## Routing Setup

### MikoPBX

1. Define an **incoming route** ([see “Incoming Routes” guide](../../manual/routing/incoming-routing.md)). In this example, all calls are routed to extension **202**:

<figure><img src="../../.gitbook/assets/inboundMikoPBX.jpg" alt=""><figcaption><p>Inbound routing rule MikoPBX</p></figcaption></figure>

If needed, define a separate route for each DID with its own destination:

<figure><img src="../../.gitbook/assets/inboundMikoPBXindividualDID.jpg" alt=""><figcaption><p>Inbound routing rule MikoPBX (Individual DID)</p></figcaption></figure>

2. Define an **outgoing route** ([see “Outbound Routes” guide](https://chatgpt.com/manual/routing/outbound-routing.md)):

<figure><img src="../../.gitbook/assets/outboundRulesMikoPBX(USA).jpg" alt=""><figcaption><p>Outbound routing MikoPBX</p></figcaption></figure>

### FreePBX

1. Go to **"Connectivity" → "Inbound Routes"** and define an inbound route:

<figure><img src="../../.gitbook/assets/inboundRouteFreePBX.jpg" alt=""><figcaption><p>Inbound routing FreePBX</p></figcaption></figure>

2. Go to **"Connectivity" → "Outbound Routes"** and define an outbound route:

<figure><img src="../../.gitbook/assets/outboundRoutingFreePBX.jpg" alt=""><figcaption><p>Outbound routing FreePBX</p></figcaption></figure>
