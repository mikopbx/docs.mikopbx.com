---
description: Listen to live Asterisk events and send AMI commands from a module worker.
---

# Interact with AMI

The Asterisk Manager Interface (AMI) is a TCP protocol exposed by Asterisk on `127.0.0.1:5038`. It lets you do two things from a module:

* **Listen** to a live stream of events (`Newchannel`, `DialBegin`, `Hangup`, `UserEvent`, peer/extension status, …).
* **Send** actions (`Originate`, `Hangup`, `SetVar`, `Command`, …) and read the response.

MikoPBX never makes you open a raw socket. The Core gives you a fully wired `MikoPBX\Core\Asterisk\AsteriskManager` instance through `MikoPBX\Core\System\Util::getAstManager()`, already connected and logged in with the system AMI credentials.

This recipe threads the running example module **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`) and anchors every pattern on two real, working modules:

* `Extensions/EXAMPLES/AMI/ModuleExampleAmi/` — the canonical AMI example.
* `Extensions/ModuleCallTracking/` — a production module that listens for a single `UserEvent` and forwards it over HTTP.

{% hint style="info" %}
A long-lived AMI listener belongs in a **background worker**, not in a controller or a REST callback. See [Background workers](../../module-developement/workers.md) for the worker lifecycle, and [AMI / AJAM](../../internal-structure/api/ami-ajam.md) for the protocol reference.
{% endhint %}

## Two connection modes: listener vs commander

`Util::getAstManager()` resolves one of two shared DI services depending on the `$events` argument:

{% code title="Core/src/Core/System/Util.php" %}
```php
public static function getAstManager(string $events = 'on'): AsteriskManager
{
    if ($events === 'on') {
        $nameService = AmiConnectionListener::SERVICE_NAME; // 'amiListener'
    } else {
        $nameService = AmiConnectionCommand::SERVICE_NAME;  // 'amiCommander'
    }
    // returns the shared, already-connected manager from the DI container
}
```
{% endcode %}

| Call                                                  | DI service     | Use it for                                                                                    |
| ----------------------------------------------------- | -------------- | --------------------------------------------------------------------------------------------- |
| `Util::getAstManager()` / `Util::getAstManager('on')` | `amiListener`  | A worker that subscribes to the event stream.                                                 |
| `Util::getAstManager('off')`                          | `amiCommander` | One-shot actions (`Originate`, `Hangup`, `Command`) where you do not want the event firehose. |

The `amiCommander` provider connects to `127.0.0.1:<AMI port>` (the port comes from `PbxSettings::AMI_PORT`, default `5038`) with events turned off:

{% code title="Core/src/Common/Providers/AmiConnectionCommand.php" %}
```php
$port = PbxSettings::getValueByKey(PbxSettings::AMI_PORT);
$am   = new AsteriskManager();
$am->connect("127.0.0.1:{$port}", null, null, 'off');
```
{% endcode %}

{% hint style="info" %}
Both services are registered as **shared** in the DI container, so repeated `getAstManager()` calls hand back the same socket. You do not create or close the connection yourself.
{% endhint %}

## Step 1 — Write the AMI listener worker

An AMI listener is a `WorkerBase` subclass whose `start()` method:

1. obtains the listener connection via `Util::getAstManager()`;
2. installs event **filters** (`setFilter()`) — without them AMI sends almost nothing;
3. registers a callback with `addEventHandler()`;
4. blocks in a `while (true)` loop on `waitUserEvent(true)`, reconnecting when the loop returns an empty array.

Here is the **ModuleBlackList** worker, modelled directly on the real example.

{% code title="Extensions/ModuleBlackList/Lib/WorkerBlackListAMI.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Common\Handlers\CriticalErrorsHandler;
use MikoPBX\Core\Asterisk\AsteriskManager;
use MikoPBX\Core\System\Util;
use MikoPBX\Core\Workers\WorkerBase;

require_once 'Globals.php';

/**
 * Background worker that listens to AMI events and dispatches them
 * to the module's main logic.
 */
class WorkerBlackListAMI extends WorkerBase
{
    /** AMI listener connection. */
    protected AsteriskManager $am;

    /**
     * Worker entry point.
     *
     * Flow: connect → set filters → register handler → loop with reconnect.
     *
     * @param array<int, string> $argv
     */
    public function start(array $argv): void
    {
        $this->am = Util::getAstManager();
        $this->setFilter();
        $this->am->addEventHandler('userevent', [$this, 'callback']);

        while (true) {
            $result = $this->am->waitUserEvent(true);
            if ($result === []) {
                // Empty result => the socket timed out / dropped. Reconnect.
                usleep(100000);
                $this->am = Util::getAstManager();
                $this->setFilter();
            }
        }
    }

    /**
     * Subscribe to the events this worker cares about.
     *
     * The first filter is mandatory: it lets WorkerSafeScriptsCore ping the
     * worker (CHECK_BY_AMI) to confirm it is alive.
     */
    private function setFilter(): void
    {
        // Mandatory: liveness ping channel for CHECK_BY_AMI monitoring.
        $pingTube = $this->makePingTubeName(self::class);
        $params   = ['Operation' => 'Add', 'Filter' => 'UserEvent: ' . $pingTube];
        $this->am->sendRequestTimeout('Filter', $params);

        // Our own application event.
        $params = ['Operation' => 'Add', 'Filter' => 'UserEvent: BlackListHit'];
        $this->am->sendRequestTimeout('Filter', $params);
    }

    /**
     * Event handler. Receives every event that passed the filters.
     *
     * @param array<string, mixed> $parameters
     */
    public function callback(array $parameters): void
    {
        // Always answer the liveness ping first and return.
        if ($this->replyOnPingRequest($parameters)) {
            return;
        }

        if (($parameters['UserEvent'] ?? '') !== 'BlackListHit') {
            return;
        }

        // Dispatch the real work to the module's main class.
        BlackListMain::handleBlackListHit($parameters);
    }
}

// Boilerplate that launches the worker process.
$workerClassname = WorkerBlackListAMI::class;
if (isset($argv) && count($argv) > 1) {
    cli_set_process_title($workerClassname);
    try {
        $worker = new $workerClassname();
        $worker->start($argv);
    } catch (\Throwable $e) {
        CriticalErrorsHandler::handleExceptionWithSyslog($e);
    }
}
```
{% endcode %}

