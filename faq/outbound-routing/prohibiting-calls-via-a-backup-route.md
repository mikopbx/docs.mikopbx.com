# Prohibiting calls via a backup route

When describing [outgoing routes](../../manual/routing/outbound-routing.md), it is allowed to use the same phone number template.&#x20;

If several routes are used with the same template, the PBX will try to make a call on each of the routes until the call is accepted.&#x20;

This functionality is not always necessary and there may be a need to disable it.&#x20;

At the moment, this can only be done through the customization of the PBX.

1. Go to the provider's settings and copy its ID in the browser's address bar:

<figure><img src="../../.gitbook/assets/ProviderID (1).png" alt=""><figcaption><p>Provider ID</p></figcaption></figure>

2. Go to the section "**System file customization**"

<figure><img src="../../.gitbook/assets/SystemFileCustomization.png" alt=""><figcaption><p>System file cuztomization menu</p></figcaption></figure>

3. Go to the editing section of the **extensions.conf** file.

<figure><img src="../../.gitbook/assets/EditExtensions.conf.png" alt=""><figcaption><p>extensions.conf file</p></figcaption></figure>

4. Add the following context to the end of the file:

```php
[SIP-1687941868-outgoing-after-dial-custom]
exten => _X!,1,Hangup()
	same => n,return
```

<figure><img src="../../.gitbook/assets/CodeForExtensions (4).png" alt=""><figcaption><p>Code in extensions.conf</p></figcaption></figure>

Save the changes. Now all calls via routes associated with the provider will occur without taking into account the backup provider. It will work out only the first of the outgoing routes.
