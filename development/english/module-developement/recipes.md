---
description: 'Capability building blocks: the recipes that compose a MikoPBX module.'
---

# Recipes

A MikoPBX module is rarely a single thing. A real module is a _combination_ of capabilities: a settings page, a background worker, a few dialplan hooks, maybe a REST resource. The **recipe system** is the vocabulary we use to talk about those capabilities one at a time.

A **recipe** is a self-contained capability. Each recipe answers three questions:

1. **What files does it add** to the module tree?
2. **Which hooks or classes does it use** from the Core?
3. **Where is the canonical working example** in the repository?

This vocabulary is shared by two consumers:

* **Hand-coders** — you pick the recipes your module needs and copy the patterns from the linked examples.
* **The `/mikopbx-module` AI skill** — it generates modules by selecting and composing the same recipes. See [what the skill generates](../ai-assisted-development/what-it-generates.md).

Everything below is anchored to the **running example** of this guide — a fictional module called **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` backed by table `m_BlackListNumbers`, front-end script `module-black-list-index.js`) — and to **real, shipping example modules** under `Extensions/EXAMPLES/`.

{% hint style="info" %}
Recipes are a documentation and generation convention, not a runtime framework. There is no `Recipe` class. When you "add the workers recipe", you are simply adding the files and the `getModuleWorkers()` hook that the recipe describes.
{% endhint %}

## The recipe catalog

| Recipe       | Adds                                                                                               | Integration surface                                                                                         |
| ------------ | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **base**     | `module.json`, `Setup/PbxExtensionSetup.php`, ≥1 Model, `Lib/{Feature}Conf.php`, `Messages/ru.php` | Always present — the skeleton every module shares                                                           |
| **ui**       | Controllers, Form, Volt views, JS, CSS                                                             | Phalcon MVC + `onBeforeHeaderMenuShow()`                                                                    |
| **rest-api** | `Lib/RestAPI/{Resource}/…` (attribute-routed)                                                      | Auto-discovered v3 controllers — no manual routes                                                           |
| **dialplan** | Methods on `Lib/{Feature}Conf.php`                                                                 | `AsteriskConfigInterface` hook methods                                                                      |
| **agi**      | `agi-bin/{script}.php`                                                                             | `MikoPBX\Core\Asterisk\AGI` + a dialplan hook                                                               |
| **workers**  | `bin/Worker*.php`                                                                                  | `getModuleWorkers()`                                                                                        |
| **firewall** | Methods on the Conf class                                                                          | `getDefaultFirewallRules()`, `onAfterIptablesReload()`, `generateFail2BanJails()`                           |
| **acl**      | Methods on the Conf class                                                                          | `onAfterACLPrepared()`, `applyACLFiltersToCDRQuery()`, `onGetControllerPermissions()`, `authenticateUser()` |
| **system**   | Methods on the Conf class                                                                          | `createCronTasks()`, `createNginxLocations()`, `onAfterPbxStarted()`, …                                     |

Every method named above is a real hook. Their full signatures and timing are documented in the [hooks reference](hooks-reference.md). This page shows how each recipe _uses_ them; the reference page is the exhaustive list.

***

## Recipe: base (always included)

Every module — including ModuleBlackList — ships the base recipe. Without it there is nothing to install.

**Files added:**

```
ModuleBlackList/
├── module.json                       # metadata: uniqid, version, dependencies
├── Setup/PbxExtensionSetup.php        # install / uninstall lifecycle
├── Models/BlackListNumbers.php        # ≥1 model (table m_BlackListNumbers)
├── Lib/BlackListConf.php              # config class — extends ConfigClass
└── Messages/ru.php                    # Russian translation keys (en.php optional)
```

**Integration surface:** the heart of the base recipe is the config class. It extends `MikoPBX\Modules\Config\ConfigClass`, which is `abstract` and itself `extends AsteriskConfigClass`. That single inheritance is what makes _every_ hook in every other recipe available to your module — dialplan, workers, firewall, ACL and system hooks are all inherited from this one base class.

{% code title="Lib/BlackListConf.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Modules\Config\ConfigClass;

class BlackListConf extends ConfigClass
{
    // Hooks from any recipe below are added here as you compose the module.
}
```
{% endcode %}

