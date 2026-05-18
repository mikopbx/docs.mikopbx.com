---
description: >-
  Automatic configuration of IP phones in MikoPBX. The module discovers phones
  on the local network, sends them a ready-to-use SIP configuration, and can
  even host firmware images for centralised upgrades.
---

# Automatic phone setup

## What the module does

The **Automatic phone setup** module (Autoprovisioning Plug & Play, PnP) takes the manual work out of configuring desktop phones. Instead of opening each phone's web interface, you bind a MAC address to an extension once — the next time the phone boots it pulls its SIP account, dial plan, BLF keys and (optionally) a firmware update from MikoPBX automatically.

**Supported brands:** Yealink, Snom, Fanvil, Grandstream, Htek.

Tested handsets include Yealink T19/T28/T46S/T48S/W52/WP530, Snom D120/D335/D385/D715/D735/D785, Fanvil X1SP/X3SP/X5U, Grandstream GXP and GRP series, Htek UC9xx. Other models from the same vendors usually work — if a particular handset needs a tweak, edit its template (see *Settings templates* below).

## Network requirements

* The module works **only inside the local network**.
* The network must pass **multicast traffic to `224.0.1.75:5060`** — that is how phones announce themselves.
* Phones must reach the PBX on **SIP `5060`** and on the **provisioning HTTP port** (default `8480`).
* If you turn on the TFTP channel, **UDP/69** must also be reachable.
* The firewall rules for the provisioning HTTP port and TFTP are opened **automatically** when you enable the module.
* No other PnP server should be running on the same LAN — phones accept settings from the first server that answers.

## Installing the module

1. Open **Modules → Module Marketplace**.
2. Install **Automatic Phone Setup Module** and reload the page.
3. Open the module page and configure each tab **before turning the module on**. The provisioning worker starts as soon as the toggle goes green.

<figure><img src="../../.gitbook/assets/settings.jpg" alt=""><figcaption></figcaption></figure>

## Module tabs

The module page has six tabs. Walk through them in order.

### 1. Phone settings

The MAC-to-extension mapping table. Each row binds one phone (by MAC address) to one MikoPBX user and one configuration template.

| Field | Purpose |
| --- | --- |
| **User** | Pick the internal MikoPBX user from the dropdown. |
| **MAC** | Phone's MAC address. Any case, separators are tolerated. |
| **Template** | One of the templates from the *Settings templates* tab. |

Phones that come up on the LAN by themselves (PnP discovery) appear in this table automatically — you only need to assign them a user and a template.

### 2. Settings templates

The text of the configuration file that the phone receives. Click **Add new** for a blank template, or **Load examples** to seed five vendor-specific examples (Yealink, Fanvil, Snom, Grandstream, Htek). Edit a template to change wording, add codecs, tweak DTMF, set the time zone — anything the vendor's config syntax supports.

Templates use placeholders that are replaced at delivery time:

| Placeholder | Replaced with |
| --- | --- |
| `{SIP_NUM}` | Extension number of the bound user. |
| `{SIP_USER_NAME}` | SIP authentication username. |
| `{SIP_PASS}` | SIP password. |
| `{PBX_HOST}` | Value of *Server Registration Address* from the PnP tab. |
| `{FIRMWARE_URL}` | Download URL of the firmware row that matches the vendor (and model, if specified). Empty when no firmware is registered. |

### 3. URI Settings

Some phones ask the PBX for a configuration file at a fixed URL (for example `y000000000028.cfg` for older Yealink firmwares). This tab lets you map an arbitrary URL to a template — when the phone requests that URL, MikoPBX answers with the matching template.

### 4. Phone book

External phonebook sources. Each row lists another MikoPBX whose user directory should be merged into the phonebook delivered to your phones.

The phonebook is currently delivered in **Yealink format** (works on Yealink and most Yealink-compatible firmwares) and **Grandstream format**. Native phonebook formats for Fanvil, Snom and Htek are planned.

> Each external PBX must allow incoming requests on its provisioning HTTP port.

### 5. PnP Settings

Top-level options for the module.

