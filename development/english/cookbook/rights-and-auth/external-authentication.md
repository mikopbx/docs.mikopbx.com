---
description: Integrate LDAP/OAuth login via the authenticateUser hook.
---

# External authentication

MikoPBX delegates web login to modules through a single hook. By implementing `authenticateUser()` on your module's `ConfigClass`, you can authenticate the operator against an LDAP directory, an OAuth/OpenID Connect identity provider, or any external IdP — and still fall back to MikoPBX's built-in credential check when your module does not recognise the user.

Throughout this recipe the running example is **ModuleBlackList** (config class `BlackListConf`). The factual anchors, however, are the production module **ModuleUsersUI**, which ships a complete external-auth implementation:

* Config class — `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`
* Authenticator — `Extensions/ModuleUsersUI/Lib/UsersUIAuthenticator.php`
* LDAP settings model — `Extensions/ModuleUsersUI/Models/LdapConfig.php`

## Where the hook fits in the login flow

Web login and the REST login action both funnel through one place: `MikoPBX\Common\Library\Auth\CredentialsValidator`. Its `authenticate()` method runs two checks, **in order**:

1. **Admin credentials first.** `checkAdminCredentials()` compares the submitted login/password against the system administrator account (`PbxSettings::WEB_ADMIN_LOGIN` / `WEB_ADMIN_PASSWORD`). If that matches, the user logs in as `admins` and modules are never consulted.
2. **Module hooks second.** Only if the admin check fails does `authenticateViaModules()` fan out to every enabled module.

{% code title="Core/src/Common/Library/Auth/CredentialsValidator.php" %}
```php
public static function authenticateViaModules(string $login, string $password): ?array
{
    // Try to authenticate via module hooks
    $moduleResults = PBXConfModulesProvider::hookModulesMethod(
        WebUIConfigInterface::AUTHENTICATE_USER,
        [$login, $password]
    );

    foreach ($moduleResults as $sessionData) {
        if (!empty($sessionData) && is_array($sessionData)) {
            return $sessionData;
        }
    }

    return null;
}
```
{% endcode %}

{% hint style="warning" %}
**You cannot override the administrator login.** Because the admin check runs first, returning a session array for the admin login from your module has no effect. The hook authenticates _additional_ (non-admin) users only.
{% endhint %}

The hook is iterated across all modules and **the first non-empty result wins**. The contract for each module is therefore:

* return a **non-empty session-data array** to log the user in with that role, or
* return **`[]`** to say "not my user" — the loop continues to the next module, and ultimately authentication fails with a normal "invalid credentials" error.

The constant and the method signature are both fixed by the interface:

{% code title="Core/src/Modules/Config/WebUIConfigInterface.php" %}
```php
// line 39
public const string AUTHENTICATE_USER = 'authenticateUser';

// line 70
public function authenticateUser(string $login, string $password): array;
```
{% endcode %}

`ConfigClass` ships a no-op implementation at `Core/src/Modules/Config/ConfigClass.php:405`, so `authenticateUser()` is a method you **override**, and every module already satisfies the interface whether or not it cares about login.

## The session-data array

The array you return becomes the user's session. The keys are defined as constants on `MikoPBX\AdminCabinet\Controllers\SessionController` — always use the constants, never bare strings:

| Constant                       | Wire value | Meaning                                                                                                      |
| ------------------------------ | ---------- | ------------------------------------------------------------------------------------------------------------ |
| `SessionController::ROLE`      | `role`     | ACL role this user gets. For module users this is a custom role string registered in `onAfterACLPrepared()`. |
| `SessionController::HOME_PAGE` | `homePage` | Path the operator is redirected to after login.                                                              |
| `SessionController::USER_NAME` | `userName` | Display login stored in the session.                                                                         |

The role string ties this recipe to the ACL recipe: the role you return here must be one your module declared. ModuleUsersUI builds it from a per-module prefix constant — `Constants::MODULE_ROLE_PREFIX` (its real value is `'UsersUIRoleID'`) — plus the access-group id, e.g. `UsersUIRoleID42`. Define your own prefix constant; do not hardcode another module's value.