{% hint style="warning" %}
The config class name is the _feature_ name plus `Conf` (`BlackListConf`), **not** the full module name. The module directory and `module.json` uniqid use the `Module` prefix (`ModuleBlackList`); the Lib class drops it. This mirrors the shipping examples — `ModuleExampleForm` → `ExampleFormConf`.
{% endhint %}

**See the working example in** `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` — specifically:

* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Setup/PbxExtensionSetup.php`
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php`
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Models/ModuleExampleForm.php`
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Messages/ru.php`

***

## Recipe: ui (web interface)

The ui recipe gives ModuleBlackList a settings page inside the MikoPBX admin cabinet.

**Files added:**

```
App/Controllers/ModuleBlackListController.php   # main controller (Phalcon MVC)
App/Forms/ModuleBlackListForm.php               # Phalcon form
App/Views/ModuleBlackList/index.volt            # Volt template
public/assets/js/src/module-black-list-index.js  # ES6+ source (babel-compiled)
public/assets/css/module-black-list-index.css    # styles
```

Asset filenames are **per controller action**, not per module: the `index` action loads `module-black-list-index.*`, a `modify` action would load `module-black-list-modify.*`. This is the convention the Core example follows (`module-example-form-index.js`, `module-example-form-modify.js`), and the controller below hardcodes those names, so they must match exactly.

### How assets are registered (important)

Modules do **not** ship a standalone `AssetProvider.php` file. The Core already provides `MikoPBX\AdminCabinet\Providers\AssetProvider`, and you reference its collection constants from inside your controller action. This is the verified pattern from the shipping example:

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\App\Controllers;

use MikoPBX\AdminCabinet\Controllers\BaseController;
use MikoPBX\AdminCabinet\Providers\AssetProvider;

class ModuleBlackListController extends BaseController
{
    private string $moduleUniqueID = 'ModuleBlackList';

