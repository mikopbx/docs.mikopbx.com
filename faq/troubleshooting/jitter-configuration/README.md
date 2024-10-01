# Jitter Configuration

Jitter configuration is usually required when there are significant issues with RTP traffic transmission, such as delayed voice, dropped audio, or the inability to hear the other party.

It is best to investigate the root cause by analyzing call dumps on the PBX, router, or upstream router. Compare the data and observe where the jitter increases.

{% hint style="success" %}
There is this [article ](troubleshooting-audio-issues.md)on sound problems.
{% endhint %}

However, in some cases, you may not be able to influence the quality of the communication channel. If the audio stream delay becomes too large, Asterisk will discard "late" packets. Increasing the Jitter value can solve this issue.

In MikoPBX, jitter can only be configured through [system files customisation](../../../manual/system/custom-files.md).

**modules.conf** - Add the following line:

```
load => func_jitterbuffer.so
```

**extensions.conf** - You can add **custom** contexts for the provider:

For internal (outgoing):

```php
[all_peers-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

For internal (incoming):

```php
[internal-users-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

For incoming calls:

```php
[SIP-1611151795-incoming-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

For outgoing calls:

```php
[SIP-1611151795-incoming-custom] 
exten => _.!,1,Set(JITTERBUFFER(fixed)=200,1500) 
    same => n,return
```

