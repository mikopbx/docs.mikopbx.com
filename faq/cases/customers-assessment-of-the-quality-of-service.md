# Customer's assessment of the quality of service

Improving the quality of the organization's service is an extremely important component of the successful operation of the company. The mechanism for evaluating the work of an operator will be useful to any business.&#x20;

In the current article, we offer an example of the implementation of the quality assessment mechanism by the client for PBX MikoPBX.

### Refinement of Dialplan

{% hint style="warning" %}
You should first upload the question/greeting record files to the PBX.

&#x20;WAV (Microsoft) signed 16-bit PCM recording format. Files can be conveniently placed in the directory: **/storage/usbdisk1/quality/sounds**.&#x20;

In dialplan, the path to the file should be specified without an extension.
{% endhint %}

To begin with, let's add the logic of processing incoming calls:

1. Go to **"Routing" → "Telephony Providers"**. Open the provider's account for editing. Copy in the address bar the ID of the provider through which subscribers call you to the company. Please note that in our example, the only provider Zadarma is used. If you have configured the connection of several providers, then the following steps must be performed for **each provider.**

In our example, the provider ID takes the form: **SIP-1687947415**

<figure><img src="../../.gitbook/assets/1 (3).png" alt=""><figcaption></figcaption></figure>

2. Go to "**System**" → "**System file customization**".

<figure><img src="../../.gitbook/assets/2 (26).png" alt=""><figcaption></figcaption></figure>

3. Open the **extensions.conf** configuration file for editing.

<figure><img src="../../.gitbook/assets/3 (21).png" alt=""><figcaption></figcaption></figure>

4. Set the "**Add to end of file**" mode.

<figure><img src="../../.gitbook/assets/4 (26).png" alt=""><figcaption></figcaption></figure>

5. In the black window, add the following code snippet:

```
[SIP-PROVIDER-SIP-1687947415-after-dial-custom]
exten => _.!,1,Goto(quality-start,s,1)
	same => n,return

[SIP-PROVIDER-SIP-1687947415-outgoing-custom]
exten => _.!,1,Set(DOPTIONS=${DOPTIONS}F(quality-start,s,1))
	same => n,return

[quality-start]
exten => _.!,1,NoOp(--- Quality assessment ---)
	same => n,ExecIf($[${M_DIALSTATUS}!=ANSWER]?return)
	; We describe the path to the conversation termination file. Thank you for the quality assessment. 
	same => n,Set(filename_bye=/storage/usbdisk1/quality/sounds/bye)
	; We describe the paths to the survey files.
	same => n,Set(filename_1=/storage/usbdisk1/quality/sounds/miko_hello)
	same => n,Set(filename_2=/storage/usbdisk1/quality/sounds/q_1)
	same => n,Set(filename_3=/storage/usbdisk1/quality/sounds/q_2)
	same => n,Set(f_num=0);
	same => n,Goto(ivr-quality,s,1)

[ivr-quality]
exten => s,1,NoOP( start ivr quality )
	same => n,Set(f_num=$[${f_num} + 1])
 	same => n,Set(filename=${filename_${f_num}})
	same => n,GotoIf($["x${filename}" == "x"]?ivr-quality,bye,1);
	same => n,Background(${filename})
	same => n,WaitExten(5)

exten => _[1-5],1,NoOP( quality is ${EXTEN})
	; AGI a script that will save the result of the evaluation of the conversation by the client.
	same => n,AGI(/storage/usbdisk1/quality/quality_agi.php)
	same => n,Goto(ivr-quality,s,1)

exten => _[06-9],Goto(ivr-quality,s,1)

exten => bye,1,ExecIf($["x${filename_bye}" != "x"]?Playback(${filename_bye}));
	same => n,Hangup()
```

In the above code snippet, you need to make the correct context name.&#x20;

Format of the created context:

```
[PROVIDER ID-outgoing-custom]
[PROVIDER ID-after-dial-custom]
```

PROVIDER ID is the value that you saved in the first step of this instruction. In our example, this is **SIP-PROVIDER**-1687947415.

### Evaluation result processing script

{% hint style="info" %}
The file should be saved to the PBX along the path **/storage/usbdisk1/quality/quality\_agi.php** You can create a file using the [WinSCP ](../troubleshooting/connecting-to-a-pbx-using-winscp.md)file manager The file should be set to execute access rights. Connect via [SSH ](../troubleshooting/connecting-to-a-pbx-using-an-ssh-client.md)and run the command&#x20;

`chmod +x /storage/usbdisk1/quality/quality_agi.php`
{% endhint %}

Implementation example quality\_agi.php:

```
#!/usr/bin/php
<?php
require_once 'phpagi.php';
require_once 'globals.php';

$agi 		= new AGI();
$linkedid   = $agi->get_variable('CDR(linkedid)', true);

$arr = [
    'quality'   => $agi->request['agi_extension'],
    'f_num'     => $agi->get_variable('f_num', true),
    'filename'  => $agi->get_variable('filename', true),
    // 'Оцененный оператор:' => $operator,
    'linkedid'  => $linkedid,
    'date'      => date('Y-m-d H:i:s'),
    'callerid'  => $agi->request['agi_callerid']
];

$file_log = '/storage/usbdisk1/quality/'.$linkedid.'.log';
Util::mw_mkdir(dirname($file_log));
file_put_contents($file_log, json_encode($arr)."\n", FILE_APPEND);
```

{% hint style="info" %}
The evaluation result will be saved in files like **/storage/usbdisk1/quality/\<ID>.log**
{% endhint %}

Example file:

```
{"quality":"1","f_num":"1","filename":"\/offload\/asterisk\/sounds\/other\/miko_hello","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:13","callerid":"79265775289"}
{"quality":"2","f_num":"2","filename":"beep","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:15","callerid":"79265775289"}
{"quality":"3","f_num":"3","filename":"\/offload\/asterisk\/sounds\/other\/out_work_times","linkedid":"mikopbx-1574331248.66","date":"2019-11-21 13:14:16","callerid":"79265775289"}
```
