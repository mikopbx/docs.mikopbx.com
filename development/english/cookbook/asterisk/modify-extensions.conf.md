---
description: Add custom contexts and dialplan rules to the generated extensions.conf.
---

# Modify extensions.conf

MikoPBX regenerates `extensions.conf` from scratch every time the dialplan is reloaded. You never edit the file directly — instead your module's configuration class returns **string fragments** that the Core splices into the right sections. This recipe shows which method feeds which `extensions.conf` section, how the fragments are ordered, and gives copy-paste-runnable examples threaded through a fictional module, **ModuleBlackList** (config class `BlackListConf`).

For the wider hook list and the underlying call mechanism, see [hooks-reference.md](../../module-developement/hooks-reference.md). For the rest of the Asterisk cookbook, see [README.md](./).

## The return-a-string-fragment model

Your config class extends `MikoPBX\Modules\Config\ConfigClass`, which in turn extends `MikoPBX\Core\Asterisk\Configs\AsteriskConfigClass` and implements `MikoPBX\Core\Asterisk\Configs\AsteriskConfigInterface`. The base class provides empty default implementations of every dialplan hook (each simply `return '';`), so you override only the ones you need.

When the Core builds `extensions.conf`, the generator calls one hook across _all_ installed modules and concatenates the non-empty return values. The relevant generators are:

* `MikoPBX\Core\Asterisk\Configs\Generators\Extensions\InternalContexts` — drives `[internal]` and the additional-contexts block.
* `MikoPBX\Core\Asterisk\Configs\ExtensionsConf` — drives `[globals]`, the channel-create dialplan, transfer contexts, the public context and the `[internal-hints]` BLF block.
* `MikoPBX\Core\Asterisk\Configs\FeaturesConf` — drives `features.conf` star codes (see the warning below).

The concatenation happens in `AsteriskConfigClass::hookModulesMethod()`. Two facts about it matter when you write fragments:

1. **Ordering is by priority.** Modules are sorted with `usort()` on `getMethodPriority($methodName)`, **ascending** — a lower number is emitted first. The default priority for an external module (`ConfigClass`) is `10000`; Core config classes use `1000`, so core dialplan is laid down before module dialplan unless you lower your number.
2. **Each external fragment is wrapped in comment markers.** A non-empty return value from an installed module is bracketed with `; ***** BEGIN BY <moduleUniqueId>` / `; ***** END BY <moduleUniqueId>` comments by `confBlockWithComments()`, so your dialplan is easy to spot in the generated file. (There is also a `same`-prefix auto-tab in `hookModulesMethod()`, but it can only fire for core config classes: they have no `moduleUniqueId`, so `confBlockWithComments()` merely `rtrim()`s their fragment, whereas an external fragment is prefixed with the `BEGIN BY` marker and can therefore never start with `same`. Always emit your own leading tab — as the examples below do — to attach a `same =>` line to the preceding `exten`.)

{% hint style="info" %}
Hooks return raw dialplan text. Terminate lines with `PHP_EOL`. Do not write the file yourself and do not call `dialplan reload` — returning a fragment is the entire contract. To trigger a regeneration after enabling/disabling your module, call `MikoPBX\Core\Asterisk\Configs\ExtensionsConf::reload()` from `onAfterModuleEnable()` / `onAfterModuleDisable()` — it regenerates `extensions.conf` and issues the `dialplan reload`.
{% endhint %}

## Which hook writes which section

| Method                   | Section in the generated file                    | Generator          |
| ------------------------ | ------------------------------------------------ | ------------------ |
| `extensionGlobals()`     | `[globals]` in `extensions.conf`                 | `ExtensionsConf`   |
| `extensionGenContexts()` | new `[your-context]` blocks in `extensions.conf` | `InternalContexts` |
| `getIncludeInternal()`   | `include =>` lines inside `[internal]`           | `InternalContexts` |
| `extensionGenInternal()` | rules inside `[internal]`                        | `InternalContexts` |
| `extensionGenHints()`    | `[internal-hints]` BLF hints                     | `ExtensionsConf`   |
| `getFeatureMap()`        | `[featuremap]` in **`features.conf`**            | `FeaturesConf`     |

{% hint style="warning" %}
`getFeatureMap()` is the odd one out: it does **not** modify `extensions.conf`. Its return value lands in the `[featuremap]` section of `features.conf` — that is how star-code features are registered. It is documented here because it is part of the same fragment-return family, but keep the target file in mind.
{% endhint %}

## Add your own context

`extensionGenContexts()` returns one or more complete `[context]` blocks. This is where you put dialplan that other contexts will `include`, `Goto` or `Gosub` into. Start the fragment with a leading `PHP_EOL` so your block does not glue onto the previous module's output.

The screening context below is a **subroutine**: it is entered by an explicit `Gosub(module-black-list,s,1)` from the incoming-route hook, never by an `include`. That is why it can safely use the `s` extension.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Modules\Config\ConfigClass;
use Modules\ModuleBlackList\Models\BlackListNumbers;

