---
description: >-
  Long-running background workers: Beanstalk and AMI patterns, registration and
  supervision via WorkerSafeScriptsCore.
---

# Workers and background processes

A **worker** is a long-running PHP process that does work outside the HTTP request cycle: listening for call events from Asterisk, draining a job queue, polling an external API, periodic housekeeping. MikoPBX ships a worker framework in `MikoPBX\Core\Workers` and a supervisor (`WorkerSafeScriptsCore`) that starts your workers, restarts them when they die, and pings them to make sure they are alive.

This page shows how a module declares its workers, the two main runtime patterns (Beanstalk queue and AMI event stream), and the rules you must follow so the supervisor can keep your process healthy.

Throughout we use the running example module **ModuleBlackList** (config class `BlackListConf`, worker `WorkerBlackListAMI`). For complete, working code, read the two example modules pinned below — every snippet on this page is distilled from them:

* Beanstalk queue worker: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/WorkerExampleFormMain.php`
* AMI event worker: `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php`

## The worker contract

Every worker extends `MikoPBX\Core\Workers\WorkerBase`, which is declared as:

{% code title="src/Core/Workers/WorkerBase.php" %}
```php
abstract class WorkerBase extends \Phalcon\Di\Injectable implements WorkerInterface
```
{% endcode %}

Because `WorkerBase` extends `Phalcon\Di\Injectable`, every worker has the dependency-injection container available through `$this->getDI()` and lazy property access (e.g. `$this->config`), exactly like a controller or a model.

`WorkerInterface` is the minimal contract. It declares only three methods you have to care about (plus the constructor):

{% code title="src/Core/Workers/WorkerInterface.php" %}
```php
interface WorkerInterface
{
    public function __construct();

    /** Keep-alive ping handler, called by the supervisor over Beanstalk. */
    public function pingCallBack(BeanstalkClient $message): void;

    /** Path of the PID file that proves this worker is running. */
    public function getPidFile(): string;

    /** Worker entry point — your main loop lives here. */
    public function start(array $argv): void;
}
```
{% endcode %}

`WorkerBase` already implements `pingCallBack()` and `getPidFile()` for you, so in practice the only method you **must** override is `start()`.

{% hint style="warning" %}
The signature is `public function start(array $argv): void`. Note the `array` type hint on `$argv` and the `void` return type — some shorthand snippets omit the `array` hint, but your override must match the interface exactly or PHP 8.4 will raise a fatal error. The argument carries the CLI arguments the supervisor used to launch the process.
{% endhint %}

### The main loop and `needRestart`

`WorkerBase` exposes a protected boolean `$needRestart` (default `false`) and a protected helper `setWorkerState(int $state)`. The canonical loop runs while a restart has **not** been requested:

```php
public function start(array $argv): void
{
    $this->setWorkerState(self::STATE_RUNNING);

    while ($this->needRestart === false) {
        pcntl_signal_dispatch();   // let queued signals run if not using async signals
        // ... do one unit of work, then block briefly ...
        sleep(1);
    }
    // falling out of the loop returns normally → clean shutdown
}
```

When the supervisor wants your worker to restart (for example after a graceful reconfigure), it sends `SIGUSR1`. `WorkerBase` handles that signal for you: the default `handleSignalUsr1()` simply sets `$this->needRestart = true`, so your loop condition becomes false and `start()` returns. The supervisor then spawns a fresh process. `SIGTERM`/`SIGINT` trigger an immediate, clean stop (the base class releases the Redis socket and calls `exit(0)`).

The worker-state constants on `WorkerBase` are `STATE_STARTING = 1`, `STATE_RUNNING = 2`, `STATE_STOPPING = 3`, and `STATE_RESTARTING = 4`. Call `setWorkerState(self::STATE_RUNNING)` once you have finished initialising and are about to enter the loop — it produces a structured syslog line and is what the supervisor's logs key off.

{% hint style="danger" %}
**Never call `die()` or `exit()` inside your own worker code.** A worker that exits on its own fights the supervisor: `WorkerSafeScriptsCore` immediately respawns it, and a tight crash loop follows. To stop or restart, set `$this->needRestart = true` and **return** from `start()` (or break out of the loop). Let `WorkerBase` own the process lifecycle and the `exit()` calls.
{% endhint %}

## Pattern 1 — Beanstalk queue worker

Use this when other parts of the system (or your own controllers) push messages onto a named queue and your worker processes them one at a time. Beanstalk is the standard MikoPBX inter-process job bus.

A Beanstalk worker subscribes to two tubes — its own work tube plus a **ping tube** the supervisor uses for keep-alive checks — and then blocks in `wait()` forever:

{% code title="Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/WorkerExampleFormMain.php" %}
```php
namespace Modules\ModuleExampleForm\Lib;

