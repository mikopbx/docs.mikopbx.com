# Setting up E-mail notifications for the Gmail mail service

{% hint style="info" %}
Always use "Application Passwords" for authorization. [See instructions](https://support.google.com/accounts/answer/6010255). [Instructions](https://support.google.com/a/answer/9003945?hl=en) for setting up smtp from Gmail
{% endhint %}

To receive notifications about missed calls by email, you need to configure the SMTP client. Detailed information about notifications in MikoPBX is reviewed [here](./). As part of this instruction, an example of setting up missed call notifications for the Gmail mail service will be considered.

1. Enter the IP address of the MikoPBX PBX in the browser and go to **System → Mail Settings**

<figure><img src="../../../.gitbook/assets/1 (12).png" alt=""><figcaption></figcaption></figure>

SMTP Client Settings for Gmail service:

* **SMPT host** - `smtp.gmail.com`
* **SMPT port** - 465 (Customer Service port)
* **SMPT login** and **Sender address**- the E-mail from which messages about missed calls will be sent
* **SMPT Password** - the email password required for authorization
* **Encryption Method** - Use TLS

<figure><img src="../../../.gitbook/assets/2 (3).png" alt=""><figcaption></figcaption></figure>

2. Save the entered settings and proceed to setting up your email account. A feature of the Gmail service is that access to your account is automatically denied to untrusted applications, which include MikoPBX, so you need to manually allow access to these applications (setup instructions are posted [here](https://support.google.com/accounts/answer/6010255)).
3. Go back to **System** → **Mail Settings**. We will send a test letter to the e-mail of any service. In case of successful testing, a test email will be sent to the email address you specified.

You can read about how to set up a letter template to create an E-mail notification [here](./).