See the working originals: `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php` (broadcasts events to the web UI over the EventBus) and `Extensions/ModuleCallTracking/Lib/WorkerCallTrackingAMI.php` (forwards a single `UserEvent` over HTTP).

### Why each piece is there

* **`require_once 'Globals.php';`** — bootstraps the module autoloader/DI so the worker can run as a standalone CLI process.
* **`setFilter()` is not optional.** A freshly logged-in AMI connection with an event filter installed receives only what you explicitly add. The Core's own example puts it bluntly: _"Without explicit filters, AMI only sends login/ping responses."_ Add `UserEvent: <name>` filters for user events and `Event: <Name>` filters for native Asterisk events.
* **`makePingTubeName()` + `replyOnPingRequest()`** come from `WorkerBase`. The monitor (`WorkerSafeScriptsCore`) pings the worker by sending a `UserEvent` whose name equals `makePingTubeName(static::class)`; `replyOnPingRequest()` detects it and answers with a `…Pong` `UserEvent`. If you skip the ping filter, the monitor concludes the worker is dead and restarts it in a loop.
* **`waitUserEvent(true)`** blocks reading the socket, runs each event through the registered handler, and returns `[]` on timeout — your cue to reconnect. It loops internally (`do { … } while (!$timeout)`) and **returns only when a ping fails**, so it does _not_ hand control back once per event. The body of your `while (true)` loop therefore executes only on disconnect. Put per-event work in the handler and periodic work in `setOnIdleCallback()`; anything you place in the outer loop expecting it to run regularly will never run at all on a healthy PBX — and will look fine in testing for exactly that reason.

{% hint style="warning" %}
**Three constraints on `addEventHandler(string $event, array|string $callback)` (`Core/src/Core/Asterisk/AsteriskManager.php:1751`).**

1. It registers a handler **only once per event name** — the name is lowercased, and if one is already registered the method returns `false` and does nothing. Register handlers in `start()` before the loop, not inside it.
2. The signature is `array|string` — **a closure is not accepted.** Pass an array callable `[$this, 'callback']` (the form used above) or a plain function-name string. Anything else is silently stored and then fails at dispatch time.
3. The two forms are **not** invoked the same way. An array callable receives a single argument — the parsed event parameters array. A string callable receives four arguments (event name, parameters, server, port). Use the array form unless you have a reason not to; then your handler signature is `function callback(array $parameters): void`.

Use `'userevent'` for user events or `'*'` to catch every event type (as `ModuleExampleAmi` does).
{% endhint %}

### Subscribing to native Asterisk events

To watch call progress instead of (or in addition to) custom user events, add `Event:` filters and handle `'*'`:

```php
private function setFilter(): void
{
    $pingTube = $this->makePingTubeName(self::class);
    $this->am->sendRequestTimeout('Filter', [
        'Operation' => 'Add',
        'Filter'    => 'UserEvent: ' . $pingTube,
    ]);

    foreach (['Newchannel', 'DialBegin', 'DialEnd', 'Hangup'] as $event) {
        $this->am->sendRequestTimeout('Filter', [
            'Operation' => 'Add',
            'Filter'    => "Event: $event",
        ]);
    }
}
```

`Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php` filters exactly this set (`Newchannel`, `Newstate`, `DialBegin`, `DialEnd`, `Hangup`, `PeerStatus`, `ExtensionStatus`, `Hold`, `Unhold`, `Bridge`) and registers its handler against `'*'` so every one of them reaches the callback.

## Step 2 — Register the worker for monitoring

Workers do not start themselves. Declare them from your config class via `getModuleWorkers()` so the Core supervisor (`WorkerSafeScriptsCore`) launches and watches them. For an AMI listener use the `CHECK_BY_AMI` strategy — that is the ping mechanism the worker already answers in `callback()`.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;

/**
 * @return array<int, array<string, string>>
 */
public function getModuleWorkers(): array
{
    return [
        [
            'type'   => WorkerSafeScriptsCore::CHECK_BY_AMI,
            'worker' => WorkerBlackListAMI::class,
        ],
    ];
}
```
{% endcode %}

`WorkerSafeScriptsCore::CHECK_BY_AMI` is the literal string `'checkWorkerAMI'` (defined in `Core/src/Core/Workers/Cron/WorkerSafeScriptsCore.php`). The other strategy, `CHECK_BY_BEANSTALK` (`'checkWorkerBeanstalk'`), is for queue workers, not AMI listeners.

Confirm against `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`, which registers `WorkerExampleAmiAMI` with exactly this shape.

{% hint style="info" %}
`CHECK_BY_AMI` is what wires the liveness ping to the `makePingTubeName()` / `replyOnPingRequest()` pair in your worker. The two go together: pick `CHECK_BY_AMI`, add the ping filter, answer the ping.
{% endhint %}

## Step 3 — Send AMI commands

To act on the PBX, grab a manager connection and call the typed methods on `AsteriskManager`. All of these are real methods on `Core/src/Core/Asterisk/AsteriskManager.php`.

### Originate a call

{% code title="Extensions/ModuleBlackList/Lib/BlackListMain.php" %}
```php
use MikoPBX\Core\System\Util;

// 'off' => the command-mode (amiCommander) connection, no event stream.
$am = Util::getAstManager('off');

$am->Originate(
    channel:     'Local/204@internal-originate', // who to call first
    exten:       '74950000000',                  // then dial this
    context:     'all_peers',
    priority:    1,
    timeout:     30000,                           // ms
    callerid:    'BlackList <204>',
    variable:    'pt1c_cid=74950000000',
    async:       true
);
```
{% endcode %}

`Originate()` accepts named parameters (`$channel, $exten, $context, $priority, $application, $data, $timeout, $callerid, $variable, $account, $async, $actionid`). Empty parameters are stripped before the request is sent, and `Async` is normalized to `'true'`/`'false'` for you.

### Hang up a channel

```php
$am = Util::getAstManager('off');
$am->Hangup('PJSIP/204-00000017');
```

### Run a CLI command or a raw action

`AsteriskManager` also exposes `Command()` (wraps the AMI `Command` action) and `sendRequestTimeout($action, $params)` for any action that has no typed wrapper:

```php
$am = Util::getAstManager('off');

// Native action with no parameters:
$response = $am->sendRequestTimeout('CoreStatus');

// Set a channel variable:
$am->SetVar('PJSIP/204-00000017', 'MY_FLAG', '1');
```

{% hint style="warning" %}
In Asterisk 20 the AMI `Command` action does not reliably return CLI output. `ExampleAmiConf::executeCliCommand()` works around this by shelling out with `Processes::mwExec("/usr/sbin/asterisk -rx " . escapeshellarg($command), …)` instead. Use a typed action (`CoreStatus`, `Originate`, …) when you need the response array; fall back to `asterisk -rx` only for CLI text output.
{% endhint %}

A complete request router — accepting either a CLI string or a multi-line `Action: …` block over REST — lives in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php` (`sendAmiCommandAction()`, `sendNativeAmiAction()`).

## Step 4 — Provision a dedicated AMI user (optional)

The shared `Util::getAstManager()` connection uses the system AMI account, which is enough for most modules. If you need your **own** AMI user (for example, a third-party app that connects directly), override `generateManagerConf()` on your config class. The Core appends its return value to `manager.conf`.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use MikoPBX\Core\Asterisk\Configs\ManagerConf;

/**
 * Append a [section] to Asterisk's manager.conf.
 */
