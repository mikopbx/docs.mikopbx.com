---
description: Changes since MikoPBX 2026.6.34
---

# MikoPBX 2026.6.99

The main focus of MikoPBX 2026.6.99 is security. Vulnerabilities that allowed commands to be run on the PBX and call recordings to be accessed without authentication have been fixed, along with Asterisk vulnerabilities in encrypted call handling. Linux, OpenSSL, Python, and network services have been updated.

Call history now handles transfers and pickups correctly, queues and providers work more reliably, and MP3 export of recordings is faster.

{% hint style="warning" %}
Back up the PBX before updating. When updating from 2026.5.6, upload the IMG image manually: the "Download" button does not work in that version. For PBXs installed before 2026.1.1, use the IMG-legacy image.
{% endhint %}

### Security

Vulnerabilities that allowed commands to be run on the PBX and call recordings to be accessed without authentication have been fixed. We recommend updating as soon as possible, especially if the PBX web interface is reachable from the internet.

Asterisk vulnerabilities in encrypted call handling and certificate validation have been fixed. Linux, OpenSSL, Python, and network services have been updated. Files, settings, and the web interface are better protected against malicious requests and content.

A list of system components and their versions has been added. Use it to check the PBX for known vulnerabilities.

### Call recordings and call history

Call history, duration, and recording links have been fixed for transfers, pickups, and forwarding configured on the phone itself. Such calls now appear correctly in call history, and each recording opens from the right call.

MP3 export of recordings is faster and produces smaller files: both speakers are mixed into one channel.

If a recording download from S3 storage is interrupted, the incomplete file no longer stays on the PBX or becomes available for playback.

When disk space runs low, the PBX first removes archived system logs and only then, if space is still short, deletes old call recordings.

### Queues and providers

After a call is transferred to a queue, it keeps ringing agents even if the transferring employee has already hung up. Previously, the call could drop in this case.

Incoming calls from some providers, including Novofon, have been fixed. Out-of-hours rules assigned to specific inbound routes are applied correctly again.

### Stability and maintenance

Module downloads have been fixed. Long names and email addresses are no longer truncated when importing employees.

Settings are read correctly even while another process is updating them. If a configuration check fails, the PBX no longer restores the configuration unnecessarily.

### Update notes

For PBXs in an LXC container, updating with the IMG image is no longer offered. Instead, the web interface shows instructions for recreating the container while keeping its data. See [Updating an LXC container](../../manual/maintenance/update/lxc.md).

The additional compressed in-memory swap (zram) introduced in 2026.6.34 has been removed. Swap works the same way as in earlier versions again.
