---
description: >-
  The MikoPBX PBXCoreREST API: v3 architecture, auth, routing and async
  processing.
---

# REST API

MikoPBX exposes a single HTTP API surface called **PBXCoreREST**. The current generation is **v3**: a RESTful, attribute-driven API rooted at `/pbxcore/api/v3/`. Routes are auto-discovered from PHP 8.4 attributes, requests are authenticated by JWT or API key, and the actual work runs asynchronously in a pool of Redis-backed worker processes.

This page describes the v3 architecture end to end. For the high-level overview of the API surface see [api/README.md](./); for shipping your own v3 endpoints from a module see [module-developement/rest-api-in-modules.md](../../module-developement/rest-api-in-modules.md); for token issuance and external integrations see [cookbook/rights-and-auth/external-authentication.md](../../cookbook/rights-and-auth/external-authentication.md).

{% hint style="info" %}
Throughout this page the host is written as `pbx.example.com`. Replace it with the IP address or hostname of your MikoPBX. We thread a fictional module, **ModuleBlackList**, through the narrative for illustration, and anchor every pattern to the real reference module in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`.
{% endhint %}

## URL scheme

Every v3 resource lives under `/pbxcore/api/v3/{resource}`. Standard CRUD maps onto HTTP verbs, and Google-API-Guide-style **custom methods** are addressed with a colon suffix:

```
GET    /pbxcore/api/v3/{resource}                  → getList    (collection)
GET    /pbxcore/api/v3/{resource}/{id}             → getRecord  (resource)
POST   /pbxcore/api/v3/{resource}                  → create
PUT    /pbxcore/api/v3/{resource}/{id}             → update     (full replace)
PATCH  /pbxcore/api/v3/{resource}/{id}             → patch      (partial)
DELETE /pbxcore/api/v3/{resource}/{id}             → delete
GET    /pbxcore/api/v3/{resource}:customMethod     → collection custom method
POST   /pbxcore/api/v3/{resource}/{id}:customMethod → resource custom method
```

The collection/resource verb routing is defined in `PBXCoreREST/Controllers/BaseRestController.php` (`$actionMapping`, `mapHttpMethodToAction()`). The custom-method colon syntax is parsed by `handleCustomRequest()` / `handleResourceCustomRequest()` in the same class.

The reference Tasks controller declares exactly these patterns — see `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Controller.php`, whose resource is `/pbxcore/api/v3/module-example-rest-api-v3/tasks` and which exposes the custom methods `:getDefault` (collection) and `/{id}:download`, `/{id}:uploadFile` (resource).

## Attribute-based routing

There is **no route table to edit**. `PBXCoreREST/Providers/RouterProvider.php` scans the `Controllers/` tree (and every enabled module's `Lib/RestAPI/` directory), reflects each class, and registers routes for any controller that carries an `#[ApiResource]` attribute. The base path comes straight from that attribute's `path` argument (`RouterProvider::getResourcePathFromAttribute()`), and the set of generated routes is driven by the `#[HttpMapping]` attribute (`RouterProvider::generateMappedRoutes()`).

The attribute set lives in `PBXCoreREST/Attributes/`:

| Attribute             | Target         | Purpose                                                                                                                                            |
| --------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `#[ApiResource]`      | class          | Declares the resource: `path`, `tags`, `description`, default `security`, `processor`, `version` (defaults to `v3`).                               |
| `#[HttpMapping]`      | class / method | Maps HTTP methods to operation names, lists `resourceLevelMethods` / `collectionLevelMethods` / `customMethods`, and sets an optional `idPattern`. |
| `#[ResourceSecurity]` | class / method | RBAC `Resource:Action` policy plus the allowed `SecurityType` requirements.                                                                        |
| `#[ApiOperation]`     | method         | OpenAPI metadata (`summary`, `description`, `operationId`, `requestBody`, `internal`).                                                             |
| `#[ApiParameterRef]`  | method         | References a parameter defined in a `DataStructure` (single source of truth).                                                                      |
| `#[ApiDataSchema]`    | method         | Binds the request/response schema to a `DataStructure` class.                                                                                      |
| `#[ApiResponse]`      | method         | Documents one HTTP response code.                                                                                                                  |

Three enums back the attributes:

