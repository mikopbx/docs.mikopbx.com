# Remove all special characters from the dialed number

Some softphones / CTI solutions transmit a phone number with special characters when dialing, for example **+371(2) 229-3042**

Such a call is very likely to fail, it will be completed by mistake.

1. To solve the problem of "filtering" characters, an additional context should be described through the [Customization of system files](../../manual/system/custom-files.md) menu

<figure><img src="../../.gitbook/assets/SystemFileCustomization.png" alt=""><figcaption><p>System file customization menu</p></figcaption></figure>

2. We will edit the **extensions.conf** file.

<figure><img src="../../.gitbook/assets/EditExtensions.conf.png" alt=""><figcaption></figcaption></figure>

3. Add the following context to the end of the file:

```php
[all_peers-custom]

exten => s,1,NoOp(Cleaning dst number)
	same => n,Set(cleanNumber=${FILTER(\*\#1234567890,${EXTEN})})
	same => n,ExecIf($["${EXTEN}" != "${cleanNumber}"]?Goto(all_peers,${cleanNumber},1))
	same => n,return
```

<figure><img src="../../.gitbook/assets/codeForExtensions.png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
According to the described rule, only the characters \*#1234567890 will remain in the dialed number
{% endhint %}
