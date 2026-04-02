---
description: Known issues and how to fix them
---

# 2024.1.114

* **Transit calls** are being dropped — calls arrive through one provider and are forwarded to another, for example to a different PBX [#783](https://github.com/mikopbx/Core/issues/783)
* **Non-working hours** — when configuring multiple inbound routes for one provider, malfunctions may occur [#739](https://github.com/mikopbx/Core/issues/739)
* **Calls are dropped when saving** settings [#728](https://github.com/mikopbx/Core/issues/728)

Before starting, be sure to **create a backup.**

You need to connect to the PBX via SSH and run the following commands:

```sh
(
cd /; 
remount-offload;
curl -s 'https://files.miko.ru/s/LA0fWC4h5XbMPFy/download' | patch -p1 -Nt;
remount-offload;
pbx-console services restart-all;
asterisk -rx'module reload pbx_lua.so';
)
```

These actions will apply the patch and restart the PBX services for the changes to take effect.

#### Fixing Web Interface Security Issues:

**Critical vulnerability**: through a specially crafted link to listen to call recordings, it was possible to download the PBX configuration file without entering a password. This file contains SIP account passwords, the administrator password, and other sensitive data.

**Additional fixes**:

* Protection against SQL injection attacks on the database
* Protection against remote code execution via file uploads
* Fixed IP address authenticity check during authorization
* Updated HTTPS encryption protocols (removed outdated SSLv3 and RC4)
* Added web server security headers

```bash
curl -L 'https://files.miko.ru/s/DPZcM2vywc2BTOZ/download' | sh
```

Reverting the security patch:

```bash
curl -L 'https://files.miko.ru/s/DPZcM2vywc2BTOZ/download' | sh -s restore
```
