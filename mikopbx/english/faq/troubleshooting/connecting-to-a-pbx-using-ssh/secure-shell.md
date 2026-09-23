---
description: SSH Connection using Google Secure Shell
---

# SSH Connection (Google Secure Shell)

{% hint style="info" %}
This method is available if you use Google Chrome for connection to the MikoPBX web interface.
{% endhint %}

### Enabling password access

1. Go to the "**System**" -> "**General Settings**" section.

<figure><img src="../../../.gitbook/assets/ENgeneralSettingsSectionMikoPBXWEB.png" alt=""><figcaption><p>"<strong>System</strong>" - "<strong>General settings</strong>" section</p></figcaption></figure>

2. Go to the "**SSH**" section. Toggle the "**Disable password logins**" switch (switch is enabled by default). Set your password for the SSH connection. **Save** the settings.

{% hint style="danger" %}
Do not use weak passwords! Always disable password-based SSH connections after you log out!
{% endhint %}

<figure><img src="../../../.gitbook/assets/ENSSHPasswordAuth.png" alt=""><figcaption><p>Password-based access</p></figcaption></figure>

### Connection using Google Secure Shell

1. In the MikoPBX Web-interface, go to "Maintenance" -> "SSH console".

<figure><img src="../../../.gitbook/assets/ENSSHConsoleSectionMikoPBXWEB.png" alt=""><figcaption><p>"Maintenance" -> "SSH Console" section</p></figcaption></figure>

2. If you don't have the "**Secure Shell**" browser extension installed, you will be redirected to the extension store. Click "**Add to Chrome**." After installation is complete, return to the web interface and repeat **Step 1**.

<figure><img src="../../../.gitbook/assets/ENgshExtension.png" alt=""><figcaption><p>Installing extension</p></figcaption></figure>

3. After redirecting to the Google Secure Shell tab, enter "**yes**" in the dialog box for a new connection. Press "**Enter**."

<figure><img src="../../../.gitbook/assets/newFingerprintSSHGoogle.png" alt=""><figcaption><p>Dialog box #1</p></figcaption></figure>

4. In the new dialog box, enter the previously set SSH password. Press "**Enter**."

<figure><img src="../../../.gitbook/assets/sshPasswordInput.png" alt=""><figcaption><p>Dialog box #2</p></figcaption></figure>

An SSH connection will be established. :tada:

<figure><img src="../../../.gitbook/assets/successfulSSHConnectionGSS.png" alt=""><figcaption><p>Successful SSH connection</p></figcaption></figure>
