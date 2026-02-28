---
description: Gmail service mail configuration
---

# Gmail Setup (oAuth2)

{% hint style="info" %}
Setting up OAuth 2.0 in Google requires using the **station's URL address**.\
The easiest way is to create a DNS record on the local server **or** add an IP address-to-domain name mapping in the `hosts` file on the device from which the configuration is being performed.
{% endhint %}

## Google Account Settings

1. Before starting the setup, you need to change some Google account parameters. To do this, go to the account management page ([link](https://myaccount.google.com/)).

<figure><img src="../../../.gitbook/assets/myaccountgooglecomUpd.png" alt=""><figcaption><p>Google Account management page</p></figcaption></figure>

2. Go to the "**Security and sign-in**" section. Make sure that two-step authentication is configured.

<figure><img src="../../../.gitbook/assets/2-stepverif_google.png" alt=""><figcaption><p>Two-step authentication setup</p></figcaption></figure>

3. Go to the Google Cloud Console, to the "**APIs & Services**" section ([link](https://console.cloud.google.com/apis/dashboard)). Create a project for the current task.

<figure><img src="../../../.gitbook/assets/googleCloudAPIs.png" alt=""><figcaption><p>"APIs &#x26; Services" section in Google Cloud</p></figcaption></figure>

4. Go to the APIs library (the "**Library**" section).

<figure><img src="../../../.gitbook/assets/googleCloudAPIsLibrary.png" alt=""><figcaption><p>"Library" section in APIs &#x26; Services</p></figcaption></figure>

5. Enter "gmail api" in the search bar. Open the Gmail API card.

<figure><img src="../../../.gitbook/assets/googleCloudGmailAPI.png" alt=""><figcaption><p>Gmail API in the Google Cloud library</p></figcaption></figure>

6. Click "**Enable**" to connect.

<figure><img src="../../../.gitbook/assets/googleCloudEnableGmailAPI.png" alt=""><figcaption><p>Enabling the API</p></figcaption></figure>

7. Go to the main **APIs & Services** page. Then click "**OAuth consent screen**".

<figure><img src="../../../.gitbook/assets/googleCloudOAuthConsentScreen.png" alt=""><figcaption><p>"OAuth consent screen" section in APIs &#x26; Services</p></figcaption></figure>

8. Create a project (click "**Get started**"). Fill in an arbitrary name and your email. Select "**Internal**" as the Audience. Click "**Create**" to finish.

<figure><img src="../../../.gitbook/assets/googleCloudAudienceInternal.png" alt=""><figcaption><p>"Audience" parameter in project creation process</p></figcaption></figure>

9. Return to the main **APIs & Services** page. Then go to the "**Credentials**" section. Click "**Create credentials**". Select "OAuth client ID" to create.

<figure><img src="../../../.gitbook/assets/googleCloudnewOAuthCredentials.png" alt=""><figcaption><p>Creating a new OAuth client ID</p></figcaption></figure>

10. Select "**Web application**" as the Application type. Then enter an arbitrary name. Click "**Create**".

<figure><img src="../../../.gitbook/assets/creatingWebApp.png" alt=""><figcaption><p>Creating a new OAuth client ID</p></figcaption></figure>

11. Add a new "**Authorized redirect URI**".

{% hint style="info" %}
Format:

<mark style="color:blue;">`https://mikopbx.station.com/pbxcore/api/v3/mail-settings/oauth2-callback`</mark>

Replace "mikopbx.station.com" with your station's URL.
{% endhint %}

<figure><img src="../../../.gitbook/assets/addingNewRedirectURl.png" alt=""><figcaption><p>Adding a new redirect URL</p></figcaption></figure>

12. An OAuth client will be created. Save the Client ID and Client Secret to your notes. You will need this data for the connection in the future.

<figure><img src="../../../.gitbook/assets/OAuthclientCreated.png" alt=""><figcaption><p>Successfully created client</p></figcaption></figure>

## Settings in MikoPBX

1. Go to the "**System**" -> "**Mail and notifications**" section:

<figure><img src="../../../.gitbook/assets/MikoPBXMailSection.png" alt=""><figcaption><p>"Mail and Notifications" section in MikoPBX</p></figcaption></figure>

2. Next, go to "SMTP Settings". Fill in the following parameters:

* **Sender address, Sender name** — your email and the name from which the emails will be sent.
* **Authentication type** — OAuth2.
* **SMTP login** — your email.
* **OAuth2 Provider** — Google/Gmail.
* **Application ID (Client ID), Secret key (Client Secret)** — the data saved from Google Cloud (step 12 from the previous section of this guide).

Leave all other settings at their default values. A more detailed description can be found in the main article about mail parameters ([link](./)).

After that, click "**Save**"!

<figure><img src="../../../.gitbook/assets/SMTPParametersGmailOAuth2ast.png" alt=""><figcaption><p>Mail parameters for connecting Gmail</p></figcaption></figure>

3. Click the blue "**Connect via OAuth2**" button. Then select your Gmail account.

<figure><img src="../../../.gitbook/assets/GoogleOauthStep1.png" alt="" width="375"><figcaption><p>Selecting a Google account</p></figcaption></figure>

4. Confirm the sign-in: click "**Continue**".

<figure><img src="../../../.gitbook/assets/GoogleOauthStep2.png" alt="" width="375"><figcaption><p>Continuing authorization</p></figcaption></figure>

5. Confirm granting the required permissions (click "**Allow**").

<figure><img src="../../../.gitbook/assets/GoogleOauthStep3.png" alt="" width="375"><figcaption><p>Granting permissions</p></figcaption></figure>

Upon successful authorization, you will see the following window.

<mark style="color:$success;">TO DO (INSERT IMG)</mark>

#### Troubleshooting

Access blocked: Authorization Error (\*\*Error 400: invalid\_request)

<figure><img src="../../../.gitbook/assets/GoogleInvalidRequest.png" alt="" width="375"><figcaption><p>Error 400: invalid_request</p></figcaption></figure>

Solution: enter the station's URL address in the MikoPBX web interface: "**Network and Firewall**" -> "**Network Interfaces**". Go to the "Network Topology" section and enter the hostname in the "**External hostname of your router**" field. (Enable "**This station is located behind a NAT router**".)

<figure><img src="../../../.gitbook/assets/GoogleInvalidRequestSolution.png" alt=""><figcaption><p>Problem solution</p></figcaption></figure>
