---
description: >-
  Conventions, PHP 8.4 idioms, and anti-patterns to avoid when building MikoPBX
  modules.
---

# Best practices

## Best practices

This page is the rulebook for writing MikoPBX modules that are clean, secure, and indistinguishable from Core code. It targets **MikoPBX 2025.1.1+, PHP 8.4, Phalcon 5.9.3**. Everything here is verified against the Core source and the example modules shipped under `Extensions/EXAMPLES/`.

The running example throughout this documentation is a fictional module **`ModuleBlackList`** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`, front-end `module-black-list-index.js`). Every rule below is also anchored to a **real** example module by repo-relative path so you can read working code.

{% hint style="info" %}
**How to read this page.** The conventions come first (naming, file layout, PHP idioms). The anti-patterns come second, each with a _Detection_ signal, the _Problem_, and a copy-paste _Fix_. Security anti-patterns (`S1`–`S6`) are priority fixes — treat them before any code-quality cleanup.
{% endhint %}

### File headers

Every PHP file in a module starts the same way and never emits a closing tag:

```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;
```

* `declare(strict_types=1);` is mandatory on **every** file (anti-pattern #10 below). It makes scalar type hints reject silent coercion — a `string` passed where `int` is declared throws `TypeError` instead of becoming `0`.
* No closing `?>`. A trailing tag risks emitting stray whitespace that corrupts headers or generated config files.

See a real header in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`.

### The DI import rule

This single import trips up almost every new module. Phalcon 5 moved the container class into its own sub-namespace:

```php
// CORRECT — Phalcon 5.9.3
use Phalcon\Di\Di;

$di = Di::getDefault();
$db = $di->get('db');
```

```php
// WRONG — Phalcon 4 era, will not resolve in 5.9.3
use Phalcon\Di;
```

Confirmed in real module code at `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php` (`$di = Di::getDefault();`) and `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv2/Lib/RestAPI/Backend/Actions/GetUsersAction.php`.

### Naming conventions

Consistent names are not cosmetic — the loader, the table-name resolver, the asset pipeline, and the REST router all derive paths from these patterns. Deviating breaks autodiscovery silently.

| Element                | Convention                                   | `ModuleBlackList` example                            |
| ---------------------- | -------------------------------------------- | ---------------------------------------------------- |
| Module ID / directory  | `Module{Feature}` (PascalCase)               | `ModuleBlackList`, dir `Extensions/ModuleBlackList/` |
| `moduleUniqueID`       | identical to Module ID                       | `ModuleBlackList`                                    |
| Config class           | `{Feature}Conf`                              | `BlackListConf`                                      |
| Main logic class       | `{Feature}Main`                              | `BlackListMain`                                      |
| Setup class            | always `PbxExtensionSetup`                   | `Setup/PbxExtensionSetup.php`                        |
| Model                  | `{Entity}`                                   | `BlackListNumbers`                                   |
| DB table               | `m_{Entity}` (auto from model)               | `m_BlackListNumbers`                                 |
| Web controller         | `Module{Feature}Controller`                  | `ModuleBlackListController`                          |
| Form                   | `Module{Feature}Form`                        | `ModuleBlackListForm`                                |
| Worker                 | `Worker{Feature}{Type}`                      | `WorkerBlackListAMI`                                 |
| REST resource classes  | `Controller` / `Processor` / `DataStructure` | `Lib/RestAPI/Numbers/Controller.php`                 |
| REST action            | `{Verb}{Entity}Action`                       | `GetListAction`, `SaveRecordAction`                  |
| JS source / compiled   | `module-{kebab}-{action}.js`                 | `public/assets/js/src/module-black-list-index.js`    |
| CSS                    | `module-{kebab}-{action}.css`                | `public/assets/css/module-black-list-index.css`      |
| Translation key prefix | `module_{feature}_`                          | `module_black_list_NumberColumn`                     |
| Dialplan context       | `[module-{kebab}-{purpose}]`                 | `[module-black-list-check]`                          |

