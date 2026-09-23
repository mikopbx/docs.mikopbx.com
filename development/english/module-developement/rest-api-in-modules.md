---
description: >-
  Exposing a module REST API: the modern v3 attribute-routed pattern with
  auto-discovered #[ApiResource] controllers, Processor + Action classes, and a
  DataStructure single source of truth that drives O
---

# REST API in modules

A module can publish its own REST endpoints under the PBX core API surface. The recommended way in MikoPBX 2025.1.1+ is the **v3 attribute-routed pattern**: you annotate a `Controller` class with PHP 8 attributes, drop it into `Lib/RestAPI/{Resource}/`, and the core router discovers it automatically. There is **no route table to register** and **no method to override** in your `Conf.php` — the controller's `#[ApiResource]` attribute is the single declaration of the endpoint.

Throughout this page we use the running example module **ModuleBlackList** (config class `BlackListConf`) and imagine it exposes a `numbers` resource for managing blocked phone numbers. For every pattern below you will also find a pointer to the real, working reference module: `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`. Read those files alongside this page — they compile and run.

{% hint style="info" %}
**v3 is the only pattern you should write for new modules.** The older v1 (`moduleRestAPICallback()`) and v2 (namespaced `Processor` registered through `getPBXCoreRESTAdditionalRoutes()`) callback styles still work for backward compatibility, but they are documented separately in [../api/rest-api.md](../internal-structure/api/rest-api.md) and are not covered in depth here.
{% endhint %}

## How auto-discovery works

The core service `MikoPBX\PBXCoreREST\Providers\RouterProvider` scans for module controllers at boot. The relevant logic lives in `Core/src/PBXCoreREST/Providers/RouterProvider.php`, method `discoverModuleControllers()`:

1. It reads the list of **enabled** modules from the `PbxExtensionModules` table (`disabled = '0'`).
2. For each enabled module `{uniqid}`, it looks for a `Lib/RestAPI` directory.
3. It recursively scans that directory for files named `*Controller.php`.
4. For each one it builds the class name `Modules\{uniqid}\Lib\RestAPI\{...}\Controller`, checks the class exists, and reads its `#[ApiResource]` attribute.
5. If the attribute is present, it generates the routes and mounts them.

{% hint style="warning" %}
**Two preconditions for discovery.** Your module must be **enabled** in the database, and the controller files must live under `Lib/RestAPI/`. A controller placed anywhere else is never scanned. The directory `{uniqid}` here is the module's unique id (for ModuleBlackList that is `ModuleBlackList`), used only to locate the folder to scan — see the note on the URL path below.
{% endhint %}

### Your Conf.php stays empty for routing

Because discovery is automatic, the module's config class does **not** implement any routing method. `RestAPIConfigInterface` (in `Core/src/Modules/Config/RestAPIConfigInterface.php`) declares `getPBXCoreRESTAdditionalRoutes()` and `moduleRestAPICallback()`, but the base `ConfigClass` already provides empty stubs, so v3 modules override neither.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Modules\Config\ConfigClass;

/**
 * No REST routing code here.
 *
 * v3 controllers under Lib/RestAPI/ are discovered automatically by
 * RouterProvider. We do NOT override getPBXCoreRESTAdditionalRoutes()
 * or moduleRestAPICallback() — those belong to the legacy v1/v2 patterns.
 */
class BlackListConf extends ConfigClass
{
    // Intentionally empty for REST routing.
}
```
{% endcode %}

The reference module does exactly this — see `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/ExampleRestAPIv3Conf.php`, whose class body is empty with the comment _"No methods needed!"_.

## Per-resource file layout

Each REST resource is a self-contained folder under `Lib/RestAPI/`. For ModuleBlackList's `numbers` resource:

```
Extensions/ModuleBlackList/Lib/RestAPI/Numbers/
├── Controller.php          # HTTP interface: #[ApiResource], #[HttpMapping], #[ApiOperation]
├── Processor.php           # Routes the action name to an Action class
├── DataStructure.php       # Single source of truth: fields, validation, OpenAPI schema
└── Actions/
    ├── GetListAction.php
    ├── GetRecordAction.php
    ├── SaveRecordAction.php    # handles create / update / patch
    └── DeleteRecordAction.php
