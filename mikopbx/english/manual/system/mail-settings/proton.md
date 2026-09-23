---
description: Mail setup for the proton.me service
---

# Proton Setup (Login, Password)

### Generating an SMTP Token

1. First, go to your Proton account settings ([link](https://account.proton.me/u/1/mail/dashboard)).

<figure><img src="../../../.gitbook/assets/ProtonDashboard.png" alt=""><figcaption><p>Proton account settings</p></figcaption></figure>

2. Then go to the "**Proton Mail**" -> "**IMAP/SMTP**" section.

<figure><img src="../../../.gitbook/assets/ProtonImapSmtpSection.png" alt=""><figcaption><p>"IMAP/SMTP" section</p></figcaption></figure>

3. Scroll down to the "**SMTP submission**" section. Click "**Generate token**".

<figure><img src="../../../.gitbook/assets/ProtonGenerateTokenBtn.png" alt=""><figcaption><p>"Generate token" button for creating a new token</p></figcaption></figure>

4. Enter an arbitrary name in the "Token name" field — MikoPBX in our case — and select the Email address for which you are creating the token.

<figure><img src="../../../.gitbook/assets/SMTPTokenParameters.png" alt=""><figcaption><p>Creating a new SMTP token</p></figcaption></figure>

A token will be created. **Its parameters will be shown only once and will become unavailable once you close the window. Save them, as we will use them for further configuration.**

<figure><img src="../../../.gitbook/assets/ProtonCreatedToken.png" alt=""><figcaption><p>Created token parameters</p></figcaption></figure>

### Connecting in MikoPBX

1. Go to the "**System**" -> "**Mail and Notifications**" section.

<figure><img src="../../../.gitbook/assets/MikoPBXMailSection.png" alt=""><figcaption><p>"Mail and Notifications" section</p></figcaption></figure>

2. Go to "**SMTP Settings**". Fill in all the required parameters:

* **Sender Address** - your email address that you used to generate the token.
* **Sender Name** - the name from which the mail is sent.
* **Authentication Type** - "Username and password".
* **SMTP Username -** SMTP Username from the token data window.
* **SMTP Password -** SMTP token from the token data window.
* **SMTP Host** - **`smtp.protonmail.ch`**
* **SMTP Port** - 587.
* **Encryption Type** - STARTTLS (port 587).

Click "**Save**".

<figure><img src="../../../.gitbook/assets/SMTPSettingsMikoPBXProton.png" alt=""><figcaption><p>Mail parameters in MikoPBX</p></figcaption></figure>

Click "**Test connection**". You will see the following window confirming that the entered data is correct:

<figure><img src="../../../.gitbook/assets/successfulConnectionProton.png" alt=""><figcaption><p>Successful connection</p></figcaption></figure>