use MikoPBX\Common\Handlers\CriticalErrorsHandler;
use MikoPBX\Core\System\BeanstalkClient;
use MikoPBX\Core\Workers\WorkerBase;

require_once 'Globals.php';

class WorkerExampleFormMain extends WorkerBase
{
    protected ExampleFormMain $templateMain;

    public function start(array $argv): void
    {
        $this->templateMain = new ExampleFormMain();

        $client = new BeanstalkClient(self::class);

        // Keep-alive: subscribe to the ping tube and answer with pingCallBack().
        $client->subscribe($this->makePingTubeName(self::class), [$this, 'pingCallBack']);

        // Real work: subscribe to the worker's own tube.
        $client->subscribe(self::class, [$this, 'beanstalkCallback']);

        while (true) {
            $client->wait();
        }
    }

    public function beanstalkCallback(BeanstalkClient $message): void
    {
        $receivedMessage = json_decode($message->getBody(), true);
        $this->templateMain->processBeanstalkMessage($receivedMessage);
    }
}
```
{% endcode %}

Key APIs, all confirmed against `src/Core/System/BeanstalkClient.php`:

* `new BeanstalkClient(string $tube = 'default')` — opens a connection; passing `self::class` names the worker's tube after its own class.
* `subscribe(string $tube, array $callback): void` — registers a `[$object, 'method']` callback for a tube. Subscribe to **both** your work tube and `makePingTubeName(self::class)`.
* `wait(float $timeout = 5): void` — blocks for the next reserved job and dispatches it to the matching callback. Loop on it.
* Inside a callback, `BeanstalkClient::getBody()` returns the payload and `reply(string $response): void` sends a response back to a requester that used `request()`.

{% hint style="warning" %}
`getBody()` is declared `: string`, but what you get back depends on how the message was published. `publish()` runs the payload through `serialize()`, and `wait()` returns the raw string **only when it parses as JSON** — otherwise it hands the callback the `unserialize()`d value. So the `json_decode($message->getBody(), true)` idiom above works when the publisher sent a JSON string (as the example module does), and breaks if the publisher passed an array straight to `publish()`. Agree on one payload format across both ends of the tube.
{% endhint %}

`makePingTubeName(string $workerClassName): string` (on `WorkerBase`) derives the ping tube name from the class name. The supervisor's `CHECK_BY_BEANSTALK` health check sends a message to this tube; `WorkerBase::pingCallBack()` answers it with a `:pong` reply. You normally just wire `pingCallBack` to the ping tube and never touch it again.

{% hint style="info" %}
The supervisor expects Beanstalk workers to answer pings. If you forget the ping-tube subscription, the health check fails and the supervisor will keep killing and respawning your worker even though it is otherwise healthy.
{% endhint %}

## Pattern 2 — AMI event worker

Use this when your module reacts to Asterisk telephony events (new channel, dial begin/end, hangup, peer status, …). The worker holds an Asterisk Manager Interface connection and runs an event loop.

Connect through `Util::getAstManager()` rather than constructing `AsteriskManager` by hand — the helper resolves the right DI service and connects to the local Asterisk manager on `localhost:5038` using the credentials from the PBX settings (the AMI host/port defaults `localhost`/`5038` are set inside `AsteriskManager`). Then install an event filter, register a handler, and reconnect whenever the event read returns empty:

{% code title="Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/WorkerExampleAmiAMI.php" %}
```php
namespace Modules\ModuleExampleAmi\Lib;