class BlackListConf extends ConfigClass
{
    /**
     * Emits a dedicated context that screens the caller against the
     * m_BlackListNumbers table via an AGI script.
     */
    public function extensionGenContexts(): string
    {
        // Nothing to generate when the blacklist is empty.
        if (BlackListNumbers::count() === 0) {
            return '';
        }

        $conf  = PHP_EOL . '[module-black-list]' . PHP_EOL;
        $conf .= 'exten => s,1,NoOp(Check ${CALLERID(num)} against blacklist)' . PHP_EOL . "\t";
        $conf .= 'same => n,AGI(' . $this->moduleDir . '/agi-bin/BlackListCheck.php)' . PHP_EOL . "\t";
        $conf .= 'same => n,ExecIf($["${BLACKLISTED}" == "1"]?Hangup(21))' . PHP_EOL . "\t";
        $conf .= 'same => n,Return()' . PHP_EOL;

        return $conf;
    }
}
```
{% endcode %}

`$this->moduleDir` is a `protected string` set by `ConfigClass` to the absolute install path of the module — use it to address bundled scripts.

{% hint style="success" %}
**See the working example** in `Extensions/ModuleSmartIVR/Lib/SmartIVRConf.php` (`extensionGenContexts()` emits a `[module_smartivr]` context and routes the configured extension into an AGI), and in `Extensions/ModuleAutoprovision/Lib/AutoprovisionConf.php` (`extensionGenContexts()` builds the `[autoprovision-internal]` context).
{% endhint %}

## Wire the context into `[internal]`

A context defined with `extensionGenContexts()` is inert until something jumps to it. To make a **feature code** dialable from a handset, put that code in its own context and pull it into `[internal]` with `getIncludeInternal()`.

{% hint style="danger" %}
**Never `include` a context that contains bare-digit or wildcard extensions into `[internal]`.** `[internal]` is where real internal numbers live, and an included `_X!` / `_XXX` / `200` pattern will shadow or collide with them. This is not theoretical: a module that included a context whose menu started at digit `1` broke a real queue numbered `1` on a live PBX — dialing the queue landed in the module menu instead.

The safe shape is the one every real module uses: the included context holds **only** the star-prefixed feature code, and anything that matches broadly lives in a separate context entered by an explicit `Gosub`/`Goto`. Compare `Extensions/ModuleSmartIVR/Lib/SmartIVRConf.php:29-57` — `getIncludeInternal()` returns `include => module_smartivr` and the context contains exactly one `exten => <configured extension>` line, nothing wildcard.
{% endhint %}

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Publish ONLY the *32 feature code in its own context, then include it.
     * The wildcard screening logic stays in [module-black-list], which is
     * entered by Gosub from the incoming-route hook and is never included.
     */
    public function extensionGenContextsMenu(): string
    {
        return PHP_EOL . '[module-black-list-menu]' . PHP_EOL
             . 'exten => *32,1,NoOp(Add last caller to blacklist)' . PHP_EOL . "\t"
             . 'same => n,AGI(' . $this->moduleDir . '/agi-bin/BlackListAdd.php)' . PHP_EOL . "\t"
             . 'same => n,Hangup()' . PHP_EOL;
    }

    /**
     * Pull only the feature-code context into [internal].
     */
    public function getIncludeInternal(): string
    {
        if (BlackListNumbers::count() === 0) {
            return '';
        }

        return 'include => module-black-list-menu' . PHP_EOL;
    }
```
{% endcode %}

(`extensionGenContextsMenu()` is not a hook — it is a private helper whose output you append to what `extensionGenContexts()` returns. Only the method names listed in `AsteriskConfigInterface` are called by the Core.)

`getIncludeInternal()` is called **before** `extensionGenInternal()` (see `InternalContexts::generateAdditionalModulesInternalContext()`), so all module `include` lines are written ahead of module-supplied rules — Asterisk evaluates includes after the explicit patterns in a context, which is the behaviour you want here.

{% hint style="success" %}
**See the working example** in `Extensions/ModuleAutoprovision/Lib/AutoprovisionConf.php` — `getIncludeInternal()` returns `include => autoprovision-internal` and the matching context is built by its `extensionGenContexts()`. `Extensions/ModuleSmartIVR/Lib/SmartIVRConf.php` uses the same pairing with `include => module_smartivr`.
{% endhint %}

## Add rules directly to `[internal]`

If you need `exten =>` rules placed _inside_ `[internal]` rather than in a separate context (for example to claim a specific feature number), return them from `extensionGenInternal()`.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Register a service number (*32) inside [internal] that lets a user
     * add the last caller to the blacklist.
     */
    public function extensionGenInternal(): string
    {
        $conf  = 'exten => *32,1,NoOp(Add last caller to blacklist)' . PHP_EOL . "\t";
        $conf .= 'same => n,AGI(' . $this->moduleDir . '/agi-bin/BlackListAdd.php)' . PHP_EOL . "\t";
        $conf .= 'same => n,Playback(beep)' . PHP_EOL . "\t";
        $conf .= 'same => n,Hangup()' . PHP_EOL;

        return $conf;
    }
