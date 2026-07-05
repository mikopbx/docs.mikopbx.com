---
description: Instructions for installing the patch that fixes security vulnerabilities in version 2024.1.114
---

# Fixing security issues in version 2024.1.114

{% hint style="danger" %}
We recommend updating your PBX to the latest available stable release.

You can find releases on GitHub [here](https://github.com/mikopbx/core/releases).
{% endhint %}

**Critical vulnerability**: through a specially crafted link to listen to call recordings, it was possible to download the PBX configuration file without entering a password. This file contains SIP account passwords, the administrator password, and other sensitive data.

**Patch fixes:**

* Protection against SQL injection attacks on the database
* Protection against remote code execution via file uploads
* Fixed IP address authenticity check during authorization
* Updated HTTPS encryption protocols (removed outdated SSLv3 and RC4)
* Added web server security headers

To install the patch, connect to your PBX using SSH (instructions are available [here](../faq/troubleshooting/connecting-to-a-pbx-using-ssh/)). Open the console and run the following command:

```bash
curl -L 'https://files.miko.ru/s/DPZcM2vywc2BTOZ/download' | sh
```

To revert the security patch, run the following command:

```bash
curl -L 'https://files.miko.ru/s/DPZcM2vywc2BTOZ/download' | sh -s restore
```
