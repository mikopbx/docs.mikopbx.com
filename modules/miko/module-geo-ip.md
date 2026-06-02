---
description: >-
  The GeoIP module blocks incoming connections to MikoPBX from selected
  countries at the firewall level. It protects the PBX from SIP scanners,
  password brute-forcing and unwanted calls. Works for both IPv4 and IPv6.
---

# GeoIP Filtering

The **GeoIP** module blocks incoming connections to your PBX from selected countries. A PBX exposed to the internet constantly receives hundreds of connection attempts from all over the world: scanners look for vulnerable SIP servers, brute-force extension passwords and try to place calls at your expense. Most of these attacks come from countries you have no telephony with.

GeoIP filtering lets you block whole countries in one click — every IP address from those countries is dropped before it can ever reach the SIP server. Filtering works for both **IPv4 and IPv6** at the same time.

{% hint style="info" %}
The module uses **ipset** — a Linux kernel mechanism for handling large address lists. It can check hundreds of thousands of subnets almost instantly, with no impact on PBX performance. ipset support is included in all standard MikoPBX builds.
{% endhint %}

## Installation and activation <a href="#install" id="install"></a>

1. Open **Modules → Module Marketplace**, find **GeoIP Filter** and click **Install**.
2. Once installed, go to **Network and firewall → GeoIP Filtering**.
3. Turn on the **Enable GeoIP filtering** switch.
4. Select the countries you want to block and click **Save**.

After it is enabled, the module automatically starts the first download of the IP address lists. Until the data is downloaded, the **Last update** field shows "Data not yet downloaded".

{% hint style="warning" %}
If the `ipset` utility is not available on the system, the module shows an **"ipset unavailable"** warning — in that case filtering is not applied to traffic. The utility is present on standard MikoPBX builds.
{% endhint %}

## Selecting countries <a href="#countries" id="countries"></a>

The module page lists **249 countries** (the ISO 3166-1 standard). Each country shows its current status — **Blocked** or **Allowed**. A country is allowed by default; blocking is enabled explicitly.

| Element | Purpose |
| --- | --- |
| **Search** | Quickly find a country by name. |
| **Status filter** | Show all countries, allowed only, or blocked only. |
| **Block all** | Block every country in the list at once. |
| **Unblock all** | Remove blocking from every country. |
| **Save** | Apply the changes. Firewall rules are updated immediately after saving. |

## Data source <a href="#source" id="source"></a>

Per-country subnet lists can be obtained from one of three sources, selected in the **Data source** field:

| Source | Notes |
| --- | --- |
| **DB-IP Lite** (recommended) | An up-to-date "country → subnet" database. A copy of the database **ships inside the module**, so the very first activation works even without internet access. |
| **RIR delegation files** | Official address-allocation files from the Regional Internet Registries (RIPE, ARIN, APNIC, etc.). |
| **ipdeny.com** | Ready-made aggregated per-country CIDR blocks. |

{% hint style="info" %}
The **DB-IP Lite** source includes an offline copy of the database inside the module. This matters for PBXs in restricted networks where downloading the database from outside is unreliable: the module uses the bundled copy first and only goes online if it is missing.
{% endhint %}

## Updating the lists <a href="#update" id="update"></a>

The IP address lists are updated **automatically once a week** (on Sundays). The date and time of the last successful update, along with the number of downloaded subnets, are shown on the module page.

To refresh the data immediately, click **Update now** — a progress indicator appears next to it. This is handy right after installation or after changing the data source.

## How it works <a href="#how" id="how"></a>

GeoIP blocking is applied in the firewall **last of all — after every allow rule**. The order in which an incoming connection is processed:

1. **Established connections** — passed.
2. **Firewall rules** (your trusted subnets) — passed.
3. **SIP provider IP addresses** — passed.
4. **GeoIP filter** — block by country.

{% hint style="warning" %}
Trusted addresses are never blocked by mistake. Even if your SIP provider's IP address belongs to a blocked country, it keeps working — because the allow rule is evaluated before the GeoIP filter.
{% endhint %}

## System requirements <a href="#requirements" id="requirements"></a>

- MikoPBX **2026.1.223** or newer.
- **ipset** support in the Linux kernel (included in all standard MikoPBX builds).
