---
description: >-
  Overview of MikoPBX APIs and integration surfaces — REST API v3, AMI/AJAM,
  AGI, and the legacy module REST callback — with guidance on when to use each.
---

# API

MikoPBX exposes four distinct integration surfaces. They are not interchangeable: each one lives at a different layer of the stack, speaks a different protocol, and is invoked from a different place. Picking the wrong surface is the single most common mistake in module development — for example, polling the REST API in a loop to watch call state when you should be subscribing to AMI events.

This page is a decision guide. Read it first, then follow the link to the child page for the surface you need.

{% hint style="info" %}
Throughout the developer docs we thread a single fictional module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`, frontend `module-black-list-index.js`), so each pattern has a concrete home. For every pattern we also point to a **real, working example module** under `Extensions/EXAMPLES/`.
{% endhint %}

## The four surfaces at a glance

| Surface                             | Protocol / transport          | Lives in                                     | Use it for                                                 |
| ----------------------------------- | ----------------------------- | -------------------------------------------- | ---------------------------------------------------------- |
| [REST API v3](rest-api.md)          | HTTPS, JSON, attribute-routed | `Core/src/PBXCoreREST/`                      | Management & CRUD, external integrations, module endpoints |
| [AMI / AJAM](ami-ajam.md)           | TCP 5038 (AMI) / HTTP (AJAM)  | `Core/src/Core/Asterisk/AsteriskManager.php` | Real-time Asterisk events and live commands in workers     |
| [AGI](agi.md)                       | Dialplan-invoked PHP scripts  | `Core/src/Core/Asterisk/AGI.php`, `agi-bin/` | Per-call, in-dialplan decision logic                       |
| Module REST callback (legacy v1/v2) | HTTPS, JSON, route-table      | module `*Conf.php`                           | Older modules; simple custom endpoints                     |

## REST API v3 — management and CRUD

The modern, recommended surface for anything that reads or writes configuration, manages resources, or is consumed by an external system (a CRM, a provisioning script, a mobile app, another server).

* **Routing** is auto-discovered from PHP 8.4 attributes (`#[ApiResource]`, `#[HttpMapping]`, `#[ResourceSecurity]`) by `RouterProvider`. You declare a resource class with attributes; no manual route table.
* **Authentication** is JWT Bearer tokens (HMAC-SHA256, 15-min access + 30-day refresh) or 64-char API keys, with `Resource:Action` RBAC. Requests from `127.0.0.1`/`::1` bypass authentication.
*   **URL shape** is conventional REST:

    ```
    GET    /pbxcore/api/v3/{resource}           → getList
    GET    /pbxcore/api/v3/{resource}/{id}      → getRecord
    POST   /pbxcore/api/v3/{resource}           → create
    PUT    /pbxcore/api/v3/{resource}/{id}      → update
    PATCH  /pbxcore/api/v3/{resource}/{id}      → patch
    DELETE /pbxcore/api/v3/{resource}/{id}      → delete
    GET    /pbxcore/api/v3/{resource}:custom    → custom method
    ```
*   **Processing is asynchronous**: the php-fpm front end queues the request on Redis (`api:requests`) and `WorkerApiCommands` (3 parallel processes) executes the action and writes the result back. Every action returns a `PBXApiResult`:

    ```php
    class PBXApiResult {
        public bool $success = false;
        public array $data;             // initialised to [] in the constructor
        public array $messages;         // ['error' => [...], 'warning' => [...]]
        public string $processor;       // set by the Processor that ran
        public string $function;        // the action name
        public ?int $httpCode = null;   // 200, 201, 400, 422, 409, 500
        public ?array $pagination = null;
        public string $reload = '';
    }
    ```

    `getResult()` also injects `pid` (the worker PID) into the wire body.

{% hint style="success" %}
**Choose REST v3 when** you are building a new module endpoint, exposing CRUD on a model, or integrating with anything outside the PBX. For ModuleBlackList you would add a v3 resource to manage rows in `m_BlackListNumbers` (add/remove/list blocked numbers from a CRM).

