---
description: >-
  The ConfigClass-based hub: features, hooks, REST API methods, and PBX core
  interaction.
---

# Module main class

Every MikoPBX module has exactly one **configuration class** — the single hub that wires the module into the PBX lifecycle. It lives in `Lib/{Feature}Conf.php`, extends `MikoPBX\Modules\Config\ConfigClass`, and is the only place the Core ever calls into your code automatically: on PBX start, on database changes, when Asterisk configs are regenerated, when an admin enables or disables the module, when a REST API request arrives, or when the web interface renders.

Throughout this guide the running example is a fictional module **ModuleBlackList** whose config class is `BlackListConf` and whose business logic lives in `BlackListMain`. For each pattern you will also find a pointer to a real, working module you can read in the repository.

{% hint style="info" %}
Read the real reference implementations alongside this page:

* `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` — REST API, workers, `manager.conf` generation, enable/disable hooks.
* `Extensions/ModuleUsersUI/Lib/UsersUIConf.php` — ACL, authentication, Volt blocks, form hooks, menu items, CDR filtering.
{% endhint %}

## Where the class sits

`{Feature}Conf` does not extend `ConfigClass` directly out of thin air — it sits at the bottom of a small inheritance chain defined in the Core:

```
{Feature}Conf
  └── MikoPBX\Modules\Config\ConfigClass            (abstract)
        └── MikoPBX\Core\Asterisk\Configs\AsteriskConfigClass
              └── Phalcon\Di\Injectable
```

Because `ConfigClass` extends `AsteriskConfigClass` (which extends `Phalcon\Di\Injectable`), your config class is dependency-injection aware out of the box: `$this->di`, `$this->config`, `$this->mikoPBXConfig`, `$this->messages`, `$this->moduleUniqueId`, and `$this->moduleDir` are all available without any wiring on your part.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Modules\Config\ConfigClass;
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;

/**
 * The single hub that wires ModuleBlackList into the PBX lifecycle.
 */
class BlackListConf extends ConfigClass
{
    // Hook applying priority — lower runs earlier. Default is 10000.
    protected int $priority = 10000;

    // ... thin hook methods that delegate to BlackListMain ...
}
```
{% endcode %}

{% hint style="info" %}
The namespace must be `Modules\{ModuleUniqueID}\Lib`. The `ConfigClass` constructor inspects the namespace via reflection and derives `$this->moduleUniqueId` and `$this->moduleDir` from it. See `ConfigClass::__construct()` in `Core/src/Modules/Config/ConfigClass.php`. Get the namespace wrong and the Core cannot locate your module directory.
{% endhint %}

## The four interfaces

`ConfigClass` implements four interfaces, one per area of the PBX it can hook into. You never implement them yourself — you simply override the methods you need; everything else inherits a safe no-op default from `ConfigClass`.

| Interface                 | Concern                                                                                            | Examples of overrides                                                                     |
| ------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `SystemConfigInterface`   | OS-level lifecycle: cron, firewall, fail2ban, nginx, PBX start, enable/disable, background workers | `createCronTasks()`, `onAfterPbxStarted()`, `onAfterModuleEnable()`, `getModuleWorkers()` |
| `RestAPIConfigInterface`  | PBXCoreREST endpoints handled under root rights                                                    | `getPBXCoreRESTAdditionalRoutes()`, `moduleRestAPICallback()`                             |
| `WebUIConfigInterface`    | Admin Cabinet: assets, ACL, menu, forms, Volt blocks, routes                                       | `onBeforeHeaderMenuShow()`, `onAfterAssetsPrepared()`, `onAfterACLPrepared()`             |
| `AsteriskConfigInterface` | Asterisk config file fragments: `extensions.conf`, `pjsip.conf`, `manager.conf`, `features.conf`   | `generateManagerConf()`, `extensionGenContexts()`, `generatePeersPj()`                    |

The class declaration in the Core spells this out:

{% code title="Core/src/Modules/Config/ConfigClass.php" %}
```php
abstract class ConfigClass extends AsteriskConfigClass implements
    SystemConfigInterface,
    RestAPIConfigInterface,
    WebUIConfigInterface,
    AsteriskConfigInterface
