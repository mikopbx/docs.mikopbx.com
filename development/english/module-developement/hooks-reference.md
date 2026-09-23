---
description: >-
  The complete verified catalog of MikoPBX module extension points (hooks):
  exact signatures, the interface that declares each constant, and the Core call
  site that fires them.
---

# Hooks reference

A MikoPBX module never registers callbacks. Instead, the Core periodically asks every installed module's `Conf` class "do you implement method X?" — and if you do, it calls it. This page is the authoritative catalog of those methods (hooks), grouped by category, each with its exact signature, the interface that declares its constant, and the **Core call site** that decides _when_ it fires.

The running example throughout the developer docs is a fictional module **ModuleBlackList** with config class `BlackListConf`. Every hook below is shown as it would appear on `BlackListConf`, and every pattern is anchored to a real, shippable example module under `Extensions/EXAMPLES/`.

For the broader picture of the `Conf` class and how to register it, see [module-class.md](module-class.md).

{% hint style="info" %}
All signatures, constants, and call sites on this page are taken verbatim from the Core source under `Core/src/`. The base class `MikoPBX\Modules\Config\ConfigClass` (in `Core/src/Modules/Config/ConfigClass.php`) already provides a no-op default body for almost every hook, so you only override the ones you need.
{% endhint %}

## The dispatch model

`BlackListConf` extends `MikoPBX\Modules\Config\ConfigClass`, which in turn extends `MikoPBX\Core\Asterisk\Configs\AsteriskConfigClass` and implements four interfaces that declare the hook constants:

* `MikoPBX\Core\Asterisk\Configs\AsteriskConfigInterface` — Asterisk config generation.
* `MikoPBX\Modules\Config\SystemConfigInterface` — system, network, cron, nginx, firewall, lifecycle.
* `MikoPBX\Modules\Config\RestAPIConfigInterface` — PBXCoreREST.
* `MikoPBX\Modules\Config\WebUIConfigInterface` — admin web interface.

A fifth interface, `MikoPBX\Modules\Config\CDRConfigInterface`, declares the CDR filter hook; modules implement its method directly even though `ConfigClass` does not formally `implements` it (the base class still ships a default body).

### Two dispatchers, two return conventions

The Core fires hooks through one of **two** dispatchers. Which one is used determines how your return value is consumed — get this wrong and your output is silently discarded.

{% tabs %}
{% tab title="Asterisk config (string concat)" %}
`AsteriskConfigClass::hookModulesMethod(string $methodName, array $arguments = []): string` — defined in `Core/src/Core/Asterisk/Configs/AsteriskConfigClass.php`.

* Merges **internal** Core config classes **and** external modules, sorts them by `getMethodPriority($methodName)`.
* Calls your method only if `method_exists()`; skips the calling class itself to prevent recursion.
* **Concatenates** every module's returned **string**, wrapping each non-empty block in `; ***** BEGIN BY <moduleUniqueId>` / `; ***** END BY <moduleUniqueId>` comment markers.
* Any thrown exception is caught and logged via `CriticalErrorsHandler`; the loop continues to the next module.

Used by the entire **Asterisk config generation** category. Your hook must return a `string` of `.conf` text; returning anything else (or nothing) is treated as "no contribution."
{% endtab %}

{% tab title="System / REST / WebUI / CDR (array result)" %}
`PBXConfModulesProvider::hookModulesMethod(string $methodName, array $arguments = []): array` — static, defined in `Core/src/Common/Providers/PBXConfModulesProvider.php`.

* Iterates **external modules only**, sorted by `getMethodPriority($methodName)`.
* Calls your method only if `method_exists()`.
* Returns an **array keyed by `moduleUniqueId`**: `[ 'ModuleBlackList' => <your return> ]`. Empty returns are dropped.
* Any thrown exception is caught and logged via `CriticalErrorsHandler`; the loop continues to the next module — exactly as in the Asterisk dispatcher.

Used by the **system / network / REST / WebUI** categories.
{% endtab %}
{% endtabs %}