```

The reference module ships the same shape for its `Tasks` resource. See `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/` — and a second, smaller `Status` resource alongside it (`.../Lib/RestAPI/Status/`) that demonstrates a read-only endpoint.

## The Controller: attributes only

The controller is a thin declaration layer. Its methods have **empty bodies** — they exist only to carry attributes that the router and the OpenAPI generator read by reflection. Actual work happens in the Processor and Actions.

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/Controller.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers;

use MikoPBX\PBXCoreREST\Controllers\BaseRestController;
use MikoPBX\PBXCoreREST\Lib\Common\CommonDataStructure;
use MikoPBX\PBXCoreREST\Attributes\{
    ApiResource,
    ApiOperation,
    ApiParameterRef,
    ApiResponse,
    ApiDataSchema,
    HttpMapping,
    SecurityType,
    ResourceSecurity
};

#[ApiResource(
    path: '/pbxcore/api/v3/module-black-list/numbers',
    tags: ['ModuleBlackList - Numbers'],
    description: 'Manage blocked phone numbers',
    processor: Processor::class
)]
#[HttpMapping(
    mapping: [
        'GET'    => ['getList', 'getRecord'],
        'POST'   => ['create'],
        'PUT'    => ['update'],
        'PATCH'  => ['patch'],
        'DELETE' => ['delete']
    ],
    resourceLevelMethods: ['getRecord', 'update', 'patch', 'delete'],
    collectionLevelMethods: ['getList', 'create'],
    idPattern: '[^/:]+'
)]
#[ResourceSecurity(
    'module-black-list-numbers',
    requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN]
)]
class Controller extends BaseRestController
{
    /** @var string The processor class to handle requests */
    protected string $processorClass = Processor::class;

    #[ApiDataSchema(schemaClass: DataStructure::class, type: 'list', isArray: true)]
    #[ApiOperation(summary: 'List blocked numbers', operationId: 'getNumbersList')]
    #[ApiParameterRef('limit', dataStructure: CommonDataStructure::class)]
    #[ApiParameterRef('offset', dataStructure: CommonDataStructure::class)]
    #[ApiResponse(200, 'OK')]
    #[ApiResponse(401, 'Unauthorized')]
    public function getList(): void {}

    #[ApiDataSchema(schemaClass: DataStructure::class, type: 'detail')]
    #[ApiOperation(summary: 'Get one blocked number', operationId: 'getNumberById')]
    #[ApiResponse(200, 'OK')]
    #[ApiResponse(404, 'Not Found')]
    public function getRecord(): void {}

    #[ApiDataSchema(schemaClass: DataStructure::class, type: 'detail')]
    #[ApiOperation(summary: 'Add a blocked number', operationId: 'createNumber')]
    #[ApiParameterRef('number', required: true)]
    #[ApiResponse(201, 'Created')]
    #[ApiResponse(422, 'Validation failed')]
    public function create(): void {}

    #[ApiDataSchema(schemaClass: DataStructure::class, type: 'detail')]
    #[ApiOperation(summary: 'Replace a blocked number', operationId: 'updateNumber')]
    #[ApiParameterRef('number', required: true)]
    #[ApiResponse(200, 'OK')]
    #[ApiResponse(404, 'Not Found')]
    public function update(): void {}

    #[ApiDataSchema(schemaClass: DataStructure::class, type: 'detail')]
    #[ApiOperation(summary: 'Partially update a blocked number', operationId: 'patchNumber')]
    #[ApiParameterRef('number')]
    #[ApiResponse(200, 'OK')]
    #[ApiResponse(404, 'Not Found')]
    public function patch(): void {}

    #[ApiOperation(summary: 'Delete a blocked number', operationId: 'deleteNumber')]
    #[ApiResponse(200, 'OK')]
    #[ApiResponse(404, 'Not Found')]
    public function delete(): void {}
}
```
{% endcode %}

