---
description: Recipes for authentication, ACL roles and per-role data filtering in modules.
---

# Rights and auth

MikoPBX lets a module take over **who can log in**, **what each role is allowed to see**, and **which data rows that role receives**. All three are driven by hooks declared in the Core web-interface and CDR interfaces, and implemented in your module configuration class (`BlackListConf` in our running **ModuleBlackList** example, extending `MikoPBX\Modules\Config\ConfigClass`).

The richest real-world implementation of every recipe on this page is **`ModuleUsersUI`**. It defines arbitrary access groups, maps each group to a MikoPBX ACL role, authenticates external (non-admin) logins, and filters the CDR table per role. Read it alongside these recipes: `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`.

{% hint style="info" %}
The minimal, copy-paste-able skeleton lives in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`. Use `ModuleUsersUI` to see the patterns at full scale, and `ModuleExampleForm` to see the smallest working shape.
{% endhint %}

## The hooks involved

Authentication and ACL hooks come from **two separate Core interfaces**. Your config class extends `ConfigClass`, which already provides empty defaults for all of them, so you only override the ones you need.

| Hook                                                                              | Interface                                     | What it does                                                                                                               |
| --------------------------------------------------------------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `authenticateUser(string $login, string $password): array`                        | `MikoPBX\Modules\Config\WebUIConfigInterface` | Validate a login/password pair against your own user store and return session data (role, home page, user name).           |
| `onAfterACLPrepared(\Phalcon\Acl\Adapter\Memory $aclList): void`                  | `MikoPBX\Modules\Config\WebUIConfigInterface` | Register extra ACL roles and `allow`/`deny` rules on the in-memory ACL list.                                               |
| `onGetControllerPermissions(string $controller, array &$permissions): void`       | `MikoPBX\Modules\Config\WebUIConfigInterface` | Contribute custom per-controller permissions when the ACL editor enumerates a controller's actions.                        |
| `applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void` | `MikoPBX\Modules\Config\CDRConfigInterface`   | Inject extra `WHERE`/`bind` parameters into the CDR query so a role only ever receives the call records it is entitled to. |

{% hint style="info" %}
Every hook name is also published as a constant on its interface, e.g. `WebUIConfigInterface::AUTHENTICATE_USER`, `WebUIConfigInterface::ON_AFTER_ACL_LIST_PREPARED`, `WebUIConfigInterface::ON_GET_CONTROLLER_PERMISSIONS`, `CDRConfigInterface::APPLY_ACL_FILTERS_TO_CDR_QUERY`. The Core calls them by name on enabled modules — you implement the method, you do not register it.
{% endhint %}

For the complete catalogue of every web-interface hook, its exact signature and when it fires, see [../../module-developement/hooks-reference.md](../../module-developement/hooks-reference.md).

## Recipe 1 — External authentication

Override `authenticateUser()` to authenticate users that do not exist in the standard MikoPBX admin account — LDAP/AD users, customer-portal accounts, or any external identity provider. On success you return an array carrying the role to assign for the session; on failure you return an empty array and the Core falls back to the next authenticator.

```php
public function authenticateUser(string $login, string $password): array
{
    // Verify against your own store, then describe the session.
    // Empty array => "not my user", let the Core continue.
}
```

`ModuleUsersUI` implements this by delegating to its `UsersUIAuthenticator`, which looks the login up in its own `UsersCredentials` model, optionally checks the password against an LDAP server, and returns the session keys defined by the Core `SessionController` constants — `SessionController::ROLE`, `SessionController::HOME_PAGE`, `SessionController::USER_NAME`. The role it assigns is its own ACL role name (see Recipe 2).

See the full walkthrough, including the LDAP path and the `getPasskeySessionData()` companion hook, in [external-authentication.md](external-authentication.md). Real example: `Extensions/ModuleUsersUI/Lib/UsersUIConf.php` and `Extensions/ModuleUsersUI/Lib/UsersUIAuthenticator.php`.

## Recipe 2 — Custom ACL roles and limited rights

Override `onAfterACLPrepared()` to add your own roles to the Phalcon ACL and grant each role access only to the controllers and actions it should reach. A role created here is what `authenticateUser()` returns, closing the loop: your authenticator decides _who_ the user is, the ACL decides _what_ that user can do.

```php
use Phalcon\Acl\Adapter\Memory as AclList;

public function onAfterACLPrepared(AclList $aclList): void
{
    // $aclList->addRole(...); $aclList->addComponent(...); $aclList->allow(...);
}
```

`ModuleUsersUI` builds one ACL role per access group, naming each role with its own prefix constant `Constants::MODULE_ROLE_PREFIX` (the literal `'UsersUIRoleID'`) followed by the group id, e.g. `UsersUIRoleID42`. The role-building logic — querying access groups, expanding linked controllers, and granting `allow($role, $controller, $actions)` — lives in `Extensions/ModuleUsersUI/Lib/UsersUIACL.php` (`UsersUIACL::modify()`), which `UsersUIConf::onAfterACLPrepared()` calls directly.

{% hint style="warning" %}
ACL roles are cached. When the data backing your roles changes, clear the cache so the Core rebuilds the ACL. `ModuleUsersUI` calls `MikoPBX\Common\Providers\AclProvider::clearCache()` from `onAfterModuleEnable()`, `onAfterModuleDisable()`, and its `modelsEventChangeData()` handler whenever a role-related model is written.
{% endhint %}

### Per-role CDR filtering

Limited rights are not only about hiding pages — a role often must also see only _its own_ call records. Override `applyACLFiltersToCDRQuery()` (from `CDRConfigInterface`) to append conditions to the CDR query before it runs.

```php
public function applyACLFiltersToCDRQuery(array &$parameters, array $sessionContext = []): void
{
    // $sessionContext['role'] holds the JWT role; return early when it is null.
    // Mutate $parameters['conditions'] / $parameters['bind'] to scope the rows.
}
```

`$sessionContext` carries the role from the JWT token (`$sessionContext['role']`, `$sessionContext['user_name']`, `$sessionContext['session_id']`). Every CDR list query — including the one behind the AdminCabinet _Call Detail Records_ page — goes through the REST API, so the role is always available there; ignore the "AdminCabinet context is empty, read `SessionProvider`" wording still present in the interface docblock. `ModuleUsersUI` recovers the access-group id by stripping `Constants::MODULE_ROLE_PREFIX` from the role, then narrows the query in `Extensions/ModuleUsersUI/Lib/UsersUICDRFilter.php`.

The full role/ACL/CDR-filter recipe — including `onGetControllerPermissions()` for exposing custom controller actions in the ACL editor — is in [limited-rights.md](limited-rights.md). Real example: `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`, `Extensions/ModuleUsersUI/Lib/UsersUIACL.php` and `Extensions/ModuleUsersUI/Lib/UsersUICDRFilter.php`.

## Where to go next

* [external-authentication.md](external-authentication.md) — implement `authenticateUser()` (and passkey/LDAP variants).
* [limited-rights.md](limited-rights.md) — define ACL roles, controller permissions, and per-role CDR filtering.
* [../../module-developement/hooks-reference.md](../../module-developement/hooks-reference.md) — full signatures and firing order for every web-interface and CDR hook.