use MikoPBX\Common\Providers\EventBusProvider;
use MikoPBX\Core\Asterisk\AsteriskManager;
use MikoPBX\Core\System\Util;
use MikoPBX\Core\Workers\WorkerBase;
use Phalcon\Di\Di;

require_once 'Globals.php';

class WorkerExampleAmiAMI extends WorkerBase
{
    protected AsteriskManager $am;

    public function start(array $argv): void
    {
        $this->am = Util::getAstManager();
        $this->setFilter();
        $this->am->addEventHandler("*", [$this, "callback"]);

        while (true) {
            $result = $this->am->waitUserEvent(true);
            if ($result === []) {
                // Connection dropped or idle timeout — reconnect and re-arm filters.
                usleep(100000);
                $this->am = Util::getAstManager();
                $this->setFilter();
            }
        }
    }

    private function setFilter(): void
    {
        // Let the supervisor's CHECK_BY_AMI ping through.
        $pingTube = $this->makePingTubeName(self::class);
        $params   = ['Operation' => 'Add', 'Filter' => 'UserEvent: ' . $pingTube];
        $this->am->sendRequestTimeout('Filter', $params);

        // Subscribe to the events this worker cares about.
        foreach (['Newchannel', 'DialBegin', 'DialEnd', 'Hangup'] as $event) {
            $this->am->sendRequestTimeout('Filter', ['Operation' => 'Add', 'Filter' => "Event: $event"]);
        }
    }

    public function callback(array $parameters): void
    {
        // ALWAYS answer the supervisor's ping first, then return.
        if ($this->replyOnPingRequest($parameters)) {
            return;
        }

        $di = Di::getDefault();
        $di->get(EventBusProvider::SERVICE_NAME)->publish('ami-events', [
            'event' => $parameters['Event'] ?? 'Unknown',
            'data'  => $parameters,
        ]);
    }
}
```
{% endcode %}

Key APIs, confirmed against `src/Core/Asterisk/AsteriskManager.php` and `src/Core/System/Util.php`:

* `Util::getAstManager(string $events = 'on'): AsteriskManager` — returns a connected manager from the DI container.
* `addEventHandler(string $event, array|string $callback): bool` — register a callback. `"*"` matches every event.
* `waitUserEvent(bool $allow_timeout = false): array` — block for the next event; returns an empty array `[]` on timeout/disconnect — your loop treats that as the signal to reconnect.

{% hint style="warning" %}
**`waitUserEvent()` does not return once per event.** Its body is a `do { … } while (!$timeout)` loop that reads the socket and dispatches each event to your registered handlers _itself_; it only returns to you when a ping fails — that is, when the connection is dead. So the code inside your `while (true)` loop runs **only on disconnect**, never per event.

The consequence bites when you put periodic work there — pruning a correlation map, flushing a buffer, refreshing a token. On a healthy PBX that code never executes, and the bug is invisible in testing precisely because everything is working. All per-event work belongs in the event handler; use `setOnIdleCallback()` for periodic work.
{% endhint %}

* `sendRequestTimeout(string $action, array $parameters = []): array` — used here to add AMI `Filter` directives. **Without explicit filters AMI only delivers login/ping responses**, so you must add a filter for each event class you want.
* `UserEvent(string $name, array $headers): array` — emits a custom UserEvent (used internally by the ping reply).

The keep-alive handshake on the AMI side is `WorkerBase::replyOnPingRequest(array $parameters): bool`. The supervisor's `CHECK_BY_AMI` check sends a `UserEvent` named after the ping tube; your event callback must call `replyOnPingRequest($parameters)` first and `return` if it handled the ping. `replyOnPingRequest()` fires back a `…Pong` UserEvent via `$this->am->UserEvent(...)`.

{% hint style="info" %}
For a deeper tour of AMI actions, events, and originate/redirect patterns, see the cookbook: [Interact with AMI](../cookbook/asterisk/interact-with-ami.md).
{% endhint %}

## Registering workers with the supervisor

Workers do not start themselves. Your module's config class advertises them through `getModuleWorkers()`. `WorkerSafeScriptsCore` calls this hook (via the `getModuleWorkers` hook on every module config class) and merges your workers into the global supervision list.

`getModuleWorkers()` is declared on `MikoPBX\Modules\Config\ConfigClass` as:

{% code title="src/Modules/Config/ConfigClass.php" %}
```php
public function getModuleWorkers(): array
{
    return [];
}
```
{% endcode %}

Override it to return an array of `['type' => …, 'worker' => …]` entries. For our ModuleBlackList example registering a single AMI worker:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;
use MikoPBX\Modules\Config\ConfigClass;
use Modules\ModuleBlackList\Lib\WorkerBlackListAMI;

class BlackListConf extends ConfigClass
{
    public function getModuleWorkers(): array
    {
        return [
            [
                'type'   => WorkerSafeScriptsCore::CHECK_BY_AMI,
                'worker' => WorkerBlackListAMI::class,
            ],
        ];
    }
}
```
{% endcode %}

