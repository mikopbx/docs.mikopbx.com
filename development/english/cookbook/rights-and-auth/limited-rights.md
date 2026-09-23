---
description: >-
  Define ACL roles, controller permissions and per-role data filtering so that
  module-managed users see only what their role allows.
---

# Limited rights

This recipe shows how a module gives the MikoPBX web interface **role-based access control (RBAC)**: how to register custom ACL roles and rules, how to contribute per-controller permission flags consumed by the front-end, and how to inject per-role `WHERE` conditions into CDR queries so each role sees only its own call records.

Three Web-UI hooks carry the whole flow. They are declared on [`WebUIConfigInterface`](../../module-developement/hooks-reference.md) and [`CDRConfigInterface`](../../module-developement/hooks-reference.md), and you override them in your module's config class:

| Hook                                                                        | Declared in            | Purpose                                                                   |
| --------------------------------------------------------------------------- | ---------------------- | ------------------------------------------------------------------------- |
| `onAfterACLPrepared(AclList $aclList)`                                      | `WebUIConfigInterface` | Add roles, components (controllers) and `allow` rules to the Phalcon ACL. |
| `onGetControllerPermissions(string $controller, array &$permissions)`       | `WebUIConfigInterface` | Add custom permission flags for a controller, returned to JavaScript.     |
| `applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = [])` | `CDRConfigInterface`   | Add per-role `WHERE` conditions to CDR list queries.                      |

{% hint style="info" %}
The complete, production-grade implementation of this recipe is the **ModuleUsersUI** module. Read it in `Extensions/ModuleUsersUI/Lib/` — the narrative below uses the fictional **ModuleBlackList** module (config class `BlackListConf`) for teaching, but every API it calls is taken from that real module and from the MikoPBX Core classes.
{% endhint %}

## How MikoPBX resolves rights

MikoPBX uses Phalcon's in-memory ACL (`Phalcon\Acl\Adapter\Memory`, aliased as `AclList`). The ACL has three kinds of entries:

* **Roles** — `Phalcon\Acl\Role`. A logged-in session carries exactly one role.
* **Components** — `Phalcon\Acl\Component`. In MikoPBX a component is a controller class name, e.g. `MikoPBX\AdminCabinet\Controllers\CallDetailRecordsController`.
* **Rules** — `allow($role, $component, $actions)` grants a role the listed controller actions (`index`, `modify`, `save`, …).

The ACL is built in `Core/src/Common/Providers/AclProvider.php` and starts from `setDefaultAction(AclEnum::DENY)` — **anything not explicitly allowed is denied**. Two roles are then registered before any module is consulted:

* `AclProvider::ROLE_ADMINS` (`'admins'`) gets `allow('admins', '*', '*')` — the administrator bypasses every check.
* `AclProvider::ROLE_GUESTS` (`'guests'`) gets `deny('guests', '*', '*')` — the anonymous default.

A module that introduces limited users defines additional roles and grants each one only the controllers and actions it should reach. When the user's session role is set to one of those custom roles, the `SecurityPlugin` denies every request that the ACL does not explicitly allow.

{% hint style="warning" %}
**Two bypasses you must know about before you rely on the ACL.** `SecurityPlugin::isAllowedAction()` (`Core/src/AdminCabinet/Plugins/SecurityPlugin.php`) reads the role from the JWT (Bearer header, or the `refreshToken` cookie mapped through Redis). When no role can be extracted it falls back to `ROLE_ADMINS` for **localhost** requests and `ROLE_GUESTS` for everything else. On top of that, `beforeDispatch()` skips the ACL check entirely for `isLocalHostRequest()`. So a request originating from `127.0.0.1` — internal workers, health checks, anything proxied without the original client address — is treated as an administrator. Never model your access control on the assumption that the ACL is the only gate for local traffic.
{% endhint %}

The ACL also grants a few things to `'*'` (every role) _after_ the module hook runs: the `Errors` controller (`show401`/`show404`/`show500`), the `Session` controller (`index`/`start`/`changeLanguage`/`end`) and the stateless password helper endpoints. You cannot revoke those from a module hook — they are added after your `onAfterACLPrepared()` returns.

{% hint style="warning" %}
The assembled ACL is **cached**. After you change anything that affects role membership or rules, clear the cache with `MikoPBX\Common\Providers\AclProvider::clearCache();`. ModuleUsersUI does this in `onAfterModuleEnable()`, `onAfterModuleDisable()` and inside `modelsEventChangeData()` when its access-group models change — see `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`.
{% endhint %}

