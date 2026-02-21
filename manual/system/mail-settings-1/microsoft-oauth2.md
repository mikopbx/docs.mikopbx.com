---
description: Mail setup for the Outlook service (outlook.com; hotmail.com)
---

# Microsoft Outlook Setup (OAuth2)

## Settings in Microsoft Entra

### Application Registration

1. Sign in to the [Microsoft Entra admin center.](https://entra.microsoft.com/)

<figure><img src="../../../.gitbook/assets/MicrosoftEntraDeshboard.png" alt=""><figcaption><p>Microsoft Entra admin center home page</p></figcaption></figure>

2. Go to "**Entra ID**" -> "**App registrations**". Then click "**New registration**" to register a new application.

<figure><img src="../../../.gitbook/assets/MicrosoftEntraNewAppRegistration.png" alt=""><figcaption><p>Registering a new application</p></figcaption></figure>

3. Select the following parameters for your application:

* **Name** - enter a name for your application.
* **Supported account types** - select "**Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**".

<figure><img src="../../../.gitbook/assets/ApplicationNameAccTypes.png" alt=""><figcaption><p>Application parameters</p></figcaption></figure>

4. Specify the Redirect URL:

* **Select a platform** — select "**Web**".
* **URL**:

```
https://192.168.100.71/pbxcore/api/v3/mail-settings/oauth2-callback
```

Replace 192.168.100.71 with your MikoPBX address.

Then click "**Register**".

<figure><img src="../../../.gitbook/assets/MicrosoftEntraRedirectURl.png" alt=""><figcaption><p>Redirect URL parameters</p></figcaption></figure>

5. The application will be created. Save the Client ID — you will need it in the future for configuration inside the MikoPBX web interface.

<figure><img src="../../../.gitbook/assets/CreatedApplicationOverview.png" alt=""><figcaption><p>Created application overview page</p></figcaption></figure>

### Granting Permissions and Creating a Client Secret

1. From the application's main page, go to "**Manage**" -> "**API permissions**".

<figure><img src="../../../.gitbook/assets/MicrosoftEntraAPIpermissions.png" alt=""><figcaption><p>"API permissions" section</p></figcaption></figure>

2. Click "**Add a permission**".

<figure><img src="../../../.gitbook/assets/MicrosoftEntraAddPermission.png" alt=""><figcaption><p>Adding permissions</p></figcaption></figure>

3. In the "**Microsoft Graph**" section, select "**Delegated Permissions**". Enter "**SMTP**" in the search bar. Check the box next to "**SMTP.Send**".

<figure><img src="../../../.gitbook/assets/APISmtp.Send.png" alt=""><figcaption><p>Granting the "SMTP.Send" permission</p></figcaption></figure>

4. Also enter "**offline**" in the search bar. Check the box next to "**offline\_access**".

Click "**Add permissions**".

<figure><img src="../../../.gitbook/assets/APIoffline_access.png" alt=""><figcaption><p>Granting the "offline_access" permission</p></figcaption></figure>

5. Next, go to "**Certificates & secrets**" -> "**Client secrets**". Click "**New client secret**".

<figure><img src="../../../.gitbook/assets/creatingNewClientSecret.png" alt=""><figcaption><p>Creating a new Secret ID</p></figcaption></figure>

6. Set the required parameters:

* **Description** - an arbitrary description.
* **Expires** - the duration for which you are issuing this client secret. It will be needed for application authentication in MikoPBX.

{% hint style="info" %}
After expiration, the created client secret will stop functioning and you will need to repeat the process of creating a new key and connecting to MikoPBX.
{% endhint %}

{% hint style="danger" %}
After creation, the Client Secret value will be shown only once. Do not forget to copy it into the MikoPBX web interface.
{% endhint %}

Click "**Add**".

<figure><img src="../../../.gitbook/assets/newClientSecret.png" alt=""><figcaption><p>Parameters for creating a new client secret</p></figcaption></figure>

7. Copy the "**Value**" (<mark style="color:$danger;">**not the Secret ID!**</mark>). It will be needed for configuration in the MikoPBX web interface.

<figure><img src="../../../.gitbook/assets/copyingSecretKeyValue.png" alt=""><figcaption><p>Copying the Value of the previously created Client Secret</p></figcaption></figure>

#### Granting Permissions to a User

For the application to work correctly, you need to grant permission to use the SMTP protocol for the user whose mailbox you are authorizing during this setup. To do this, follow these steps:

1. Go to the organization's admin center ([link](https://admin.microsoft.com/)).

<figure><img src="../../../.gitbook/assets/MicrosoftAdminCenterDashboard.png" alt=""><figcaption><p>Microsoft Admin Center home page</p></figcaption></figure>

2. Go to "**Users**" -> "**Active Users**". Click on the name of the user account under which the application is being created.

<figure><img src="../../../.gitbook/assets/MicrosoftAdminCenterActiveUsers.png" alt=""><figcaption><p>"Active Users" section in Microsoft Admin Center</p></figcaption></figure>

3. In the account, go to the "**Mail**" section and select "**Manage email apps**".

<figure><img src="../../../.gitbook/assets/MicrosoftAdminCenterUserMail.png" alt=""><figcaption><p>"Mail" section in the user account</p></figcaption></figure>

4. Make sure that "**Authenticated SMTP**" is allowed. Save the changes by clicking "**Save changes**".

<figure><img src="../../../.gitbook/assets/AllowAuthSMTP.png" alt=""><figcaption><p>Allowing Authenticated SMTP for the selected user</p></figcaption></figure>

### Settings in MikoPBX

1. Go to the MikoPBX web interface. Then "**System**" -> "**Mail and Notifications**" -> "**SMTP Settings**".

Fill in all the required fields:

* **Sender address, Sender name** — your email and the name from which the emails will be sent.
* **Authentication type** — OAuth2.
* **SMTP login** — your email.
* **OAuth2 Provider** — Microsoft/Outlook.
* **Application ID (Client ID), Secret key (Client Secret)** — data from Microsoft Entra.

Leave all other settings at their default values. A more detailed description can be found in the main article about mail parameters ([link](https://docs.mikopbx.com/mikopbx/manual/system/mail-settings-1)).

After that, click "**Save**"!

<mark style="color:$success;">To do: paste screenshot.</mark>

2. Click "**Connect via OAuth2**". Sign in to your Microsoft account. Then confirm granting all requested permissions.

Upon successful authorization, you will see the corresponding window.

<mark style="color:$success;">To do: paste screenshot.</mark>