See the real, working registrations in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php` (registers both a Beanstalk worker and an AMI worker) and `Extensions/EXAMPLES/AMI/ModuleExampleAmi/Lib/ExampleAmiConf.php`.

### The four check types

The `type` you pick tells `WorkerSafeScriptsCore` how to verify the worker is alive. The constants are defined in `src/Core/Workers/Cron/WorkerSafeScriptsCore.php`:

| Constant                                        | String value           | How the supervisor checks                                                   |
| ----------------------------------------------- | ---------------------- | --------------------------------------------------------------------------- |
| `WorkerSafeScriptsCore::CHECK_BY_BEANSTALK`     | `checkWorkerBeanstalk` | Sends a ping over the worker's Beanstalk ping tube and waits for the pong.  |
| `WorkerSafeScriptsCore::CHECK_BY_AMI`           | `checkWorkerAMI`       | Sends an AMI `UserEvent` ping and waits for the `…Pong` reply.              |
| `WorkerSafeScriptsCore::CHECK_BY_REDIS`         | `checkWorkerRedis`     | Reads the worker's Redis heartbeat key (used by `WorkerRedisBase` workers). |
| `WorkerSafeScriptsCore::CHECK_BY_PID_NOT_ALERT` | `checkPidNotAlert`     | Confirms the PID-file process exists; no ping, no alert noise.              |

Match the type to the pattern: a Beanstalk worker uses `CHECK_BY_BEANSTALK`, an AMI worker uses `CHECK_BY_AMI`, a `WorkerRedisBase` pool worker uses `CHECK_BY_REDIS`, and a plain periodic PID-based worker uses `CHECK_BY_PID_NOT_ALERT`.

### How supervision works

`WorkerSafeScriptsCore` is a singleton that runs its checks in parallel (PHP Fibers) and:

* Starts every registered worker that is not currently running.
* Restarts a worker that fails its check or whose PID is gone.
* Tracks crash loops. `WorkerBase::startWorker()` records each crash in a Redis counter (`module:crashes:{ModuleUniqueID}` for module workers, `core:crashes:{Class}` for core workers, both with a 30-minute TTL) via `WorkerBase::recordModuleCrash()` / `recordCoreWorkerCrash()`. When the supervisor sees a module's counter exceed `CRASH_LOOP_THRESHOLD = 100` in that window, it disables the offending module via `PbxExtensionUtils::forceDisableModule()` rather than respawning it forever (core workers are never disabled — the supervisor only suppresses their respawns until the TTL expires). This is exactly why the `die()`/`exit()` anti-pattern is dangerous — a worker that exits on its own keeps tripping the supervisor's restart path and can get your whole module disabled.

{% hint style="warning" %}
Crash counting only happens when the worker is launched through `WorkerBase::startWorker()`. The inline bootstrap shown below (the form both example modules use) catches startup failures with `CriticalErrorsHandler::handleExceptionWithSyslog()`, which logs to syslog/Sentry but does **not** increment the crash counter. So the auto-disable safety net is tied to the `startWorker()` launch path; if you adopt the inline bootstrap, a crashing worker is logged and respawned but not auto-disabled. Either way, never `exit()` from your own loop.
{% endhint %}

You never instantiate `WorkerSafeScriptsCore` yourself; you only feed it through `getModuleWorkers()`.

## The self-exec bootstrap

Each worker file is **both** a class definition and a runnable CLI script: the supervisor launches it as `php WorkerBlackListAMI.php start`. The bottom of the file is the bootstrap that turns the file into an executable entry point. Use the exact form from the example modules:

```php
// ... class WorkerBlackListAMI extends WorkerBase { ... } ...

