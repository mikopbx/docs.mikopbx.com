# Callback: call the client back and route into an IVR

### Introduction <a href="#introduction" id="introduction"></a>

The "callback" scenario is useful when an incoming call is expensive for the subscriber (international or long-distance calls, roaming) or when the company, rather than the client, should pay for the connection. As soon as the client gets through, MikoPBX **drops the incoming call and calls the subscriber back itself**, and after they answer it routes them into an IVR (voice menu) — from there the call proceeds through the usual internal routing.

{% hint style="info" %}
This is not the same as the [Missed-call auto-processing](../../modules/miko/module-callback-queues.md) module: that one records a _missed_ call and makes an _agent_ call the client back. Here the callback is placed **immediately and automatically**, without an operator, routing the subscriber into an IVR.
{% endhint %}

It is implemented with two parts: a custom dialplan context in `extensions.conf` (a wrapper with a pause before dialing) and a PHP-AGI dialplan application that schedules the callback.

### 1. Add a custom context to extensions.conf <a href="#custom_context" id="custom_context"></a>

Go to the "[**System file customization**](../../manual/system/custom-files.md)" section, open the **extensions.conf** file for editing and append the wrapper context:

```php
[callback-originate]
exten => _[+0-9].,1,Wait(4)
 same => n,Goto(outgoing,${EXTEN},1)
```

The context waits 4 seconds (so the incoming leg can terminate cleanly) and then sends the call into the `outgoing` context — that is, it dials the subscriber's number through your **outbound routes**.

### 2. Create a dialplan application <a href="#dialplan_app" id="dialplan_app"></a>

Add a new dialplan application (see [**Dialplan Applications**](../../manual/modules/dialplan-applications.md)), code type — "**PHP-AGI script**", and paste the code:

```php
<?php
require_once 'Globals.php';

use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;
use MikoPBX\Core\System\SystemMessages;

const CALLBACK_WHITELIST    = [];                   // empty = for everyone
const CALLBACK_IVR_EXTEN    = '2003';               // IVR number in MikoPBX
const CALLBACK_IVR_CONTEXT  = 'internal';
const CALLBACK_ORIG_CONTEXT = 'callback-originate'; // wrapper context with Wait
const CALLBACK_TIMEOUT      = 60000;                // ms: Wait(4) + dialing

$agi    = new AGI();
$raw    = (string)$agi->get_variable('CALLERID(num)', true);
$caller = preg_replace('/\D/', '', $raw);

if ($caller === '') {
    SystemMessages::sysLogMsg('Callback', "Empty CID ($raw)", LOG_WARNING);
    return;
}
if (CALLBACK_WHITELIST !== [] && !in_array($caller, CALLBACK_WHITELIST, true)) {
    return; // not our CID — the call will follow the usual route
}

// 1. Schedule the callback (the AMI action lives in the core, independent of AGI).
try {
    $am = Util::getAstManager('off');
    $am->Originate(
        'Local/' . $caller . '@' . CALLBACK_ORIG_CONTEXT . '/n',
        CALLBACK_IVR_EXTEN,
        CALLBACK_IVR_CONTEXT,
        1,
        null, null,
        CALLBACK_TIMEOUT,
        $caller,
        '__ISCALLBACK=1',
        null,
        true
    );
    SystemMessages::sysLogMsg('Callback', "Scheduled callback $caller -> IVR " . CALLBACK_IVR_EXTEN, LOG_NOTICE);
} catch (\Throwable $e) {
    SystemMessages::sysLogMsg('Callback', 'Originate failed: ' . $e->getMessage(), LOG_ERR);
    return; // do not drop the incoming call if originate did not go out
}

// 2. Terminate the incoming call the regular MikoPBX way (it sets M_DIALSTATUS and Hangup itself).
$agi->exec_goto('internal', 'hangup', '1');
```

### 3. Configure the constants <a href="#configuration" id="configuration"></a>

| Constant | Purpose |
| --- | --- |
| `CALLBACK_WHITELIST` | Whitelist of incoming CIDs allowed to use callback. An empty array — call **everyone** who gets through back |
| `CALLBACK_IVR_EXTEN` | The IVR number (or other destination) the subscriber reaches after answering the callback |
| `CALLBACK_IVR_CONTEXT` | The destination context. For internal numbers and IVRs — `internal` |
| `CALLBACK_ORIG_CONTEXT` | The name of the custom wrapper context from step 1 |
| `CALLBACK_TIMEOUT` | Timeout (ms) to reach the subscriber, with a margin for `Wait(4)` |

### 4. Route an incoming call to the application <a href="#incoming_route" id="incoming_route"></a>

In the "[**Incoming routes**](../../manual/routing/incoming-routing.md)" section, set the created dialplan application as the destination (for the needed provider or DID). After that every such incoming call will be dropped and called back automatically.

### Important points <a href="#important_points" id="important_points"></a>

1. The callback goes out through your outbound routes — **the company pays for it**. Restrict access with `CALLBACK_WHITELIST`, otherwise the callback can be abused for free calls at your expense.
2. The subscriber's number is taken from `CALLERID(num)` and stripped of non-digit characters. It must be in a format accepted by your outbound routes and [number templates](../outbound-routing/number-templates/README.md); normalize it in the `[callback-originate]` context if needed.
3. The IVR with the number `CALLBACK_IVR_EXTEN` (in the example `2003`) must exist in MikoPBX (see [Basic IVR example](basic-ivr-example.md)).
4. Loop protection: the callback is tagged with the `__ISCALLBACK=1` variable — use it in the custom dialplan if you need to prevent re-entering the script on the callback leg.
5. All events are logged via `SystemMessages::sysLogMsg('Callback', ...)` — search by the `Callback` tag in the system log.
6. The script is **not a complete product**, but is open for customization.
