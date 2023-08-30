# Limit the number of authorizations per SIP account

1. Go to edit the employee account.

<figure><img src="../../.gitbook/assets/extensionsSection.png" alt=""><figcaption><p>Editing an employee's account</p></figcaption></figure>

2. In the "**Extra oprions**" field, add:

```
[aor]
max_contacts = 1
```

<figure><img src="../../.gitbook/assets/extraOptions.png" alt=""><figcaption><p>Additional parameters in the employee account</p></figcaption></figure>

{% hint style="success" %}
The last registration will work. The softphone/phone that registered last and will receive calls. Outgoing calls can be made by each end device.
{% endhint %}