Asset names carry the controller **action** suffix (`-index`, `-modify`, …) because the Core loads them per action — see `ModuleExampleFormController::indexAction()` / `modifyAction()` in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`.

#### Namespaces

The PHP namespace mirrors the directory layout exactly (PSR-4):

```
Modules\ModuleBlackList\Setup            → Setup/PbxExtensionSetup.php
Modules\ModuleBlackList\Lib              → Lib/BlackListConf.php, Lib/BlackListMain.php
Modules\ModuleBlackList\Models           → Models/BlackListNumbers.php
Modules\ModuleBlackList\App\Controllers  → App/Controllers/ModuleBlackListController.php
Modules\ModuleBlackList\App\Forms        → App/Forms/ModuleBlackListForm.php
```

#### REST API path scheme

```
/pbxcore/api/v3/module-{kebab}/{resource}
/pbxcore/api/v3/module-{kebab}/{resource}/{id}
/pbxcore/api/v3/module-{kebab}/{resource}/{id}:{action}
```

```
GET    /pbxcore/api/v3/module-black-list/numbers
POST   /pbxcore/api/v3/module-black-list/numbers
GET    /pbxcore/api/v3/module-black-list/numbers/123
DELETE /pbxcore/api/v3/module-black-list/numbers/123
```

See the live v3 implementation in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/`.

### PHP 8.4 idioms