public function generateManagerConf(): string
{
    $settings = BlackListNumbers::findFirst();
    if ($settings === null) {
        return '';
    }

    $conf  = "[{$settings->ami_user}]" . PHP_EOL;
    $conf .= "secret={$settings->ami_password}" . PHP_EOL;
    // Localhost-only is the safe default; widen deliberately if you must.
    $conf .= 'deny=0.0.0.0/0.0.0.0' . PHP_EOL;
    $conf .= 'permit=127.0.0.1/255.255.255.255' . PHP_EOL;
    $conf .= 'read=all' . PHP_EOL;
    $conf .= 'write=all' . PHP_EOL;
    $conf .= PHP_EOL;

    return $conf;
}

/**
 * Reload Asterisk manager so the new user takes effect, then start workers.
 */
public function onAfterModuleEnable(): void
{
    ManagerConf::reload();

    $main = new BlackListMain();
    $main->startAllServices();
}

/**
 * Reload manager on disable so the AMI user is removed.
 */
public function onAfterModuleDisable(): void
{
    ManagerConf::reload();
}
```
{% endcode %}

`generateManagerConf()` is a real hook declared in `Core/src/Core/Asterisk/Configs/AsteriskConfigInterface.php` and overridden here. `ManagerConf::reload()` (`Core/src/Core/Asterisk/Configs/ManagerConf.php:258-270`) regenerates `manager.conf` — which is what invokes your `generateManagerConf()` hook again — plus `http.conf`, then issues `module reload manager` and `module reload http`. `ExampleAmiConf::generateManagerConf()` (`Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php:269`) is the working version of the hook itself.

{% hint style="warning" %}
Older module code — including `ExampleAmiConf::onAfterModuleEnable()` at line 292 of that same file — reloads the manager with `System::invokeActions(['manager' => 0])`. **Do not copy that part.** The method is marked `@deprecated` in `Core/src/Core/System/System.php:61-69` (the docblock points at `WorkerModelsEvents::invokeAction()`). Call the specific config class's `reload()` instead — `ManagerConf::reload()` here, `ExtensionsConf::reload()` for the dialplan. The `PBX` class is deprecated for the same reason.
{% endhint %}

{% hint style="danger" %}
`read=all` / `write=all` grants full AMI privileges. Keep `deny=0.0.0.0/0.0.0.0` plus an explicit `permit=127.0.0.1/...` so the account is reachable only from localhost, and narrow the `read`/`write` classes to what your module actually needs.
{% endhint %}

## Reference: methods used in this recipe

All confirmed against the pinned Core sources.

| Symbol                                                                      | Where                                                | Purpose                                                                   |
| --------------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------- |
| `Util::getAstManager(string $events = 'on')`                                | `Core/src/Core/System/Util.php`                      | Get the shared listener (`'on'`) or commander (`'off'`) connection.       |
| `AsteriskManager::sendRequestTimeout(string $action, array $params = [])`   | `AsteriskManager.php`                                | Send any AMI action and read the response array.                          |
| `AsteriskManager::waitUserEvent(bool $allow_timeout = false)`               | `AsteriskManager.php`                                | Block on the event stream; returns `[]` on timeout.                       |
| `AsteriskManager::addEventHandler(string $event, array\|string $callback)`  | `AsteriskManager.php`                                | Register a per-event handler (once per name).                             |
| `AsteriskManager::Originate(...)`                                           | `AsteriskManager.php`                                | Originate a call (named args).                                            |
| `AsteriskManager::Hangup(string $channel)`                                  | `AsteriskManager.php`                                | Hang up a channel.                                                        |
| `AsteriskManager::SetVar(string $channel, string $variable, string $value)` | `AsteriskManager.php`                                | Set a channel variable.                                                   |
| `AsteriskManager::Command(string $command, ?string $actionid = null)`       | `AsteriskManager.php`                                | Run a CLI command via the `Command` action.                               |
| `WorkerBase::makePingTubeName(string $class)`                               | `Core/src/Core/Workers/WorkerBase.php`               | Compute this worker's liveness ping name.                                 |
| `WorkerBase::replyOnPingRequest(array $parameters)`                         | `WorkerBase.php`                                     | Answer a monitor ping; returns `true` if handled.                         |
| `WorkerSafeScriptsCore::CHECK_BY_AMI`                                       | `WorkerSafeScriptsCore.php`                          | Monitoring strategy string `'checkWorkerAMI'`.                            |
| `ManagerConf::reload()`                                                     | `Core/src/Core/Asterisk/Configs/ManagerConf.php:258` | Regenerate `manager.conf` + `http.conf` and reload both Asterisk modules. |
| `PbxSettings::AMI_PORT`                                                     | `Core/src/Common/Models/PbxSettings.php`             | AMI TCP port setting (default `5038`).                                    |

## See also

* [Asterisk integration overview](./)
* [Background workers](../../module-developement/workers.md)
* [AMI / AJAM](../../internal-structure/api/ami-ajam.md)
