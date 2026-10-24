---
description: Changes since MikoPBX 2026.5.6
---

# MikoPBX 2026.6.34

MikoPBX 2026.6.34 fixes call recording in complex transfer scenarios and makes employee call forwarding more reliable. Web interface and REST API protection has been strengthened, and Cloud.ru Evolution is now supported. Installation from LiveCD and downloading updates from the web interface work again.

### Call recordings and call history

If a transfer is rejected in a custom dialplan subroutine, for example with `Busy()` in `[internal-users-custom]`, the agent's recording resumes once the call returns. Previously, the part of the conversation after a failed transfer was lost.

A transfer answered via `*8` pickup is now stored in call history as an answered call with its own recording. Previously, such a conversation appeared as missed, and its recording file was left on disk unlinked from the call.

When recordings are played or downloaded as MP3, WAV, or OGG, the converted file is cached. Replaying and seeking no longer start a new conversion, and temporary copies no longer pile up on disk: the cache is cleaned automatically.

### Call forwarding and SIP

Employee call forwarding settings are applied more reliably. They are no longer lost if telephony was unavailable at the moment of saving, and a disabled forward stops working immediately, without restarting the PBX.

{% hint style="warning" %}
If a custom dialplan reads forwarding from the Asterisk database (`DB(FW...)`), it must be updated: the values are now stored in the global variables `FW_<number>`, `FW_BUSY_<number>`, `FW_UNAV_<number>`, and `FW_TIME_<number>`.
{% endhint %}

Two providers with inbound registration and the same login can no longer be saved, nor can a provider whose login matches an extension number. Previously, this mistake stopped all trunks and employees. Manual `set_var` parameters in advanced settings are applied again, including several lines in a row.

The TLS transport in the employee card is deprecated and is no longer offered for new employees. To connect over TLS, set the SIP client username to `number-TLS`, for example `105-TLS`, with the same password. Employees that already use TLS get a warning in their card and in notifications.

The PBX's external NAT address is no longer derived from its hostname. Previously, this could break calls behind NAT and TLS connections. The link in the weak SIP password warning opens the right employee card again.

### Security

The built-in attack protection now scans the whole request body, including `DELETE` requests. The scan runs after blocklist and rate limit checks and does not add load during brute-force attempts. REST API requests larger than 4 MB are rejected with a clear error.

Internal recording view links are no longer reachable from the network, and system log archive downloads are restricted to generated archives only. On Alibaba Cloud, a hostname containing the instance ID is no longer used: the ID serves as the initial password and must not appear on the login page.

### Cloud, storage, and updates

Automatic PBX setup in Cloud.ru Evolution has been added, including SSH keys from the virtual machine parameters.

Swap now uses a compressed area in RAM. The swap file on disk is used only when that area fills up, which reduces wear on flash storage.

Downloading updates from the web interface works again: previously, clicking the button did not start the download. Installation from LiveCD works again: the installer no longer exits on its own without waiting for disk selection and confirmation.

The system log keeps nine archived parts instead of two, so a busy PBX retains a longer history. After a factory reset, logs continue to be written.

### For developers

Module REST APIs return proper HTTP error codes instead of `200` for `result: false`. Empty JSON objects in responses are no longer turned into arrays. A timeout returns `504`, and an overloaded queue returns `503`.

Chunked file uploads send blocks of at most 10 MB. The module update notice appears again after the cache is cleared. Updating an API key's last-used time works on read-only installations, such as LXC.