* `ActionType` — `READ`, `WRITE`, `ADMIN`, `SENSITIVE`. Permission inheritance is defined in `ActionType::getAllowedActions()` (admin ⊇ write ⊇ read; sensitive ⊇ read).
* `SecurityType` — `LOCALHOST`, `BEARER_TOKEN`, `PUBLIC`.
* `ParameterLocation` — `PATH`, `QUERY`, `HEADER`, `COOKIE` (OpenAPI 3.1 locations).

A minimal v3 controller header looks like this (modelled on the real Tasks controller; here for the fictional **ModuleBlackList**):

{% code title="Lib/RestAPI/Numbers/Controller.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers;

use MikoPBX\PBXCoreREST\Controllers\BaseRestController;
use MikoPBX\PBXCoreREST\Attributes\{ApiResource, HttpMapping, ResourceSecurity, SecurityType};

#[ApiResource(
    path: '/pbxcore/api/v3/module-black-list/numbers',
    tags: ['ModuleBlackList - Numbers'],
    description: 'Blacklisted phone numbers managed by ModuleBlackList',
    processor: Processor::class
)]
#[HttpMapping(
    mapping: [
        'GET'    => ['getList', 'getRecord'],
        'POST'   => ['create'],
        'PUT'    => ['update'],
        'PATCH'  => ['patch'],
        'DELETE' => ['delete'],
    ],
    resourceLevelMethods: ['getRecord', 'update', 'patch', 'delete'],
    collectionLevelMethods: ['getList', 'create'],
    idPattern: '[^/:]+'
)]
#[ResourceSecurity('module-black-list-numbers', requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN])]
class Controller extends BaseRestController
{
    protected string $processorClass = Processor::class;

    public function getList(): void {}
    public function getRecord(): void {}
    public function create(): void {}
    public function update(): void {}
    public function patch(): void {}
    public function delete(): void {}
}
```
{% endcode %}

The controller methods are intentionally empty: they exist only so reflection can read their attributes. The actual routing target is `handleCRUDRequest()` / `handleCustomRequest()` in `BaseRestController`, which forwards to the `$processorClass`.

{% hint style="info" %}
For the full module-side build-out (Processor, DataStructure, Action classes, `module.json`), follow [module-developement/rest-api-in-modules.md](../../module-developement/rest-api-in-modules.md) and copy the working tree under `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`.
{% endhint %}

## Authentication

`PBXCoreREST/Middleware/AuthenticationMiddleware.php` runs `before` every route (it implements `Phalcon\Mvc\Micro\MiddlewareInterface`; its `call()` entry method is attached on the `before` event in `RouterProvider::attachMiddleware()`). It resolves access in this order: public-endpoint check → Bearer token (JWT or API key, followed by the RBAC/ACL check inside the Bearer branch) → localhost bypass. Because the Bearer branch runs first, a request that presents an _invalid_ token is rejected even when it originates from localhost.

### Localhost bypass

A request that carries no Bearer token and originates from `127.0.0.1` or `::1` skips authentication and ACL entirely (`AuthenticationMiddleware::call()` calls `$request->isLocalHostRequest()`). This is how internal core processes and Nginx Lua helpers reach the API; it is **not** available to remote callers.

### JWT (Bearer tokens)

The primary remote auth mechanism is JWT. Tokens are minted by `POST /pbxcore/api/v3/auth:login` (`PBXCoreREST/Controllers/Auth/RestController.php`) and signed with **HMAC-SHA256** (`alg: HS256`) by `PBXCoreREST/Lib/Auth/JWTHelper.php`. Two lifetimes are involved (`JWTHelper::ACCESS_TOKEN_TTL` / `REFRESH_TOKEN_TTL`):

* **Access token** — 900 seconds (15 minutes). Sent on every request in the `Authorization: Bearer <token>` header.
* **Refresh token** — 2 592 000 seconds (30 days). Stored as an `httpOnly`, `SameSite=Strict` cookie and exchanged for a new access token at `POST /pbxcore/api/v3/auth:refresh` (token is rotated on each refresh).

Obtain an access token:

{% tabs %}
{% tab title="BASH" %}
```bash
curl -X POST 'https://pbx.example.com/pbxcore/api/v3/auth:login' \
  -H 'Content-Type: application/json' \
  -c auth-cookies.txt \
  --data '{"login":"admin","password":"adminpassword"}'
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
$host   = 'pbx.example.com';
$client = new \GuzzleHttp\Client();