## Step 1 — register roles and rules with `onAfterACLPrepared`

`onAfterACLPrepared(AclList $aclList): void` runs while MikoPBX builds the ACL (the result is then cached). Use it to add roles, components and `allow` rules. The signature is fixed by `WebUIConfigInterface`:

{% code title="Core/src/Modules/Config/WebUIConfigInterface.php" %}
```php
public function onAfterACLPrepared(AclList $aclList): void;
```
{% endcode %}

For **ModuleBlackList** we define one role per managed access group. Each role gets the controllers it is allowed to use; a group flagged `fullAccess` is granted everything with `allow($role, '*', '*')`.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Common\Providers\AclProvider;
use MikoPBX\Modules\Config\ConfigClass;
use Modules\ModuleBlackList\Models\BlackListAccessGroups;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Component;
use Phalcon\Acl\Role as AclRole;

class BlackListConf extends ConfigClass
{
    /** Prefix that makes our role names unique across the system. */
    public const ROLE_PREFIX = 'BlackListRoleID';

    /**
     * Add this module's roles and rules to the system ACL.
     */
    public function onAfterACLPrepared(AclList $aclList): void
    {
        foreach (BlackListAccessGroups::find() as $group) {
            // 1) Build a unique role name from the access group id.
            $role = self::ROLE_PREFIX . $group->id;
            $aclList->addRole(new AclRole($role, (string)$group->name));

            // 2) Full-access groups may reach everything.
            if ($group->fullAccess === '1') {
                $aclList->allow($role, '*', '*');
                continue;
            }

            // 3) Otherwise grant only the listed controller actions.
            //    $group->rights() returns rows of {controller, actions(JSON)}.
            foreach ($group->rights() as $right) {
                $actions = json_decode($right->actions, true) ?: [];
                $controller = $right->controller; // FQCN of an AdminCabinet controller

                // Register the controller as a component, then allow the actions.
                $aclList->addComponent(new Component($controller), $actions);
                $aclList->allow($role, $controller, $actions);
            }
        }
    }

    /**
     * Drop the cached ACL whenever the module is toggled.
     */
    public function onAfterModuleEnable(): void
    {
        AclProvider::clearCache();
    }

    public function onAfterModuleDisable(): void
    {
        AclProvider::clearCache();
    }
}
```
{% endcode %}

Key points, all verified against `Extensions/ModuleUsersUI/Lib/UsersUIACL.php`:

* A component **must be added** (`addComponent`) before you can `allow` a role on it. ModuleUsersUI builds an `$actionsArray[$controller] = [...actions]` map and then loops `addComponent(new Component($controller), $actions)` followed by `allow($role, $controller, $actions)`.
* `allow($role, '*', '*')` is the real escape hatch ModuleUsersUI uses for its `fullAccess` groups.
* Roles must be **globally unique**. ModuleUsersUI prefixes every role with `UsersUIRoleID` (constant `Constants::MODULE_ROLE_PREFIX` in `Extensions/ModuleUsersUI/Lib/Constants.php`); ModuleBlackList uses `BlackListRoleID`. This prefix is also how you recover the access-group id later (Step 3).

{% hint style="info" %}
ModuleUsersUI additionally lets each enabled module contribute "always allowed" and "linked" controllers via optional static methods on a `Modules\<Uniqid>\Lib\<Uniqid>ACL` class (`getAlwaysAllowed()`, `getAlwaysDenied()`, `getLinkedControllerActions()`). Those are a ModuleUsersUI convention, not a Core hook — see `UsersUIACL::addRulesFromModules()` if you ship a module that should be governed by ModuleUsersUI's access groups.
{% endhint %}

## Step 2 — expose permission flags with `onGetControllerPermissions`

The web front-end asks the Core `AclController` "what may the current user do on this controller?" so it can show or hide buttons. The Core computes the standard three-level flags itself (`index`, `getNewRecords`, `modify`, `edit`, `save`, `delete`, `copy`) from the `SecurityPlugin`, then lets modules add **custom** flags through this hook.

{% code title="Core/src/Modules/Config/WebUIConfigInterface.php" %}
```php
public function onGetControllerPermissions(string $controller, array &$permissions): void;
```
{% endcode %}

The contract, confirmed in `Core/src/AdminCabinet/Controllers/AclController.php`:

* `$controller` is the **fully-qualified controller class name** (e.g. `MikoPBX\AdminCabinet\Controllers\CallDetailRecordsController`).
* `$permissions` arrives as an **empty array** scoped to your module. Whatever you put in it is returned to JavaScript under `data.custom`. You do **not** touch the built-in flags — those are computed separately.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use MikoPBX\AdminCabinet\Controllers\CallDetailRecordsController;
use MikoPBX\AdminCabinet\Plugins\SecurityPlugin;

public function onGetControllerPermissions(string $controller, array &$permissions): void
{
    // Only contribute flags for the controllers we care about.
    if ($controller !== CallDetailRecordsController::class) {
        return;
    }

    $securityPlugin = new SecurityPlugin();
    $securityPlugin->setDI($this->di);

    // A custom flag the module's JS reads to toggle an "Export blacklist" button.
    $permissions['canExportBlackList'] =
        $securityPlugin->isAllowedAction($controller, 'save');
}
```
{% endcode %}