### Three result-handling styles

Independently of the dispatcher, individual call sites consume your hook in one of three ways:

1. **String return, concatenated** — the Asterisk family (e.g. `generatePeersPj`, `extensionGenContexts`). Return a `.conf` fragment.
2. **Array return, keyed by module** — e.g. `getDefaultFirewallRules`, `getModuleWorkers`, `getPBXCoreRESTAdditionalRoutes`. Return your data; the Core merges it.
3. **By-reference mutation** — the call site passes an argument by reference and **ignores your return value**. You must mutate the argument in place: `createCronTasks(array &$tasks)`, `onBeforeHeaderMenuShow(array &$menuItems)`, `applyACLFiltersToCDRQuery(array &$parameters, ...)`, `onGetControllerPermissions(string $controller, array &$permissions)`, `onAfterACLPrepared` (the call site passes `[&$acl]`), and `onAfterRoutesPrepared` (the call site passes `[&$router]`). A `return` here does nothing.

{% hint style="warning" %}
Two hooks below — `getWafExemptions` and the `#[WafExempt]` attribute path — are **not** dispatched through `hookModulesMethod` at all. They are read directly by `WafRegistry` via `method_exists()` iteration. They are documented in their own section.
{% endhint %}

### Priority

```php
public function getMethodPriority(string $methodName = ''): int
{
    // Lower number = earlier. Default in ConfigClass is 10000.
    return match ($methodName) {
        AsteriskConfigInterface::GENERATE_PEERS_PJ => 9000, // run before others
        SystemConfigInterface::CREATE_CRON_TASKS   => 11000,
        default                                    => 10000,
    };
}
```

Both dispatchers `usort` modules by `getMethodPriority($methodName)` ascending before calling them. Override it on `BlackListConf` to control ordering relative to other modules.

***

## 1. Asterisk config generation

Declared in `AsteriskConfigInterface` (`Core/src/Core/Asterisk/Configs/AsteriskConfigInterface.php`). Dispatched by `AsteriskConfigClass::hookModulesMethod`, so every hook returns a **`string`** that is concatenated into the generated `.conf` file. Default bodies live in `AsteriskConfigClass`.

| Constant                             | Method signature                                              | When the Core calls it                                                                                                             |
| ------------------------------------ | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `EXTENSION_GEN_CONTEXTS`             | `extensionGenContexts(): string`                              | Building extra context sections in `extensions.conf` — `Core/src/Core/Asterisk/Configs/Generators/Extensions/InternalContexts.php` |
| `EXTENSION_GEN_INTERNAL`             | `extensionGenInternal(): string`                              | Adding rules to the `[internal]` context — `InternalContexts.php`                                                                  |
| `GENERATE_INCOMING_ROUT_BEFORE_DIAL` | `generateIncomingRoutBeforeDial(string $rout_number): string` | Per incoming route, before the dial command — `Core/src/Core/Asterisk/Configs/Generators/Extensions/IncomingContexts.php`          |
| `GENERATE_OUT_ROUT_CONTEXT`          | `generateOutRoutContext(array $rout): string`                 | Per outgoing route context — `Core/src/Core/Asterisk/Configs/Generators/Extensions/OutgoingContext.php`                            |
| `GENERATE_PUBLIC_CONTEXT`            | `generatePublicContext(): string`                             | Building the `[public-direct-dial]` section — `Core/src/Core/Asterisk/Configs/ExtensionsConf.php`                                  |
| `GENERATE_PEERS_PJ`                  | `generatePeersPj(): string`                                   | Generating peer endpoints in `pjsip.conf` — `Core/src/Core/Asterisk/Configs/SIPConf.php`                                           |
| `GET_FEATURE_MAP`                    | `getFeatureMap(): string`                                     | Building the `[featuremap]` section of `features.conf` — `Core/src/Core/Asterisk/Configs/FeaturesConf.php`                         |
| `GENERATE_MANAGER_CONF`              | `generateManagerConf(): string`                               | Adding AMI users to `manager.conf` — `Core/src/Core/Asterisk/Configs/ManagerConf.php`                                              |
| `GET_METHOD_PRIORITY`                | `getMethodPriority(string $methodName = ''): int`             | Both dispatchers, before any hook, to sort modules                                                                                 |