See the working example in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/` (controllers under `App/Controllers/`, action logic under `Lib/RestAPI/Tasks/`). The module-authoring walkthrough is on the [REST API in modules](../../module-developement/rest-api-in-modules.md) page.
{% endhint %}

→ Full reference: [REST API v3](rest-api.md)

## AMI / AJAM — real-time Asterisk events and commands

The Asterisk Manager Interface (AMI) is a long-lived TCP socket to Asterisk. It is how you **watch the PBX live** — channel state, dial events, queue membership, hangups — and how you send **immediate** control commands (originate a call, hang up a channel, redirect, add/remove a queue member).

* **Transport.** AMI listens on TCP — port `5038` by default, configured via `PbxSettings::AMI_PORT` (see `Core/src/Core/Asterisk/Configs/ManagerConf.php`, which generates `manager.conf`). The HTTP-flavoured variant **AJAM** is enabled by `webenabled = yes` in the same `[general]` block and is served by Asterisk's built-in HTTP server (`Core/src/Core/Asterisk/Configs/HttpConf.php`, `bindport` = `PbxSettings::AJAM_PORT`, URL prefix `asterisk`).
*   **Client.** Core wraps AMI in `MikoPBX\Core\Asterisk\AsteriskManager`. Key methods:

    ```php
    $am = new AsteriskManager();
    $am->connect($server, $username, $secret, $events);

    // Live command/response
    $am->Originate(/* ... */);
    $am->Hangup($channel);
    $am->QueueAdd($queue, $interface, $penalty);
    $peers = $am->getPjSipPeers();   // [['id'=>'201','state'=>'OK', ...]]

    // Event-driven — the callback must be an array callable [$object, 'method'],
    // never a closure: addEventHandler() is typed array|string.
    $am->addEventHandler('Hangup', [$this, 'onHangup']);
    $am->waitResponse(true);
    ```

{% hint style="warning" %}
AMI is for code that runs **inside the PBX** — typically a long-running module worker that holds the connection open and reacts to events. Do **not** poll the REST API in a loop to discover call state; subscribe to AMI events instead.
{% endhint %}

{% hint style="success" %}
**Choose AMI/AJAM when** your module needs to react to live call activity. ModuleBlackList could run a worker that listens for incoming-call events and reacts in real time.

See the working example in `Extensions/EXAMPLES/AMI/ModuleExampleAmi/` — note the dedicated worker `Lib/WorkerExampleAmiAMI.php` that owns the AMI connection.
{% endhint %}

→ Full reference: [AMI / AJAM](ami-ajam.md)

## AGI — in-call decision logic

The Asterisk Gateway Interface runs a script **from the dialplan, during a call**. Asterisk hands the script the channel and call variables; the script makes a decision and tells Asterisk what to do next (continue, set a variable, play a file, hang up). This is the right place for per-call logic that must execute _inside_ the call flow.

*   Core's AGI client is `MikoPBX\Core\Asterisk\AGI` (built on `AGIBase`). It exposes the standard AGI verbs as PHP methods:

    ```php
    $agi = new AGI();
    $cid = $agi->get_variable('CALLERID(num)', true);
    $agi->set_variable('__BLACKLISTED', '1');
    $agi->verbose("BlackList check for {$cid}", 1);
    $agi->hangup();   // reject the call
    ```
* Real Core AGI scripts live in `Core/src/Core/Asterisk/agi-bin/` (`check_redirect.php`, `meetme_dial.php`, `unpark_call.php`, `get_park_info.php`) — these are the canonical examples of how a dialplan-invoked script is structured. A module ships its own scripts in its `agi-bin/` directory and wires them into the dialplan from its config class.

{% hint style="success" %}
**Choose AGI when** you need a decision _within_ a call before it is routed. ModuleBlackList’s natural fit: an AGI script invoked early in the incoming context that looks up the caller in `m_BlackListNumbers` and hangs up if matched. A module injects this into the dialplan via a hook like `extensionGenInternal()` on its `BlackListConf` (see the dialplan-generation hooks in `Core/src/Core/Asterisk/Configs/AsteriskConfigInterface.php`).
{% endhint %}

→ Full reference: [AGI](agi.md)

## Legacy module REST callback (v1 / v2)

Before REST API v3, modules exposed HTTP endpoints under `/pbxcore/api/...` in one of two ways, both declared in the module's config class. These still work and you will encounter them in existing modules, but **new modules should use REST API v3**.

*   **v1 — single callback.** The config class implements one method that dispatches on the action name:

    ```php
    // Routes: GET|POST /pbxcore/api/modules/{ModuleName}/{actionName}
    public function moduleRestAPICallback(array $request): PBXApiResult
    ```

    See `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv1/Lib/ExampleRestAPIv1Conf.php`.
*   **v2 — explicit route table + controllers.** The config class returns a route definition array mapping URLs to controller actions, supporting public (no-auth) routes:

    ```php
    // Route tuple: [ControllerClass, ActionMethod, RequestTemplate, HttpMethod, RootUrl, NoAuth]
    public function getPBXCoreRESTAdditionalRoutes(): array
    ```

    See `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv2/Lib/ExampleRestAPIv2Conf.php` (routes like `/pbxcore/api/module-example-rest-api-v2/{actionName}`).

{% hint style="warning" %}
**Use the legacy callback only** when maintaining an existing v1/v2 module. For anything new, prefer attribute-routed REST API v3 — it gives you authentication, RBAC, OpenAPI generation, and the queue backpressure protection for free. The module-authoring guide covers all three patterns side by side: [REST API in modules](../../module-developement/rest-api-in-modules.md).
{% endhint %}

## ARI — advanced call control (when AMI is not enough)

For programmatic media and call-control beyond what AMI/AGI offer (building channels, bridges, and stasis applications), Asterisk's REST Interface (ARI) is available. It is served over the same built-in HTTP server as AJAM — `HttpConf` enables the HTTP server when **either AJAM or ARI is enabled** — and modules can contribute to its configuration through the `GENERATE_ARI_CONF` hook on `AsteriskConfigInterface`. ARI is an advanced, specialised surface; most modules will not need it. Reach for it only when AMI events plus AGI decisions cannot express the call flow you need.

## Decision summary

{% hint style="info" %}
* **Read/write configuration, CRUD, external integration** → [REST API v3](rest-api.md)
* **React to live call events / send immediate commands from a worker** → [AMI / AJAM](ami-ajam.md)
* **Make a decision inside an active call from the dialplan** → [AGI](agi.md)
* **Advanced channel/bridge/media control** → ARI
* **Maintaining an old module's HTTP endpoint** → legacy v1/v2 callback (see [REST API in modules](../../module-developement/rest-api-in-modules.md))
{% endhint %}