The relevant Core slice that calls this hook:

{% code title="Core/src/AdminCabinet/Controllers/AclController.php" %}
```php
// Standard flags computed by Core (index/modify/save/edit/delete/copy/...)
$permissions = [ /* ... */ ];

// Modules add custom flags here:
$customPermissions = [];
PBXConfModulesProvider::hookModulesMethod(
    WebUIConfigInterface::ON_GET_CONTROLLER_PERMISSIONS,
    [$controllerClass, &$customPermissions]
);
$permissions['custom'] = $customPermissions ?: [];
```
{% endcode %}

{% hint style="warning" %}
This hook only affects what the UI **renders**. It is a convenience for show/hide logic — it is not a security boundary. Real enforcement happens in the ACL (Step 1) and, for REST, in the security attributes (see the note at the end). Always back a hidden button with a real ACL rule.
{% endhint %}

## Step 3 — filter CDR data per role with `applyACLFiltersToCDRQuery`

A limited role that can open the _Call Detail Records_ page should still see only its own calls. CDR list queries are funnelled through one hook before execution:

{% code title="Core/src/Modules/Config/CDRConfigInterface.php" %}
```php
public function applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void;
```
{% endcode %}

`$parameters` is the Phalcon query-builder array (`conditions`, `bind`) that is about to run. You mutate it in place to add your `WHERE` clause.

`$sessionContext` is filled from the caller's JWT by `Core/src/PBXCoreREST/Controllers/BaseController.php` (lines 368-387) and carries:

* `$sessionContext['role']` — the user's role,
* `$sessionContext['user_name']` — the login,
* `$sessionContext['session_id']` — the token/session id.

{% hint style="warning" %}
The interface docblock still describes an "AdminCabinet context" in which `$sessionContext` arrives empty and you are told to read the role from `SessionProvider`. That branch is **dead in 2025.1.1**. The hook has exactly one call site — `Core/src/PBXCoreREST/Lib/Cdr/GetListAction.php:225` — and the AdminCabinet _Call Detail Records_ page reaches it over the REST API too: its controller is a static shell and the grid fetches `/pbxcore/api/v3/cdr` (`Core/sites/admin-cabinet/assets/js/src/CallDetailRecords/call-detail-records-index.js:264`), so the JWT is always present. `ModuleUsersUI` reflects this — it simply returns early when `$sessionContext['role']` is null. Write the same shape; do not build a `SessionProvider` fallback.
{% endhint %}

The hook is invoked from `Core/src/PBXCoreREST/Lib/Cdr/GetListAction.php`:

{% code title="Core/src/PBXCoreREST/Lib/Cdr/GetListAction.php" %}
```php
PBXConfModulesProvider::hookModulesMethod(
    CDRConfigInterface::APPLY_ACL_FILTERS_TO_CDR_QUERY,
    [&$parameters, $sessionContext]
);
```
{% endcode %}