Compare every attribute here against the verified original in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Controller.php`.

### The attributes, one by one

All of these live in the `MikoPBX\PBXCoreREST\Attributes` namespace (`Core/src/PBXCoreREST/Attributes/`).

| Attribute             | Where           | Purpose                                                                                                                                                            |
| --------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `#[ApiResource]`      | class           | Marks the controller as discoverable. Carries `path`, `tags`, `description`, and `processor`. The router reads `path` verbatim.                                    |
| `#[HttpMapping]`      | class           | Maps each HTTP verb to a list of operation names, and declares which operations are collection-level vs resource-level. `idPattern` constrains the `{id}` segment. |
| `#[ResourceSecurity]` | class or method | RBAC + access channel. First argument is the resource permission name; `requirements` lists `SecurityType` cases.                                                  |
| `#[ApiOperation]`     | method          | OpenAPI summary/description/`operationId`. May also carry `requestBody` for uploads.                                                                               |
| `#[ApiDataSchema]`    | method          | Binds a response shape to a `DataStructure` class (`type: 'list'` or `'detail'`, `isArray`).                                                                       |
| `#[ApiParameterRef]`  | method          | References a parameter defined in a `DataStructure` (or `CommonDataStructure` for shared pagination params) and can mark it `required`.                            |
| `#[ApiResponse]`      | method          | Documents an HTTP status + description for OpenAPI.                                                                                                                |

{% hint style="warning" %}
**The URL path is a string you write, not a value the framework derives.** `RouterProvider::getResourcePathFromAttribute()` returns the `path` argument of `#[ApiResource]` exactly as written. The router uses the module's `{uniqid}` only to _find the folder to scan_ — it never computes the path from it. The `/pbxcore/api/v3/module-{slug}/{resource}` shape is therefore a **naming convention you follow when you author the attribute**, so module endpoints stay collision-free and easy to recognise. Pick a stable slug and write the full path yourself.
{% endhint %}

### URL scheme

By convention the path follows:

```
/pbxcore/api/v3/module-{slug}/{resource}[/{id}[:{action}]]
```

For ModuleBlackList:

```bash
# Collection
GET    /pbxcore/api/v3/module-black-list/numbers
POST   /pbxcore/api/v3/module-black-list/numbers

# Single record
GET    /pbxcore/api/v3/module-black-list/numbers/42
PUT    /pbxcore/api/v3/module-black-list/numbers/42
PATCH  /pbxcore/api/v3/module-black-list/numbers/42
DELETE /pbxcore/api/v3/module-black-list/numbers/42
```

Custom (non-CRUD) operations use a colon suffix. The reference module demonstrates both forms on its `Tasks` resource:

```bash
# Collection-level custom method
GET    /pbxcore/api/v3/module-example-rest-api-v3/tasks:getDefault

# Resource-level custom method
GET    /pbxcore/api/v3/module-example-rest-api-v3/tasks/42:download
POST   /pbxcore/api/v3/module-example-rest-api-v3/tasks/42:uploadFile
```

The `:{action}` routes are generated by `RouterProvider::generateMappedRoutes()` for every HTTP verb that has any operations in `#[HttpMapping]` (it mounts the `handleCustomRequest` / `handleResourceCustomRequest` route shapes alongside the plain CRUD routes); the `customMethods` list is consulted later, at dispatch time, to tell a custom action from a CRUD one. The `idPattern` `[^/:]+` deliberately excludes the colon so `{id}` and `{action}` parse correctly. This is all handled for you — you only declare the operations.

## Security types

`#[ResourceSecurity]` controls who may call the resource. The available channels are the cases of `MikoPBX\PBXCoreREST\Attributes\SecurityType` (`Core/src/PBXCoreREST/Attributes/SecurityType.php`):

| Case                         | Wire value     | Meaning                                                                                                             |
| ---------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `SecurityType::LOCALHOST`    | `localhost`    | Requests from `127.0.0.1`/`::1` bypass token auth. Not exposed in OpenAPI.                                          |
| `SecurityType::BEARER_TOKEN` | `bearer_token` | Requires `Authorization: Bearer <token>` — either a short-lived JWT or a long-lived API Key. Documented in OpenAPI. |
| `SecurityType::PUBLIC`       | `public`       | No authentication at all (OAuth callbacks, webhooks). Use sparingly.                                                |

The reference controller declares `#[ResourceSecurity('module-example-rest-api-v3-tasks', requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN])]`, which is the right default for a module: callable from the box itself and from authenticated API clients, but never anonymously.

{% hint style="info" %}
The first argument (`'module-black-list-numbers'`) is the **resource permission name** used by the RBAC layer (`Resource:Action`). Keep it unique per resource and stable across releases so API-key path/permission restrictions keep working.
{% endhint %}

## The Processor: action routing

The Processor is a tiny dispatcher. `BaseRestController` resolves the HTTP request to an action name (from `#[HttpMapping]`) and calls `Processor::callBack($request)`; the Processor switches on `$request['action']` and delegates to an Action class, returning a `PBXApiResult`.

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/Processor.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers;

