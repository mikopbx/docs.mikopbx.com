---
description: >-
  MikoPBX implementation of the PHP-AGI library and how to write AGI scripts
  that hook into the Asterisk dialplan from a module.
---

# AGI

AGI (Asterisk Gateway Interface) lets a PHP script take control of a live call. Asterisk launches the script from the dialplan, hands it the channel's stdin/stdout, and the script reads channel variables, plays prompts, sets caller-ID, redirects the call, and so on. MikoPBX ships a self-contained PHP-AGI implementation so that module code can do all of this in plain PHP, with full access to the MikoPBX ORM and DI container.

The implementation is two classes under `MikoPBX\Core\Asterisk`:

* `MikoPBX\Core\Asterisk\AGIBase` (`Core/src/Core/Asterisk/AGIBase.php`) — the transport layer. Its constructor opens `php://stdin`/`php://stdout`, parses the AGI request header into `$this->request`, and exposes the low-level `evaluate(string $command)` method that writes one AGI command and parses the response into `['code' => ..., 'result' => ..., 'data' => ...]`.
* `MikoPBX\Core\Asterisk\AGI` (`Core/src/Core/Asterisk/AGI.php`) — the public API. It extends `AGIBase` and adds the named command methods (`answer()`, `verbose()`, `get_variable()`, `set_variable()`, `exec()`, `stream_file()`, `getData()`, …).

{% hint style="danger" %}
**There is no `AGI\AgiClient`, no `getVariable()`, and no `setVariable()`.** Some recipe/skill text shows a camelCase API such as `(new AGI\AgiClient)->getVariable('X')` or `->setVariable('X', 'Y')`. **That class and those two methods do not exist anywhere in the MikoPBX source.** The real class is `MikoPBX\Core\Asterisk\AGI`, and the variable accessors are the snake\_case `get_variable(string $variable, bool $getvalue = false)` and `set_variable(string $variable, string $value)`. Using the camelCase names will fatal with "Call to undefined method".
{% endhint %}

{% hint style="info" %}
The class mixes naming styles by design — most AGI primitives are snake\_case (`get_variable`, `set_variable`, `exec_dial`, `stream_file`), while a few helper methods are camelCase (`getData`, `getCallerIdName`, `databasePut`). Always confirm a method name against `Core/src/Core/Asterisk/AGI.php` before using it.
{% endhint %}

## The verified API surface

Every method below is defined in `Core/src/Core/Asterisk/AGI.php`. Signatures are copied verbatim from source.

