---
description: Recipes for integrating modules with the Asterisk dialplan, AMI and configs.
---

# Asterisk

This chapter collects task-oriented recipes for the part of a MikoPBX module that touches **Asterisk** itself: injecting your own dialplan into the generated `extensions.conf`, reacting to live calls through the Asterisk Manager Interface (AMI), and adding sections to the configuration files the core regenerates on every reload.

Everything here is built on one mechanism. When MikoPBX rebuilds a configuration file, the core generator that owns that file walks **every enabled module** and calls a named hook method on it, then splices the returned text into the file. Your module never edits `extensions.conf` or `manager.conf` directly — it returns a string from a hook, and the core decides where it lands.

Throughout the cookbook we thread a single fictional module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`, front-end asset `module-black-list-index.js`), so the recipes read as one continuous story. Each recipe is also anchored to a real, working example module under `Extensions/EXAMPLES/`.

## How a module plugs into Asterisk

A module's config class extends `MikoPBX\Modules\Config\ConfigClass`. That base class in turn extends `AsteriskConfigClass` and implements four interfaces — `SystemConfigInterface`, `RestAPIConfigInterface`, `WebUIConfigInterface` and `AsteriskConfigInterface` (`Core/src/Modules/Config/ConfigClass.php:44-48`) — so your class inherits an empty, no-op implementation of **every** Asterisk hook. You override only the hooks you care about and ignore the rest.

{% code title="Lib/BlackListConf.php" %}
```php
namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Modules\Config\ConfigClass;

class BlackListConf extends ConfigClass
{
    /**
     * Injected verbatim into the [internal] context of extensions.conf.
     * The core ExtensionsConf generator calls this on every enabled module.
     */
    public function extensionGenInternal(): string
    {
        return "exten => *789,1,NoOp(BlackList check)" . PHP_EOL
             . "\tsame => n,Hangup()" . PHP_EOL;
    }
}
```
{% endcode %}

The working reference for this pattern is `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`. It extends `ConfigClass` and overrides `generateManagerConf()` to add an AMI user — proof that a module participates by **implementing a hook**, never by subclassing a core generator.

{% hint style="info" %}
The hook **names** are the method names. They are declared as constants in `MikoPBX\Core\Asterisk\Configs\AsteriskConfigInterface` (for example `EXTENSION_GEN_INTERNAL = 'extensionGenInternal'`, `GENERATE_MANAGER_CONF = 'generateManagerConf'`). The core generator passes the matching string to `hookModulesMethod()`, which calls that method on each module. You implement the method; you do not register anything.
{% endhint %}

### Who calls your hook

Each Asterisk config file is produced by a generator named after it (`ExtensionsConf` → `extensions.conf`, `ManagerConf` → `manager.conf`, and so on). Inside `generateConfigProtected()` the generator builds the base file and then calls `hookModulesMethod()` once per extension point. A few concrete examples confirmed in the core source:

| Core generator                               | Calls hook                             | Lands in                                                                                  |
| -------------------------------------------- | -------------------------------------- | ----------------------------------------------------------------------------------------- |
| `Generators/Extensions/InternalContexts.php` | `extensionGenInternal`                 | `[internal]` context of `extensions.conf`                                                 |
| `Generators/Extensions/IncomingContexts.php` | `generateIncomingRoutBeforeDial`       | each incoming route, before the system dials                                              |
| `Generators/Extensions/IncomingContexts.php` | `generateIncomingRoutAfterDialContext` | the `[<provider>-incoming]` summary context, after the default action's `Dial()`/`Goto()` |
| `Generators/Extensions/OutgoingContext.php`  | `generateOutRoutContext`               | each outgoing route context                                                               |
| `Configs/ManagerConf.php`                    | `generateManagerConf`                  | a user section in `manager.conf`                                                          |

The full catalogue of hook methods — dialplan contexts, includes, hints, feature codes, PJSIP overrides, AMI users — lives in [the hooks reference](../../module-developement/hooks-reference.md). Use it as the authoritative map; the recipes below cover the three you will reach for first.

## Priority: who runs first

When more than one module implements the same hook, ordering matters — one module may need to set a channel variable that another reads. `hookModulesMethod()` sorts the participating modules by `getMethodPriority()` before calling them, in **ascending** order:

{% code title="Core/src/Core/Asterisk/Configs/AsteriskConfigClass.php" %}
```php
usort($arrObjects, function ($a, $b) use ($methodName) {
    return $a->getMethodPriority($methodName) - $b->getMethodPriority($methodName);
});
```
{% endcode %}

So a **lower number runs earlier**. The default a module inherits, set in `ConfigClass`, is `protected int $priority = 10000;` (the base `AsteriskConfigClass` declares `1000`, but `ConfigClass` raises it). To run before the average module, lower it; to run last, raise it. You can also override `getMethodPriority(string $methodName)` to use a different priority for a specific hook while leaving the rest at the default.

{% hint style="warning" %}
Priority orders modules **relative to each other within one hook**; it does not move your output between contexts. A `Hangup()` you return from `extensionGenInternal()` always lands in `[internal]` regardless of priority. Choose the right hook first, then tune priority only when two modules collide.
{% endhint %}

## The three recipes

### 1. Hook on an incoming call

React to a call the moment it arrives on an inbound route — before MikoPBX dials the destination. You implement `generateIncomingRoutBeforeDial($rout_number)` (or the `…PreSystem` / `…System` variants — all three fire once per route) and return dialplan that runs in the incoming context. The fourth hook, `generateIncomingRoutAfterDialContext($uniqId)`, is different: it is emitted once in the provider's summary context, not per route. This is where ModuleBlackList drops a call from a barred number, or sets a channel variable for later steps. The core invokes these hooks from `Core/src/Core/Asterisk/Configs/Generators/Extensions/IncomingContexts.php`.

➡️ [Hook on an incoming call](hook-on-incoming-call.md)

### 2. Interact with AMI

Two distinct jobs share the name "AMI", and a module typically does both:

* **Declare an AMI user** so something can connect. You override `generateManagerConf()` and return a `[user]` section for `manager.conf`. The example in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` emits exactly this, scoped to localhost.
* **Be the AMI client at runtime** — connect with `MikoPBX\Core\Asterisk\AsteriskManager`, send actions (`sendRequestTimeout()`, `Originate()`, `Hangup()`), and subscribe to live events (`addEventHandler()`) from a background worker. The reference worker is `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php`.