use MikoPBX\PBXCoreREST\Lib\PBXApiResult;
use Modules\ModuleBlackList\Lib\RestAPI\Numbers\Actions\{
    GetListAction,
    GetRecordAction,
    SaveRecordAction,
    DeleteRecordAction
};
use Phalcon\Di\Injectable;

class Processor extends Injectable
{
    public static function callBack(array $request): PBXApiResult
    {
        $res = new PBXApiResult();
        $res->processor = __METHOD__;
        $action = $request['action'] ?? '';

        switch ($action) {
            case 'getList':
                $res = GetListAction::main($request['data'] ?? []);
                break;
            case 'getRecord':
                $res = GetRecordAction::main($request['data'] ?? []);
                break;
            case 'create':
            case 'update':
            case 'patch':
                // create/update/patch share one action; it inspects the ID to
                // decide create vs update, and the HTTP method for PATCH semantics.
                $res = SaveRecordAction::main($request['data'] ?? []);
                break;
            case 'delete':
                $res = DeleteRecordAction::main($request['data'] ?? []);
                break;
            default:
                $res->messages['error'][] = "Unknown action - $action in " . __CLASS__;
                break;
        }

        $res->function = $action;
        return $res;
    }
}
```
{% endcode %}

This mirrors `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Processor.php` exactly (which also routes `download`, `uploadFile`, and `getDefault`).

{% hint style="info" %}
Actions are static (`Action::main($data)`) and are **not** `Injectable` across the worker-queue boundary. If an action needs the authenticated session or forwarded HTTP headers, the Processor must forward them explicitly from `$request['sessionContext']` / `$request['httpHeaders']`. See the _Non-obvious behaviour_ notes in `Core/src/PBXCoreREST/AGENTS.md` and the filter in `Core/src/PBXCoreREST/Http/ForwardedHeaderFilter.php`.
{% endhint %}

## DataStructure: the single source of truth

`DataStructure` is where every field is defined **once**: its type, validation bounds, default, example, and OpenAPI description. The class extends `MikoPBX\PBXCoreREST\Lib\Common\AbstractDataStructure` and implements `MikoPBX\PBXCoreREST\Lib\Common\OpenApiSchemaProvider`. From the central `getParameterDefinitions()` method, the framework derives:

* the OpenAPI 3.1 request/response schemas (via `getListItemSchema()` / `getDetailSchema()`),
* the sanitization rules (`getSanitizationRules()`, inherited from `AbstractDataStructure`),
* the parameters referenced by `#[ApiParameterRef]` in the controller.

`getParameterDefinitions()` returns three buckets: `request` (writable fields), `response` (read-only fields like `id`, timestamps), and `related` (nested object schemas).

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/DataStructure.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers;

use MikoPBX\PBXCoreREST\Lib\Common\AbstractDataStructure;
use MikoPBX\PBXCoreREST\Lib\Common\OpenApiSchemaProvider;

class DataStructure extends AbstractDataStructure implements OpenApiSchemaProvider
{
    public static function getParameterDefinitions(): array
    {
        return [
            // Writable fields — accepted in POST/PUT/PATCH bodies.
            'request' => [
                'number' => [
                    'type'        => 'string',
                    'description' => 'rest_param_blacklist_number',
                    'minLength'   => 3,
                    'maxLength'   => 32,
                    'required'    => true,
                    'example'     => '74951234567',
                ],
                'comment' => [
                    'type'        => 'string',
                    'description' => 'rest_param_blacklist_comment',
                    'maxLength'   => 255,
                    'default'     => '',
                    'example'     => 'Robocaller',
                ],
            ],
            // Read-only fields — present only in responses.
            'response' => [
                'id' => [
                    'type'        => 'integer',
                    'description' => 'rest_schema_blacklist_id',
                    'readOnly'    => true,
                    'example'     => 42,
                ],
            ],
            // Nested object schemas referenced by $ref (none here).
            'related' => [],
        ];
    }

    public static function getListItemSchema(): array
    {
        $defs = self::getParameterDefinitions();
        $properties = ($defs['request'] ?? []) + ($defs['response'] ?? []);
        return ['type' => 'object', 'properties' => $properties];
    }

    public static function getDetailSchema(): array
    {
        return self::getListItemSchema();
    }

    public static function getRelatedSchemas(): array
    {
        return self::getParameterDefinitions()['related'] ?? [];
    }

