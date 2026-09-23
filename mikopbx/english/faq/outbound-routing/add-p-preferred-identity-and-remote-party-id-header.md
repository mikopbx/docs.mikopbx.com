# Add P-Preferred-Identity and Remote-Party-ID header

Some providers require sending an additional SIP header when initiating an outgoing call. In this article we will describe a way to install it.

1. To solve the problem, additional contexts should be described through the **System File Customization** menu.

<figure><img src="../../.gitbook/assets/SystemFileCustomization.png" alt=""><figcaption><p>System File Customization menu</p></figcaption></figure>

2. We will edit the **extensions.conf** file.

<figure><img src="../../.gitbook/assets/EditExtensions.conf.png" alt=""><figcaption><p>Extensions.conf file</p></figcaption></figure>

3. Add the following text to the end of the file:

```php
[SIP-TRUNK-A2DDBADA-outgoing-custom]
exten => _X!,1,Dial(PJSIP/${number}@${PROVIDER_ID},600,${DOPTIONS}TKU(dial_answer)b(dial_create_chan_${PROVIDER_ID}_custom,s,1))
    same => n,ExecIf($["${DIALSTATUS}" = "ANSWER"]?Hangup())
    same => n,ExecIf($["${DIALSTATUS}" = "BUSY"]?Busy(2))
    same => n,return
    
[dial_create_chan_SIP-TRUNK-A2DDBADA_custom] 
exten => s,1,Gosub(lua_${ISTRANSFER}dial_create_chan,${EXTEN},1)
    same => n,Set(pt1c_is_dst=1) 
    same => n,Set(OUTGOING_CID=32672293042)
    same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,P-Preferred-Identity)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,Remote-Party-ID)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,Set(__PT1C_SIP_HEADER=${UNDEFINED}) 
    same => n,Set(CHANNEL(hangup_handler_wipe)=hangup_handler,s,1) 
    same => n,return
```

<figure><img src="../../.gitbook/assets/CodeForExtensionsConf (1).png" alt=""><figcaption><p>Code for extrensions.conf</p></figcaption></figure>

{% hint style="danger" %}
**Pay attention:**

1. Replace all occurrences of "**SIP-TRUNK-A2DDBADA**" with your provider ID. You can find it in the browser address bar when editing the provider account in the MikoPBX web interface.
2. Instead of **"**[**sip:${OUTGOING\_CID}@127.0.0.1**](https://sip:$%7BOUTGOING_CID%7D@127.0.0.1)**"**, the required header value should be set
3. The provider ID format depends on the MikoPBX version: current versions use `SIP-TRUNK-XXXX`, older ones use `SIP-PROVIDER-XXXX` or `SIP-XXXX`. That is why the example does not extract the provider from the context name but takes it from the `${PROVIDER_ID}` variable, which MikoPBX sets before calling the context. Versions earlier than 2024.1.114 do not set this variable — replace `${PROVIDER_ID}` with your provider ID explicitly.
{% endhint %}

#### When using the "User Groups" module:

```php
[SIP-TRUNK-A2DDBADA-outgoing-ug-custom]
exten => _X!,1,Dial(PJSIP/${number}@${PROVIDER_ID},600,${DOPTIONS}TKU(dial_answer)b(dial_create_chan_custom,s,1))
	same => n,ExecIf($["${DIALSTATUS}" = "ANSWER"]?Hangup())
	same => n,ExecIf($["${DIALSTATUS}" = "BUSY"]?Busy(2))
    same => n,return
    
[dial_create_chan_custom] 
exten => s,1,Gosub(lua_${ISTRANSFER}dial_create_chan,${EXTEN},1)
    same => n,Set(pt1c_is_dst=1) 
    same => n,Set(GR_VARS=${DB(UsersGroups/${FROM_PEER})}) 
    same => n,Set(tmpName=${CUT(GR_VARS,\,,2)})
    same => n,Set(tmpName=${CUT(tmpName,\_,3)})
	same => n,ExecIf($["${GR_VARS}x" != "x"]?Exec(Set(${GR_VARS}))) 
	same => n,ExecIf($["${GR_PERM_ENABLE}" == "1" && "${GR_ID_${tmpName}}" != "1"]?return) 
	same => n,ExecIf($["${GR_PERM_ENABLE}" == "1" && "${GR_CID_${tmpName}}x" != "x"]?MSet(GR_OLD_CALLERID=${CALLERID(num)},OUTGOING_CID=${GR_CID_${tmpName}}))
	same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,P-Preferred-Identity)=<sip:${OUTGOING_CID}@127.0.0.1>))
	same => n,ExecIf($["${OUTGOING_CID}x" != "x"]?Set(PJSIP_HEADER(add,Remote-Party-ID)=<sip:${OUTGOING_CID}@127.0.0.1>))
    same => n,Set(__PT1C_SIP_HEADER=${UNDEFINED}) 
    same => n,Set(CHANNEL(hangup_handler_wipe)=hangup_handler,s,1) 
    same => n,return
```

{% hint style="warning" %}
The peculiarity of this option is that the value "OUTGOING\_CID" will be taken from the settings of user groups. For each group, you can assign your own value of the outgoing caller id. For example, the Westcall provider thus allows you to control the value of the callerid that the client sees.
{% endhint %}