    public function indexAction(): void
    {
        $headerCollectionCSS = $this->assets->collection(AssetProvider::HEADER_CSS);
        $headerCollectionCSS->addCss(
            'css/cache/' . $this->moduleUniqueID . '/module-black-list-index.css',
            true
        );

        $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
        $footerCollectionJS->addJs(
            'js/cache/' . $this->moduleUniqueID . '/module-black-list-index.js',
            true
        );
    }
}
```
{% endcode %}

A module controller extends the Core class `MikoPBX\AdminCabinet\Controllers\BaseController` (there is no separate "module controller" base class). `AssetProvider::HEADER_CSS` and `AssetProvider::FOOTER_JS` are the verified collection names; `$moduleUniqueID` is a property you declare on the controller (set to your module's uniqid) so cached assets are namespaced per module.

### How the menu item is added

The sidebar entry is **not** a `MenuProvider.php` file either — it is the `onBeforeHeaderMenuShow()` hook on your config class. The Core calls it while building the admin menu and passes the menu array by reference:

{% code title="Lib/BlackListConf.php" %}
```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $menuItems['module_blacklist_MenuItem'] = [
        'caption'   => 'module_blacklist_MenuItem',
        'iconclass' => 'ban',
        'submenu'   => [
            '/module-black-list/index' => [
                'caption'   => 'module_blacklist_SubMenuItem',
                'iconclass' => 'list',
                'action'    => 'index',
                'param'     => '',
                'style'     => '',
            ],
        ],
    ];
}
```
{% endcode %}

The `caption` values are translation keys resolved from `Messages/ru.php` (and `en.php`).

{% hint style="info" %}
**Post-generation step:** JS under `public/assets/js/src/` is ES6+ source and must be transpiled with babel into `public/assets/js/`. The `/mikopbx-module` skill runs this for you; by hand, use the babel pipeline.
{% endhint %}

**See the working example in** `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`:

* Controller — `App/Controllers/ModuleExampleFormController.php` (the `indexAction()` / `modifyAction()` asset pattern shown above is copied verbatim from here)
* Form — `App/Forms/ModuleExampleFormForm.php`
* View — `App/Views/ModuleExampleForm/index.volt`
* JS source — `public/assets/js/src/module-example-form-index.js`
* Menu hook — `Lib/ExampleFormConf.php` (`onBeforeHeaderMenuShow()`)

***

## Recipe: rest-api (REST API v3)

This recipe exposes ModuleBlackList data over HTTP. The 2025 v3 pattern is **attribute-routed and auto-discovered** — you write PHP 8 attributes on a controller, and `ControllerDiscovery` registers the routes. **You never register a route manually**, and there is **no** Conf hook for this.

**Files added per resource:**

```
Lib/RestAPI/Numbers/Controller.php       # HTTP layer — PHP 8 attributes only
Lib/RestAPI/Numbers/Processor.php        # routes requests to Action classes
Lib/RestAPI/Numbers/DataStructure.php    # parameter + OpenAPI schema
Lib/RestAPI/Numbers/Actions/GetListAction.php
Lib/RestAPI/Numbers/Actions/GetRecordAction.php
Lib/RestAPI/Numbers/Actions/SaveRecordAction.php
Lib/RestAPI/Numbers/Actions/DeleteRecordAction.php
```

The controller declares routing entirely through attributes; its method bodies are intentionally empty (`{}`) because the Processor and Actions do the work:

{% code title="Lib/RestAPI/Numbers/Controller.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers;

use MikoPBX\PBXCoreREST\Controllers\BaseRestController;
use MikoPBX\PBXCoreREST\Attributes\{
    ApiResource,
    HttpMapping,
    ResourceSecurity,
    SecurityType,
    ApiOperation,
    ApiResponse
};

#[ApiResource(
    path: '/pbxcore/api/v3/module-black-list/numbers',
    tags: ['Module BlackList - Numbers'],
    description: 'Blacklisted number management',
    processor: Processor::class
)]
#[HttpMapping(
    mapping: [
        'GET'    => ['getList', 'getRecord'],
        'POST'   => ['create'],
        'DELETE' => ['delete'],
    ],
    resourceLevelMethods: ['getRecord', 'delete'],
    collectionLevelMethods: ['getList', 'create'],
)]
#[ResourceSecurity(
    'module-black-list-numbers',
    requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN]
)]
class Controller extends BaseRestController
{
    protected string $processorClass = Processor::class;

    #[ApiOperation(summary: 'rest_numbers_GetList', operationId: 'getNumbersList')]
    #[ApiResponse(200, 'rest_response_200_list')]
    public function getList(): void {}

    // ... getRecord(), create(), delete() — empty bodies, attributes only.
}
```
{% endcode %}

Key facts, all verified against the shipping v3 example:

* `#[ApiResource]` marks the class auto-discoverable; `ControllerDiscovery` scans `Lib/RestAPI` for `*Controller.php` and `RouterProvider` registers the routes.
* `#[HttpMapping]` maps HTTP verbs to method names and splits them into resource-level (`/numbers/{id}`) vs collection-level (`/numbers`) operations.
* `#[ResourceSecurity]` declares auth requirements with `SecurityType` enum values — `SecurityType::LOCALHOST`, `SecurityType::BEARER_TOKEN`.
* `BaseRestController` is the base class; `$processorClass` points at your `Processor`.

For the full attribute set, the 7-phase Action lifecycle, and `DataStructure` / `OpenApiSchemaProvider`, see [REST API in modules](rest-api-in-modules.md).

**See the working example in** `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`:

* `Lib/RestAPI/Tasks/Controller.php`
* `Lib/RestAPI/Tasks/Processor.php`
* `Lib/RestAPI/Tasks/DataStructure.php`
* `Lib/RestAPI/Tasks/Actions/SaveRecordAction.php`

***

## Recipe: dialplan (Asterisk dialplan hooks)

The dialplan recipe adds **no new files** — it adds methods to `Lib/BlackListConf.php`. Because `ConfigClass` implements `AsteriskConfigInterface`, every method below is a real, overridable hook. Implement only the ones you need.