    // getSanitizationRules() is inherited from AbstractDataStructure and is
    // generated from getParameterDefinitions() — do not duplicate it here.
}
```
{% endcode %}

The verified original is `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/DataStructure.php`. Note how it tags response-only fields with `'readOnly' => true` and rewrites the description prefix from `rest_schema_*` to `rest_param_*` when emitting request parameters — those `rest_*` keys are translation keys resolved through your module's `Messages/` files (see [translations.md](translations.md)).

## The 7-phase Action pattern

The Action class holds the business logic. MikoPBX standardises a **7-phase** order for create/update/patch in `SaveRecordAction`, documented in `Core/src/PBXCoreREST/AGENTS.md` and supported by the helper base `Core/src/PBXCoreREST/Lib/Common/AbstractSaveRecordAction.php` (`sanitizeInputData()`, `validateRequiredFields()`, `applyDefaults()`, `validateRecordExistence()`, `executeInTransaction()`):

{% hint style="warning" %}
**`applyDefaults()` is two different methods.** `AbstractSaveRecordAction::applyDefaults(array $data, array $defaults)` is protected and takes **two** arguments; `AbstractDataStructure::applyDefaults(array $data)` is public and takes **one**. The example below calls the DataStructure one — `DataStructure::applyDefaults($clean)`. Writing `self::applyDefaults($clean)` inside your Action resolves to the inherited two-argument version and fails with `ArgumentCountError`.
{% endhint %}

1. **Sanitize** — run `DataStructure::getSanitizationRules()` over the input. Never trust raw user data.
2. **Determine operation** — new vs existing record (presence of `id`), and which HTTP verb is in play (`$data['httpMethod']`). This must come _before_ any verb-dependent validation. For PUT/PATCH against a missing record, `validateRecordExistence()` returns a 404.
3. **Validate required** — fail fast on missing mandatory fields, **per verb**. POST and PUT require the mandatory fields; PATCH must not, because a partial body legitimately omits everything it does not intend to change.
4. **Apply defaults** — **CREATE only**. Applying defaults on update/patch would clobber the caller's existing values.
5. **Schema validate** — validate the complete dataset _after_ defaults.
6. **Business logic / save** — wrap writes in `executeInTransaction()`. For PATCH, write each field only when it is actually present in the payload (`isset()` / `array_key_exists()`), so a partial body never blanks out untouched columns.
7. **Format response** — return a consistent `PBXApiResult` (HTTP 201 on create, 200 on update).

{% hint style="warning" %}
Three phases are conditional on the operation: **Phase 3 (required fields are enforced for POST/PUT but not PATCH)**, **Phase 4 (defaults are CREATE-only)** and **Phase 6 (PATCH writes only present fields)**. The working example of this ordering is the reference module's `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/SaveRecordAction.php` — study its phase comments, its `$httpMethod !== 'PATCH'` gate and its `array_key_exists()` / `isset()` guards for PATCH support.

Do **not** copy `Core/src/PBXCoreREST/Lib/ApiKeys/SaveRecordAction.php` for this: it predates PATCH support, runs phases 2 and 3 in the opposite order, and calls `validateRequiredFields()` unconditionally — so a PATCH that omits a mandatory field is rejected with 422.

Phase 3 is the one that is easy to get wrong, and the framework will not catch it: `validateRequiredFields()` knows nothing about the verb, so an unconditional `if (empty($clean['title']))` guard makes every PATCH fail with 422 even though Phase 6 would have preserved the untouched columns perfectly well. Gate it:

```php
if ($httpMethod !== 'PATCH' && empty($clean['title'])) {
    // ... 422
}
```
{% endhint %}

### The reference SaveRecordAction is a full, DB-backed example

The example module's `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/SaveRecordAction.php` implements all seven phases for real: it sanitizes and validates the input, generates a public `uniqid` with `Tasks::generateUniqueID('TASK')`, and persists the `Tasks` model inside `executeInTransaction()`. Its sibling actions (`GetListAction`, `GetRecordAction`, `DeleteRecordAction`) are DB-backed too, so the module is a consistent end-to-end CRUD you can install and exercise (create a task, list it, fetch it by id or `uniqid`, delete it). Your own module follows the same shape — load/create your model (e.g. `BlackListNumbers` mapping table `m_BlackListNumbers`), save it inside `executeInTransaction()`, and return the persisted record.

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/Actions/SaveRecordAction.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers\Actions;

use MikoPBX\Common\Providers\TranslationProvider;
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;
use MikoPBX\PBXCoreREST\Lib\Common\AbstractSaveRecordAction;
use Modules\ModuleBlackList\Models\BlackListNumbers;
use Modules\ModuleBlackList\Lib\RestAPI\Numbers\DataStructure;

class SaveRecordAction extends AbstractSaveRecordAction
{
    public static function main(array $data): PBXApiResult
    {
        $result = self::createApiResult(__METHOD__);

        // PHASE 1: SANITIZE — sanitizeInputData() is inherited from
        // AbstractSaveRecordAction. The last argument lists free-text fields
        // that need extra dangerous-content checks.
        try {
            $clean = self::sanitizeInputData(
                $data,
                DataStructure::getSanitizationRules(),
                ['comment']
            );
        } catch (\Exception $e) {
            $result->messages['error'][] = $e->getMessage();
            return $result;
        }
        // Preserve the ID — it is usually not part of the sanitization rules.
        if (isset($data['id'])) {
            $clean['id'] = $data['id'];
        }

        // PHASE 2: DETERMINE OPERATION
        // Resolve the verb and the target record BEFORE validating, because
        // required-field rules differ per verb (see PHASE 3).
        $httpMethod  = strtoupper((string)($data['httpMethod'] ?? 'POST'));
        $isNewRecord = empty($clean['id']);
        $record = $isNewRecord
            ? new BlackListNumbers()
            : BlackListNumbers::findFirstById($clean['id']);

        if (!$isNewRecord && $record === null) {
            $result->messages['error'][] = 'Record not found';
            $result->httpCode = 404;
            return $result;
        }

        // PHASE 3: VALIDATE REQUIRED (POST/PUT only — PATCH is partial)
        // Emit a translation KEY through TranslationProvider, not a raw key and
        // not a hardcoded English string: module Messages/*.php catalogs are
        // merged into the global dictionary, so the caller gets the message in
        // the PBX language.
        if ($httpMethod !== 'PATCH' && empty($clean['number'])) {
            $result->messages['error'][] = TranslationProvider::translate(
                'module_black_list_number_required'
            );
            $result->httpCode = 422;
            return $result;
        }

        // PHASE 4: APPLY DEFAULTS (CREATE ONLY!)
        if ($isNewRecord) {
            $clean = DataStructure::applyDefaults($clean);
        }

        // PHASE 5: SCHEMA VALIDATION
        $schemaErrors = DataStructure::validateInputData($clean);
        if (!empty($schemaErrors)) {
            $result->messages['error'] = $schemaErrors;
            $result->httpCode = 422;
            return $result;
        }

        // PHASE 6: SAVE — wrap writes in a transaction and write only the
        // fields that were sent (PATCH-safe).
        try {
            $saved = self::executeInTransaction(function () use ($record, $clean) {
                foreach (['number', 'comment'] as $field) {
                    if (array_key_exists($field, $clean)) {
                        $record->$field = $clean[$field];
                    }
                }
                if (!$record->save()) {
                    throw new \Exception(implode(', ', $record->getMessages()));
                }
                return $record;
            });
        } catch (\Exception $e) {
            return self::handleError($e, $result);
        }

        // PHASE 7: RESPONSE
        $result->data = ['id' => (int)$saved->id, 'number' => $saved->number];
        $result->success = true;
        $result->httpCode = $isNewRecord ? 201 : 200;
        return $result;
    }
}
```
{% endcode %}