| Method                                                               | Signature                                                                                                                | Purpose                                                                                                                                          |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `verbose`                                                            | `verbose(string $message, int $level = 1): array`                                                                        | Write a message to the Asterisk console / verbose log.                                                                                           |
| `answer`                                                             | `answer(): array`                                                                                                        | Answer the channel if not already answered.                                                                                                      |
| `hangup`                                                             | `hangup(string $channel = ''): array`                                                                                    | Hang up the current (or named) channel.                                                                                                          |
| `noop`                                                               | `noop(string $string = ''): array`                                                                                       | No-op; useful for tracing.                                                                                                                       |
| `get_variable`                                                       | `get_variable(string $variable, bool $getvalue = false): array\|string`                                                  | Read a channel/dialplan variable. With `$getvalue = true` it returns the trimmed string value; otherwise the full `['result','data',...]` array. |
| `set_variable`                                                       | `set_variable(string $variable, string $value): array`                                                                   | Set a channel variable (`SET VARIABLE`).                                                                                                         |
| `set_var`                                                            | `set_var(string $pVariable, string\|int\|float $pValue): array`                                                          | Set a variable via the `Set()` dialplan application.                                                                                             |
| `exec`                                                               | `exec(string $application, mixed $options): array`                                                                       | Run any Asterisk dialplan application. `$options` may be a string or array (joined with commas).                                                 |
| `exec_dial`                                                          | `exec_dial(string $type, string $identifier, ?int $timeout = null, ?string $options = null, ?string $url = null): array` | Convenience wrapper over `exec('Dial', ...)`.                                                                                                    |
| `exec_goto`                                                          | `exec_goto(string $a, ?string $b = null, ?string $c = null): array`                                                      | Jump to `context,extension,priority`.                                                                                                            |
| `exec_absolutetimeout`                                               | `exec_absolutetimeout(int $seconds = 0): array`                                                                          | Set the absolute call timeout.                                                                                                                   |
| `stream_file`                                                        | `stream_file(string $filename, string $escape_digits = '', int $offset = 0): array`                                      | Play a sound file; stops on the first DTMF digit.                                                                                                |
| `getData`                                                            | `getData(string $filename, ?int $timeout = null, ?int $max_digits = null): array`                                        | Play a prompt and collect multiple DTMF digits (IVR input).                                                                                      |
| `wait_for_digit`                                                     | `wait_for_digit(int $timeout = -1): array`                                                                               | Wait up to `$timeout` ms for a single DTMF digit.                                                                                                |
| `set_callerid`                                                       | `set_callerid(string $cid): array`                                                                                       | Change the channel caller-ID string.                                                                                                             |
| `set_music`                                                          | `set_music(bool $enabled = true, string $class = ''): array`                                                             | Toggle music-on-hold.                                                                                                                            |
| `getCallerIdName`                                                    | `getCallerIdName(string $number): string`                                                                                | Read `CALLERID(name)`; returns `''` when it equals `$number`.                                                                                    |
| `database_get` / `databasePut` / `database_del` / `database_deltree` | see source                                                                                                               | AstDB family/key access.                                                                                                                         |
| `evaluate`                                                           | `evaluate(string $command): array`                                                                                       | Low-level: send a raw AGI command (inherited from `AGIBase`).                                                                                    |

The constructor also populates `$agi->request` — the AGI request header sent by Asterisk at script start. Commonly used keys (from `AGIBase`):

* `agi_callerid` — caller number
* `agi_extension` — dialed extension
* `agi_channel` — channel name (e.g. `PJSIP/2001-00000001`)
* `agi_uniqueid` — call unique id
* `agi_context`, `agi_priority`, `agi_dnid`, `agi_language`

## The running example: ModuleBlackList

Throughout this chapter we use a fictional module **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` backing table `m_BlackListNumbers`). The goal: when an external call arrives, an AGI script looks the caller number up in `m_BlackListNumbers` and, if it is blacklisted, hangs the call up.

A real, working module that follows exactly the same shape is **ModulePhoneBook** — read it end-to-end before building your own:

* Launcher script: `Extensions/ModulePhoneBook/agi-bin/agi_phone_book.php`
* AGI logic class: `Extensions/ModulePhoneBook/Lib/PhoneBookAgi.php`
* Dialplan hook: `Extensions/ModulePhoneBook/Lib/PhoneBookConf.php`

## How AGI integration works in a module

There are four moving parts. Get all four right and Asterisk will call your PHP on every matching leg of a call.

### 1. Ship the script under `agi-bin/`

Place the launcher script at `Extensions/ModuleBlackList/agi-bin/agi_black_list.php`. Keep it thin — a launcher that bootstraps and delegates to a class in `Lib/`. This mirrors `agi-bin/agi_phone_book.php`, whose entire body is:

{% code title="Extensions/ModulePhoneBook/agi-bin/agi_phone_book.php" %}
```php
#!/usr/bin/php
<?php

use Modules\ModulePhoneBook\Lib\PhoneBookAgi;

require_once 'Globals.php';