Other Asterisk hooks declared by the same interface (all `(): string` unless noted, all dispatched the same way): `extensionGenHints`, `extensionGenCreateChannelDialplan`, `extensionGenInternalTransfer`, `getIncludeInternalTransfer`, `getIncludeInternal`, `extensionGenInternalUsersPreDial`, `extensionGenAllPeersContext`, `extensionGlobals`, `generateIncomingRoutBeforeDialPreSystem(string $rout_number)`, `generateIncomingRoutBeforeDialSystem(string $rout_number)`, `generateIncomingRoutAfterDialContext(string $uniqId)`, `generateOutRoutAfterDialContext(array $rout)`, `generatePeerPjAdditionalOptions(array $peer)`, `generateModulesConf`, and the two PJSIP option overrides which return an **array**: `overridePJSIPOptions(string $extension, array $options): array` and `overrideProviderPJSIPOptions(string $uniqid, array $options): array`.

{% hint style="info" %}
**`generateAriConf` has no default body.** Only the constant `AsteriskConfigInterface::GENERATE_ARI_CONF = 'generateAriConf'` (`Core/src/Core/Asterisk/Configs/AsteriskConfigInterface.php:62`) and the call site in `Core/src/Core/Asterisk/Configs/AriConf.php` exist — the interface does not declare the method and `AsteriskConfigClass` does not implement it. Declare `generateAriConf(): string` on `BlackListConf` yourself if your module needs to contribute to `ari.conf`; the returned string is concatenated like any other Asterisk hook.
{% endhint %}

### Config-generation lifecycle members

`AsteriskConfigClass` also exposes four members that are part of the same family but are rarely thought of as "hooks". They are real and overridable:

| Constant                | Method signature               | When the Core calls it                                                                                                                                                                          |
| ----------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GENERATE_CONFIG`       | `generateConfig(): void`       | Regenerating all Asterisk configs — `Core/src/Core/System/Configs/PbxConf.php`. Override `generateConfigProtected()` rather than this wrapper if you only want to write your own `.conf` files. |
| `GET_SETTINGS`          | `getSettings(): void`          | Called from `generateConfig()` before generation, so you can load your module settings into properties                                                                                          |
| `GET_DEPENDENCE_MODELS` | `getDependenceModels(): array` | Returns the model classes your module cares about; used to decide whether a model change is relevant to you before `modelsEventChangeData` work is done                                         |
| —                       | `getMessages(): array`         | Reads back the `$this->messages` array your hooks filled in; the installer and `PbxExtensionState` surface those strings to the admin                                                           |

{% code title="Lib/BlackListConf.php" %}
```php
use MikoPBX\Modules\Config\ConfigClass;

class BlackListConf extends ConfigClass
{
    /**
     * Adds a dialplan branch in the [internal] context that rejects
     * blacklisted callers. Concatenated into extensions.conf.
     */
    public function extensionGenInternal(): string
    {
        $conf  = "exten => _X.,1,GotoIf(\$[\${DB_EXISTS(blacklist/\${CALLERID(num)})}]?reject)\n";
        $conf .= "same => n(reject),Hangup(21)\n";
        return $conf;
    }
}
```
{% endcode %}

See the working `generateManagerConf` implementation in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`. For more dialplan patterns, see [../cookbook/asterisk/README.md](../cookbook/asterisk/).

***

## 2. Firewall, fail2ban, WAF

Firewall and fail2ban hooks are declared in `SystemConfigInterface` and dispatched by `PBXConfModulesProvider::hookModulesMethod`. WAF exemptions use a **separate, direct** dispatch path.

