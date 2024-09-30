# Black and white lists

1. Go to **System** → **System file custmization**.

<figure><img src="../../.gitbook/assets/SystemFileCustomization.png" alt=""><figcaption><p>System file customization menu</p></figcaption></figure>

2. Open the **extensions.conf** configuration file for editing.

<figure><img src="../../.gitbook/assets/EditExtensions.conf.png" alt=""><figcaption><p>extensions.conf</p></figcaption></figure>

3. Set the "**Add to end of file**" mode and paste the following code:

```php
[add-trim-prefix-clid-custom]
exten => _X!,1,NoOp(...)
    ; Blacklist of numbers. The call will be completed.. 
    same => n,ExecIf($["${CALLERID(num)}" == "491771789427"]?Hangup())
    same => n,ExecIf($["${CALLERID(num)}" == "4915735987904"]?Hangup())
    same => n,ExecIf($["${CALLERID(num)}" == "4915735996685"]?Hangup())
    same => n,return
```

<figure><img src="../../.gitbook/assets/CodeForExtensions (6).png" alt=""><figcaption><p>Code for extensions.conf</p></figcaption></figure>

The whitelist of numbers sometimes needs to be described for specific providers:

```php
[Provider-ID-incoming-custom]
exten => _X!,1,NoOp(...)
    ; Whitelist of numbers. 
    same => n,ExecIf($["${CALLERID(num)}" == "491771789427"]?return)
    same => n,ExecIf($["${CALLERID(num)}" == "4915735996685"]?return)
    same => n,ExecIf($["${CALLERID(num)}" == "4915735987904"]?return)
    same => n,Hangup()
```

* **PROVIDER-ID** - the value that you can find in the address bar at the time of provider configuration

<figure><img src="../../.gitbook/assets/ProviderID (3).png" alt=""><figcaption><p>Provider ID</p></figcaption></figure>