* **Extension Template** — the dial pattern users dial to self-bind a phone. The installer seeds something like `*123*200` (a free feature code plus a typical extension length). Edit to taste.
* **Server Registration Address** — the IP address or DNS name of this PBX. Phones use this as their SIP server and to fetch configuration files. Use an address that is reachable from every phone's network.
* **Provisioning HTTP port** — TCP port on which the module serves configuration files. Default `8480`. This is a dedicated port that bypasses the global HTTPS redirect, so it works even on PBXes that force HTTPS on the admin UI.
* **Blacklist of MAC Addresses** — MAC addresses that will be ignored. **Blacklist wins over whitelist.**
* **Whitelist of MAC Addresses** — MAC addresses allowed to auto-provision. If both lists are empty, every phone on the LAN is served.
* **Additional Parameters** — extra vendor-specific lines that are merged into the generated configuration. See *Additional Parameters* below.
* **Enable TFTP provisioning (UDP/69)** — turns on the built-in TFTP server. Phones that prefer DHCP option 66 over multicast PnP (typical for Snom and some Fanvil firmwares) pull their configuration via TFTP instead of HTTP. Use it on a trusted LAN only — TFTP traffic is unencrypted.

### 6. Firmware

A built-in repository for phone firmware files. Drag a firmware blob onto the upload area, fill in vendor and model, save — MikoPBX hosts the file and exposes it through the `{FIRMWARE_URL}` placeholder.

| Field | Notes |
| --- | --- |
| **Vendor** | `yealink`, `snom`, `fanvil`, `grandstream` or `htek`. |
| **Model** | Free-form (for example `T46S`). Leave empty for a vendor-wide fallback. |
| **Version** | Free-form (for example `66.86.0.15`). |
| **Notes** | Admin comment. |
| **File** | The firmware blob itself. |

**Limits:** up to **80 MB per file**, **300 MB total** across all rows.

**Allowed file extensions per vendor:**

| Vendor | Extensions |
| --- | --- |
| Yealink | `.rom`, `.bin` |
| Snom | `.bin`, `.fw` |
| Fanvil | `.z`, `.bin` |
| Grandstream | `.bin` |
| Htek | `.rom`, `.bin` |

Each row has Edit and Delete actions; editing lets you upload a replacement firmware in place.

When a phone fetches its configuration the module picks the firmware row matching its vendor and model. If no model-specific row exists, the vendor-wide fallback (model field empty) is used. The result lands in the template via `{FIRMWARE_URL}`. Vendor generators also emit a vendor-native line automatically:

* **Yealink** — `firmware.url = …`
* **Snom** — `<firmware perm="RW">…</firmware>`
* **Fanvil** — `App URL : …`
* **Grandstream** — `<P192>…</P192>` plus `<P237>1</P237>` (forces a firmware check on every reboot)
* **Htek** — `auto_image_url = …`

## Additional Parameters

The **Additional Parameters** field on the *PnP Settings* tab accepts vendor-specific lines, grouped into INI-style sections. Each section is merged into the corresponding vendor's generated config.

### Yealink

```ini
[yealink]
features.headset_prior = 1
features.intercom.allow = 1
```

### Snom

Use `[snom]` for lines that go inside the root `<settings>` node, and `[snom-phone-settings]` for lines inside `<phone-settings>`.

```ini
[snom]
time_24_format = on

[snom-phone-settings]
dialplan_active = on
```

### Fanvil

Fanvil splits its configuration into named modules. Extend them with `[fanvil]`, `[fanvil-sip]`, `[fanvil-tele]` or `[fanvil-autoupdate]`.

```ini
[fanvil-sip]
SIP1 Default DTMF = AUTO
```

### Grandstream

Grandstream uses XML P-codes. Add or override any code via `[grandstream]`:

```ini
[grandstream]
<P3>caller_id</P3>
```

## Star-code self-binding

When the module is enabled, the dial pattern from *Extension Template* (for example `*123*200`) becomes a working extension. Dial it from a fresh phone:

1. The phone places the call to that pattern.
2. MikoPBX looks up the caller's IP, resolves the MAC address, and adds the phone to *Phone settings*.
3. The phone is rebooted automatically and pulls its fresh configuration on next boot.

The user is then bound to the phone — no admin action required.

## Troubleshooting

### The phone does not pick up its configuration

