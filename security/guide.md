---
description: How to protect MikoPBX from hacking and unauthorized access
---

# Securing MikoPBX (In dev)

IP PBX systems are increasingly being targeted by attackers. Criminals gain access to your telephony and make calls at your expense - to premium numbers and international destinations. This can result in losses of tens or hundreds of thousands of dollars within just a few hours.

Beyond direct financial losses, a compromised PBX can be used by fraudsters to make calls on behalf of your organization - for example, calling citizens while impersonating banks or government agencies. Victims see your company's phone number, causing reputational damage and potential investigations by law enforcement.

Go through every item in this guide - even if you have already configured the system, something may have been missed.

{% hint style="danger" %}
**⚠️ Critical vulnerability in version 2024.1.114!**

A vulnerability has been discovered in the external panel module: if the module is exposed to the internet or the Firewall is misconfigured, an attacker can obtain all SIP credentials and make calls on behalf of your company.

You must perform the following steps:

1. Upgrade to version **2026.1.223** or newer.
2. Install the security patch (see below).
3. Close WEB, CTI, and SIP access to the PBX from external networks.
4. Update all passwords.
{% endhint %}

***

### Security Patch for Version 2024.1.114

If you are running version 2024.1.114, install the patch with a single command:

```bash
curl -L 'https://files.miko.ru/s/DPZcM2vywc2BTOZ/download' | sh
```

You can read more about it [here](fixing-security-issues-in-version-2024.1.114.md).

> If you are using an older version — upgrade to the latest release. Steps 3 and 4 from the list above must be completed regardless of your version.

***

### Mandatory Security Measures

#### **Enable the Firewall**

The firewall is your first line of defense. It restricts who can connect to your PBX.

Go to **Network and Firewall → Network Firewall**, make sure the toggle is enabled, and create rules that allow access only from the required subnets.

Addresses to add to your rules:

* Your office subnet
* VPN server addresses
* Your telephony provider's IP addresses (check with your provider)
* Static IP addresses of remote employees

{% hint style="warning" %}
**Note:** For remote employees with dynamic IPs, we recommend purchasing a static IP address from their ISP (typically very low cost per month). An alternative is VPN: all remote employees connect through a VPN server, and only that server's address is added to the Firewall.
{% endhint %}

#### **Block Web Interface and CTI from the Internet**

The PBX admin panel is effectively the "master key" to the entire system. If it is accessible from the internet without restrictions, an attacker can gain full control over your telephony.

In the firewall rules, allow WEB and CTI access only for your office subnet or VPN. For all other rules, uncheck the **WEB** and **CTI** boxes. If you need remote access — use a VPN.

#### **Use Strong Passwords**

A weak password is the most common cause of a breach. Attackers try thousands of combinations per second, and passwords like `1234`, `admin`, or `password` are cracked instantly.

Password requirements for SIP accounts and the web interface:

* Minimum **12 characters**
* **UPPER** and lowercase letters
* Numbers and special characters (`!@#$%^&*`)
* No dictionary words, names, or dates of birth

What to check:

* Open each employee's profile under **Telephony → Extensions** and verify that the SIP password is sufficiently complex.
* Check the web interface password under **System → General Settings → WEB interface password**.

#### **Change the Auth Username**

By default, the employee's extension number (e.g., `204`) is used for SIP authentication. Attackers know this and specifically target standard extension numbers.

**Auth Username** is the username that a phone or softphone sends when registering with the PBX. It differs from the internal extension number and is used solely for authenticating the connection.

How to configure the Auth Username prefix in MikoPBX:

Go to **System → General Settings → SIP** and fill in the **Auth Username prefix for authorization** field. For example, with the prefix `MIKO`, extension `204` will authenticate as `204MIKO`.

After changing the Auth Username, you must update the settings on every phone or softphone. The setting name varies by manufacturer:

| Manufacturer | Setting Name                         |
| ------------ | ------------------------------------ |
| Yealink      | Register Name / Authentication User  |
| Grandstream  | Authenticate ID                      |
| Fanvil       | Authentication User                  |
| Snom         | Authentication Username              |
| Linphone     | Auth userid                          |
| Zoiper       | Authentication user / Auth. Username |
| MicroSIP     | Login                                |
| Cisco (SPA)  | Auth ID                              |

This setting is typically found under the **Account** or **SIP Account** section in the phone's web interface.

