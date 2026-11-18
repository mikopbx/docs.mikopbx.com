---
description: Changes since MikoPBX 2026.6.99
---

# MikoPBX 2026.8.2

The main focus of MikoPBX 2026.8.2 is security and fixes after 2026.6.99. Vulnerabilities in module access rights, provider settings, and DHCP response handling have been fixed. Forwarding to voicemail, Busy, and Hangup works again, modules can be installed from the marketplace again, and the click at the start of prompts and music on hold is gone.

{% hint style="warning" %}
Back up the PBX before updating. After the update, web interface users need to sign in again once. When updating from 2026.5.6, upload the IMG image manually. For PBXs installed before 2026.1.1, use the IMG-legacy image.
{% endhint %}

### Security

A user with a restricted role can no longer call module actions without permission. The module menu item form can no longer be used to change system settings or inject markup into the menu.

Values in provider settings and off-work calendar settings can no longer change the telephony configuration behind the interface. Routes received from a DHCP server are validated: a forged reply on the local network can no longer run commands on the PBX.

Each PBX now gets its own login session encryption key. Employee CSV import data, including phone passwords, is kept in a protected location and deleted after an hour. On internal errors, the interface shows a generic message without service details.

### Calls and sound

Forwarding to voicemail, Busy, Hangup, and DID to User works again. Previously, the call kept ringing the employee instead of being forwarded. These options are available again in the selectors of incoming routes, IVR menus, queues, and forwarding.

The click at the start of IVR menus, system prompts, and music on hold has been removed. Existing files are repaired automatically when the PBX starts.

Music on hold plays again on new installs. If you delete all your own tracks, the PBX uses the built-in set.

### Modules

Installing modules from the marketplace works again.

### Network filters and access

A network filter used by an employee, provider, AMI user, or API key can no longer be deleted. Previously, deleting the filter removed the address restriction from such records.

API keys with a network filter work from the permitted networks again. Previously, they were rejected from any address.

In LXC containers, banned addresses are now actually blocked from the web interface, and the whitelist applies right after a restart.

### Other fixes

New installs no longer try to send notifications through an unconfigured mail server. Disks and partitions are detected correctly even when there are more than nine of them or their names are similar.

### Update notes

If an employee, provider, AMI user, or API key was bound to a network filter that has already been deleted, after the update it can connect only from local addresses. Each such record is listed in the system log - assign the right filter to it.