| Constant                     | Method signature                    | When the Core calls it                                                                                    |
| ---------------------------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `GET_DEFAULT_FIREWALL_RULES` | `getDefaultFirewallRules(): array`  | Rebuilding firewall rules — `Core/src/Common/Models/FirewallRules.php`                                    |
| `ON_AFTER_IPTABLES_RELOAD`   | `onAfterIptablesReload(): void`     | After iptables rules are applied, before the final DROP — `Core/src/Core/System/Configs/IptablesConf.php` |
| `GENERATE_FAIL2BAN_JAILS`    | `generateFail2BanJails(): string`   | Building fail2ban jails — `Core/src/Core/System/Configs/Fail2BanConf.php`                                 |
| `GENERATE_FAIL2BAN_FILTERS`  | `generateFail2BanFilters(): string` | Building fail2ban filter files — `Fail2BanConf.php`                                                       |

{% hint style="info" %}
**`generateFail2BanFilters` has no default body.** Only the constant `SystemConfigInterface::GENERATE_FAIL2BAN_FILTERS = 'generateFail2BanFilters'` and a call site exist — `ConfigClass` does not declare the method. Implement it on `BlackListConf` yourself. From the call site in `Fail2BanConf.php` (`generateModulesFilters()`), the Core takes each module's returned **string** and writes it to `<filterPath>/<uncamelized moduleUniqueId>.conf`. Return the full body of a fail2ban filter definition.
{% endhint %}

### WAF exemptions (direct dispatch — not `hookModulesMethod`)

`getWafExemptions(): array` is declared and given a no-op default in `ConfigClass`, but the Core does **not** route it through `hookModulesMethod`. Instead `MikoPBX\PBXCoreREST\Lib\Waf\WafRegistry` (`Core/src/PBXCoreREST/Lib/Waf/WafRegistry.php`) iterates module instances and calls it directly after a `method_exists($instance, 'getWafExemptions')` check.

Return a map of URI → scope list (shorthand, exact match) or URI → `['scopes' => [...], 'prefix' => bool]` (verbose). Valid scopes: `body-scan`, `request-line`, `rate-limit`.

```php
public function getWafExemptions(): array
{
    return [
        '/pbxcore/api/black-list/webhook' => ['body-scan'],
        '/pbxcore/api/black-list/items'   => [
            'scopes' => ['body-scan', 'rate-limit'],
            'prefix' => true,
        ],
    ];
}
```

As an alternative, REST controllers annotated with the `#[WafExempt]` attribute are published automatically by `WafRegistry` (class-level → prefix match, method-level → exact match), so you do not need to override `getWafExemptions` for those routes.

***

## 3. Cron, system, network, nginx, model events

Declared in `SystemConfigInterface` (`Core/src/Modules/Config/SystemConfigInterface.php`), dispatched by `PBXConfModulesProvider::hookModulesMethod`.

| Constant                      | Method signature                                           | When the Core calls it                                                                                  |
| ----------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `CREATE_CRON_TASKS`           | `createCronTasks(array &$tasks): void`                     | Regenerating crontab — `Core/src/Core/System/Configs/CronConf.php`                                      |
| `CREATE_NGINX_LOCATIONS`      | `createNginxLocations(): string`                           | Building nginx config — `Core/src/Core/System/Configs/NginxConf.php`                                    |
| `CREATE_NGINX_SERVERS`        | `createNginxServers(): string`                             | Building nginx server blocks — `NginxConf.php`                                                          |
| `ON_AFTER_PBX_STARTED`        | `onAfterPbxStarted(): void`                                | After Asterisk/PBX has started — `Core/src/Core/System/Configs/PbxConf.php`                             |
| `ON_AFTER_NETWORK_CONFIGURED` | `onAfterNetworkConfigured(): void`                         | After all interfaces, routes and DNS are set up — `Core/src/Core/System/Network.php` (`lanConfigure()`) |
| `MODELS_EVENT_CHANGE_DATA`    | `modelsEventChangeData($data): void`                       | After each model change — `Core/src/Core/Workers/WorkerModelsEvents.php`                                |
| `MODELS_EVENT_NEED_RELOAD`    | `modelsEventNeedReload(array $plannedReloadActions): void` | After a batch of model changes, when a reload is planned — `WorkerModelsEvents.php`                     |