➡️ [Interact with AMI](interact-with-ami.md)

### 3. Modify extensions.conf

Add your own dialplan to the file that drives every call. Depending on what you need, you return a string from `extensionGenInternal()` (extensions in `[internal]`), `extensionGenContexts()` (a whole new context), `extensionGenHints()` (BLF/presence hints), or `extensionGlobals()` (global variables). ModuleBlackList uses this recipe to publish its `*789` maintenance extension and a private `[blacklist-check]` context that the incoming-call hook jumps into.

➡️ [Modify extensions.conf](modify-extensions.conf.md)

## Verifying your dialplan

After a module change, trigger a reload and inspect the result on the box with the Asterisk CLI:

{% code title="bash" %}
```bash
# Show what actually landed in the dialplan
asterisk -rx "dialplan show internal"
asterisk -rx "dialplan show blacklist-check"

# Confirm your AMI user was written to manager.conf
asterisk -rx "manager show users"
```
{% endcode %}

Your contributed block is wrapped in `; ***** BEGIN BY <moduleUniqueId>` / `; ***** END BY <moduleUniqueId>` comment markers in the generated file (see `confBlockWithComments()` in `AsteriskConfigClass.php`), which makes it easy to spot which module produced which lines.

## Where to go next

* The complete list of every hook, its signature, and the file it writes: [hooks reference](../../module-developement/hooks-reference.md).
* Building the admin UI that stores the data these hooks read: [Forms cookbook](../forms/).
* Then work the three recipes above in the order that fits your module — most start with recipe 3 (publish a context), add recipe 1 (act on incoming calls), and reach for recipe 2 only when they need a live AMI connection.