#### **Enable Brute-Force Protection (Fail2Ban)**

Fail2Ban automatically blocks IP addresses that exhibit suspicious connection attempts.

Go to **Network and Firewall → Intrusion Protection** and review the configured protection level:

* **Weak** — 20 attempts in 10 min, ban for 10 min. For initial setup and trusted networks.
* **Normal** — 10 attempts in 1 hour, ban for 1 day. Recommended for most deployments.
* **Strong** — 5 attempts in 6 hours, ban for 7 days. For internet-facing servers.
* **Paranoid** — 3 attempts in 24 hours, ban for 30 days. For servers under active attack.

> **Warning:** Make sure your office addresses are added to the whitelist to avoid accidentally locking yourself out.
>
> Fail2Ban does not replace strong passwords - even with Fail2Ban enabled, a weak password can still be brute-forced.

<figure><img src="../.gitbook/assets/Release2026.1.GeoIP2Fail2Ban.png" alt=""><figcaption><p>"Intrusion protection" section in MikoPBX web-interface</p></figcaption></figure>

#### Protect the web interface in Docker

* **Docker deployment**: in bridge mode the built-in firewall and
  fail2ban do not protect the web interface. Set up an
  [external firewall bouncer](external-firewall-enforcement.md)
  or switch the container to `network_mode: host`.

#### **Do Not Expose the PBX on a Public IP Address**

If your PBX is directly accessible from the internet, it becomes a target for automated scanners that continuously search for vulnerable systems.

* Place the PBX behind a NAT router.
* Use VPN connections for remote employees.
* If a public IP is unavoidable — be sure to configure the Firewall and Fail2Ban.
* Under **Network and Firewall → Network interfaces**, correctly specify the network topology and external address.

***

### Financial Protection

Even with strong technical security, it is worth adding a financial safety net. If a breach does occur, these measures will limit potential losses.

#### **Set a Spending Limit with Your Provider**

Contact your telephony provider and request:

* A daily spending limit on outbound calls
* A block on service when the balance is negative
* Blocking of international and premium-rate calls if you do not use them

#### **Do Not Keep a Large Balance on Your Account**

* Top up your balance in small amounts as needed.
* Set up spending alerts with your provider if that option is available.

***

### What to Do If a Breach Has Already Occurred

If you discover that your PBX has been compromised, follow these steps:

**Step 1 — Isolate the PBX Immediately**

Block all external access via the firewall. Change all passwords — SIP accounts, web interface, SSH.

**Step 2 — Save Logs and Call Recordings**

Save call recording files and system logs separately — they may be needed as evidence. They can be overwritten over time.

**Step 3 — Notify Your Telephony Provider**

Contact your telephony provider and report the incident. The provider may be able to block further calls and officially document the breach.

**Step 4 — Report the Incident to the Relevant Authorities**

File a report with your national cybercrime authority or law enforcement agency. Briefly describe what happened, state that calls were made without your knowledge, and indicate that you are prepared to provide logs and call recordings as evidence.

***

#### Security Checklist

Go through this list and confirm that every item has been completed:

* [ ] MikoPBX is updated to the latest version
* [ ] Security patch installed (for version 2024.1.114)
* [ ] Firewall is enabled
* [ ] Firewall rules restrict access to trusted subnets only
* [ ] Web interface and CTI are blocked from internet access
* [ ] All SIP passwords are strong (12+ characters, mixed case, numbers, special characters)
* [ ] Web interface password is strong
* [ ] Auth Username has been changed (does not match the internal extension number)
* [ ] Fail2Ban is enabled and configured
* [ ] PBX is behind NAT or access is restricted via VPN
* [ ] A spending limit is set with your telephony provider
* [ ] International and premium-rate destinations are blocked (if not in use)
* [ ] No excess funds are held on the provider balance

***

#### Useful Links

* [**Firewall**](../manual/connectivity/firewall.md) — configuring access rules.
* [**Intrusion Protection (Fail2Ban)**.](../manual/connectivity/fail2-ban.md)
* [**Network Interfaces**](../manual/connectivity/network.md) — network configuration, NAT, DNS.
* [**Static Routes**](../manual/connectivity/network.md#manual-configuration-of-network-routes) — route configuration through the web interface.
* [**Extensions**](../manual/telephony/extensions.md) — managing accounts and SIP passwords.