$resp = $client->post("https://$host/pbxcore/api/v3/auth:login", [
    'headers' => ['Content-Type' => 'application/json'],
    'json'    => ['login' => 'admin', 'password' => 'adminpassword'],
]);

$body        = json_decode((string)$resp->getBody(), true);
$accessToken = $body['data']['accessToken'] ?? null;   // JWT, valid 15 min
$expiresIn   = $body['data']['expiresIn'] ?? null;      // seconds
```
{% endtab %}
{% endtabs %}

`auth:login` returns the access token in the body and sets the refresh token as an `httpOnly` cookie. Use the access token as a Bearer credential on subsequent calls:

```bash
curl 'https://pbx.example.com/pbxcore/api/v3/module-example-rest-api-v3/tasks' \
  -H 'Authorization: Bearer <accessToken>'
```

{% hint style="warning" %}
The `auth` controller handles cookies in the web (php-fpm) context, so its methods diverge slightly from the generic worker flow — see `PBXCoreREST/Controllers/Auth/RestController.php`. Everything else in the API goes through the async worker queue described below.
{% endhint %}

### API keys

For machine-to-machine integrations that should not expire every 15 minutes, MikoPBX supports long-lived API keys. A key is a 64-character hex string (`ApiKeys::generateApiKey()` → `bin2hex(random_bytes(32))`) stored **bcrypt-hashed** in the `m_ApiKeys` table (`Common/Models/ApiKeys.php`, `setSource('m_ApiKeys')`; the hash is written in `PBXCoreREST/Lib/ApiKeys/SaveRecordAction.php` via `password_hash($key, PASSWORD_BCRYPT)`). The plaintext key is shown **once** at creation and never stored.

A request presents the key with either header (`PBXCoreREST/Http/Request.php::getBearerToken()` accepts both, preferring the first):

```bash
# Canonical form
curl 'https://pbx.example.com/pbxcore/api/v3/module-example-rest-api-v3/tasks' \
  -H 'Authorization: Bearer 0123ab...<64 hex chars>'

# Fallback header
curl 'https://pbx.example.com/pbxcore/api/v3/module-example-rest-api-v3/tasks' \
  -H 'X-Api-Key: 0123abc...<64 hex chars>'
```

{% hint style="info" %}
The web-cabinet session flow (`POST /admin-cabinet/session/start`) still exists for the browser UI and legacy scripts, but for programmatic v3 access prefer JWT or an API key. Legacy session example, with the generic host:

```bash
curl 'https://pbx.example.com/admin-cabinet/session/start' \
  -X POST --cookie-jar auth-cookies.txt \
  -H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' \
  -H 'X-Requested-With: XMLHttpRequest' \
  --data 'login=admin&password=adminpassword'