1. **Check the address.** Open the phone's web interface and confirm it is using your PBX IP as the provisioning server. Wrong subnet or DHCP option 66 pointing elsewhere are the two most common reasons.
2. **Check the port.** From the same network as the phone, open `http://<pbx-ip>:8480/` in a browser. You should not see "connection refused". If you do, the provisioning port is closed — recheck firewall rules and that the module toggle is on.
3. **Check the MAC list.** If the phone's MAC is on the blacklist (or the whitelist exists and does not include it), the module refuses to serve it.
4. **Reboot the phone.** Some firmwares only request a new config on power cycle.

### Yealink — read the phone log

1. Open the phone's web UI → **Settings → Configuration**.
2. Set log level to **6**.
3. Reboot and export the log.

Look for `Connect Error` lines:

```text
LIBD[528]: HTTP<5+notice> URL :
LIBD[528]: HTTP<3+error > Connect Error
AUTP[528]: AUTP<3+error > http to file failed, code = -3, msg = Connect Failed
```

If you see `Connect Error`, the phone could not reach the PBX over HTTP — recheck *Server Registration Address* and the provisioning HTTP port.

### Detailed PnP logging

To see every PnP packet the module receives, paste this into **Additional Parameters**:

```ini
[debug]
verbose = 1
```

The module then logs every multicast packet, the matched MAC, and whether the whitelist / blacklist allowed it. Turn the flag off in normal operation — the output is noisy.

### Firmware

* The phone fails to upgrade. Confirm that the `{FIRMWARE_URL}` placeholder ended up in the actual config the phone received (download the file from `http://<pbx-ip>:8480/...` in a browser). If it is empty, you forgot to upload a firmware row for that vendor.
* The upload form rejects the file. Check the extension list above and the per-file/total quotas.

### TFTP

If you turned on the TFTP channel and a phone still cannot fetch its config, double-check that DHCP option 66 (TFTP server name) on your router points at the PBX IP, and that UDP/69 is not blocked between the phone's network and the PBX.

## Reference: what the module generates by default

The snippets below show the full body of the configuration file the module produces for an empty template (no overrides, no *Additional Parameters*). Use them as a reference when comparing what your phone actually received — download the file in a browser from `http://<pbx-ip>:8480/...` and diff against the relevant example.

### Yealink

```ini
#!version:1.0.0.1
account.1.enable = 1
account.1.label = Askozia (204)
account.1.display_name = 204
account.1.auth_name = 204
account.1.user_name = 204
account.1.password = 1c9709222690713dd
account.1.sip_server_host = 172.16.156.223
account.1.sip_server_port = 5060
account.1.transport = 0
account.1.codec.1.enable = 1
account.1.codec.1.payload_type = PCMU
account.1.codec.1.priority = 1
account.1.codec.1.rtpmap = 0
account.1.cid_source = 4
voice_mail.number.1 = *001
phone_setting.lcd_logo.mode = 0
auto_provision.dhcp_option.enable = 0
features.intercom.allow = 1
features.intercom.mute = 0
features.intercom.tone = 1
features.intercom.barge = 1
features.dtmf.transfer = ##
features.dtmf.replace_tran = 1
features.headset_prior = 1
```

### Snom

```xml
<?xml version="1.0" encoding="utf-8"?>
<settings>
    <time_24_format perm="R">off</time_24_format>
    <phone-settings>
        <user_pname idx="1" perm="RW">203</user_pname>
        <user_name idx="1" perm="RW">203</user_name>
        <user_realname idx="1" perm="RW">Irina Smirnova</user_realname>
        <user_pass idx="1" perm="RW">3256157a99f176eb959ef9c1fdd947f0</user_pass>
        <user_host idx="1" perm="RW">172.16.32.225</user_host>
        <user_srtp idx="1" perm="RW">off</user_srtp>
        <user_mailbox idx="1" perm="RW">*001</user_mailbox>
    </phone-settings>
</settings>
```

### Fanvil

```text
<<VOIP CONFIG FILE>>Version:2.0002
PNP Enable         :0
<SIP CONFIG MODULE>
SIP1 Enable Reg    :1
SIP1 Phone Number  :203
SIP1 Display Name  :Irina Smirnova
SIP1 Register Addr :172.16.156.223
SIP1 Register Port :5060
SIP1 Register User :203
SIP1 Register Pswd :3256157a99f176eb959ef9c1fdd947f0
```
