---
description: Description and configuration of Firewall rules in MikoPBX
---

# Firewall

The **Firewall** in MikoPBX is an interface for configuring the system's firewall. Here, administrators can create and manage network traffic filtering rules, controlling access to MikoPBX and protecting it from unauthorized access and network threats. Configuring the firewall ensures the security of the telephone system, preventing potential attacks and ensuring stable operation in the organization's network infrastructure.

In MikoPBX, all local subnets can be described in the "**Network and Firewall**" → "**Firewall**" section. The firewall is designed to restrict access to the station by traffic type and subnets.

<figure><img src="../../.gitbook/assets/new1 (4).png" alt=""><figcaption><p>Section "Network and Firewall" -> "Firewall" in MikoPBX</p></figcaption></figure>

To add a new rule, you need to click on the button:

<figure><img src="../../.gitbook/assets/new2 (2).png" alt=""><figcaption><p>Button for creating a new rule</p></figcaption></figure>

## General settings

You can give the rule any custom name. To the right of the subnet address, there is a field for Subnet Mask in CIDR format.&#x20;

<figure><img src="../../.gitbook/assets/new3 (1).png" alt=""><figcaption><p>Rule parameters</p></figcaption></figure>

## Available services

* **SIP\&RTP** - registration of phones and voice traffic. Session Initiation Protocol is used for establishing connections between VoIP phones.
* &#x20;**WEB** - access to the administrative interface for configuring the PBX. SSH - root access to the system.&#x20;
* **SSH** (Secure Shell) allows accessing the MikoPBX console.
* &#x20;**AMI** - access to Asterisk Manager API via telnet. Asterisk Manager Interface (AMI) provides access to Asterisk via TCP/IP protocol.&#x20;
* **AJAM** - access to Asterisk Manager API via HTTP or HTTPS.
* &#x20;**ICMP** - communication check using the 'ping' command.&#x20;
* **CTICLIENT** - connection of the telephony panel 2 for 1C.

<figure><img src="../../.gitbook/assets/new4.png" alt=""><figcaption><p>"Available service" section</p></figcaption></figure>

## Advanced Options

* Each subnet has a flag 'Is it a VPN or a local network'. When this flag is set, MikoPBX will present itself as a local IP to all local subnets instead of external ones.&#x20;
* The flag 'Never block addresses from this network' should be enabled only for trusted subnets. If this flag is enabled, intrusion prevention rules will not apply to this subnet

<figure><img src="../../.gitbook/assets/new5.png" alt=""><figcaption><p>"Advanced options" section</p></figcaption></figure>

## Behaviour in Docker containers

In Docker bridge mode the MikoPBX built-in firewall and fail2ban **do
not protect the web interface**: the container cannot manage host
iptables, and HTTP clients arrive from the `docker0` gateway. SIP
protection continues to work (UDP DNAT preserves the source IP).

To protect the web interface in Docker, choose one of:

* `network_mode: host` for the container (when the host is dedicated
  to the PBX);
* An external CrowdSec-compatible bouncer in front of the MikoPBX API —
  see
  [External firewall for Docker](../../security/external-firewall-enforcement.md).