{% hint style="warning" %}
**`createCronTasks` is a by-reference hook.** The call site invokes it as `hookModulesMethod(SystemConfigInterface::CREATE_CRON_TASKS, [&$tasks])` and ignores the return value. Append your entries to `$tasks` in place.
{% endhint %}

{% code title="Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\System\Util;

public function createCronTasks(array &$tasks): void
{
    $php       = Util::which('php');
    $workerPath = "{$this->moduleDir}/bin/cleanup-blacklist.php";
    // minute hour day month weekday command
    $tasks[] = "*/5 * * * * {$php} -f {$workerPath} > /dev/null 2>&1\n";
}
```
{% endcode %}

See `modelsEventChangeData` and `getModuleWorkers` implemented in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php`.

### `getModuleWorkers` and worker check types

`getModuleWorkers(): array` (declared in `SystemConfigInterface`) returns an array of `['type' => ..., 'worker' => WorkerClass::class]` entries that `WorkerSafeScripts` keeps alive. The `type` is a constant from `MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore` (`Core/src/Core/Workers/Cron/WorkerSafeScriptsCore.php`):

* `WorkerSafeScriptsCore::CHECK_BY_BEANSTALK`
* `WorkerSafeScriptsCore::CHECK_BY_AMI`
* `WorkerSafeScriptsCore::CHECK_BY_REDIS` (value `'checkWorkerRedis'`)
* `WorkerSafeScriptsCore::CHECK_BY_PID_NOT_ALERT`

```php
use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;

public function getModuleWorkers(): array
{
    return [
        [
            'type'   => WorkerSafeScriptsCore::CHECK_BY_REDIS,
            'worker' => BlackListWorker::class,
        ],
    ];
}
```

For the full worker lifecycle, see [workers.md](workers.md).

***

## 4. REST API, Web UI, CDR

REST hooks are declared in `RestAPIConfigInterface`, Web UI hooks in `WebUIConfigInterface`, and the CDR filter in `CDRConfigInterface`. All are dispatched by `PBXConfModulesProvider::hookModulesMethod`.

| Constant                             | Method signature                                                                  | When the Core calls it                                                                                                                                                                                                     |
| ------------------------------------ | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GET_PBXCORE_REST_ADDITIONAL_ROUTES` | `getPBXCoreRESTAdditionalRoutes(): array`                                         | Registering REST routes — `Core/src/PBXCoreREST/Providers/RouterProvider.php` (the site that actually adds your routes); also read by `Core/src/PBXCoreREST/Http/Request.php` to decide which URIs may skip authentication |
| `MODULE_RESTAPI_CALLBACK`            | `moduleRestAPICallback(array $request): PBXApiResult`                             | Handling a REST request under root rights — `Core/src/PBXCoreREST/Lib/PbxExtensionsProcessor.php`                                                                                                                          |
| `AUTHENTICATE_USER`                  | `authenticateUser(string $login, string $password): array`                        | Login attempt that the core could not authenticate — `Core/src/Common/Library/Auth/CredentialsValidator.php`                                                                                                               |
| `GET_PASSKEY_SESSION_DATA`           | `getPasskeySessionData(string $login): array`                                     | Resolving session params for a passkey login — `Core/src/PBXCoreREST/Lib/Passkeys/AuthenticationFinishAction.php`                                                                                                          |
| `ON_AFTER_ACL_LIST_PREPARED`         | `onAfterACLPrepared(AclList $aclList): void`                                      | Building the ACL list — `Core/src/Common/Providers/AclProvider.php` (passes `[&$acl]`)                                                                                                                                     |
| `ON_BEFORE_HEADER_MENU_SHOW`         | `onBeforeHeaderMenuShow(array &$menuItems): void`                                 | Rendering the header/sidebar menu — `Core/src/AdminCabinet/Library/Elements.php`                                                                                                                                           |
| `ON_VOLT_BLOCK_COMPILE`              | `onVoltBlockCompile(string $controller, string $blockName, View $view): string`   | Compiling a Volt template block — `Core/src/AdminCabinet/Providers/VoltProvider.php`                                                                                                                                       |
| `APPLY_ACL_FILTERS_TO_CDR_QUERY`     | `applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void` | Before a CDR query executes — `Core/src/PBXCoreREST/Lib/Cdr/GetListAction.php`                                                                                                                                             |

Other Web UI / REST hooks (all dispatched the same way): `onAfterAssetsPrepared(Manager $assets, Dispatcher $dispatcher): void` (`Core/src/AdminCabinet/Providers/AssetProvider.php`), `onBeforeExecuteRoute(Dispatcher $dispatcher): void` and `onAfterExecuteRoute(Dispatcher $dispatcher): void` (`Core/src/AdminCabinet/Controllers/BaseController.php`), `onBeforeExecuteRestAPIRoute(Micro $app): void` and `onAfterExecuteRestAPIRoute(Micro $app): void` (`Core/src/PBXCoreREST/Providers/RouterProvider.php`).

Two more Web UI hooks are **by-reference** — they appear in the by-reference group above, not here:

| Constant                        | Method signature                                                            | When the Core calls it                                                                                                                                                                                             |
| ------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ON_AFTER_ROUTES_PREPARED`      | `onAfterRoutesPrepared(Router $router): void`                               | Building the admin router — `Core/src/Common/Providers/RouterProvider.php` (passes `[&$router]`)                                                                                                                   |
| `ON_GET_CONTROLLER_PERMISSIONS` | `onGetControllerPermissions(string $controller, array &$permissions): void` | Building the permission list for a controller — `Core/src/AdminCabinet/Controllers/AclController.php` (passes `[$controllerClass, &$customPermissions]`; whatever you append lands under `$permissions['custom']`) |

{% hint style="info" %}
**`getPasskeySessionData` has no default body.** Only the constant `WebUIConfigInterface::GET_PASSKEY_SESSION_DATA = 'getPasskeySessionData'` and the call site in `AuthenticationFinishAction.php` exist; `ConfigClass` does not declare the method. Implement it on `BlackListConf` if your module owns passkey logins. The call site invokes it as `hookModulesMethod(..., [$login])` and takes the first non-empty array return as the session parameters, so the inferred signature is `getPasskeySessionData(string $login): array`.
{% endhint %}

### Documented corrections

These three signatures are frequently misremembered. The values below are verified against the primary source:

* **`applyACLFiltersToCDRQuery`** — the second parameter has a default: `array $sessionContext = []`. In the AdminCabinet context this array is empty (use `SessionProvider`); in the REST API context it carries `role`, `user_name`, and `session_id` from the JWT. **By-reference**: mutate `$parameters`, do not rely on the return value.
* **`onBeforeFormInitialize`** — the third parameter `$options` is **untyped**: `onBeforeFormInitialize(Form $form, $entity, $options): void`. (`$entity` is also untyped.)
* **`CHECK_BY_REDIS`** — the constant's value is the string `'checkWorkerRedis'`, not `'redis'`.

{% code title="Lib/BlackListConf.php" %}
```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $menuItems['module_black_list_Menu'] = [
        'caption'   => 'module_black_list_Menu',
        'iconclass' => 'ban',
        'submenu'   => [
            '/module-black-list/index' => [
                'caption'   => 'module_black_list_Numbers',
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

See `onBeforeHeaderMenuShow`, `getModuleWorkers`, and `moduleRestAPICallback` implemented in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php`, and a `getPBXCoreRESTAdditionalRoutes` route table in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv2/Lib/ExampleRestAPIv2Conf.php`. For end-to-end recipes, see [recipes.md](recipes.md).

***

## 5. Peer and provider status changes

Two more hooks fire when the Core detects that a SIP extension or a provider changed registration state. They are dispatched by `PBXConfModulesProvider::hookModulesMethod` like everything in sections 2–4, but they are unusual in two ways.

{% hint style="warning" %}
**No interface constant and no default body.** Unlike every other hook on this page, these two are dispatched by a **bare string literal** at the call site — there is no `SystemConfigInterface::…` constant to reference, and `ConfigClass` does not declare the methods. Spell the method name exactly as shown or the `method_exists()` check silently skips you.
{% endhint %}

| Method signature                                   | When the Core calls it                                                                                              |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `onExtensionStatusChange(array $changeData): void` | An internal extension's registration state changed — `Core/src/PBXCoreREST/Lib/Extensions/GetAllStatusesAction.php` |
| `onProviderStatusChange(array $changeData): void`  | A SIP/IAX provider's registration state changed — `Core/src/PBXCoreREST/Lib/Providers/GetAllStatusesAction.php`     |

Both receive the same payload shape:

```php
public function onProviderStatusChange(array $changeData): void
{
    // $changeData = [
    //     'changes'   => [...],   // the per-entity state transitions
    //     'timestamp' => 1735689600,
    //     'source'    => 'GetAllStatusesAction',
    // ];
    foreach ($changeData['changes'] as $change) {
        // React — notify a CRM, flip a UI badge, write a log line.
    }
}
```

The call site wraps the dispatch in its own `try`/`catch` and logs failures to syslog, so an exception in your handler will not break status polling — but it will also not be reported to the admin. Log deliberately.

***

## 6. Module lifecycle

Declared in `SystemConfigInterface`, dispatched by `PBXConfModulesProvider::hookModulesMethod`, fired by `Core/src/Modules/PbxExtensionState.php` (and `Core/src/Core/Workers/Libs/WorkerModelsEvents/Actions/ReloadModuleStateAction.php`).

| Constant                   | Method signature                | When the Core calls it                       |
| -------------------------- | ------------------------------- | -------------------------------------------- |
| `ON_BEFORE_MODULE_ENABLE`  | `onBeforeModuleEnable(): bool`  | Before enabling; **return `false` to veto**  |
| `ON_AFTER_MODULE_ENABLE`   | `onAfterModuleEnable(): void`   | After the module is enabled                  |
| `ON_BEFORE_MODULE_DISABLE` | `onBeforeModuleDisable(): bool` | Before disabling; **return `false` to veto** |
| `ON_AFTER_MODULE_DISABLE`  | `onAfterModuleDisable(): void`  | After the module is disabled                 |

{% hint style="danger" %}
`onBeforeModuleEnable` and `onBeforeModuleDisable` return **`bool`**. Returning `false` aborts the enable/disable transition — use this to refuse activation when a dependency or license check fails. The default body in `ConfigClass` returns `true`.
{% endhint %}

{% code title="Lib/BlackListConf.php" %}
```php
public function onBeforeModuleEnable(): bool
{
    // Refuse to enable until the lookup table has been migrated.
    if (!\Modules\ModuleBlackList\Models\BlackListNumbers::isTableReady()) {
        $this->messages['error'][] = 'm_BlackListNumbers table is not ready';
        return false;
    }
    return true;
}

public function onAfterModuleDisable(): void
{
    // Stop background workers, flush AstDB keys, etc.
}
```
{% endcode %}

See `onAfterModuleEnable` / `onAfterModuleDisable` in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`.

***

## Related pages

* [module-class.md](module-class.md) — the `Conf` class, its constructor, and `$moduleUniqueId` / `$moduleDir`.
* [workers.md](workers.md) — background workers returned by `getModuleWorkers`.
* [recipes.md](recipes.md) — task-oriented hook recipes.
* [../cookbook/asterisk/README.md](../cookbook/asterisk/) — Asterisk dialplan patterns for the config-generation hooks.