$type = $argv[1] ?? 'in';
PhoneBookAgi::setCallerId($type);
```
{% endcode %}

{% hint style="info" %}
`require_once 'Globals.php'` bootstraps the MikoPBX CLI runtime. `Globals.php` (`Core/src/Core/Config/Globals.php`) returns immediately unless `PHP_SAPI === 'cli'`, then creates a Phalcon `FactoryDefault\Cli` DI container, loads the class autoloader (`Common/Config/ClassLoader.php`), and calls `RegisterDIServices::init()`. After that line your namespaces autoload and DI services (database, config, logging) are available. It is resolved on the include path, so write the literal `require_once 'Globals.php';` — do not hardcode an absolute path.
{% endhint %}

### 2. Put the logic in a `Lib/` class

The launcher delegates to a class under `Lib/`. ModulePhoneBook's `PhoneBookAgi` extends `Phalcon\Di\Injectable` (giving it access to the DI container), instantiates the AGI client with `new AGI()`, reads the channel via `$agi->request`, and writes back with `$agi->set_variable(...)`. Note the snake\_case accessor:

{% code title="Extensions/ModulePhoneBook/Lib/PhoneBookAgi.php" %}
```php
namespace Modules\ModulePhoneBook\Lib;

use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;
use Modules\ModulePhoneBook\Models\PhoneBook;
use Phalcon\Di\Injectable;

