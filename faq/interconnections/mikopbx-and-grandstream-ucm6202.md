# Integration of MikoPBX and Grandstream UCM6202

In this example, there are two departments within a company, where the Administration will use the Grandstream PBX system, while the Sales Department will use MikoPBX and will need integration with 1C. Below is a guide for integrating both PBX systems:

1. **Administration**: Internal numbering plan **1XX** is managed by Grandstream.
2. **Sales Department**: Internal numbering plan **3XX** is managed by MikoPBX.
3. **Grandstream**: Connected to the external line for city calls.
4. **MikoPBX**: Should use Grandstream’s external line for making city calls.
5. Users in **1XX** can call **3XX** numbers.
6. Users in **3XX** can call **1XX** numbers.

***

## Grandstream UCM6202 Setup

### Trunk

1. Go to "**Extensions / Trunk**" → "**VoIP Trunk**", and click "**Add SIP Trunk**".
2. For "**Provider Name**", use a descriptive name like **MikoPBX**.
3. Set the "**Host Name**" to the address of MikoPBX.
4. Set "**Transport**" to **UDP**.
5. Enable "**Keep Original CID**".
6. Click "**Save**".

#### **Outbound to \_2XX**

This rule allows Grandstream users (**1XX**) to call MikoPBX internal numbers (**2XX**).

1. Go to "**Extensions / Trunk**" → "**Outbound Routes**".
2. Add a new route and set the **Trunk** to "**SIP Trunks – MikoPBX**".
3. Set **Calling Rule Name** to **MikoPBX**.
4. Set **Pattern** to:

```
_2XX
_90000099
```

#### **Inbound to \_1XX**

This rule allows MikoPBX users (**2XX**) to call Grandstream internal numbers (**1XX**).

1. Go to "**Extensions / Trunk**" → "**Inbound Routes**".
2. Select the trunk "**SIP Trunks – MikoPBX**".
3. Add a route with the **Pattern** = **\_1XX**.
4. Set **Allowed DID Destination** to **Extension**.

**Outbound to \[78]XX...**

This rule allows MikoPBX users to make external calls through Grandstream.

1. Go to "**Extensions / Trunk**" → "**Inbound Routes**".
2. Select the trunk "**SIP Trunks – MikoPBX**".
3. Add a route with **Pattern** = **\[78]XXXXXXXXXX**.

### Extensions

Ensure that the correct CallerID is set for the Grandstream extensions under "**Extensions / Trunk**" → "**Extension**".

***

### MikoPBX Setup

#### Provider

1. Go to "**Routing**" → "**Providers**" and add a new provider.
2. Set the **Provider Name** to something descriptive like **Grandstream**.
3. Set the **Account Type** to "Authentication by IP".
4. Set **Host or IP Address** to the address of Grandstream.

#### Inbound Routing

This rule allows Grandstream users (**1XX**) to call MikoPBX internal numbers (**2XX**).

1. Go to "**Routing**" → "**Inbound Routes**".
2. Add a new rule for provider "**Grandstream**" and set the **DID** pattern to "**2XX**".

#### Outbound to 1XX

This rule allows MikoPBX users (**2XX**) to call Grandstream users (**1XX**).

1. Go to "**Routing**" → "**Outbound Routes**".
2. Create a rule where numbers starting with "**1**" and consisting of three digits are routed through "**Grandstream**".

#### Outbound to City Numbers

This rule allows MikoPBX to make external calls through Grandstream.

1. Go to "**Routing**" → "**Outbound Routes**".
2. Create a rule for numbers starting with "**7**" or "**8**" and consisting of 10 digits, to be routed through "**Grandstream**".

***

By following these steps, you will integrate MikoPBX and Grandstream UCM6202, allowing communication between the two systems and external call routing.
