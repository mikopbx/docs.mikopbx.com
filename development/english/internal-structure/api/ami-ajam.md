---
description: Interacting with Asterisk via AMI (TCP 5038) and AJAM (HTTP).
---

# AMI / AJAM

MikoPBX talks to Asterisk through two manager interfaces:

* **AMI** (Asterisk Manager Interface) — a line-oriented TCP protocol on port **5038**. This is the primary channel used by core workers and modules to send commands (`Originate`, `Hangup`, `Redirect`, …) and to subscribe to events (`Newchannel`, `Hangup`, `UserEvent`, …).
* **AJAM** (Asterisk JavaScript Asynchronous Manager) — the same manager actions, exposed over **HTTP** (default port **8088**, TLS **8089**) under the `/asterisk/` path prefix. Convenient for shell scripts, web clients, and ad-hoc testing with `curl`.

From PHP you almost never open the socket yourself. You ask the DI container for a ready, authenticated [`AsteriskManager`](ami-ajam.md#asteriskmanager-the-ami-client) instance. Raw protocol (telnet / `curl`) is useful for debugging and for one-off operations from the PBX shell.

{% hint style="info" %}
**Running example.** Throughout the developer docs we follow a fictional module **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`). Where this page shows a module calling AMI, imagine it lives in `BlackListMain`. For a working, real-world AMI consumer pattern see the cookbook page [Interact with AMI](../../cookbook/asterisk/interact-with-ami.md).
{% endhint %}

## AsteriskManager: the AMI client

The AMI client is `MikoPBX\Core\Asterisk\AsteriskManager`.

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
namespace MikoPBX\Core\Asterisk;

class AsteriskManager
{
    public function connect(
        ?string $server = null,
        ?string $username = null,
        ?string $secret = null,
        string $events = 'on'
    ): bool;

    public function disconnect(): void;
    public function loggedIn(): bool;
}
```
{% endcode %}

The constructor accepts an optional path to a config file (parsed with `parse_ini_file`) and an `$optconfig` array. When values are missing it falls back to defaults: `server=localhost`, `port=5038`, `username=phpagi`, `secret=phpagi`. `connect()` accepts a `server:port` string (e.g. `127.0.0.1:5038`), checks that the Asterisk process is actually listening, opens the socket with a 2-second timeout, and logs in. On a successful `Login` response `loggedIn()` returns `true`.

### Getting an instance from the DI container

Do **not** instantiate `AsteriskManager` directly in module code. Use the helper `MikoPBX\Core\System\Util::getAstManager()`, which returns a shared, already-connected instance from the DI container.

{% code title="Core/src/Core/System/Util.php" %}
```php
use MikoPBX\Core\System\Util;

// Command mode — send actions, do not receive events (non-blocking).
$am = Util::getAstManager('off');

// Listener mode — receive events.
$amEvents = Util::getAstManager('on');
```
{% endcode %}

`getAstManager()` maps the `$events` argument to one of two DI services:

| `$events` | DI service name | Provider class                                   | Events     |
| --------- | --------------- | ------------------------------------------------ | ---------- |
| `'on'`    | `amiListener`   | `MikoPBX\Common\Providers\AmiConnectionListener` | Subscribed |
| `'off'`   | `amiCommander`  | `MikoPBX\Common\Providers\AmiConnectionCommand`  | Suppressed |

Both providers register a **shared** service that connects to `127.0.0.1:{AMI_PORT}` and logs in as the internal `phpagi` user (the `null` username/secret fall through to the constructor defaults). `amiCommander` connects with `events='off'` so a command-only client is never blocked draining the event stream; `amiListener` connects with `events='on'` for code that needs to consume events.

{% code title="Core/src/Common/Providers/AmiConnectionCommand.php" %}
```php
public const string SERVICE_NAME = 'amiCommander';

public function register(DiInterface $di): void
{
    $di->setShared(self::SERVICE_NAME, function () {
        $port = PbxSettings::getValueByKey(PbxSettings::AMI_PORT);
        $am   = new AsteriskManager();
        $am->connect("127.0.0.1:{$port}", null, null, 'off');
        return $am;
    });
}
```
{% endcode %}

{% hint style="warning" %}
The shared instances log in as `phpagi`, which is restricted to **localhost** (`permit=127.0.0.1`, `deny=0.0.0.0/0`). They work only on the PBX itself. To reach AMI from another host you must create an [`AsteriskManagerUsers`](ami-ajam.md#ami-users-asteriskmanagerusers) record whose network filter permits the caller, then connect with that user's credentials.
{% endhint %}

## Sending commands

Two low-level primitives back every action:

* **`sendRequest(string $action, array $parameters = []): array`** — writes the action and blocks on `waitResponse()` until a `Response:` block arrives. Use for request/response actions.
* **`sendRequestTimeout(string $action, array $parameters = []): array`** — same, but transparently reconnects on a dead socket and returns `[]` on timeout. Used internally by the list/status helpers below. It auto-fills an `ActionID` of `"{$action}_" . getmypid()`.

The raw CLI bridge:

* **`Command(string $command, ?string $actionid = null): array`** — runs an Asterisk CLI command (the `Command` AMI action), e.g. `core show channels`, `pjsip show endpoints`.

```php
$am  = Util::getAstManager('off');
$res = $am->Command('core show version');
echo $res['data'] ?? '';
```

### Channel and call operations

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
public function Originate(
    ?string $channel,
    ?string $exten = null,
    ?string $context = null,
    ?int $priority = null,
    ?string $application = null,
    ?string $data = null,
    ?int $timeout = null,
    ?string $callerid = null,
    ?string $variable = null,
    ?string $account = null,
    bool $async = true,
    ?string $actionid = null
): array;

public function Hangup(string $channel): array;
public function Redirect(string $channel, string $extrachannel, string $exten, string $context, int $priority): array;
public function ExtensionState(string $exten, string $context, ?string $actionid = null): array;
public function Status(string $channel, ?string $actionid = null): array;
public function SetVar(string $channel, string $variable, string $value): array;
public function GetVar(string $channel, string $variable, ?string $actionId = null, bool $retArray = true): array|string;
public function AbsoluteTimeout(string $channel, int $timeout): array;
```
{% endcode %}

**`GetChannels(bool $group = true): array`** wraps the `CoreShowChannels` action. With `$group = true` (default) it returns channels grouped by `Linkedid`; with `false` it returns a flat list of channel names.

```php
$am       = Util::getAstManager('off');
$channels = $am->GetChannels(false);   // ['PJSIP/201-00000001', ...]
```

Originate a call from extension `201` to `203` (the BlackList module might do this to ring an administrator):

```php
$am = Util::getAstManager('off');
$am->Originate(
    channel:  'Local/201@internal-originate',
    context:  'all_peers',
    exten:    '203',
    priority: 1,
    callerid: '201',
    variable: 'pt1c_cid=203',
    async:    true
);
```

### PJSIP helpers

These call PJSIP manager actions and normalize the multi-event responses into plain arrays:

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
public function getPjSipPeers(): array;     // [['id'=>'201','state'=>'OK','detailed-state'=>'Not in use'], ...]
public function getPjSipPeer(string $peer, string $prefix = ''): array;
public function getPjSipPeerContacts(string $peer): array;  // all contacts incl. -WS / -TLS variants
public function getPjSipRegistry(): array;  // outbound registration status
```
{% endcode %}

```php
$am    = Util::getAstManager('off');
$peers = $am->getPjSipPeers();
foreach ($peers as $peer) {
    echo "{$peer['id']} => {$peer['state']}\n";
}
```

`getPjSipRegistry()` issues `PJSIPShowRegistrationsOutbound` and returns rows of `id`, `state`, `host`, `username` for each outbound provider registration.

### Queues

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
public function QueueAdd(string $queue, string $interface, int $penalty = 0): array;
public function QueueRemove(string $queue, string $interface): array;
public function QueueStatus(?string $actionid = null): array;
public function Queues(): array;
```
{% endcode %}

### Call recording

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
public function MixMonitor(string $channel, string $file, string $options, string $command = '', string $ActionID = ''): array;
public function StopMixMonitor(string $channel, string $ActionID = ''): array;
public function Monitor(string $channel, ?string $file = null, ?string $format = null, ?bool $mix = null): array;
public function StopMonitor(string $channel): array;
public function ChangeMonitor(string $channel, string $file): array;
```
{% endcode %}

`MixMonitor` records both legs of a call into a single mixed file and is the preferred recorder; `Monitor` records the legs separately. MikoPBX uses `MixMonitor` for standard call recording.

## Working with events

Register handlers, then enter a blocking loop that dispatches each incoming event to the matching handler.

{% code title="Core/src/Core/Asterisk/AsteriskManager.php" %}
```php
public function addEventHandler(string $event, array|string $callback): bool;
public function waitResponse(bool $allow_timeout = false): array;
public function waitUserEvent(bool $allow_timeout = false): array;
public function Events(string $eventMask): array;
public function UserEvent(string $name, array $headers): array;
public function processEvent(array $parameters): mixed;
```
{% endcode %}

* **`addEventHandler($event, $callback)`** registers a handler. `$event` is lower-cased internally, so `'Hangup'` and `'hangup'` are equivalent; `'*'` is a catch-all. `$callback` must be an **array callable** `[$object, 'method']` (or a plain function-name string) — see the warning below. It returns `false` if a handler for that event is already registered.
* **`Events(string $eventMask)`** turns the event stream on/off at runtime (`'on'`, `'off'`, or a mask such as `'system,call,log'`).
* **`UserEvent(string $name, array $headers)`** emits a custom `UserEvent` — the standard way for an AGI script or worker to push a message onto the AMI bus.
* **`waitUserEvent()`** is a blocking loop tuned for `UserEvent` consumers; it also supports an idle callback (`setOnIdleCallback()`).

{% hint style="warning" %}
The method is **`addEventHandler`** (camelCase). There is no `add_event_handler` alias.
{% endhint %}

{% hint style="danger" %}
**Do not pass a closure to `addEventHandler()`.** The parameter is typed `array|string $callback` (`Core/src/Core/Asterisk/AsteriskManager.php:1751`), so `addEventHandler('Hangup', fn($p) => …)` fails immediately with a `TypeError` — a `Closure` is not an `array` and not a `string`.

Use the array callable `[$object, 'method']`. The string form technically works, but it is dispatched differently: `processEvent()` invokes an **array** handler as `call_user_func($handler, $parameters)` — one argument — while a **string** handler is called as `$handler($eventName, $parameters, $server, $port)` — four arguments. So the array form is the only one whose signature matches the `function (array $parameters)` shape used throughout these docs and in the example modules.
{% endhint %}

A minimal event-listening loop, of the kind a long-running worker uses. Note that the handlers are methods on the worker object, registered as array callables:

```php
use MikoPBX\Core\System\Util;
use MikoPBX\Core\Asterisk\AsteriskManager;

class BlackListAmiListener
{
    protected AsteriskManager $am;

    public function run(): void
    {
        $this->am = Util::getAstManager('on');

        $this->am->addEventHandler('Newchannel', [$this, 'onNewChannel']);
        $this->am->addEventHandler('Hangup', [$this, 'onHangup']);

        // Block forever, dispatching events to the handlers above.
        while (true) {
            $this->am->waitUserEvent();
        }
    }

    public function onNewChannel(array $parameters): void
    {
        // $parameters['Channel'], $parameters['CallerIDNum'], ...
    }

    public function onHangup(array $parameters): void
    {
        // react to call teardown
    }
}
```

{% hint style="info" %}
A blocking event loop belongs in a background worker, not in a web request or a config generator. See the workers documentation in [Workers](../../module-developement/workers.md) for how to run one as a managed process, and [Debugging a PHP worker](../../module-developement/debuging/debug-php-worker.md) for attaching to it.
{% endhint %}

## manager.conf — how AMI is configured

The `manager.conf` file is generated by `MikoPBX\Core\Asterisk\Configs\ManagerConf`.

{% code title="Core/src/Core/Asterisk/Configs/ManagerConf.php" %}
```php
$amiPort = PbxSettings::getValueByKey(PbxSettings::AMI_PORT);

$conf = "[general]\n" .
    "enabled = yes\n" .
    "port = {$amiPort};\n" .
    "bindaddr = 0.0.0.0\n" .
    "displayconnects = no\n" .
    "allowmultiplelogin = yes\n" .
    "webenabled = yes\n" .          // enables AJAM over the HTTP server
    "timestampevents = yes\n" .
    'channelvars=' . implode(',', $vars) . "\n" .
    "httptimeout = 60\n\n";
```
{% endcode %}

Key facts to internalize:

* The `[general]` section (with `enabled = yes` and `port = {AMI_PORT}`) is written **unconditionally**. The AMI listener is always on.
* `PbxSettings::AMI_PORT` (key `AMIPort`, default `5038`) sets the TCP port.
* `PbxSettings::AMI_ENABLED` (key `AMIEnabled`, default `1`) gates **only** the block that writes the database-defined `AsteriskManagerUsers`. When it is `'1'`, each enabled manager user becomes a `manager.conf` section with its `secret`, `permit`/`deny` ACLs, and read/write class permissions. The internal `phpagi` user is **always** appended regardless of this setting.
* `webenabled = yes` is what allows the same manager actions to be reached over HTTP (AJAM).

### Module hook: generateManagerConf

A module can append its own `manager.conf` sections by implementing the `generateManagerConf` hook. `ManagerConf` collects every module's contribution via the hook constant `AsteriskConfigInterface::GENERATE_MANAGER_CONF`:

{% code title="Core/src/Core/Asterisk/Configs/ManagerConf.php" %}
```php
$conf .= $this->hookModulesMethod(AsteriskConfigInterface::GENERATE_MANAGER_CONF);
$this->saveConfig($conf, $this->description);
```
{% endcode %}

So `BlackListConf` could expose a dedicated manager user by returning an extra section:

{% code title="Modules/ModuleBlackList/Lib/BlackListConf.php" %}
```php
public function generateManagerConf(): string
{
    return "[blacklist_monitor]\n"
         . "secret=ChangeMe\n"
         . "deny=0.0.0.0/0.0.0.0\n"
         . "permit=127.0.0.1/255.255.255.255\n"
         . "read=call,cdr\n"
         . "write=call,originate\n"
         . "eventfilter=!Event: Newexten\n\n";
}
```
{% endcode %}

The `generateManagerConf` hook constant and the `generateConfig()` → `generateConfigProtected()` → `saveConfig()` flow are documented with the other hooks in [the module class reference](../../module-developement/module-class.md).

After regeneration, `ManagerConf::reload()` rewrites both `manager.conf` and `http.conf` and runs `module reload manager` / `module reload http` in Asterisk.

### AMI users: AsteriskManagerUsers

Database-defined AMI accounts are the model `MikoPBX\Common\Models\AsteriskManagerUsers`, backed by table **`m_AsteriskManagerUsers`**. Each row carries the `username`, `secret`, a `networkfilterid` (linking to a `NetworkFilters` row that provides the `permit`/`deny` ACLs), per-class permissions (`call`, `cdr`, `originate`, `reporting`, `agent`, `config`, `dialplan`, `dtmf`, `log`, `system`, `command`, `verbose`, `user`, each `read`/`write`/`readwrite`/none), and optional `eventfilter` lines. Setting `networkfilterid` to `localhost` forces a localhost-only ACL.

## http.conf — how AJAM is configured

AJAM rides on the Asterisk built-in HTTP server, configured by `MikoPBX\Core\Asterisk\Configs\HttpConf`.

{% code title="Core/src/Core/Asterisk/Configs/HttpConf.php" %}
```php
$ajamEnabled = intval(PbxSettings::getValueByKey(PbxSettings::AJAM_ENABLED)) === 1;
$ariEnabled  = intval(PbxSettings::getValueByKey(PbxSettings::ARI_ENABLED)) === 1;
$isEnable    = $ajamEnabled || $ariEnabled;

$port    = PbxSettings::getValueByKey(PbxSettings::AJAM_PORT);
$enabled = ($isEnable) ? 'yes' : 'no';
$conf    = "[general]".PHP_EOL .
    "enabled=$enabled".PHP_EOL .
    "bindaddr=0.0.0.0".PHP_EOL .
    "bindport={$port}".PHP_EOL .
    "prefix=asterisk".PHP_EOL .
    "enablestatic=yes".PHP_EOL.PHP_EOL;
```
{% endcode %}

* The HTTP server is enabled when **either** `AJAM_ENABLED` (key `AJAMEnabled`, default `1`) **or** `ARI_ENABLED` (key `ARIEnabled`, default `0`) is on.
* `PbxSettings::AJAM_PORT` (key `AJAMPort`, default `8088`) is the HTTP port; `PbxSettings::AJAM_PORT_TLS` (key `AJAMPortTLS`, default `8089`) is the HTTPS port. When a TLS port is set, `HttpConf` provisions the certificate via `SslCertificateService::prepareAsteriskCertificates('asterisk-http')` and writes `tlsenable`/`tlsbindaddr`/`tlscertfile`/`tlsprivatekey`.
* **`prefix=asterisk`** is what makes every AJAM URL start with `/asterisk/` — e.g. `/asterisk/rawman` and `/asterisk/mxml`. Keep this prefix in mind when crafting `curl` calls.

`HttpConf::reload()` regenerates `http.conf` and runs `module reload http`.

## Raw protocol examples

These run from the PBX shell (connect via SSH first). They use the localhost-only internal account; replace `${PBX_HOST}` with a remote address only if you have an `AsteriskManagerUsers` record permitting it.

### Originate

{% tabs %}
{% tab title="Telnet (AMI)" %}
Connect to the AMI socket:

```bash
busybox telnet 127.0.0.1 5038
```

Authenticate:

```
Action: Login
Username: phpagi
Secret: phpagi
Events: off

```

Originate a call from `201` to `203`:

```
Action: Originate
Channel: Local/201@internal-originate
Context: all_peers
Exten: 203
Priority: 1
Callerid: 201
Variable: SIPADDHEADER="Call-Info:\;answer-after=0",pt1c_cid=203,ALLOW_MULTY_ANSWER=1

```

* **SIPADDHEADER** — header added to the INVITE sent to extension `201`.
* **pt1c\_cid** — destination number.
* **ALLOW\_MULTY\_ANSWER** — if `201` has multiple registrations, `SIPADDHEADER` is sent to all contacts.
{% endtab %}

{% tab title="curl (AJAM)" %}
AJAM exposes the same actions over HTTP under the `/asterisk/` prefix:

```bash
h='127.0.0.1';            # ${PBX_HOST} — localhost when run on the PBX
u='phpagi';
p='phpagi';
port='8088';

from='201';
to='203';

# Login — capture the session cookie
tcookie="$(curl -I "http://${h}:${port}/asterisk/rawman?action=login&username=${u}&secret=${p}" \
  | grep -i 'Set-Cookie' | awk -F': ' '{ print $2 }')";

# Originate a call from ${from} to ${to}
curl --cookie "${tcookie}" \
  "http://${h}:${port}/asterisk/rawman?action=originate&channel=Local/${from}@internal-originate&exten=${to}&context=all_peers&priority=1&callerid=${from}";
```
{% endtab %}

{% tab title="Call file" %}
Build the call file:

```bash
cat > /tmp/originate.call <<'EOF'
Channel: Local/201@internal-originate
Context: all_peers
Extension: 203
Priority: 1
Callerid: 201
Archive: yes
Setvar: SIPADDHEADER=Call-Info:\;answer-after=0
Setvar: pt1c_cid=203
Setvar: ALLOW_MULTY_ANSWER=1
EOF
```

Move it into the Asterisk outgoing spool directory (Asterisk processes and removes it). The spool root is the resolved media mount point (`Directories::getDir(Directories::AST_SPOOL_DIR)`, i.e. `<mountpoint>/mikopbx/astspool`); on a typical USB-disk install the mount point is `/storage/usbdisk1`:

```bash
mv /tmp/originate.call /storage/usbdisk1/mikopbx/astspool/outgoing
```
{% endtab %}
{% endtabs %}

### Redirect (blind transfer)

Forwarding without consultation:

```
Action: Redirect
Channel: PJSIP/201-000001
Context: internal-transfer
Exten: 203
Priority: 1
```

* **PJSIP/201-000001** — channel to forward.
* **internal-transfer** — always use this context for redirects.
* **203** — destination number.
* The channel that was bridged with `PJSIP/201-000001` is hung up.

### Attended transfer

Call forwarding with consultation (raw `Atxfer` action — there is no `AsteriskManager` wrapper for it):

```
Action: Atxfer
Channel: PJSIP/201-000001
Context: internal-transfer
Exten: 203
Priority: 1
```

* **PJSIP/201-000001** — the channel that transfers the call; it is connected to `203` for consultation.
* When `PJSIP/201-000001` hangs up, the bridged channel is connected to `203`.

## See also

* [API reference overview](./)
* [Cookbook: Interact with AMI](../../cookbook/asterisk/interact-with-ami.md)
* [Module class & config hooks](../../module-developement/module-class.md)
* [Workers](../../module-developement/workers.md)
* [Debugging a PHP worker](../../module-developement/debuging/debug-php-worker.md)
