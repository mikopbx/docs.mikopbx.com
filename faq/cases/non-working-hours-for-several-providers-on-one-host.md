# Setting up individual non-working hours for several providers on one host

## Task <a href="#postanovka_zadachi" id="postanovka_zadachi"></a>

We have two accounts from the Zadarma provider, configured in MikoPBX following this [guide](registering-multiple-accounts-from-one-provider.md).

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Two accounts</p></figcaption></figure>

Each Zadarma number needs to have its own non-working hours. For example, the number +60 17 308 9237 has working hours from 9:00 AM to 6:00 PM, while the number +60 17 308 2584 has working hours from 8:00 AM to 8:00 PM.

## Solution <a href="#reshenie" id="reshenie"></a>

1. Go to **System** → **System File Customization**.

<figure><img src="../../.gitbook/assets/systemFileCustomization.png" alt=""><figcaption></figcaption></figure>

2. Open the **extensions.conf** configuration file for editing. Set the mode to "**Append to the end of the file**". In the black editing window, add the following code snippet:

```php
[public-direct-dial-custom]
exten => _.!,1,NoOp(check time)
    same => n,Gosub(check-out-work-time-custom,${FROM_DID},1)
    same => n,return
	
[check-out-work-time-custom]
exten => 584611,1,NoOp(check time)
    same => n,ExecIfTime(00:00-09:00,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,ExecIfTime(18:00-23:59,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,return
exten => 420296,1,NoOp(check time)
    same => n,ExecIfTime(00:00-08:00,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,ExecIfTime(20:00-23:59,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
    same => n,return
```

<figure><img src="../../.gitbook/assets/codeForExtensionsConf.png" alt=""><figcaption><p>Code at the end of the extensions.conf file</p></figcaption></figure>

In the above code fragment, you need to specify the logins for your provider accounts. In our example, the following data was used:

*   **584611** - the login for the provider account linked to the phone number +60 17 308 9237.

    The working hours are from 9:00 to 18:00. Therefore, two intervals for non-working hours need to be set: 00:00-09:00 and 18:00-23:59.
*   **420296** - the login for the provider account linked to the phone number +60 17 308 2584.

    The working hours are from 8:00 to 20:00. Therefore, two intervals for non-working hours need to be set: 00:00-08:00 and 20:00-23:59.

Below, the fragments highlighted in red are the ones you need to modify:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Fragments to change</p></figcaption></figure>

Let's break down the **ExecIfTime** command in more detail. This command executes the specified Asterisk application if the current time matches the given time specification. In our case, the command plays an audio file located in the directory **/offload/asterisk/sounds/other/out\_work\_times** in MikoPBX. Command syntax:

```
ExecIfTime(times,weekdays,mdays,months?appname[(appargs)])
```

* **times** - Time ranges in 24-hour format
* **weekdays** - Days of the week (mon, tue, wed, thu, fri, sat, sun)
* **mdays** - Days of the month (1-31)
* **months** - Months (jan, feb, mar, apr, etc.)
* **appname\[(appargs)]** - the Asterisk command with its call parameters

In our example, a time range and a range of weekdays are specified. Instead of specifying days of the month and months, a \* is used, meaning "for all other cases."

```
ExecIfTime(00:00-08:00,mon-fri,*,*?Macro(playback-exit,/offload/asterisk/sounds/other/out_work_times))
```