```
{% endcode %}

The full, signature-by-signature catalogue of every overridable method across all four interfaces lives in [hooks-reference.md](hooks-reference.md). This page covers the architecture and the handful of overrides almost every module needs.

## Priority and ordering

`ConfigClass` declares a single protected field that controls when your module's hooks run relative to other installed modules:

{% code title="Core/src/Modules/Config/ConfigClass.php" %}
```php
// The module hook applying priority
protected int $priority = 10000;
```
{% endcode %}

When the Core fans a hook out to all modules, it sorts them by priority — **lower numbers run earlier**. Most modules leave the default `10000`. If a single module needs different ordering for different hooks (run its cron generator early but its `pjsip.conf` peer generator late), override `getMethodPriority()` and key off the interface constants rather than hard-coded strings:

```php
use MikoPBX\Modules\Config\SystemConfigInterface;
use MikoPBX\Core\Asterisk\Configs\AsteriskConfigInterface;

public function getMethodPriority(string $methodName = ''): int
{
    return match ($methodName) {
        SystemConfigInterface::CREATE_CRON_TASKS   => 200,   // run early
        AsteriskConfigInterface::GENERATE_PEERS_PJ => 50000, // run late
        default => $this->priority,
    };
}
```

The constants `SystemConfigInterface::CREATE_CRON_TASKS` (`'createCronTasks'`) and `AsteriskConfigInterface::GENERATE_PEERS_PJ` (`'generatePeersPj'`) are defined in the Core interfaces — use them so a future method rename does not silently break your ordering.

## Anti-pattern #3: keep Conf.php thin

{% hint style="danger" %}
**The single most important rule for this class.** `{Feature}Conf.php` is a _wiring_ file, not a _logic_ file. Each hook method should be a few lines that delegate to your `{Feature}Main.php` (and other helper classes). A `Conf.php` that grows past a few hundred lines of business logic is [anti-pattern #3 — monolithic classes](best-practices.md). It becomes impossible to unit-test, because the Core can only reach it through the hook entry points.
{% endhint %}

The reference modules follow this rule. In `ExampleAmiConf`, every lifecycle hook hands off to `ExampleAmiMain`:

{% code title="Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php" %}
```php
public function onAfterModuleEnable(): void
{
    System::invokeActions(['manager' => 0]);

    $templateMain = new ExampleAmiMain();
    $templateMain->startAllServices();
}
```
{% endcode %}

For ModuleBlackList you would do exactly the same: `BlackListConf` reacts to the hook, `BlackListMain` does the work.

```php
public function onAfterModuleEnable(): void
{
    (new BlackListMain())->startAllServices();
}
```

## The overrides almost every module needs

### modelsEventChangeData($data)

Called from `WorkerModelsEvents` after **each** database model change anywhere in the PBX. Use it to react to settings changes — restart workers, clear caches, regenerate state. The `$data` array carries `model` (the fully-qualified model class) and `recordId` (the changed primary key).

{% hint style="warning" %}
This fires for _every_ model change on the system, not just your module's. Always guard with a `model`/`recordId` check before doing any work, or you will run expensive logic on unrelated edits.
{% endhint %}

```php
use MikoPBX\Common\Models\PbxSettings;

public function modelsEventChangeData($data): void
{
    // Restart workers when the PBX language changes, to reload translations.
    if (
        $data['model'] === PbxSettings::class
        && $data['recordId'] === 'PBXLanguage'
    ) {
        (new BlackListMain())->startAllServices(true);
    }
}
```

See the working example in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`. `ModuleUsersUI` uses the same hook to clear its ACL cache when its own models change — see `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`:

{% code title="Extensions/ModuleUsersUI/Lib/UsersUIConf.php" %}
```php
public function modelsEventChangeData($data): void
{
    $cacheInterfereModels = [
        AccessGroups::class,
        AccessGroupsRights::class,
    ];

    if (in_array($data['model'], $cacheInterfereModels)) {
        AclProvider::clearCache();
    }

    if ($data['model'] === Users::class) {
        UsersCredentials::cleanOrphanRecords();
    }
}
```
{% endcode %}

### getModuleWorkers()

Returns the list of long-running background workers the Core should keep alive for your module. Despite being about processes rather than the OS, it is declared on `SystemConfigInterface` (`Core/src/Modules/Config/SystemConfigInterface.php`, constant `GET_MODULE_WORKERS`). `WorkerSafeScriptsCore` monitors and restarts them. Each entry is `['type' => ..., 'worker' => WorkerClass::class]`.

The `type` selects the liveness-check strategy and is one of the constants defined in `Core/src/Core/Workers/Cron/WorkerSafeScriptsCore.php`:

| Constant                                        | Value                  | Use for                                             |
| ----------------------------------------------- | ---------------------- | --------------------------------------------------- |
| `WorkerSafeScriptsCore::CHECK_BY_AMI`           | `checkWorkerAMI`       | Workers driven by Asterisk Manager Interface events |
| `WorkerSafeScriptsCore::CHECK_BY_BEANSTALK`     | `checkWorkerBeanstalk` | Workers consuming a Beanstalk queue                 |
| `WorkerSafeScriptsCore::CHECK_BY_REDIS`         | `checkWorkerRedis`     | Workers consuming a Redis queue                     |
| `WorkerSafeScriptsCore::CHECK_BY_PID_NOT_ALERT` | `checkPidNotAlert`     | Plain PID-based liveness without alerting           |

{% code title="Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php" %}
```php
public function getModuleWorkers(): array
{
    return [
        [
            'type'   => WorkerSafeScriptsCore::CHECK_BY_AMI,
            'worker' => WorkerExampleAmiAMI::class,
        ],
    ];
}
```
{% endcode %}

For ModuleBlackList, register your queue worker the same way:

```php
public function getModuleWorkers(): array
{
    return [
        [
            'type'   => WorkerSafeScriptsCore::CHECK_BY_BEANSTALK,
            'worker' => WorkerBlackList::class,
        ],
    ];
}
```

Worker implementation is covered in detail on [workers.md](workers.md).

### moduleRestAPICallback(array $request): PBXApiResult

The entry point for PBXCoreREST requests routed to your module. It runs **under root rights**, so treat every input as untrusted. The method must return a `MikoPBX\PBXCoreREST\Lib\PBXApiResult` — an object with `bool $success`, `array $data`, `array $messages`, and `string $processor` (see `Core/src/PBXCoreREST/Lib/PBXApiResult.php`).

Dispatch on the `action` field with an explicit `match` — never call a method named by user input (that is security anti-pattern S6).

{% code title="Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php" %}
```php
public function moduleRestAPICallback(array $request): PBXApiResult
{
    $action = $request['action'] ?? '';
    $data   = $request['data'] ?? [];

    return match ($action) {
        'sendCommand' => $this->sendAmiCommandAction($data),
        'check'       => $this->checkAction(),
        'reload'      => $this->reloadAction(),
        default       => $this->createErrorResult("Unknown action: {$action}")
    };
}
```
{% endcode %}

A ModuleBlackList equivalent stays equally thin, delegating the real lookup to `BlackListMain`:

```php
public function moduleRestAPICallback(array $request): PBXApiResult
{
    $res            = new PBXApiResult();
    $res->processor = __METHOD__;

    switch (strtoupper($request['action'] ?? '')) {
        case 'CHECK':
            $res = (new BlackListMain())->checkModuleWorkProperly();
            break;
        case 'ISBLOCKED':
            $res->success = true;
            $res->data    = ['blocked' => (new BlackListMain())->isBlocked($request['number'] ?? '')];
            break;
        default:
            $res->success             = false;
            $res->messages['error'][] = 'API action not found in ' . __METHOD__;
    }

    return $res;
}
```

To expose your own URL paths (rather than the implicit module callback route), override `getPBXCoreRESTAdditionalRoutes()`. Both methods are described fully in [hooks-reference.md](hooks-reference.md).

{% hint style="danger" %}
Only mark a route `noAuth = true` (the 6th element of a route tuple) for endpoints that expose nothing sensitive and change no state — this is security anti-pattern S1. Anything that touches credentials, CDR data, calls, or settings must require authentication.
{% endhint %}

### onBeforeHeaderMenuShow(array &$menuItems)

Adds entries to the Admin Cabinet sidebar. The argument is passed **by reference** — mutate `$menuItems` in place. Keys are either a controller class (for a direct link) or a custom string (for a submenu group). `ModuleUsersUI` adds a "My Profile" link only for users who logged in through the module:

{% code title="Extensions/ModuleUsersUI/Lib/UsersUIConf.php" %}
```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $role = $this->extractUserRole();

    if ($role !== null && strpos($role, Constants::MODULE_ROLE_PREFIX) === 0) {
        $menuItems[UserProfileController::class] = [
            'caption'   => 'module_usersui_MyProfile',
            'iconclass' => 'user circle',
            'action'    => 'index',
            'param'     => '',
            'style'     => '',
        ];
    }
}
```
{% endcode %}

A submenu group (as ModuleBlackList would use for its admin pages) is keyed by a custom string and carries a `submenu` array:

```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $menuItems['ModuleBlackListAdmin'] = [
        'caption'   => 'module_blacklist_MainMenuItem',
        'iconclass' => 'ban',
        'submenu'   => [
            '/module-black-list/index' => [
                'caption'   => 'module_blacklist_Numbers',
                'iconclass' => 'list',
                'action'    => 'index',
                'param'     => '',
                'style'     => '',
            ],
        ],
    ];
}
```

### onAfterModuleEnable() / onAfterModuleDisable()

Called after an admin toggles the module in the web interface. This is where you reload the parts of the PBX your module affects and start or stop your services. Reload through the Core's `PBX` / `System` helpers — never shell out to Asterisk directly (anti-pattern #2).

`ExampleAmiConf` reloads the Asterisk manager (because it added an AMI user via `generateManagerConf()`) and starts its workers:

{% code title="Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php" %}
```php
public function onAfterModuleEnable(): void
{
    System::invokeActions(['manager' => 0]);
    (new ExampleAmiMain())->startAllServices();
}

public function onAfterModuleDisable(): void
{
    System::invokeActions(['manager' => 0]);
}
```
{% endcode %}

`UsersUIConf` instead clears its ACL cache and prunes orphaned records — a good illustration that "enable/disable" means "make the system consistent with the new state", whatever that means for your module:

{% code title="Extensions/ModuleUsersUI/Lib/UsersUIConf.php" %}
```php
public function onAfterModuleEnable(): void
{
    UsersCredentials::cleanOrphanRecords();
    AccessGroupCDRFilter::cleanOrphanRecords();
    AclProvider::clearCache();
}

public function onAfterModuleDisable(): void
{
    AclProvider::clearCache();
}
```
{% endcode %}

There are also `onBeforeModuleEnable(): bool` and `onBeforeModuleDisable(): bool` guards — return `false` to veto the action (for example, refuse to enable while a prerequisite is missing).

## How the Core calls into your class

Knowing _when_ each hook fires makes the class easier to reason about:

* **PBX boot** — `onAfterPbxStarted()` after the dialplan and services are up.
* **Config regeneration** — the `AsteriskConfigInterface` generators (`generateManagerConf()`, `extensionGenContexts()`, `generatePeersPj()`, …) are collected from every module and concatenated into the relevant `.conf` file, ordered by priority.
* **Database changes** — `modelsEventChangeData($data)` per change, then `modelsEventNeedReload(array $plannedReloadActions)` once the batch settles.
* **Admin Cabinet request** — `onBeforeExecuteRoute()` / `onAfterExecuteRoute()`, asset preparation (`onAfterAssetsPrepared()`), ACL build (`onAfterACLPrepared()`), menu render (`onBeforeHeaderMenuShow()`), Volt compilation (`onVoltBlockCompile()`).
* **REST API request** — routed into `moduleRestAPICallback()` (or a custom route), wrapped by `onBeforeExecuteRestAPIRoute()` / `onAfterExecuteRestAPIRoute()`.
* **Admin toggles the module** — `onBeforeModuleEnable()` → enable → `onAfterModuleEnable()` (and the disable counterparts).

Every one of these methods has a no-op default in `ConfigClass`, so an empty module is valid — you pay only for the hooks you override.

## Checklist

* [ ] Class is named `{Feature}Conf`, lives in `Lib/`, and is in namespace `Modules\{ModuleUniqueID}\Lib`.
* [ ] Extends `MikoPBX\Modules\Config\ConfigClass`.
* [ ] File starts with `declare(strict_types=1);` (anti-pattern #20).
* [ ] Every hook method is thin and delegates to `{Feature}Main` (anti-pattern #3).
* [ ] REST dispatch uses an explicit `match`/`switch`, never `$this->$action()` (security S6).
* [ ] Reloads go through `PBX` / `System` helpers, never raw `shell_exec` (anti-pattern #2).
* [ ] No `MikoPBXVersion.php` — this is a 2025.1.1+ module (anti-pattern #1).

## Related pages

* [module-anatomy.md](module-anatomy.md) — where `Lib/{Feature}Conf.php` fits in the overall directory layout.
* [hooks-reference.md](hooks-reference.md) — the complete signature-by-signature catalogue of every overridable hook.
* [workers.md](workers.md) — implementing the worker classes returned by `getModuleWorkers()`.
* [best-practices.md](best-practices.md) — the full anti-pattern list referenced above.