| Hook                                                   | When it fires                                 | Returns                 |
| ------------------------------------------------------ | --------------------------------------------- | ----------------------- |
| `extensionGenContexts()`                               | Build custom dialplan contexts                | `string` (context text) |
| `extensionGenInternal()`                               | Add rules to `[internal]`                     | `string`                |
| `getIncludeInternal()`                                 | `#include` a custom context into `[internal]` | `string`                |
| `generateIncomingRoutBeforeDial(string $rout_number)`  | Modify an incoming route before `Dial()`      | `string`                |
| `generateIncomingRoutAfterDialContext(string $uniqId)` | Modify an incoming route after `Dial()`       | `string`                |
| `generateOutRoutContext(array $rout)`                  | Modify an outgoing route after `EXTEN` is set | `string`                |
| `generateOutRoutAfterDialContext(array $rout)`         | Modify an outgoing route after `Dial()`       | `string`                |
| `extensionGlobals()`                                   | Add `[globals]` variables                     | `string`                |
| `extensionGenHints()`                                  | Add BLF hints                                 | `string`                |
| `getFeatureMap()`                                      | Add star-codes to `featuremap`                | `string`                |

For ModuleBlackList, the natural hook is `generateIncomingRoutBeforeDial()` — intercept the inbound call before it rings and route blacklisted callers into an AGI check:

{% code title="Lib/BlackListConf.php" %}
```php
public function generateIncomingRoutBeforeDial(string $rout_number): string
{
    return "\t" . 'same => n,AGI(module-black-list.php)' . "\n";
}
```
{% endcode %}

{% hint style="warning" %}
These methods return **dialplan text** that the Core stitches into the generated `extensions.conf`. Return an empty string (`''`) when a given call does not concern your module — never `null`.
{% endhint %}

**See working examples in** real shipping modules:

* `Extensions/ModulePhoneBook/Lib/PhoneBookConf.php` — AGI injection on incoming calls
* `Extensions/ModuleAutoDialer/Lib/AutoDialerConf.php` — multi-context dialplan generation

The verified hook _signatures_ live in `Core/src/Core/Asterisk/Configs/AsteriskConfigInterface.php`.

***

## Recipe: agi (AGI scripts)

The agi recipe pairs with the dialplan recipe: a dialplan hook calls `AGI(script.php)`, and that script reads/writes channel variables and runs dialplan applications.

**Files added:**

```
agi-bin/module-black-list.php     # AGI script (symlinked into Asterisk's agi-bin on install)
```

### Correct AGI API

{% hint style="danger" %}
Use the Core class `MikoPBX\Core\Asterisk\AGI` and its **snake\_case** methods: `get_variable()`, `set_variable()`, `exec()`. Instantiate it with `new AGI()`.

Do **not** write `AGI\AgiClient`, `getVariable()` or `setVariable()` — that class and those camelCase methods do not exist in this codebase. Every shipping AGI script (`ModuleAutoDialer`, `ModuleQualityAssessment`, `ModulePhoneBook`, …) uses `new AGI()` with snake\_case methods. Verify against `Core/src/Core/Asterisk/AGI.php`.
{% endhint %}

{% code title="agi-bin/module-black-list.php" %}
```php
#!/usr/bin/php
<?php

declare(strict_types=1);

use MikoPBX\Core\Asterisk\AGI;
use Modules\ModuleBlackList\Models\BlackListNumbers;

require_once 'Globals.php';

$agi = new AGI();

// Read a channel variable (second arg true => return the value only).
$callerNum = $agi->get_variable('CALLERID(num)', true);

// Always bind parameters — never interpolate channel data into a condition.
$blocked = BlackListNumbers::findFirst([
    'conditions' => 'number = :num:',
    'bind'       => ['num' => $callerNum],
]);
if ($blocked !== null) {
    // Mark the call and route it via a dialplan application.
    $agi->set_variable('BLACKLISTED', '1');
    $agi->exec('Playback', 'silence/1');
    $agi->exec('Hangup', '');
}

exit(0);
```
{% endcode %}

The method signatures, verified in `Core/src/Core/Asterisk/AGI.php`:

* `get_variable(string $variable, bool $getvalue = false): array|string` — pass `true` to get the trimmed value directly.
* `set_variable(string $variable, string $value): array`
* `exec(string $application, mixed $options): array`

