# Setting up the "Paging" function

Paging through phones, i.e. transmitting a voice message through several phones via speakerphone. For example, a manager can quickly call a meeting.

{% hint style="info" %}
This instruction is suitable for phones:

* **Linksys**
* **Cisco**
* **Telephone** (Softphone)
* **Grandstream**
* **microsip** (Softphone)
* **Yealink**
* **Fanvil**&#x20;
* **Snom**&#x20;
{% endhint %}

1. Go to the "**System file customization**" section

<figure><img src="../../.gitbook/assets/SystemFileCustomization.png" alt=""><figcaption><p>"System file customization" section</p></figcaption></figure>

2. Open the file "**/var/spool/cron/crontabs/root**" for editing

<figure><img src="../../.gitbook/assets/crontabsroot.png" alt=""><figcaption><p>"crontabs/root" file</p></figcaption></figure>

3. Add the following code to the end of the file:

```php
*/1 * * * * /bin/touch /etc/asterisk/confbridge.conf > /dev/null 2> /dev/null
```

<figure><img src="../../.gitbook/assets/codeForCrontabsRoot.png" alt=""><figcaption><p>The code for the "crontabs/root" file</p></figcaption></figure>

4. Proceed to editing the "**modules.conf**" file

<figure><img src="../../.gitbook/assets/modulesConf.png" alt=""><figcaption><p>"Modules.conf" file</p></figcaption></figure>

5. Add the following code to the end of the file:

```php
load => bridge_softmix.so
load => app_confbridge.so
load => app_page.so
```

<figure><img src="../../.gitbook/assets/codeForModules.png" alt=""><figcaption><p>Code for modules.conf</p></figcaption></figure>

6. Go to editing the file "**extensions.conf**"

<figure><img src="../../.gitbook/assets/EditExtensions.conf.png" alt=""><figcaption><p>extensions.conf file</p></figcaption></figure>

7. Add the following code to the end of the "**extensions.conf**" file

```php
[paging-users] 
exten => _X!,1,Set(dС=${PJSIP_DIAL_CONTACTS(${EXTEN})})
  same => n,ExecIf($["${dС}x" != "x"]?Dial(${dС},,b(paging_create_chan,s,1)))

[paging_create_chan] 
exten => s,1,Set(PJSIP_HEADER(add,Call-Info)=\;answer-after=0) 
  same => n,return
```

<figure><img src="../../.gitbook/assets/codeForExtensions.conf.png" alt=""><figcaption><p>Code for extensions.conf</p></figcaption></figure>

8. Go to the "**Dialplan Applications**" section, create a new dialplan

<figure><img src="../../.gitbook/assets/newDialplan (1).png" alt=""><figcaption><p>New dialplan</p></figcaption></figure>

9. Assign an internal number, for example 2200110. Set the code type: "**Asterisk Dialplan**"

<figure><img src="../../.gitbook/assets/SettingsForDialplan.png" alt=""><figcaption><p>Dialplan Settings</p></figcaption></figure>

10. Go to the "**Programme Code**" tab, insert the following code:

```php
1,Page(Local/202@paging-users&Local/203@paging-users)
```

<figure><img src="../../.gitbook/assets/codeForDialplan (2).png" alt=""><figcaption><p>Code for dialplan</p></figcaption></figure>

{% hint style="warning" %}
In the application code, describe the contacts to whom you should call. Contacts are listed with a **&** separator.
{% endhint %}