Write modern PHP everywhere **except** Phalcon model column properties (see the [model exception](best-practices.md#the-phalcon-model-exception) immediately after).

#### Typed properties on non-model classes

`Conf`, `Main`, `Worker`, controllers, and service classes get full property types:

```php
class BlackListMain extends PbxExtensionBase
{
    public string $logPath = '';
    public readonly string $moduleUniqueId;
}
```

#### Constructor property promotion

```php
final class BlackListNotifier
{
    public function __construct(
        private readonly string $apiUrl,
        private readonly int $timeout = 5,
    ) {}
}
```

#### `match` over `switch` and over dynamic dispatch

Use `match` for request routing inside Conf hooks. It is exhaustive, returns a value, and uses strict comparison. This is the canonical REST callback pattern.

{% code title="Lib/BlackListConf.php" %}
```php
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;

public function moduleRestAPICallback(array $request): PBXApiResult
{
    $action = $request['action'] ?? '';
    $data   = $request['data'] ?? [];

    return match ($action) {
        'check'  => $this->checkAction($data),
        'reload' => $this->reloadAction(),
        default  => $this->createErrorResult("Unknown action: {$action}"),
    };
}

/**
 * There is no inherited error-result helper — each module defines its own.
 */
private function createErrorResult(string $message): PBXApiResult
{
    $result = new PBXApiResult();
    $result->processor = __METHOD__;
    $result->success   = false;
    $result->messages['error'] = [$message];

    return $result;
}
```
{% endcode %}

{% hint style="warning" %}
`createErrorResult()` is **not** a method of `MikoPBX\Modules\Config\ConfigClass` — do not assume it is inherited. It is a small private helper that each module declares for itself, which is why it is spelled out above. The shipping examples do the same: `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` and `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv1/Lib/ExampleRestAPIv1Conf.php` each define their own copy.
{% endhint %}

This is exactly the shape used in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` (`moduleRestAPICallback`). `match` is also the prescribed fix for the dynamic-dispatch security hole — see [S6](best-practices.md#s6-dynamic-dispatch-from-user-input).

#### Named arguments and backed enums

```php
enum CallDirection: string
{
    case Incoming = 'incoming';
    case Outgoing = 'outgoing';
}

$direction = CallDirection::from($channelVar);   // throws on unknown value
```

Named arguments make multi-flag calls self-documenting:

```php
Processes::processPHPWorker(
    className: WorkerBlackListAMI::class,
);
```

#### Full type declarations on methods

Every method declares parameter types and a return type — including `void` and `: never`. Anti-pattern [#11](best-practices.md#11-low-untyped-methods-and-properties-and-the-one-place-types-break-the-orm) flags untyped methods, and covers the one exception where a type declaration actually breaks the ORM.

```php
public function isBlocked(string $number): bool { /* ... */ }
private function refreshCache(): void { /* ... */ }
```

### The Phalcon model exception

{% hint style="warning" %}
**Model column properties are the one place you do NOT apply the PHP 8.4 typed-property rules.** This is intentional and matches the Core models — it is NOT an anti-pattern, and code reviewers/linters must not flag it.
{% endhint %}

SQLite stores everything as text, and Phalcon hydrates model columns from those text values. The Core models therefore follow this exact pattern:

{% code title="Models/BlackListNumbers.php" %}
```php
class BlackListNumbers extends ModelsBase
{
    // Primary key — ALWAYS untyped. Phalcon assigns it after insert.
    public $id;

    // String columns — nullable, with a string default.
    public ?string $number = '';

    // Booleans / small integers stored as string ('0' / '1').
    public ?string $enabled = '1';

    // Integer foreign keys — nullable int, default null.
    public ?int $userid = null;
}
```
{% endcode %}

Why each rule holds:

* **`public $id;` untyped** — the row has no `id` until after `save()`; a typed property would have to be uninitialized then suddenly populated, fighting the ORM.
* **`?string` columns with `''` default** — values arrive as strings from SQLite; nullable absorbs `NULL` columns.
* **Integers-as-strings (`'0'`/`'1'`)** — flag columns round-trip as text; declaring them `?string` avoids lossy coercion.
* **Nullable int FKs (`?int … = null`)** — an unset foreign key is genuinely `NULL`.

This is confirmed verbatim in the Core models `src/Common/Models/Sip.php` (`public $id;`, `public ?string $disabled = '0';`), `src/Common/Models/Extensions.php` (`public ?int $userid = null;`), and `src/Common/Models/CallQueues.php`. Use those as your reference, not example modules — see also [Data model](data-model.md).

***

## Code-quality anti-patterns

Each entry lists how to spot it, why it hurts, and the fix. They are ordered by severity, and the numbers are local to this page.

### #1 \[CRITICAL] `MikoPBXVersion.php` in new modules

**Detection.** A `MikoPBXVersion.php` file or `MikoPBXVersion::getDefaultDi()` calls in a module whose `module.json` declares `min_pbx_version` ≥ 2025.1.1.

**Problem.** `MikoPBXVersion` was a legacy compatibility shim. On a 2025.1.1+ baseline it is dead weight, and it hides the real DI container behind an indirection.

**Fix.** Delete the file and call the container directly:

```php
// BEFORE (legacy)
$di = MikoPBXVersion::getDefaultDi();

// AFTER (modern)
use Phalcon\Di\Di;

$di = Di::getDefault();
```

`Di::getDefault()` is the pattern used throughout the examples, e.g. `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php`.

### #2 \[CRITICAL] `shell_exec` for reloads instead of framework methods

**Detection.** `shell_exec(...)` or `exec(...)` in `Lib/*Conf.php` invoking `asterisk -rx`, `safe_asterisk`, or service scripts.

**Problem.** Shelling out to reload Asterisk bypasses MikoPBX process management, skips config regeneration, races with other reloads, and runs unescaped (see [S3](best-practices.md#s3-command-injection)).

**Fix.** Use the framework's reload entry points. The two you will actually call from a module are:

```php
use MikoPBX\Core\System\System;
use MikoPBX\Core\System\Processes;

// Reload Asterisk Manager (AMI) — applies a manager.conf user, etc.
System::invokeActions(['manager' => 0]);

// Reload cron jobs after touching scheduled tasks.
System::invokeActions(['cron' => 0]);

// (Re)start this module's own workers — see #9 below.
foreach ($this->getModuleWorkers() as $worker) {
    Processes::processPHPWorker($worker['worker']);
}
```

This is the real pattern in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` (`onAfterModuleEnable()` calls `System::invokeActions(['manager' => 0])`) driving `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiMain.php` (`startAllServices()` loops over `getModuleWorkers()` and calls `Processes::processPHPWorker()`).

{% hint style="warning" %}
`System::invokeActions()` recognizes only the `manager` and `cron` keys — other keys are silently ignored. It is a thin wrapper that dispatches to `WorkerModelsEvents::invokeAction(ReloadManagerAction::class)` / `ReloadCrondAction::class`, and is itself marked **`@deprecated`** in favour of calling `MikoPBX\Core\Workers\WorkerModelsEvents::invokeAction(...)` directly. The shipped example modules still use the `System::invokeActions(['manager' => 0])` wrapper, so it remains the most readable choice for module code today; prefer the `WorkerModelsEvents::invokeAction()` form if you want the non-deprecated call. For dialplan and SIP regeneration, the modern Core APIs are `MikoPBX\Core\Asterisk\Configs\ExtensionsConf::reload()` and `MikoPBX\Core\Asterisk\Configs\SIPConf::reload()`. The old facades `PBX::dialplanReload()` / `PBX::sipReload()` (and the whole `MikoPBX\Core\System\PBX` class) are **`@deprecated`** — do not use them in new code. The one non-deprecated reload method on `PbxConf` is `MikoPBX\Core\System\Configs\PbxConf::coreReload()`.
{% endhint %}

### #3 \[HIGH] Monolithic `Conf` class

**Detection.** `Lib/{Feature}Conf.php` over \~500 lines, or business logic (HTTP calls, parsing, ORM loops) living inside hook methods.

**Problem.** The `Conf` class exists to implement [hooks](module-class.md) and delegate. Stuffing logic into it makes it untestable and couples lifecycle events to behavior.

**Fix.** Extract behavior into `{Feature}Main`. Hooks stay thin:

```php
// Lib/BlackListConf.php
public function onAfterModuleEnable(): void
{
    System::invokeActions(['manager' => 0]);
    (new BlackListMain())->startAllServices();
}
```

`Extensions/EXAMPLES/AMI/ModuleExampleAmi/` is the canonical split: `ExampleAmiConf.php` (hooks) delegates to `ExampleAmiMain.php` (logic).

### #4 \[HIGH] Phantom model fields

**Detection.** Code reads `$record->someField` where `someField` is not a declared column property on the model.

**Problem.** Phalcon returns `null` for an undeclared property access instead of erroring, so the bug is silent — branches misfire, data goes unsaved.

**Fix.** Only access properties that are declared with column annotations on the model. If you need a new field, add it to the model **and** to the installer's migration (see [Data model](data-model.md)). Reference real models in `src/Common/Models/`.

### #5 \[MEDIUM] `die()` / `exit()` in library and worker classes

**Detection.** `die(` or `exit(` in `Lib/*.php` or `bin/*.php` (worker code).

**Problem.** Hard exits skip destructors and `finally` blocks, prevent graceful worker shutdown, and make the code untestable.

**Fix.** In a worker, log and flag for restart; in a library, throw.

```php
// In a worker (bin/WorkerBlackListAMI.php)
$this->logger->writeError('Settings not set');
$this->needRestart = true;
return;

// In a library (Lib/BlackListMain.php)
throw new \RuntimeException('Settings not set');
```

### #6 \[MEDIUM] `1*$variable` casting idiom

**Detection.** `1*$var` or `1*shell_exec(...)`.

**Problem.** A PHP 4-era trick to coerce to a number. Obscure and bypasses strict typing intent.

**Fix.** Cast explicitly:

```php
$count = (int) $var;
```

### #7 \[MEDIUM] `md5(print_r(...))` for change detection

**Detection.** `md5(print_r($data, true))`.

**Problem.** `print_r()` output is ambiguous — distinct structures can render identically — so the hash misses real changes.

**Fix.** Hash a canonical, unambiguous encoding:

```php
$hash = md5(json_encode($data, JSON_THROW_ON_ERROR));
```

### #8 \[MEDIUM] File-based IPC instead of Redis

**Detection.** `file_put_contents(... json_encode ...)` paired with `file_get_contents(... json_decode ...)` used to pass state between processes.

**Problem.** Files have no atomicity or TTL — workers race, and stale state accumulates.

**Fix.** Use the shared Redis service from the DI container, with an expiry:

```php
$redis = Di::getDefault()->get('redis');
$redis->setex("blacklist_state:{$id}", 600, json_encode($data, JSON_THROW_ON_ERROR));
```

### #9 \[MEDIUM] Manual worker killing

**Detection.** `Processes::killByName(...)` inside `Lib/*Conf.php` to restart workers.

**Problem.** Manually killing PIDs fights the safe-scripts watchdog, which will restart workers on its own schedule, producing flapping.

**Fix.** Let the framework manage worker lifecycle. Enumerate your workers through `getModuleWorkers()` (declared in `MikoPBX\Modules\Config\ConfigClass`) and hand each to `Processes::processPHPWorker()`, which restarts-or-starts as needed:

```php
foreach ($this->getModuleWorkers() as $worker) {
    Processes::processPHPWorker($worker['worker']);
}
```

This is `startAllServices()` in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiMain.php`.

### #10 \[LOW] Missing `strict_types` and wrong DI import

Covered above under [File headers](best-practices.md#file-headers) and [The DI import rule](best-practices.md#the-di-import-rule). Detection: `grep -L "declare(strict_types" *.php` finds files missing the declaration; `use Phalcon\Di;` (without `\Di`) is the wrong import.

### #11 \[LOW] Untyped methods and properties — and the one place types break the ORM

**Detection.** A method with no parameter types or no return type, or a non-model class property declared bare (`public $logPath;`). `grep -nE 'function [a-zA-Z]+\([^)]*\)\s*\{' Lib/*.php` finds methods missing a return type.

**Problem.** Without declared types, `declare(strict_types=1)` has nothing to enforce: a `null` flows into a string concatenation, an empty array satisfies a "count" parameter, and the failure surfaces three call frames later as a nonsense value rather than a `TypeError` at the boundary. On a PHP 8.4 baseline there is no reason to give that up — and static analysis (phpstan) can only reason about what you declare.

**Fix.** Type every parameter, every return, and every non-model property:

```php
// BEFORE — nothing is checked
public function isBlocked($number)
{
    return $this->cache[$number];
}

// AFTER — the boundary is enforced
public function isBlocked(string $number): bool
{
    return isset($this->cache[$number]);
}
```

#### The exception: never type a Phalcon model's primary key

This rule stops at the model layer, and getting it wrong produces a **fatal error rather than a warning** — which is why it is worth stating as its own anti-pattern rather than a footnote.

```php
// WRONG — fatal
class BlackListNumbers extends ModelsBase
{
    public int $id;
}

// RIGHT — untyped, as in every Core model
class BlackListNumbers extends ModelsBase
{
    public $id;
}
```

A typed, non-nullable `int $id` is **uninitialized** on a freshly constructed model — PHP typed properties have no implicit default. A new record has no `id` until the INSERT returns one, so any read of the property before that point raises:

```
Error: Typed property BlackListNumbers::$id must not be accessed before initialization
```

Phalcon reads the primary key while deciding whether `save()` is an INSERT or an UPDATE, so the very first `save()` on a new record blows up. Declaring `public ?int $id = null;` avoids the fatal but still fights the ORM's own hydration; the Core convention is the plain untyped `public $id;`.

This is not theory — it is what every Core model does: `Core/src/Common/Models/Sip.php:68` and `Core/src/Common/Models/Extensions.php:79` both declare `public $id;`. The same file pair shows the rest of the column convention: `Sip.php:82` (`public ?string $disabled = '0';`) and `Extensions.php:107` (`public ?int $userid = null;`).

The full rationale for all four column patterns is in [The Phalcon model exception](best-practices.md#the-phalcon-model-exception) above.

***

## Security anti-patterns

{% hint style="danger" %}
Modules run inside the PBX with full filesystem and DB access; AGI scripts run as **root**. A vulnerable module compromises the whole appliance. Fix every `S`-level finding **before** any code-quality work. For the authorization model and how endpoints are protected, see [Rights and authentication](../cookbook/rights-and-auth/).
{% endhint %}

### S1 \[CRITICAL] Unauthenticated endpoints

**Detection.** A REST route registered with `noAuth = true` — the 6th element (`$additionalRoute[5]`) of a route array returned by `getPBXCoreRESTAdditionalRoutes()`. When that flag is `true`, `Request::thisIsModuleNoAuthRequest()` matches the URI and lets the request through **without** authentication.

**Problem.** An unauthenticated route that mutates state, originates calls, or returns sensitive data is an open door.

**Fix.** Never set `noAuth = true` for endpoints that modify settings, originate calls, expose credentials/tokens, touch CDR or recordings, or serve files by user path. Keep them behind the default auth, or verify an HMAC/token inside the handler.

### S2 \[CRITICAL] SQL injection in `findFirst` / `find`

**Detection.** String interpolation inside a condition: `Model::findFirst("col='{$x}'")`.

**Problem.** User input concatenated into the WHERE clause is classic SQL injection.

**Fix.** Always use parameterized binds:

```php
// BEFORE (vulnerable)
BlackListNumbers::findFirst("number='{$userInput}'");

// AFTER (safe)
BlackListNumbers::findFirst([
    'conditions' => 'number = :num:',
    'bind'       => ['num' => $userInput],
]);
```

### S3 \[CRITICAL] Command injection

**Detection.** A variable inside `shell_exec` / `Processes::mwExec` / `exec` without `escapeshellarg()`. Especially dangerous in `agi-bin/` (root) and workers handling external data (CDR filenames, SIP headers, API payloads).

**Problem.** Unescaped input becomes shell syntax — arbitrary command execution.

**Fix.** Wrap **every** variable:

```php
// BEFORE (vulnerable)
shell_exec("$sox $inputFile $outputFile");

// AFTER (safe)
shell_exec(sprintf(
    '%s %s %s',
    $sox,
    escapeshellarg($inputFile),
    escapeshellarg($outputFile),
));
```

### S4 \[CRITICAL] Path traversal / arbitrary file read

**Detection.** `fopen` / `file_get_contents` / `readfile` / `fpassthru` on a user-supplied path with no validation against an allowed base directory.

**Problem.** `../../etc/...` escapes the intended directory and reads arbitrary files.

**Fix.** Resolve the real path and confirm it is inside the allowlisted base:

```php
$resolved    = realpath($userInput);
$allowedBase = realpath('/storage/usbdisk1/mikopbx/astspool/monitor/');

if ($resolved === false || !str_starts_with($resolved, $allowedBase)) {
    $this->sendError(403);
    return;
}
```

### S5 \[CRITICAL] Reflected XSS

**Detection.** `$_REQUEST` / `$_GET` / `$_POST` echoed into HTML or inline JS without escaping.

**Problem.** Attacker-controlled markup executes in the admin's browser.

**Fix.** Escape on output:

```php
<?php echo htmlspecialchars($input ?? '', ENT_QUOTES, 'UTF-8'); ?>
```

### S6 \[HIGH] Dynamic dispatch from user input

**Detection.** `$this->$action(...)` / `self::$method(...)` where the name comes from the request.

**Problem.** A user-controlled method name exposes **every** public method of the class as a callable endpoint.

**Fix.** Replace variable dispatch with an explicit `match` allowlist (the same idiom used for [REST routing](best-practices.md#match-over-switch-and-over-dynamic-dispatch)):

```php
$result = match ($request['action'] ?? '') {
    'check'  => $this->checkAction($data),
    'reload' => $this->reloadAction(),
    default  => throw new \InvalidArgumentException('Unknown action'),
};
```

***

### Quick checklist before you ship

* [ ] Every `.php` file: `<?php` + `declare(strict_types=1);`, no closing tag.
* [ ] `use Phalcon\Di\Di;` — never `use Phalcon\Di;`.
* [ ] Non-model classes: typed properties, promoted constructors, full method types.
* [ ] Models: untyped `public $id;`, nullable `?string`/`?int` columns (the [exception](best-practices.md#the-phalcon-model-exception)).
* [ ] No `MikoPBXVersion`, no `PBX::*Reload()` facade — use `System::invokeActions()`, `Processes::processPHPWorker()`, or the modern `*Conf::reload()` methods.
* [ ] No `shell_exec`/`exec` for reloads; no `die()`/`exit()` in libs or workers.
* [ ] `Conf` delegates to `Main`; no monolith.
* [ ] All shell arguments wrapped in `escapeshellarg()`.
* [ ] All ORM lookups parameterized; all output escaped; no unauthenticated mutating routes.

Related reading: [The module configuration class](module-class.md) · [Data model](data-model.md) · [Rights and authentication](../cookbook/rights-and-auth/).