class PhoneBookAgi extends Injectable
{
    public static function setCallerID(string $type): void
    {
        try {
            $agi = new AGI();

            // Channel request header, populated by the AGI constructor
            $number = ($type === 'in')
                ? $agi->request['agi_callerid']
                : $agi->request['agi_extension'];

            $result = PhoneBook::findFirstByNumber(PhoneBook::cleanPhoneNumber($number, true));

            if ($result !== null && !empty($result->call_id)) {
                $agi->set_variable('CALLERID(name)', $result->call_id);
            }
        } catch (\Throwable $e) {
            Util::sysLogMsg('PhoneBookAGI', $e->getMessage(), LOG_ERR);
        }
    }
}
```
{% endcode %}

### 3. Symlink happens automatically on install

You do **not** copy the script into Asterisk's directory yourself. When the module is installed/enabled, `MikoPBX\Core\Modules\PbxExtensionUtils::createAgiBinSymlinks(string $moduleUniqueID)` (`Core/src/Modules/PbxExtensionUtils.php`) globs `<moduleDir>/agi-bin/*.php`, symlinks each file into the Asterisk AGI directory, and makes them executable:

```php
// Core/src/Modules/PbxExtensionUtils.php (excerpt)
$agiBinDir       = Directories::getDir(Directories::AST_AGI_BIN_DIR); // /var/lib/asterisk/agi-bin
$moduleAgiBinDir = "$moduleDir/agi-bin";
foreach (glob("$moduleAgiBinDir/*.{php}", GLOB_BRACE) as $file) {
    Util::createUpdateSymlink($file, $agiBinDir . '/' . basename($file));
}
Processes::mwExec("$chmod +x $agiBinDir/*");
```

So `Extensions/ModuleBlackList/agi-bin/agi_black_list.php` becomes `/var/lib/asterisk/agi-bin/agi_black_list.php`. The AGI directory constant is `Directories::AST_AGI_BIN_DIR`, which maps to `/var/lib/asterisk/agi-bin` (`Core/src/Core/System/Directories.php`).

### 4. Emit the dialplan `AGI(...)` call from your config class

The script only runs when the dialplan executes an `AGI()` application. Your module's config class (extending `ConfigClass`) implements one of the dialplan-generation hooks and returns a context fragment. ModulePhoneBook does this in two places, both passing an argument (`in` / `out`) that ends up in `$argv[1]`:

{% code title="Extensions/ModulePhoneBook/Lib/PhoneBookConf.php" %}
```php
// Hook into incoming routes before the Dial application
public function generateIncomingRoutBeforeDial($rout_number): string
{
    return "same => n,AGI({$this->moduleDir}/agi-bin/agi_phone_book.php,in)" . PHP_EOL;
}

// Or inject a standalone context
public function extensionGenContexts(): string
{
    return '[phone-book-out]' . PHP_EOL .
        'exten => ' . ExtensionsConf::ALL_NUMBER_EXTENSION .
        ",1,AGI({$this->moduleDir}/agi-bin/agi_phone_book.php,out)\n\t" .
        'same => n,return' . PHP_EOL;
}
```
{% endcode %}

For ModuleBlackList, `BlackListConf::generateIncomingRoutBeforeDial()` would return `"same => n,AGI({$this->moduleDir}/agi-bin/agi_black_list.php,in)" . PHP_EOL;`. See the full set of dialplan hooks in the [cookbook recipe on hooking incoming calls](../../cookbook/asterisk/hook-on-incoming-call.md).

{% hint style="info" %}
`{$this->moduleDir}` is the module's own install directory, so you can reference the script either through the symlink (`agi_black_list.php`) or via the absolute module path as shown above. ModulePhoneBook uses the absolute `{$this->moduleDir}/agi-bin/...` form.
{% endhint %}

## A minimal ModuleBlackList AGI script

Putting it together — the launcher plus a `Lib` class that uses the ORM to decide whether to hang up:

{% code title="Extensions/ModuleBlackList/agi-bin/agi_black_list.php" %}
```php
#!/usr/bin/php
<?php

use Modules\ModuleBlackList\Lib\BlackListAgi;

require_once 'Globals.php';

BlackListAgi::checkIncoming();
```
{% endcode %}

{% code title="Extensions/ModuleBlackList/Lib/BlackListAgi.php" %}
```php
namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;
use Modules\ModuleBlackList\Models\BlackListNumbers;
use Phalcon\Di\Injectable;

class BlackListAgi extends Injectable
{
    public static function checkIncoming(): void
    {
        try {
            $agi    = new AGI();
            $caller = $agi->request['agi_callerid'] ?? '';

            // Parameterized lookup through the Phalcon ORM (never string-concat SQL)
            $blocked = BlackListNumbers::findFirst([
                'conditions' => 'number = :num:',
                'bind'       => ['num' => $caller],
            ]);

            if ($blocked !== null) {
                $agi->verbose("BlackList: blocking call from {$caller}", 2);
                $agi->exec('Playback', 'ss-noservice');
                $agi->hangup();
            }
        } catch (\Throwable $e) {
            Util::sysLogMsg('BlackListAGI', $e->getMessage(), LOG_ERR);
        }
    }
}
```
{% endcode %}

## Reading and writing channel variables

`get_variable()` is the single way to read both channel variables and dialplan functions (e.g. `DEVICE_STATE(...)`, `DIALPLAN_EXISTS(...)`). Pass `true` as the second argument to get the trimmed string value directly instead of the full result array. This example asks Asterisk whether an internal number exists and what its device state is:

```php
<?php

use MikoPBX\Core\Asterisk\AGI;

require_once 'Globals.php';

function getExtensionStatus(AGI $agi, string $number): array
{
    $state   = $agi->get_variable("DEVICE_STATE(PJSIP/$number)", true);
    $dExists = $agi->get_variable("DIALPLAN_EXISTS(internal,$number,1)", true);
    $agi->verbose("DEVICE_STATE: {$state} DIALPLAN_EXISTS: {$dExists}", 2);

    $stateTable = [
        'UNKNOWN'     => ['Status' => -1, 'StatusText' => 'Unknown'],
        'INVALID'     => ['Status' => -1, 'StatusText' => 'Unknown'],
        'NOT_INUSE'   => ['Status' => 0,  'StatusText' => 'Idle'],
        'INUSE'       => ['Status' => 1,  'StatusText' => 'In Use'],
        'BUSY'        => ['Status' => 2,  'StatusText' => 'Busy'],
        'UNAVAILABLE' => ['Status' => 4,  'StatusText' => 'Unavailable'],
        'RINGING'     => ['Status' => 8,  'StatusText' => 'Ringing'],
        'ONHOLD'      => ['Status' => 16, 'StatusText' => 'On Hold'],
    ];

    if ($state === 'INVALID' && $dExists === '1') {
        return $stateTable['NOT_INUSE'];
    }

    return $stateTable[$state] ?? $stateTable['UNKNOWN'];
}

$agi = new AGI();
$status = getExtensionStatus($agi, '2001');
```

{% hint style="warning" %}
`verbose()` and `answer()` are lowercase. Older snippets that wrote `$this->Verbose(...)` or `$agi->Answer()` were relying on PHP's historical case-insensitive method resolution — do not copy that style. Use the exact casing from `Core/src/Core/Asterisk/AGI.php`, and always call methods on the `$agi` instance (there is no global `$agi` magic).
{% endhint %}

## IVR: collecting DTMF input

`getData()` plays a prompt and collects multiple digits — the building block of an IVR menu. The result digits are in `['result']`:

```php
<?php

use MikoPBX\Core\Asterisk\AGI;

require_once 'Globals.php';

$agi = new AGI();
$agi->answer();

// Play <module-dir>/sounds/menu and collect up to 4 digits, 3000 ms timeout
$ivrMenu     = '/storage/usbdisk1/mikopbx/custom_modules/module-black-list/menu';
$result      = $agi->getData($ivrMenu, 3000, 4);
$selectedNum = $result['result'];

$agi->exec(
    'Dial',
    "Local/{$selectedNum}@internal/n,30,TtekKHhU(dial_answer)b(dial_create_chan,s,1)"
);
```

## Connecting a call

`exec_dial()` and `exec_goto()` are typed wrappers over `exec()`:

```php
$agi = new AGI();
$agi->answer();

// Send the call to extension 2001 in the 'internal' context, priority 1
$agi->exec_goto('internal', '2001', '1');

// Or dial a local channel for 30 seconds
$agi->exec_dial('Local', '2001@internal/n', 30, 'tT');
```

## Security

AGI scripts run with the privileges of the Asterisk process and act on attacker-influenced input (`agi_callerid`, `agi_dnid`, DTMF). Treat every value in `$agi->request` and every DTMF result as untrusted.

* **Never concatenate request data into SQL.** Use the Phalcon ORM with bound parameters (`'conditions' => 'number = :num:', 'bind' => ['num' => $caller]`) as shown above. This is the idiomatic MikoPBX data path — `PhoneBookAgi` queries entirely through model statics, never raw SQL.
* **Validate before you act.** Whitelist caller/extension formats (e.g. `preg_match('/^\+?\d{3,15}$/', $caller)`) before using a number to branch the call or build a query.
* **Escape anything that reaches a shell.** If you must call out to a system command, wrap every interpolated value in `escapeshellarg()`. Prefer not shelling out at all from an AGI context.
* **Validate file paths.** When `stream_file()`/`getData()` filenames are derived from input, restrict them to a known directory and basename — never let request data select an arbitrary path.
* **Fail safe.** Wrap the logic in `try/catch` and log via `Util::sysLogMsg(...)` (as `PhoneBookAgi` does) so an exception in your script does not drop the call silently.

## Debugging

AGI scripts run inside Asterisk, not from your shell, so a syntax error or fatal will show up as the call failing rather than as visible PHP output. See [Debugging PHP-AGI scripts](../../module-developement/debuging/debug-php-agi.md) for how to run a script standalone with a fake request header, tail the logs, and use `verbose()` for trace output.

## Related pages

* [API overview](./) — the module API surface.
* [Hook on incoming call](../../cookbook/asterisk/hook-on-incoming-call.md) — the dialplan hooks that emit your `AGI(...)` line.
* [Debugging PHP-AGI scripts](../../module-developement/debuging/debug-php-agi.md).