```
{% endcode %}

{% hint style="success" %}
**See the working example** in `Extensions/ModuleAutoDialer/Lib/AutoDialerConf.php` — `extensionGenInternal()` emits per-extension `exten => ...,AGI(...)` rules into `[internal]`.
{% endhint %}

## Declare global variables

`extensionGlobals()` returns `name=value` lines appended to the `[globals]` section of `extensions.conf`. Use it for constants your dialplan references with `${VAR}`.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Expose the module install dir to the dialplan as a global.
     */
    public function extensionGlobals(): string
    {
        return 'BLACKLIST_MODULE_DIR=' . $this->moduleDir . PHP_EOL;
    }
```
{% endcode %}

{% hint style="success" %}
**See the working example** in `Core/src/Core/Asterisk/Configs/ResParkingConf.php` — `extensionGlobals()` exports parking-lot globals into `[globals]`.
{% endhint %}

## Add BLF hints

`extensionGenHints()` feeds the `[internal-hints]` context, which backs Busy Lamp Field subscriptions on phones. Return `exten => <num>,hint,<device-or-custom>` lines.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Publish a custom-device hint so a BLF key can show whether
     * blacklist screening is currently active.
     */
    public function extensionGenHints(): string
    {
        return 'exten => *32,hint,Custom:blacklist-active' . PHP_EOL;
    }
```
{% endcode %}

{% hint style="success" %}
**See the working example** in `Core/src/Core/Asterisk/Configs/ConferenceConf.php` — `extensionGenHints()` generates conference-room hints for `[internal-hints]`.
{% endhint %}

## Register a star code (features.conf)

`getFeatureMap()` adds entries to the `[featuremap]` section of **`features.conf`** (via `FeaturesConf`), registering an in-call feature code that Asterisk's `DYNAMIC_FEATURES` machinery can invoke. Return `featurename => code` lines — the feature name is on the **left**, the DTMF code on the right, exactly as in `ResParkingConf::getFeatureMap()`, which returns `"parkcall => $this->ParkingFeature"` (`Core/src/Core/Asterisk/Configs/ResParkingConf.php:252-255`).

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Bind an in-call star code that hands the active call to a custom
     * application defined elsewhere in the dialplan.
     */
    public function getFeatureMap(): string
    {
        return 'blacklistdrop => *32' . PHP_EOL;
    }
```
{% endcode %}

{% hint style="warning" %}
Do not confuse this with `extensionGenInternal()` above. `extensionGenInternal()` puts a _dialable_ `*32` extension into `extensions.conf`; `getFeatureMap()` registers a _mid-call_ DTMF feature in `features.conf`. They are independent mechanisms; pick the one that matches the behaviour you want.
{% endhint %}

{% hint style="success" %}
**See the working example** in `Core/src/Core/Asterisk/Configs/ResParkingConf.php` — `getFeatureMap()` returns the parking feature code mapping for `[featuremap]`.
{% endhint %}

## Control ordering with getMethodPriority()

Because fragments are concatenated in ascending priority order, you override `getMethodPriority()` when your dialplan must run before or after another module's. Return a number lower than `10000` to emit earlier, higher to emit later. The signature is defined on `AsteriskConfigInterface`:

```php
public function getMethodPriority(string $methodName = ''): int;
```

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
    /**
     * Make our [internal] include/rules land before other modules' so that
     * blacklist screening happens first.
     */
    public function getMethodPriority(string $methodName = ''): int
    {
        return match ($methodName) {
            AsteriskConfigInterface::GET_INCLUDE_INTERNAL,
            AsteriskConfigInterface::EXTENSION_GEN_INTERNAL => 100,
            default => parent::getMethodPriority($methodName),
        };
    }
```
{% endcode %}

Reference the method names through the constants on `AsteriskConfigInterface` (`GET_INCLUDE_INTERNAL`, `EXTENSION_GEN_INTERNAL`, `EXTENSION_GEN_CONTEXTS`, `EXTENSION_GLOBALS`, `EXTENSION_GEN_HINTS`, `GET_FEATURE_MAP`, …) rather than raw strings — that is exactly the value `hookModulesMethod()` passes in.

## Verify the result

After installing the module and reloading the dialplan, inspect the generated file on the PBX. Your fragments appear between the `BEGIN BY` / `END BY` comment markers:

```bash
# Show the generated context and the matching include line
grep -n "module-black-list\|BEGIN BY ModuleBlackList" /etc/asterisk/extensions.conf

# Confirm the star code reached features.conf
grep -n "blacklistdrop" /etc/asterisk/features.conf

# Ask Asterisk to confirm the context loaded
asterisk -rx "dialplan show module-black-list"
```

{% hint style="info" %}
If a hook throws, `hookModulesMethod()` logs the exception via `CriticalErrorsHandler` and skips that module's fragment rather than aborting the whole generation — so a buggy module degrades to "no dialplan" instead of breaking every module's dialplan. Check the system log if your fragment silently fails to appear.
{% endhint %}

## Related pages

* [Asterisk cookbook overview](./)
* [Hooks reference](../../module-developement/hooks-reference.md)