Channel request data is also available as `$agi->request['agi_callerid']` (and similar `agi_*` keys), as used by `ModuleAutoDialer`.

{% hint style="danger" %}
AGI scripts run as **root** and every value they read from the channel is attacker-influenced (`CALLERID(num)` comes straight off the SIP wire). Bind every ORM parameter as shown above, and wrap every shell argument in `escapeshellarg()`. See anti-patterns [S2](best-practices.md#s2-critical-sql-injection-in-findfirst-find) and [S3](best-practices.md#s3-critical-command-injection).
{% endhint %}

**See working examples in** real shipping modules:

* `Extensions/ModuleQualityAssessment/agi-bin/quality_agi.php`
* `Extensions/ModuleAutoDialer/agi-bin/get-client-info.php`
* `Extensions/ModulePhoneBook/agi-bin/agi_phone_book.php`

***

## Recipe: workers (background processes)

The workers recipe gives ModuleBlackList a long-running background process — for example, periodically syncing the blacklist from an external feed.

**Files added:**

```
Lib/WorkerBlackListMain.php       # worker class (extends WorkerBase)
```

(The shipping example keeps its worker classes under `Lib/`; `getModuleWorkers()` references the worker by its fully-qualified class name, so any autoloadable location works.)

**Integration surface — the `getModuleWorkers()` hook:**

{% code title="Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;

public function getModuleWorkers(): array
{
    return [
        [
            'type'   => WorkerSafeScriptsCore::CHECK_BY_BEANSTALK,
            'worker' => WorkerBlackListMain::class,
        ],
    ];
}
```
{% endcode %}

`WorkerSafeScriptsCore` supervises and restarts your worker. The verified `type` constants are:

| Constant                                        | Value                  | Use case                                   |
| ----------------------------------------------- | ---------------------- | ------------------------------------------ |
| `WorkerSafeScriptsCore::CHECK_BY_BEANSTALK`     | `checkWorkerBeanstalk` | Beanstalk queue / event processing         |
| `WorkerSafeScriptsCore::CHECK_BY_AMI`           | `checkWorkerAMI`       | Real-time AMI call-event tracking          |
| `WorkerSafeScriptsCore::CHECK_BY_REDIS`         | `checkWorkerRedis`     | Workers that report liveness through Redis |
| `WorkerSafeScriptsCore::CHECK_BY_PID_NOT_ALERT` | `checkPidNotAlert`     | Long-running daemons (PID monitored)       |

A module can register multiple workers — the shipping `ExampleFormConf::getModuleWorkers()` returns both a Beanstalk worker and an AMI worker.

For the worker base class, the `start($argv)` entry point, Beanstalk subscription and ping/restart loop, see [workers](workers.md).

**See working examples in**:

* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/WorkerExampleFormMain.php`
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/WorkerExampleFormAMI.php`
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php` (the `getModuleWorkers()` definition)

***

## Recipe: firewall (firewall + fail2ban)

Adds methods to the Conf class to open ports and define fail2ban jails when the module is enabled.

{% code title="Lib/BlackListConf.php" %}
```php
public function getDefaultFirewallRules(): array
{
    return [
        'ModuleBlackList' => [
            'rules' => [
                ['name' => 'BlackListApiPort', 'protocol' => 'tcp', 'portfrom' => 8080, 'portto' => 8080],
            ],
            'action' => 'allow',
        ],
    ];
}

public function onAfterIptablesReload(): void
{
    // Inject custom iptables rules after the standard ruleset is applied.
}

public function generateFail2BanJails(): string
{
    return <<<JAIL
    [module-black-list]
    enabled  = true
    port     = 8080
    filter   = module-black-list
    logpath  = /var/log/module-black-list.log
    maxretry = 5
    bantime  = 3600
    JAIL;
}
```
{% endcode %}

`getDefaultFirewallRules()`, `onAfterIptablesReload()` and `generateFail2BanJails()` are all real hooks on the config class. Full timing in the [hooks reference](hooks-reference.md).

***

## Recipe: acl (access control)

Adds methods to the Conf class that integrate with the MikoPBX role/permission system. All four are verified hooks:

```php
// Add ACL roles and rules to the prepared ACL list.
// In ConfigClass the type is aliased: use Phalcon\Acl\Adapter\Memory as AclList;
public function onAfterACLPrepared(\Phalcon\Acl\Adapter\Memory $aclList): void

// Filter CDR queries by the current user's role.
public function applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void

// Declare which permissions a controller exposes.
public function onGetControllerPermissions(string $controller, array &$permissions): void

// Plug in external authentication (LDAP, OAuth, …).
public function authenticateUser(string $login, string $password): array
```

For ModuleBlackList you would use `onGetControllerPermissions()` to gate the settings page behind a custom permission.

**See working examples in** `Extensions/ModuleUsersUI/Lib/UsersUIConf.php` and `Extensions/ModuleUsersGroups/Lib/UsersGroupsConf.php`.

***

## Recipe: system (system integration)

Adds methods to the Conf class for cron, nginx and lifecycle integration. All verified hooks:

```php
// Register cron tasks (append crontab lines to $tasks).
public function createCronTasks(array &$tasks): void

// Add nginx location blocks (e.g. proxy a webhook to a module worker).
public function createNginxLocations(): string

// Add a full nginx server block on a custom port.
public function createNginxServers(): string

// Lifecycle callbacks.
public function onAfterPbxStarted(): void
public function onAfterNetworkConfigured(): void

// React to changes in core models.
public function modelsEventChangeData($data): void
```

A typical ModuleBlackList use is a nightly cleanup via `createCronTasks()`:

{% code title="Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\System\Util;

public function createCronTasks(array &$tasks): void
{
    $workerPath = $this->moduleDir . '/bin/cleanup.php';
    $phpPath    = Util::which('php');
    $tasks[]    = "0 1 * * * {$phpPath} -f {$workerPath} > /dev/null 2>&1\n";
}
```
{% endcode %}

`modelsEventChangeData()` is how a module reacts to core configuration changes. The shipping example restarts its services when the PBX language changes — see `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php`.

***

## The recipe combination matrix

Modules are built by combining recipes. The base recipe is always present; everything else is additive. Common shapes:

| Module type          | Recipes                                                  | Example                                   |
| -------------------- | -------------------------------------------------------- | ----------------------------------------- |
| Simple settings page | base + ui                                                | A page with a few options                 |
| REST service (no UI) | base + rest-api                                          | Headless integration endpoint             |
| Call processing      | base + ui + dialplan + agi                               | Caller lookup / routing (ModuleBlackList) |
| CRM integration      | base + ui + rest-api + workers + dialplan                | Bi-directional sync + screen-pop          |
| Security module      | base + ui + firewall + dialplan                          | Fraud / blocklist with port control       |
| Full-featured module | base + ui + rest-api + dialplan + agi + workers + system | Everything                                |

### Worked example: ModuleBlackList

ModuleBlackList is **base + ui + dialplan + agi + system**:

* **base** — `module.json`, `BlackListConf`, `BlackListNumbers` model (`m_BlackListNumbers`), `Messages/ru.php`, Setup class.
* **ui** — a settings page to manage blacklisted numbers, menu item via `onBeforeHeaderMenuShow()`, assets via the controller, `module-black-list-index.js`.
* **dialplan** — `generateIncomingRoutBeforeDial()` injects an AGI call on inbound calls.
* **agi** — `agi-bin/module-black-list.php` checks the caller against `BlackListNumbers` using `new AGI()` and snake\_case methods.
* **system** — `createCronTasks()` runs nightly cleanup.

Each recipe contributed its files and its hooks independently; together they form one coherent module.

***

## Where to go next

* [Hooks reference](hooks-reference.md) — exhaustive, verified signatures for every hook named here.
* [REST API in modules](rest-api-in-modules.md) — the full v3 attribute/Action lifecycle.
* [Workers](workers.md) — worker base class and supervision details.
* [What the AI skill generates](../ai-assisted-development/what-it-generates.md) — how `/mikopbx-module` composes these same recipes.