The action extends `AbstractSaveRecordAction`, so `createApiResult()`, `sanitizeInputData()`, `executeInTransaction()` and `handleError()` are inherited helpers; `applyDefaults()` and `validateInputData()` are called statically on the `DataStructure` (they live on `AbstractDataStructure`). Use these rather than rolling your own — the reference module's `Tasks/Actions/SaveRecordAction.php` is the implementation to copy.

{% hint style="info" %}
Emitting the error through `TranslationProvider::translate()` above is a **recommendation**, not a description of what Core does today: Core's own `ApiKeys/SaveRecordAction.php` still hardcodes English strings such as `'Description is required'`. There is no core precedent to look for — the module-side benefit is real (module `Messages/*.php` catalogs are merged into the global dictionary, and `translate()` falls back to the raw key if lookup fails), so use it in new module code.
{% endhint %}

For model and migration details (the `BlackListNumbers` model and `m_BlackListNumbers` table) see [data-model.md](data-model.md).

## Calling the endpoint

From an authenticated client (API Key shown):

{% tabs %}
{% tab title="Create (POST)" %}
```bash
curl -X POST https://pbx.example.com/pbxcore/api/v3/module-black-list/numbers \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"number":"74951234567","comment":"Robocaller"}'
# → 201 Created
# { "success": true, "data": { "id": 42, "number": "74951234567" }, "messages": [] }
```
{% endtab %}

