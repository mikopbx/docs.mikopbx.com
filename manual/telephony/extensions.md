---
description: Setting Primary Phone Numbers
---

# Extensions

## Connecting softphones

* <mark style="color:red;">MicroSIP</mark>
* <mark style="color:red;">Jitsi</mark>
* [Zoiper](../../faq/softphones/zoiper.md)
* <mark style="color:red;">PhonerLite</mark>
* <mark style="color:red;">Телефон (Mac OS)</mark>
* <mark style="color:red;">WebRTC tutorial using SIPML5</mark>
* <mark style="color:red;">Настройка телеграмм как SIP софтфон</mark>

## Connecting telephones

* <mark style="color:red;">Yealink T19</mark>
* <mark style="color:red;">Yealink T21</mark>
* <mark style="color:red;">Yealink T28</mark>
* <mark style="color:red;">Snom D120</mark>

## Extensions List

The "**Extensions**" section displays a list of internal user accounts for **employees**. On the left side of each employee, the status of the authorized device is displayed. If the device is successfully authorized under the respective internal user account, a green circle is shown; otherwise, it appears gray.

<figure><img src="../../.gitbook/assets/2 (29).png" alt=""><figcaption></figcaption></figure>

In the search bar, you can find the desired contact. You can search by the employee's name, internal number, mobile number, or email address.

<figure><img src="../../.gitbook/assets/3 (4).png" alt=""><figcaption></figcaption></figure>

The form also provides the ability to sort the list of employees by name, internal number, mobile number, or email address. There are buttons for copying the account password to the clipboard, editing the account, and deleting the account.

<figure><img src="../../.gitbook/assets/4 (10).png" alt=""><figcaption></figcaption></figure>

## Adding an Employee

To add a new employee, simply click on the "**Add new extension**" button.

<figure><img src="../../.gitbook/assets/5 (26).png" alt=""><figcaption></figcaption></figure>

## Main Account Settings

<figure><img src="../../.gitbook/assets/6 (10).png" alt=""><figcaption></figcaption></figure>

On the "Basic Parameters" tab, you can configure the general settings for an employee's internal account:

* **Username**: This value will be used for name substitution and displayed in the corresponding field on the phone screen.&#x20;
* **Internal Number**: This is the employee's internal extension number, which is also used as the login when connecting the phone.&#x20;
* **Mobile Number**: It is used for additional routing purposes.&#x20;
* **Email Address**: It is used for email notifications.&#x20;
* **Password for SIP**

{% hint style="warning" %}
Please ensure that the passwords for SIP accounts meet the following requirements:

* The password length should be greater than eight characters.&#x20;
* The password should contain both uppercase and lowercase letters. The password should include numbers and special characters: "-", "\_", "\[]", "{}", "@", ";".
* By setting complex passwords, you can enhance the security of the user accounts and protect them from unauthorized access.
{% endhint %}

## Advanced Account Settings

Accesses the Advanced drop-down list.

<figure><img src="../../.gitbook/assets/9 (5).png" alt=""><figcaption></figcaption></figure>

**Redefining the set string**

<figure><img src="../../.gitbook/assets/8 (11).png" alt=""><figcaption></figcaption></figure>

In the "**Redefining the set string**" field, enter the dialing rule for mobile numbers according to your provider's requirements.

#### Call recording

<figure><img src="../../.gitbook/assets/10 (6).png" alt=""><figcaption></figcaption></figure>

If you want employees to have the ability to record conversations, you can enable the **Сall recording** feature.

#### **DTMF Mode**

<figure><img src="../../.gitbook/assets/11 (3).png" alt=""><figcaption></figcaption></figure>

The setting determines how DTMF (Dual Tone Multi-Frequency) signals are transmitted over the SIP protocol. DTMF signals are used, for example, when dialing phone numbers or interacting with IVR systems.

#### Transport protocol

<figure><img src="../../.gitbook/assets/12 (2).png" alt=""><figcaption></figcaption></figure>

This setting allows you to specify the transport protocol used for this account. The transport protocol determines how data is transmitted over the network. The most common transport protocols used in SIP (Session Initiation Protocol) are UDP (User Datagram Protocol), TCP (Transmission Control Protocol), and TLS (Transport Layer Security)

#### Network filter

<figure><img src="../../.gitbook/assets/14 (1).png" alt=""><figcaption></figcaption></figure>

The subnet described in the "Network Firewall" section specifies the allowed subnet for this account. It determines which IP addresses or networks are permitted to connect to this account. Connections originating from other subnets will result in authentication errors.

#### Manual additional attributes for SIP

<figure><img src="../../.gitbook/assets/15 (1).png" alt=""><figcaption></figcaption></figure>

This field is used to modify/override the configuration files of Asterisk. You can override almost all parameters. For example, when using **chan\_pjsip**, a SIP account for an employee is described by the following sections:

{% code lineNumbers="true" %}
```
[***]
type = aor
max_contacts = 10
; ----

[***]
type = auth
; ----

[***]
type = endpoint
context = all_peers
; ----

[acl_***] 
deny = 0.0.0.0/0.0.0.0
permit = 0.0.0.0/0.0.0.0
; ----
```
{% endcode %}

To override fields in the sections, you should fill in the "Additional Parameters" field as follows:

```
[acl]
; Describe access parameters from different subnets [acl_***]

[auth] 
; Describe authentication parameters for outbound calls

[aor]
; Edit AOR section for the endpoint

[endpoint] 
; Edit endpoint parameters
```



## Routing Settings

<figure><img src="../../.gitbook/assets/17 (1).png" alt=""><figcaption></figcaption></figure>

On this tab, you can set rules for call forwarding when the employee is unavailable, busy, or does not answer.

Set the time **period** in seconds during which the call will be directed to the employee's **internal** account. If the employee cannot answer the call within the specified time, indicate to which number the call should be forwarded. By default, the call will be redirected to the employee's mobile number.

You can also specify the numbers to which the call should be redirected in case of busy and unavailable status.

Feel free to configure these parameters according to your preferences and requirements.





\