```
{% endhint %}

### Session context forwarded to workers

When a request carries a Bearer token, `BaseController::prepareRequestMessage()` attaches a `sessionContext` envelope to the queued message (`auth_type`, `token_id` = the `m_ApiKeys.id` that validated the request, `origin`, `remote_addr`, and for JWTs `user_name` / `role` / `session_id`). Workers receive it under `$request['sessionContext']`; localhost/public requests arrive without it (treat as `[]`). Processors must forward this bag explicitly to their Action class — Actions are not `Injectable` across the queue boundary.

## Asynchronous processing flow

PBXCoreREST controllers do **not** execute business logic in the php-fpm worker. They serialize the request, push it onto a Redis queue, and poll for the result. The heavy lifting happens in a separate pool of CLI workers.

```
HTTP request
  → RouterProvider (route discovered from #[ApiResource] / #[HttpMapping])
    → AuthenticationMiddleware (public → JWT/API key → ACL → localhost bypass)
      → BaseRestController::handleCRUDRequest()/handleCustomRequest()
        → BaseController::sendRequestToBackendWorker()
            • queue-length check → HTTP 503 if overloaded (backpressure)
            • RPUSH onto Redis list  api:requests
            • poll Redis key  api:response:{request_id}
              → WorkerApiCommands (pool of 3 processes, BLPOP api:requests)
                  • drop stale requests (age > API_REQUEST_TTL)
                  • route via {Resource}ManagementProcessor::callback()
                    → Action class (e.g. 7-phase SaveRecordAction)
                      → PBXApiResult
                  • SETEX api:response:{request_id} (TTL 120s)
        → Http\Response::send()  →  HTTP response
```

### Enqueue side (`BaseController::sendRequestToBackendWorker`)

`PBXCoreREST/Controllers/BaseController.php` builds the message (`prepareRequestMessage()`), stamps it with a unique `request_id` and a `created_at` timestamp, then:

1. **Backpressure / fast-fail.** Before pushing, it compares the current queue length (`lLen api:requests`) against `PbxSettings::API_QUEUE_MAX_LENGTH` (default `50`, key `APIQueueMaxLength`). If the queue is over the threshold it immediately returns **HTTP 503** and frees the php-fpm worker instead of blocking. The threshold is cached for 10 seconds to avoid a Redis round-trip on every request.
2. **Enqueue.** `RPUSH api:requests` with the JSON message (`WorkerApiCommands::REDIS_API_QUEUE`).
3. **Async shortcut.** If the request is async, the controller responds 200 immediately and the client receives the result later over an nchan channel.
4. **Poll for response.** Otherwise it polls `api:response:{request_id}` (`WorkerApiCommands::REDIS_API_RESPONSE_PREFIX`) with a graduated backoff: 10 ms for the first second, 50 ms for the next 4 s, 100 ms for the next 10 s, then 250 ms for the remainder, until the `maxTimeout` (minimum 30 s) elapses. On arrival it deletes the response key and maps the result to an HTTP status code (200, 201, 422 for validation failures, 409 for conflicts, etc., honouring an explicit `httpCode` if the Action set one).

### Worker side (`WorkerApiCommands`)

`PBXCoreREST/Workers/WorkerApiCommands.php` runs as a pool — `public int $maxProc = 3` — each instance blocking on `BLPOP [api:requests, api:failed:jobs]`. For each dequeued job it:

* **Drops stale requests.** `isStaleRequest()` compares `created_at` age against `PbxSettings::API_REQUEST_TTL` (default `35` seconds, key `APIRequestTTL`); a request older than the TTL is dropped without a response, because the client has already timed out. Debug and async requests bypass this check.
* **Resets ORM state** between jobs (`resetOrmStateBeforeJob()`) so a worker never serves a stale not-found lookup or a leaked transaction.
* **Resolves the processor** named in the message and calls its static `callback(array $request): PBXApiResult` (`prepareProcessor()` / `executeRequest()`). SQLite "database is locked" errors are retried with backoff (`executeWithRetry()`), and failed jobs are retried up to `MAX_JOB_ATTEMPTS` (3) before being parked in `api:failed:jobs:data`.
* **Writes the response** with `SETEX api:response:{request_id}` (TTL 120 s — `REDIS_RESPONSE_TTL`). Responses larger than 1 MB are gzip-compressed into a side Redis key or a temp file and dereferenced by the controller.

### Routing inside the worker: ManagementProcessor → Action

Each resource ships a Processor whose static `callback()` dispatches the message's `action` to a dedicated Action class — typically with a PHP 8.4 `enum` + `match`. Core resources name the class `{Resource}ManagementProcessor`; **modules** name it plainly `Processor` inside the resource folder and point `#[ApiResource(processor: Processor::class)]` at it. The example below uses the Core naming:

```php
enum NumbersAction: string {
    case GET_LIST = 'getList';
    case CREATE   = 'create';
    case UPDATE   = 'update';
    case PATCH    = 'patch';
    case DELETE   = 'delete';
}

class NumbersManagementProcessor extends \Phalcon\Di\Injectable
{
    public static function callback(array $request): PBXApiResult
    {
        return match (NumbersAction::tryFrom($request['action'])) {
            NumbersAction::GET_LIST => GetListAction::main($request['data']),
            NumbersAction::CREATE,
            NumbersAction::UPDATE,
            NumbersAction::PATCH    => SaveRecordAction::main($request),
            NumbersAction::DELETE   => DeleteRecordAction::main($request['data']),
            default                 => throw new \RuntimeException('Unknown action'),
        };
    }
}
```

See the working processor at `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Processor.php` and its Action classes under `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/`.

### Forwarded HTTP headers

Worker Actions run in a CLI process and cannot reach Phalcon's `Request`, so `BaseController::prepareRequestMessage()` pre-extracts a **filtered** subset of inbound headers into the message under `httpHeaders` (lowercased keys). The policy is in `PBXCoreREST/Http/ForwardedHeaderFilter.php`: `Authorization`, `Cookie`, `Set-Cookie`, `Proxy-Authorization`, **`X-Api-Key`** and the `Authentication-*` prefix family are stripped unconditionally (`X-Api-Key` because `getBearerToken()` accepts it as a credential). The public-safe allow-list is an explicit enumeration — `X-Forwarded-For`, `X-Forwarded-Proto`, `X-Forwarded-Host`, `X-Forwarded-Port`, `X-Real-IP`, `User-Agent`, `Referer`, `Origin`, `Host`, `Accept-Language` — not an `X-Forwarded-*` wildcard; the reserved prefixes `X-Mikopbx-*` and `X-Module-*` also pass through. Surviving values are lowercased by key and truncated at 1024 bytes.

Actions receive the bag only when the Processor forwards it — there is no automatic second parameter. The one Core action that takes it declares:

```php
// PBXCoreREST/Lib/System/CheckClientIpVisibilityAction.php
public static function main(array $sessionContext = [], array $httpHeaders = []): PBXApiResult
{
    $ua  = $httpHeaders['user-agent'] ?? '';
    $xff = $httpHeaders['x-forwarded-for'] ?? '';
    // ...
}
```

Its Processor passes `$request['sessionContext'] ?? []` and `$request['httpHeaders'] ?? []` explicitly. Every other action — including all of the reference module's — uses the plain single-argument `main(array $data)` form.

A module may introduce its own forwarded header simply by naming it under `X-Module-<YourModule>-`; no core edits are required.

## DataStructure: the single source of truth

Per-resource parameter definitions live in one place — a `DataStructure` class extending `AbstractDataStructure`. It declares every request/response field with its type, validation rules, sanitization rule and OpenAPI example. By convention each resource's DataStructure also exposes a `createFromModel()` that shapes a model row into the response payload — that one is **not** inherited: it is a per-resource method you write yourself (`AbstractDataStructure` supplies `getSanitizationRules()`, `applyDefaults()`, `validateInputData()` and the `createFromSchema()`/`createForList()` builders, but no `createFromModel()`). The `#[ApiParameterRef]` attributes on the controller reference these definitions by name, so docs, validation, sanitization and the OpenAPI schema never drift apart.

```php
class DataStructure extends \MikoPBX\PBXCoreREST\Lib\Common\AbstractDataStructure
{
    public static function getParameterDefinitions(): array
    {
        return [
            'request' => [
                'number' => [
                    'type'        => 'string',
                    'description' => 'rest_blacklist_number',
                    'required'    => true,
                    'minLength'   => 3,
                    'maxLength'   => 32,
                    'sanitize'    => 'text',
                    'example'     => '+15551234567',
                ],
            ],
            'response' => [
                'represent' => ['type' => 'string'],
            ],
        ];
    }
}
```

The reference implementation is `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/DataStructure.php`.

### The 7-phase SaveRecordAction

Create / Update / Patch all funnel through one Action that runs seven explicit phases (see the Core abstraction in `PBXCoreREST/Lib/Common/AbstractSaveRecordAction.php` and the module example `.../Tasks/Actions/SaveRecordAction.php`):

1. **Sanitize** input per the DataStructure rules.
2. **Determine operation** — new vs. existing record, and which HTTP verb is in play (`$data['httpMethod']`). This must come first, because the next phase depends on the verb.
3. **Validate required** fields — for POST and PUT only. PATCH must skip this: a partial body legitimately omits everything it does not intend to change.
4. **Apply defaults** — on CREATE only (never on update/patch).
5. **Schema validation** — after defaults are applied.
6. **Save** inside `executeInTransaction()`; for PATCH, only fields actually present in the payload are written (`isset()` / `array_key_exists()`).
7. **Response** — build the result via the resource's `createFromModel()` and set HTTP 201 (create) or 200 (update/patch).

{% hint style="warning" %}
Some older Core actions — `PBXCoreREST/Lib/ApiKeys/SaveRecordAction.php` is the clearest example — run phases 2 and 3 the other way round and call `validateRequiredFields()` **unconditionally**. That is the pre-PATCH shape: because the check knows nothing about the HTTP verb, any PATCH that omits a mandatory field is rejected with 422 even though phase 6 would have preserved the untouched columns perfectly well. Do not copy it. Follow the order above, which is what the reference module implements — see `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/SaveRecordAction.php`.
{% endhint %}

## Response envelope

Worker Actions return a `PBXApiResult` (`PBXCoreREST/Lib/PBXApiResult.php`). Its `getResult()` produces the wire body, and `Http\Response::send()` then merges a `meta` block and adds an `E-Tag` header. The v3 envelope is:

```json
{
  "result": true,
  "data": [
    { "id": "1", "represent": "+15551234567" },
    { "id": "2", "represent": "+15557654321" }
  ],
  "messages": [],
  "function": "getList",
  "processor": "Modules\\ModuleBlackList\\Lib\\RestAPI\\Numbers\\Processor",
  "pid": 16729,
  "meta": {
    "timestamp": "2025-01-25T08:23:02-03:00",
    "hash": "68fa513e851d9452aeed7f8f16ad61cb8705c4e5"
  }
}
```

Notes confirmed from `PBXApiResult::getResult()` and `Http/Response::send()`:

* `result` (boolean) is the success flag; `data` holds the payload; `messages` carries `error` / `warning` / `info` arrays.
* `function` and `processor` identify the Action that ran; `pid` is the worker PID.
* `httpCode` and `pagination` keys appear only when the Action sets them (list endpoints add `pagination`).
* `meta.timestamp` and `meta.hash` are added by the HTTP layer on send.

Error responses (`Response::setPayloadError()`) use the same skeleton with `result: false` and the detail under `messages.error`:

```json
{
  "result": false,
  "data": [],
  "messages": { "error": ["Service temporarily unavailable, please retry later"] },
  "function": "",
  "processor": "",
  "pid": 16730
}
```

### HTTP status codes you will see

| Code  | Meaning                                                    | Source                                                |
| ----- | ---------------------------------------------------------- | ----------------------------------------------------- |
| `200` | Success (read / update / delete)                           | `BaseController`                                      |
| `201` | Resource created                                           | `SaveRecordAction` phase 7                            |
| `400` | Bad request (malformed custom method, missing method name) | `BaseRestController`                                  |
| `401` | Missing/invalid Bearer token                               | `AuthenticationMiddleware`                            |
| `403` | Authenticated but not permitted (ACL)                      | `AuthenticationMiddleware`                            |
| `404` | Endpoint or record not found                               | `RouterProvider::attachMiddleware()` notFound handler |
| `405` | HTTP method / custom method not allowed                    | `BaseRestController`                                  |
| `409` | Conflict (constraint violation)                            | Action via `httpCode`                                 |
| `422` | Validation error                                           | Action via `httpCode`                                 |
| `503` | Queue overloaded — retry with backoff                      | `BaseController` backpressure                         |

## A complete v3 call

Putting it together against the reference Tasks resource (a `getDefault` collection custom method, then a `create`):

```bash
# 1. Authenticate
TOKEN=$(curl -s -X POST 'https://pbx.example.com/pbxcore/api/v3/auth:login' \
  -H 'Content-Type: application/json' \
  --data '{"login":"admin","password":"adminpassword"}' \
  | php -r 'echo json_decode(file_get_contents("php://stdin"), true)["data"]["accessToken"];')

# 2. Get default values for a new task (collection custom method)
curl -s 'https://pbx.example.com/pbxcore/api/v3/module-example-rest-api-v3/tasks:getDefault' \
  -H "Authorization: Bearer $TOKEN"

# 3. Create a task
curl -s -X POST 'https://pbx.example.com/pbxcore/api/v3/module-example-rest-api-v3/tasks' \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"title":"Call back customer","status":"new","priority":1}'
```

{% hint style="success" %}
Because routing, validation and the OpenAPI schema are all generated from the controller attributes and the `DataStructure`, a v3 endpoint is fully described by the code itself. The OpenAPI document MikoPBX serves is produced from these same attributes — there is no separate spec to maintain.
{% endhint %}

## See also

* [api/README.md](./) — API surface overview.
* [module-developement/rest-api-in-modules.md](../../module-developement/rest-api-in-modules.md) — build a v3 REST API inside a module.
* [cookbook/rights-and-auth/external-authentication.md](../../cookbook/rights-and-auth/external-authentication.md) — issuing tokens and integrating external systems.