See [Roles, rights and ACL](./) for how to register the role itself, and the [Hooks reference](../../module-developement/hooks-reference.md) for the full list of web-UI hooks.

## Step 1 — Store external-IdP configuration in a module Model

Authentication settings (LDAP server, base DN, bind account, OAuth client id and secret) belong in a module-owned table, not in code. Create a model that extends `MikoPBX\Modules\Models\ModulesModelsBase` and pins its table name in `initialize()`.

{% code title="Extensions/ModuleBlackList/Models/ExternalAuthConfig.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Models;

use MikoPBX\Modules\Models\ModulesModelsBase;

class ExternalAuthConfig extends ModulesModelsBase
{
    /**
     * @Primary
     * @Identity
     * @Column(type="integer", nullable=false)
     */
    public $id;

    /** @Column(type="string", length=1, nullable=false, default="0") */
    public ?string $enabled = '0';

    /** @Column(type="string", nullable=false) */
    public $serverName;

    /** @Column(type="string", nullable=false) */
    public $serverPort;

    /** @Column(type="string", nullable=false) */
    public $baseDN;

    /** @Column(type="string", nullable=false) */
    public $bindLogin;

    /**
     * Bind (service) account password used to query the directory.
     * This is NOT an end-user password — it is the credential the module
     * uses to look users up. Protect access to this table with module ACL.
     *
     * @Column(type="string", nullable=false)
     */
    public $bindPassword;

    public function initialize(): void
    {
        $this->setSource('m_BlackListExternalAuthConfig');
        parent::initialize();
    }
}
```
{% endcode %}

{% hint style="warning" %}
**Column typing rules — these are not stylistic.** The primary key must stay **untyped** (`public $id;`): giving it an `int` type causes a fatal during `save()`. String columns are either left untyped or declared `?string` with an empty-string (or other non-null) default — never a non-nullable typed property. `LdapConfig` shows both shapes side by side: untyped `public $serverName;`, `public $baseDN;` alongside `public ?string $tlsMode = 'none';` and `public ?string $verifyCert = '0';` (`Extensions/ModuleUsersUI/Models/LdapConfig.php:32-140`).
{% endhint %}

{% hint style="info" %}
Store only the **service/bind** account credentials and connection parameters here. End-user passwords are never stored: for LDAP you verify them by binding to the directory, and for OAuth you never see them at all. The table holding the bind/service secret should be reachable only through your module's protected controllers.
{% endhint %}

The production reference is `Extensions/ModuleUsersUI/Models/LdapConfig.php`, whose table is `m_ModuleUsersUI_LDAP_Config`. It carries the full set of LDAP fields — `serverName`, `serverPort`, `tlsMode` (`none`/`starttls`/`ldaps`), `verifyCert`, `caCertificate`, `administrativeLogin`, `administrativePassword`, `baseDN`, `userFilter`, `userIdAttribute`, `organizationalUnit` and `ldapType`.

## Step 2 — Implement `authenticateUser()` on the ConfigClass

Read your configuration, then branch on the IdP type. The two common branches:

* **LDAP** — bind to the directory with the supplied login/password. There is no local hash to compare; a successful bind _is_ the proof of identity.
* **Local fallback / password hash** — if you keep a local password hash, verify it with `Phalcon\Encryption\Security::checkHash()`, exactly as `CredentialsValidator` does for the admin account.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\AdminCabinet\Controllers\SessionController;
use MikoPBX\Modules\Config\ConfigClass;
use Modules\ModuleBlackList\Models\ExternalAuthConfig;

class BlackListConf extends ConfigClass
{
    /**
     * Role prefix owned by this module. Combine it with an internal id to
     * build a unique ACL role string registered in onAfterACLPrepared().
     */
    public const ROLE_PREFIX = 'BlackListRoleID';

    /**
     * Authenticate a non-admin user against the external IdP.
     *
     * @param string $login    Login entered on the web login form.
     * @param string $password Password entered on the web login form.
     * @return array Session data (role, homePage, userName) on success,
     *               or [] to fall through to core / other modules.
     */
    public function authenticateUser(string $login, string $password): array
    {
        $config = ExternalAuthConfig::findFirst();
        if ($config === null || $config->enabled !== '1') {
            // Not configured: defer to core authentication.
            return [];
        }

        if (!$this->checkExternalLogin($config, $login, $password)) {
            // Wrong credentials or unknown user — not handled here.
            return [];
        }

        // Identity proven. Map the directory user onto a MikoPBX role.
        return [
            SessionController::ROLE      => self::ROLE_PREFIX . '1',
            SessionController::HOME_PAGE => '/admin-cabinet/extensions/index',
            SessionController::USER_NAME => $login,
        ];
    }

    /**
     * Bind to the LDAP directory to verify the user's password.
     */
    private function checkExternalLogin(
        ExternalAuthConfig $config,
        string $login,
        string $password
    ): bool {
        if ($password === '') {
            // RFC 4513: an empty password causes an "unauthenticated bind",
            // which succeeds without verifying anything. Reject it explicitly.
            return false;
        }

        $uri  = sprintf('ldap://%s:%s', $config->serverName, $config->serverPort);
        $conn = @ldap_connect($uri);
        if ($conn === false) {
            return false;
        }
        ldap_set_option($conn, LDAP_OPT_PROTOCOL_VERSION, 3);
        ldap_set_option($conn, LDAP_OPT_REFERRALS, 0);

        $userDn = sprintf('uid=%s,%s', ldap_escape($login, '', LDAP_ESCAPE_DN), $config->baseDN);

        // A successful bind with the user's own DN + password proves identity.
        $ok = @ldap_bind($conn, $userDn, $password);
        ldap_unbind($conn);

        return $ok;
    }
}
```
{% endcode %}

{% hint style="danger" %}
**Never short-circuit on an empty password.** Most LDAP servers treat a bind with an empty password as an _unauthenticated bind_ that returns success. Always reject an empty password before binding, as shown above.
{% endhint %}

### How ModuleUsersUI does it (real example)

`Extensions/ModuleUsersUI/Lib/UsersUIConf.php` keeps `authenticateUser()` thin and delegates to a dedicated authenticator:

{% code title="Extensions/ModuleUsersUI/Lib/UsersUIConf.php" %}
```php
public function authenticateUser(string $login, string $password): array
{
    $authenticator = new UsersUIAuthenticator($login, $password);
    return $authenticator->authenticate() ?? [];
}
```
{% endcode %}

`UsersUIAuthenticator::authenticate()` (in `Extensions/ModuleUsersUI/Lib/UsersUIAuthenticator.php`) looks the login up in its own `UsersCredentials`/`AccessGroups` tables, then either binds against LDAP (`LdapConfig::findFirst()` + an LDAP helper) or verifies a locally stored hash, and on success returns the `role` / `homePage` / `userName` array.

{% hint style="info" %}
The production authenticator still routes its local-password branch through a legacy version-shim helper. On MikoPBX 2025.1.1+ you do **not** need that shim — verify a stored hash directly with `Phalcon\Encryption\Security`:

```php
use Phalcon\Encryption\Security;

$security = new Security();
if ($security->checkHash($password, $userPasswordHash)) {
    // local credentials valid
}
```
{% endhint %}

### OAuth / OpenID Connect variant

For OAuth/OIDC the browser typically completes the provider's flow first and your module receives an authorization code or an already-validated token. In that case `authenticateUser()` validates the token (introspection or signature/`userinfo` check), maps the verified subject/claims to a MikoPBX role, and returns the same session-data array. If no token is present, return `[]` so the standard form login still works.

## Step 3 — WebAuthn passkeys: `getPasskeySessionData()`

Passkey (WebAuthn) login is a separate path. The cryptographic challenge is verified by the Core action `Core/src/PBXCoreREST/Lib/Passkeys/AuthenticationFinishAction.php`. By the time your module is called, the user's identity is **already proven** — so the hook takes only the login and **no password**:

{% code title="Core/src/Modules/Config/WebUIConfigInterface.php" %}
```php
// line 41 — the constant, and nothing else
public const string GET_PASSKEY_SESSION_DATA = 'getPasskeySessionData';
```
{% endcode %}

{% hint style="warning" %}
**This hook is not symmetric with `authenticateUser()`.** The interface declares only the _constant_ at line 41 — there is **no** `public function getPasskeySessionData(string $login): array;` declaration in `WebUIConfigInterface`, and **no** no-op base implementation in `ConfigClass` (compare `authenticateUser()`, which has both). So this is a method you **add** to your config class, not one you override; your IDE will not offer to generate it, and misspelling it fails silently.

It works at all only because `PBXConfModulesProvider::hookModulesMethod()` guards every dispatch with `method_exists($configClassObj, $methodName)` and skips modules that lack the method (`Core/src/Common/Providers/PBXConfModulesProvider.php:94-115`). Match the signature exactly — `getPasskeySessionData(string $login): array` — since nothing in the type system will check it for you.
{% endhint %}

The Core action fans the login out to modules to build the session:

{% code title="Core/src/PBXCoreREST/Lib/Passkeys/AuthenticationFinishAction.php" %}
```php
$moduleResults = PBXConfModulesProvider::hookModulesMethod(
    WebUIConfigInterface::GET_PASSKEY_SESSION_DATA,
    [$login]
);

foreach ($moduleResults as $sessionParams) {
    if (is_array($sessionParams) && !empty($sessionParams)) {
        return $sessionParams;
    }
}
```
{% endcode %}

Your implementation returns the **same** session-data shape, by login alone:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
/**
 * Build session data for a passkey login.
 * Called only AFTER WebAuthn verification — no password is involved.
 *
 * @param string $login Login bound to the verified passkey credential.
 * @return array Session data (role, homePage, userName) or [].
 */
public function getPasskeySessionData(string $login): array
{
    $config = ExternalAuthConfig::findFirst();
    if ($config === null || $config->enabled !== '1') {
        return [];
    }

    // No password to check here — identity is already proven by WebAuthn.
    // Just resolve the login to a role and home page.
    return [
        SessionController::ROLE      => self::ROLE_PREFIX . '1',
        SessionController::HOME_PAGE => '/admin-cabinet/extensions/index',
        SessionController::USER_NAME => $login,
    ];
}
```
{% endcode %}

ModuleUsersUI implements the same idea by reusing its authenticator's "resolve session by login, no password" path:

{% code title="Extensions/ModuleUsersUI/Lib/UsersUIConf.php" %}
```php
public function getPasskeySessionData(string $login): array
{
    $authenticator = new UsersUIAuthenticator($login, '');
    return $authenticator->getSessionParamsForLogin() ?? [];
}
```
{% endcode %}

{% hint style="info" %}
Passkeys themselves are stored centrally by Core (the passkey credential is bound to a login). Your module only maps that login to a role and home page — it does not store the credential.
{% endhint %}

## How the session is consumed afterwards

Once login succeeds, MikoPBX issues a JWT and the role you returned drives every subsequent ACL decision in `MikoPBX\AdminCabinet\Plugins\SecurityPlugin`. On each request `SecurityPlugin` extracts the role (from the `Authorization: Bearer` header on AJAX calls, or from the `refreshToken` cookie mapped through Redis on page loads) and calls `isAllowedAction($controller, $action)` against the ACL. So the role string from `authenticateUser()` must be one your module registered in `onAfterACLPrepared()` — otherwise the user logs in but is denied everywhere.

The same JWT/role machinery backs the REST API; see [REST API](../../internal-structure/api/rest-api.md) for how external clients obtain and present tokens.

## Checklist

* [ ] Settings live in a `ModulesModelsBase` model with `setSource()`; only the bind/service secret is stored, never end-user passwords.
* [ ] `authenticateUser(string $login, string $password): array` returns a `role`/`homePage`/`userName` array on success, `[]` otherwise.
* [ ] Empty passwords are rejected before any LDAP bind.
* [ ] Local hash checks use `Phalcon\Encryption\Security::checkHash()` — no version shims.
* [ ] The returned `role` matches a role registered in `onAfterACLPrepared()`.
* [ ] `getPasskeySessionData(string $login): array` returns the same shape, by login only, with no password parameter.

## Related pages

* [Roles, rights and ACL](./)
* [REST API](../../internal-structure/api/rest-api.md)
* [Hooks reference](../../module-developement/hooks-reference.md)