{% tab title="Patch (PATCH)" %}
```bash
curl -X PATCH https://pbx.example.com/pbxcore/api/v3/module-black-list/numbers/42 \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"comment":"Confirmed spam"}'
# → 200 OK — only "comment" is updated; "number" is untouched
```
{% endtab %}

{% tab title="From localhost" %}
```bash
# On the PBX itself, localhost bypasses token auth (SecurityType::LOCALHOST).
curl http://127.0.0.1/pbxcore/api/v3/module-black-list/numbers
```
{% endtab %}
{% endtabs %}

Every response is the standard `PBXApiResult` envelope (`Core/src/PBXCoreREST/Lib/PBXApiResult.php`): `success` (bool), `data` (array), `messages` (`error`/`warning` lists), optional `httpCode` and `pagination`.

{% hint style="warning" %}
`httpCode` is optional, and that is a trap. When you return an unsuccessful `PBXApiResult` without setting it, the response goes out as **422 Unprocessable Entity** — regardless of the `#[ApiResponse(404, ...)]` attributes you declared on the controller. Attributes only describe the API; they do not set anything at runtime. Every early-return branch in a read or delete action must assign the status itself:

```php
$result->messages['error'][] = 'Record not found';
$result->httpCode = 404;   // without this line the client sees 422
```

Use 400 for a malformed request (missing path segment), 404 for a well-formed request against a resource that does not exist, and 422 for a well-formed request whose payload fails validation.
{% endhint %}

## Legacy patterns (for reference only)

If you are maintaining an older module you may encounter two earlier styles. Both are dispatched through `RestAPIConfigInterface` callbacks and are fully described in [../api/rest-api.md](../internal-structure/api/rest-api.md):

* **v1 — `moduleRestAPICallback()`** (interface constant `RestAPIConfigInterface::MODULE_RESTAPI_CALLBACK`). A single callback in the module's `Conf.php` dispatches by inspecting the request. Simple, but no OpenAPI and no structure.
* **v2 — namespaced Processor via `getPBXCoreRESTAdditionalRoutes()`** (interface constant `RestAPIConfigInterface::GET_PBXCORE_REST_ADDITIONAL_ROUTES`). The module returns an explicit route table that `RouterProvider` merges in. Better organised than v1, but you still hand-register every route.

{% hint style="danger" %}
Do not write new modules against v1 or v2. They exist only so the router can keep dispatching pre-2025 modules. The v3 attribute pattern gives you auto-discovery, OpenAPI 3.1, RBAC via `#[ResourceSecurity]`, and the structured 7-phase action flow with zero route bookkeeping.
{% endhint %}

## Lifecycle hooks around requests

Two interface hooks let a module observe every REST request, regardless of pattern: `onBeforeExecuteRestAPIRoute(Micro $app)` and `onAfterExecuteRestAPIRoute(Micro $app)` (constants `RestAPIConfigInterface::ON_BEFORE_EXECUTE_RESTAPI_ROUTE` and `ON_AFTER_EXECUTE_RESTAPI_ROUTE`). `RouterProvider::attachModuleHooks()` fires them via `PBXConfModulesProvider::hookModulesMethod()` on the Phalcon Micro `beforeExecuteRoute` / `afterExecuteRoute` events. These belong to the module config class and are covered with the other module hooks in [hooks-reference.md](hooks-reference.md).

## See also

* [../api/rest-api.md](../internal-structure/api/rest-api.md) — the full REST API reference, including the v1/v2 legacy dispatch details.
* [../api/README.md](../internal-structure/api/) — API overview and authentication.
* [recipes.md](recipes.md) — end-to-end module recipes.
* [hooks-reference.md](hooks-reference.md) — every module lifecycle hook, including the REST request hooks above.
* [data-model.md](data-model.md) — defining the model and migration behind a resource.
* Reference module: `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/` — read `Lib/RestAPI/Tasks/` and `Lib/RestAPI/Status/` as working v3 examples.
