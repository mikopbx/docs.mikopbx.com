---
description: Run custom logic on an incoming call using dialplan generation hooks.
---

# Hook on incoming call

When a call arrives from a provider, MikoPBX routes it through a per-route incoming context generated in `extensions.conf`. A module participates in that context by implementing dialplan-generation hooks on its configuration class (the subclass of `MikoPBX\Modules\Config\ConfigClass`). Each hook returns a **string of dialplan lines** that MikoPBX splices into the right place in the incoming context — most usefully an `AGI(...)` line that runs your PHP script, so you can inspect or modify the call before it is dialed to its destination.

This recipe shows how the fictional **ModuleBlackList** rejects blacklisted callers, and points to the real modules that ship these hooks today.

{% hint style="info" %}
All hooks below are defined on `AsteriskConfigClass` (the base for module config classes) and are invoked by the core dialplan generator `Core/src/Core/Asterisk/Configs/Generators/Extensions/IncomingContexts.php`. After you change what a hook returns, regenerate the dialplan — see [Reloading the dialplan](hook-on-incoming-call.md#reloading-the-dialplan).
{% endhint %}

## The incoming hooks and when they fire

A single incoming route produces one context. The core builds it in this order, calling each module hook in turn. The three "before dial" hooks fire **before** the `Dial()` to the route's destination; the "after dial" hook fires **after**.

| Hook                                      | Signature                       | Fires                                                                           |
| ----------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------- |
| `generateIncomingRoutBeforeDialPreSystem` | `(string $rout_number): string` | First, before MikoPBX's own system logic                                        |
| `generateIncomingRoutBeforeDialSystem`    | `(string $rout_number): string` | After PreSystem, still before `Dial()`                                          |
| `generateIncomingRoutBeforeDial`          | `(string $rout_number): string` | Last of the "before" group, immediately before `Dial()` — **the common choice** |
| `generateIncomingRoutAfterDialContext`    | `(string $uniqId): string`      | After the default action's `Dial()`/`Goto()` in the provider summary context    |

The exact ordering is visible in `IncomingContexts.php`:

```php
// Core/src/Core/Asterisk/Configs/Generators/Extensions/IncomingContexts.php
$rout_data .= $this->hookModulesMethod(AsteriskConfigInterface::GENERATE_INCOMING_ROUT_BEFORE_DIAL_PRE_SYSTEM, [$rout_number]);
$rout_data .= $this->hookModulesMethod(AsteriskConfigInterface::GENERATE_INCOMING_ROUT_BEFORE_DIAL_SYSTEM, [$rout_number]);
$rout_data .= $this->hookModulesMethod(AsteriskConfigInterface::GENERATE_INCOMING_ROUT_BEFORE_DIAL, [$rout_number]);
// ... Dial() to the route destination ...
$rout_data .= $this->hookModulesMethod(AsteriskConfigInterface::GENERATE_INCOMING_ROUT_AFTER_DIAL_CONTEXT, [$uniqId]);
```

The `$rout_number` argument is the DID (the dialed inbound number) of the route being generated; `""` denotes the default/any route.

{% hint style="warning" %}
The three "before dial" hooks fire once **per incoming route**. The after-dial hook does not. It is emitted from `createSummaryDialplanGoto()` inside the `[<provider>-incoming]` summary context, and only when a **default action** with a non-empty `extension` exists — a route set to _Playback_ or _Busy_ never reaches it. `$uniqId` there is the provider id, falling back to the incoming route's uniqid (`IncomingContexts::createSummaryDialplan()`, lines 532-565).
{% endhint %}

{% hint style="info" %}
Use `$rout_number` to scope your logic. Returning an empty string for a route means "add nothing here" — that is how `generateIncomingRoutBeforeDial` can target only certain DIDs and leave the rest untouched.
{% endhint %}

### Channel variables available at the "before dial" hook point

By the time the before-dial hooks run, the core has already set these channel variables in the context (see the same `IncomingContexts.php`):

| Variable            | Meaning                                             |
| ------------------- | --------------------------------------------------- |
| `${FROM_DID}`       | The dialed inbound number (same as `${EXTEN}` here) |
| `${FROM_CHAN}`      | The originating channel name                        |
| `${M_CALLID}`       | The channel `callid`                                |
| `${CALLERID(num)}`  | The caller's number                                 |
| `${CALLERID(name)}` | The caller's display name                           |

Your emitted dialplan (and any AGI script it launches) can read and modify these.

## Step 1 — emit an AGI line before the call is dialed

The most common pattern is to drop a single `AGI()` line into the incoming context that runs a PHP script bundled with your module. The script does the work — lookup, blocking, caller-ID rewriting — and the dialplan continues afterward.

`ConfigClass` exposes the module's installation directory as the protected property `$this->moduleDir`, so you can reference your script by absolute path.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Core\Asterisk\Configs\ExtensionsConf;
use MikoPBX\Modules\Config\ConfigClass;

/**
 * Dialplan integration for ModuleBlackList.
 */
class BlackListConf extends ConfigClass
{
    /**
     * Inject an AGI check right before the inbound call is dialed.
     *
     * @param string $rout_number The DID of the incoming route ("" = default route).
     * @return string Dialplan lines to append, or "" to add nothing.
     */
    public function generateIncomingRoutBeforeDial(string $rout_number): string
    {
        // The agi_callerid / agi_extension request fields are read inside the script.
        return "same => n,AGI({$this->moduleDir}/agi-bin/agi_black_list.php,in)" . PHP_EOL;
    }

    /**
     * Force a dialplan reload after the module is enabled so the hook takes effect.
     */
    public function onAfterModuleEnable(): void
    {
        ExtensionsConf::reload();
    }
}
```
{% endcode %}

{% hint style="success" %}
**Real example — verbatim from production.** `ModulePhoneBook` uses exactly this hook to inject a per-call AGI on incoming calls. See `Extensions/ModulePhoneBook/Lib/PhoneBookConf.php`:

```php
public function generateIncomingRoutBeforeDial($rout_number): string
{
    return "same => n,AGI({$this->moduleDir}/agi-bin/agi_phone_book.php,in)" . PHP_EOL;
}
```

`ModuleSmartIVR` does the same in `Extensions/ModuleSmartIVR/Lib/SmartIVRConf.php`, emitting `same => n,AGI({$this->moduleDir}/agi-bin/SmartIVR_AGI.php)`.
{% endhint %}

Notes on the emitted line:

* Use `same => n,...` (relative priority) — the line is appended inside an existing `exten => ...` block, so `n` continues from the previous priority.
* Always terminate the returned string with `PHP_EOL`; multiple lines are simply concatenated.
* The second AGI argument (`in`) is passed to the script as `$argv[1]` — handy for sharing one script between the incoming and outgoing paths.

## Step 2 — write the AGI script (correct AGI API)

The script runs as a standalone PHP process that talks AGI over stdin/stdout. Instantiate the core `MikoPBX\Core\Asterisk\AGI` class and use its **snake\_case** methods.

{% hint style="danger" %}
Use the real API: the class is `AGI`, and the methods are `get_variable()` / `set_variable()` / `set_var()`. There is **no** `AgiClient` class and **no** `getVariable()` camelCase method — those do not exist in MikoPBX and will fatal. Confirm signatures in `Core/src/Core/Asterisk/AGI.php`.
{% endhint %}

The request fields (`agi_callerid`, `agi_extension`, `agi_channel`, `agi_uniqueid`, …) are parsed at construction time and exposed on `$agi->request` (see `AGIBase::readRequestData()` in `Core/src/Core/Asterisk/AGIBase.php`).

{% code title="Extensions/ModuleBlackList/agi-bin/agi_black_list.php" %}
```php
#!/usr/bin/php
<?php

use Modules\ModuleBlackList\Lib\BlackListMain;
use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;

require_once 'Globals.php';

$type = $argv[1] ?? 'in';

try {
    $agi = new AGI();

    // For inbound calls the caller's number is in agi_callerid.
    $number = ($type === 'in')
        ? $agi->request['agi_callerid']
        : $agi->request['agi_extension'];

    if (BlackListMain::isBlocked($number)) {
        // Optional: log to the Asterisk console for diagnostics.
        $agi->verbose("ModuleBlackList: blocking {$number}", 3);

        // Mark the call (readable later in the dialplan via ${BLACKLISTED}).
        $agi->set_variable('BLACKLISTED', '1');

        // Hang up the current channel.
        $agi->hangup();
    }
} catch (\Throwable $e) {
    Util::sysLogMsg('ModuleBlackList', $e->getMessage(), LOG_ERR);
}
```
{% endcode %}

`set_variable()` sets a channel variable (e.g. `CALLERID(name)`), `get_variable()` reads one, `verbose()` writes to the Asterisk console/log, `hangup()` ends the channel. The convenience helper `getCallerIdName(string $number): string` returns `CALLERID(name)` when it differs from the number.

{% hint style="success" %}
**Real example.** `Extensions/ModulePhoneBook/Lib/PhoneBookAgi.php` reads `$agi->request['agi_callerid']` on incoming calls and rewrites the display name with `$agi->set_variable('CALLERID(name)', $result->call_id)`. That file is the canonical, production reference for an incoming-call AGI script.
{% endhint %}

## Step 3 (optional) — route into a custom context

Instead of (or in addition to) a bare `AGI()` line, you can publish your own dialplan context with `extensionGenContexts()` and jump into it from the before-dial hook. This keeps complex per-call logic in a self-contained block and lets you reuse it from multiple routes.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\Asterisk\Configs\ExtensionsConf;

/**
 * Publish a [module-black-list-in] context in extensions.conf.
 */
public function extensionGenContexts(): string
{
    return '[module-black-list-in]' . PHP_EOL
        . 'exten => ' . ExtensionsConf::ALL_NUMBER_EXTENSION
        . ",1,AGI({$this->moduleDir}/agi-bin/agi_black_list.php,in)\n\t"
        . 'same => n,return' . PHP_EOL;
}

/**
 * From the incoming route, Gosub into the custom context.
 */
public function generateIncomingRoutBeforeDial(string $rout_number): string
{
    return 'same => n,Gosub(module-black-list-in,${EXTEN},1)' . PHP_EOL;
}
```
{% endcode %}

`ExtensionsConf::ALL_NUMBER_EXTENSION` is the catch-all match pattern (a real constant in `Core/src/Core/Asterisk/Configs/ExtensionsConf.php`) used to accept any dialed number in a custom context.

{% hint style="success" %}
**Real example.** `PhoneBookConf::extensionGenContexts()` publishes a `[phone-book-out]` context with an AGI line, and `generateOutRoutContext()` wires the outbound side into it — the same `extensionGenContexts()` + hook pattern, applied to outgoing routes.
{% endhint %}

## Step 4 (optional) — act after the call ends

To run logic once the call is over (e.g. record an outcome), implement `generateIncomingRoutAfterDialContext(string $uniqId)`. It is appended after the `Dial()` in the incoming context, so `${M_DIALSTATUS}` (`ANSWER`, `BUSY`, `NOANSWER`, …) is already set.

```php
public function generateIncomingRoutAfterDialContext(string $uniqId): string
{
    return 'same => n,AGI(' . $this->moduleDir
        . '/agi-bin/agi_black_list.php,after)' . PHP_EOL;
}
```

## Reloading the dialplan

Dialplan-generation hooks only take effect after `extensions.conf` is regenerated and Asterisk reloads it. Trigger it after enabling the module (as in `onAfterModuleEnable()` above), or any time your settings change:

```php
use MikoPBX\Core\System\PBX;

PBX::dialplanReload();
```

## Verify it works

1. Enable ModuleBlackList in the web UI (or call `PBX::dialplanReload()`).
2.  Inspect the generated context:

    ```bash
    asterisk -rx "dialplan show" | grep -A20 "agi_black_list"
    ```
3.  Place an inbound test call and watch the console:

    ```bash
    asterisk -rvvvvv
    ```

    You should see your `verbose()` line and, for a blacklisted number, the hangup.

## See also

* [Asterisk cookbook overview](./)
* [Module hooks reference](../../module-developement/hooks-reference.md)
* [AGI API reference](../../internal-structure/api/agi.md)