// Start worker process
$workerClassname = WorkerBlackListAMI::class;
if (isset($argv) && count($argv) > 1) {
    cli_set_process_title($workerClassname);
    try {
        $worker = new $workerClassname();
        $worker->start($argv);
    } catch (\Throwable $e) {
        \MikoPBX\Common\Handlers\CriticalErrorsHandler::handleExceptionWithSyslog($e);
    }
}
```

Three things matter here:

1. **`require_once 'Globals.php';`** near the top of the file — this bootstraps autoloading and the DI container so the worker can resolve services. Both example workers do this.
2. **`cli_set_process_title($workerClassname)`** — names the OS process after the worker class so `ps`, `pkill -f WorkerBlackListAMI`, and the supervisor's PID matching all work.
3. **`CriticalErrorsHandler::handleExceptionWithSyslog($e)`** in a `catch (\Throwable)` — any fatal during startup is logged to syslog (and Sentry) instead of dying silently.

The `count($argv) > 1` guard ensures the worker only starts when the file is executed as a script **with at least one argument**, not when it is merely autoloaded as a class definition. Note that this inline form does not inspect _which_ argument was passed — any argument starts it. Only `WorkerBase::startWorker()` actually checks for the literal `start` (`$argv[1] === 'start'`); the supervisor passes `start` either way.

{% hint style="info" %}
`WorkerBase::startWorker(array $argv, bool $setProcName = true)` performs the same sequence (title, construct, `start()`, crash recording) for core workers. Module workers conventionally inline the bootstrap shown above, matching the example modules. Either form is correct as long as you never call `exit()` from inside your own loop.
{% endhint %}

## Debugging a worker

Because workers are long-running, hot-patching a `.php` file under your module does **not** affect an already-running worker — it keeps the old class definitions in memory. After changing worker code, force a respawn:

```bash
# WorkerSafeScriptsCore starts a fresh process within a few seconds.
pkill -TERM -f WorkerBlackListAMI
```

For interactive step-through debugging, attach to the worker process. See [Debug a PHP worker](debuging/debug-php-worker.md) for the full setup.

## See also

* [The module config class](module-class.md) — where `getModuleWorkers()` lives.
* [Hooks reference](hooks-reference.md) — the full list of config-class hooks the core calls.
* [Interact with AMI](../cookbook/asterisk/interact-with-ami.md) — AMI actions and events in depth.
* [Debug a PHP worker](debuging/debug-php-worker.md) — attaching a debugger to a running worker.