ModuleBlackList recovers the access-group id from the role prefix (the inverse of Step 1) and rewrites the query conditions:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
public function applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void
{
    // The role always comes from the JWT-derived sessionContext.
    // It is null for the unrestricted administrator.
    $role = $sessionContext['role'] ?? null;

    if (empty($role) || strpos($role, self::ROLE_PREFIX) !== 0) {
        return; // not one of our roles -> no filtering
    }

    // Recover the access group id: "BlackListRoleID42" -> "42".
    $accessGroupId = str_replace(self::ROLE_PREFIX, '', $role);

    // Resolve which extension numbers this group may see.
    $allowedNumbers = $this->resolveAllowedNumbers($accessGroupId); // string[]

    if (count($allowedNumbers) === 0) {
        // Nothing allowed -> return an empty result set.
        $parameters['conditions'] = '1=0';
        return;
    }

    $oldConditions = $parameters['conditions'] ?? '';

    // Bind the array and constrain the query to the allowed numbers.
    $parameters['bind']['allowedNumbers'] = $allowedNumbers;
    $parameters['conditions'] =
        '(src_num IN ({allowedNumbers:array}) OR dst_num IN ({allowedNumbers:array}))';

    // Preserve any conditions the Core already set.
    if (!empty($oldConditions)) {
        $parameters['conditions'] .= ' AND (' . $oldConditions . ')';
    }
}
```
{% endcode %}

The mutation pattern above is exactly what ModuleUsersUI does in `Extensions/ModuleUsersUI/Lib/UsersUICDRFilter.php`:

* `src_num`/`dst_num` are the CDR table columns for source and destination extension numbers.
* `{allowedNumbers:array}` is Phalcon's bound-array placeholder, paired with `$parameters['bind']['allowedNumbers']`.
* ModuleUsersUI supports several modes (`selected`, `outgoing-selected`, `not-selected`, `all`) and falls back to `conditions = '1=0'` when a restricted group has no extensions — read its `applyCDRFilterRules()` for the full logic.
* ModuleUsersUI splits the dispatch (`UsersUIConf::applyACLFiltersToCDRQuery()` reads `$sessionContext['role']`, then delegates to `UsersUICDRFilter`). Keep your config-class hook thin and put the query logic in a helper.

{% hint style="info" %}
`applyACLFiltersToCDRQuery` only narrows the data set. The user must already be **allowed to reach** the CDR controller via Step 1; otherwise the page is denied before any query runs.
{% endhint %}

## Note: REST RBAC with the `ResourceSecurity` attribute

The hooks above secure the **web interface and CDR lists**. For your module's **REST controllers**, MikoPBX 2025.1.1+ uses a resource-based attribute, `MikoPBX\PBXCoreREST\Attributes\ResourceSecurity`, applied to a controller class or an action method:

{% code title="Extensions/ModuleBlackList/App/Controllers/Api/BlackListController.php" %}
```php
use MikoPBX\PBXCoreREST\Attributes\ResourceSecurity;
use MikoPBX\PBXCoreREST\Attributes\ActionType;
use MikoPBX\PBXCoreREST\Attributes\SecurityType;

#[ResourceSecurity(
    'blacklist',
    action: ActionType::WRITE,
    requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN]
)]
class BlackListController
{
    // ...
}
```
{% endcode %}

The constructor is `ResourceSecurity(string $resource, ?ActionType $action = null, ?array $requirements = null, bool $optional = false, string $description = '', array $extensions = [])`. `ActionType` is one of `READ`, `WRITE`, `ADMIN`, `SENSITIVE`; `SecurityType` is one of `LOCALHOST`, `BEARER_TOKEN`, `PUBLIC` (`Core/src/PBXCoreREST/Attributes/`). For a real usage, see the Core controller that declares `#[ResourceSecurity('employees', requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN])]` in `Core/src/PBXCoreREST/Controllers/Employees/RestController.php`. Full REST RBAC details live in the [REST API reference](../../internal-structure/api/rest-api.md).

## Where call-level isolation fits

ACL governs _who can open which page and see which records_. If you instead need to restrict _which numbers a group may dial_, that is enforced in the dialplan, not the ACL. The **ModuleUsersGroups** module (`Extensions/ModuleUsersGroups/`) is the reference for that: it builds group-isolation dialplan contexts and outbound-rule restrictions (`Extensions/ModuleUsersGroups/Lib/UsersGroupsConf.php`, model `AllowedOutboundRules`). It does **not** implement the ACL hooks described here — use it as the companion pattern when limited rights must extend to call routing.

## Checklist

1. Override `onAfterACLPrepared()` to add a uniquely-prefixed role per group, register each allowed controller as a component, and `allow` the actions (`allow($role, '*', '*')` for full access).
2. Clear the ACL cache with `AclProvider::clearCache()` on enable/disable and whenever your role/rights models change.
3. Override `onGetControllerPermissions()` to add custom UI flags under `data.custom` (UI hint only — never your sole guard).
4. Override `applyACLFiltersToCDRQuery()` to add per-role CDR `WHERE` clauses; read the role from `$sessionContext['role']` and return early when it is null.
5. Guard your REST controllers with `#[ResourceSecurity(...)]`.

## See also

* [Rights and authentication overview](./)
* [Module hooks reference](../../module-developement/hooks-reference.md)
* [REST API reference](../../internal-structure/api/rest-api.md)
