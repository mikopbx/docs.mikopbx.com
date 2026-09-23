---
description: >-
  Architecture overview of the MikoPBX core engine: the six MikoPBX\ namespaces,
  the Phalcon 5 / SQLite / Redis platform, and the core data-model graph.
---

# Core

MikoPBX is a free PBX system for SMB built on Asterisk. The PHP side of the product — everything you import as `MikoPBX\…` — lives in a single Composer application called **Core**. This page is the map: it shows how the code is split into namespaces, what each area is responsible for, which framework and storage back the engine, and how the core database models relate to one another. Use it as the orientation layer before you read the deeper chapters on the [admin interface](admin-interface.md), the [REST API](api/), and the [module data model](../module-developement/data-model.md).

Throughout this documentation we develop one running example module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` on table `m_BlackListNumbers`, front-end assets named per action — `module-black-list-index.js`, `module-black-list-modify.js`). Where a pattern has a corresponding shipped sample, we point you to a real example module under `Extensions/EXAMPLES/`.

## Platform

The platform is fixed by `Core/composer.json`. There is no abstraction layer hiding the versions — modules run inside the same PHP process and DI container as Core, so you target these exact baselines:

{% code title="Core/composer.json" %}
```json
{
    "name": "mikopbx/core",
    "require": {
        "php": "^8.4",
        "ext-phalcon": "^5.9.3",
        "ext-sqlite3": "*",
        "ext-redis": "*",
        "ext-pcntl": "*",
        "ext-posix": "*"
    },
    "autoload": {
        "psr-4": {
            "MikoPBX\\": "src/"
        }
    }
}
```
{% endcode %}

| Concern       | Choice                                                            |
| ------------- | ----------------------------------------------------------------- |
| Language      | PHP 8.4+ (Composer platform pins `php` to `8.4`, requires `^8.4`) |
| Framework     | Phalcon 5 (`ext-phalcon ^5.9.3`) — MVC, ORM, DI container         |
| Main database | SQLite (`ext-sqlite3`), WAL journaling                            |
| Cache / IPC   | Redis (`ext-redis`)                                               |
| Process model | Long-running daemons via `ext-pcntl` / `ext-posix`                |
| Autoload      | PSR-4, `MikoPBX\` → `src/`                                        |

{% hint style="info" %}
Phalcon is a C extension. Its services (`db`, `router`, `dispatcher`, models metadata, cache) are registered in a **dependency-injection container**, and that container is the same one your module code receives. Everything in Core is discoverable through the DI — see [Common/Providers](core.md#common-the-shared-foundation) below.
{% endhint %}

## The six namespaces of `MikoPBX\`

All PHP source lives under `Core/src/` and maps one-to-one to a top-level namespace. Each major area carries its own `AGENTS.md` describing local conventions — those are the authoritative, in-tree references for that subsystem.

```
Core/src/
├── Common/        →  MikoPBX\Common\        Models, Providers, Library, Messages
├── Core/          →  MikoPBX\Core\          Asterisk config + AMI/AGI/CDR, System, Workers
├── PBXCoreREST/   →  MikoPBX\PBXCoreREST\   REST API v3 infrastructure
├── AdminCabinet/  →  MikoPBX\AdminCabinet\  Web admin MVC (Phalcon + Volt + Fomantic UI)
├── Modules/       →  MikoPBX\Modules\       Extension framework (this is your home)
└── Service/       →  MikoPBX\Service\       Main entry point, License
```

### Common — the shared foundation

`MikoPBX\Common\` (`Core/src/Common/`) is everything the rest of the engine depends on:

* **`Models/`** — the SQLite-backed ORM models (see the [model graph](core.md#core-data-model-graph) below). All extend `MikoPBX\Common\Models\ModelsBase`.
* **`Providers/`** — the DI service providers. There are **34** provider classes (`Core/src/Common/Providers/*.php`) registering services such as `db`, `managedCache`, `redis`, `logger`, `pbxConfModules`, `eventBus`, `license`, and more. See `Core/src/Common/Providers/AGENTS.md`.
* **`Library/`** — reusable helpers and value objects.
* **`Messages/`** — translation arrays. There are **26** language directories under `Core/src/Common/Messages/` (for example `en`, `ru`, `de`, `zh_Hans`, `pt_BR`), each holding PHP files that return key→string maps. The catalogue is declared by `LanguageProvider::AVAILABLE_LANGUAGES`.

### Core — telephony and the operating system

`MikoPBX\Core\` (`Core/src/Core/`) is the engine that turns database rows into a running Asterisk PBX on a Linux appliance:

* **`Asterisk/`** — generators that emit Asterisk config files plus the live integration classes `AsteriskManager` (AMI), `AGI` / `AGIBase`, `AstDB`, and `CdrDb`. The generator base class lives at `Core/src/Core/Asterisk/Configs/AsteriskConfigClass.php` — and that is the class every module config inherits from (see [Modules](core.md#modules-the-extension-framework)).
* **`System/`** — boot, network, storage, certificates, cloud provisioning, and the container entry point (`Core/src/Core/System/`).
* **`Workers/`** — the long-running daemons (`Core/src/Core/Workers/`). All extend `WorkerBase` and implement `WorkerInterface`. Examples shipped in the tree include `WorkerModelsEvents` (reacts to model changes and regenerates config), `WorkerCdr`, `WorkerCallEvents`, `WorkerMarketplaceChecker`, and `WorkerS3Upload`.

### PBXCoreREST — the internal/external REST API

`MikoPBX\PBXCoreREST\` (`Core/src/PBXCoreREST/`) is **REST API v3**: attribute-based routing, async Redis-queue processing, and OpenAPI auto-generation. Resource controllers extend shared base controllers. This is also the transport that Core uses internally (for example the event bus posts through it). Full details are in [the API chapter](api/) and in `Core/src/PBXCoreREST/AGENTS.md`.

### AdminCabinet — the web administration MVC

`MikoPBX\AdminCabinet\` (`Core/src/AdminCabinet/`) is the operator web UI: a Phalcon MVC application with Volt templates, Fomantic UI, and jQuery + ES6 on the front end. One controller per section, all extending `MikoPBX\AdminCabinet\Controllers\BaseController` (which extends `Phalcon\Mvc\Controller`); the module is wired through a Phalcon `ModuleDefinitionInterface`. Your module's settings page plugs into this MVC — see [the admin interface chapter](admin-interface.md).

### Modules — the extension framework

`MikoPBX\Modules\` (`Core/src/Modules/`) is where your module attaches to the core. The three pillars are:

* **`PbxExtensionBase`** — `Core/src/Modules/PbxExtensionBase.php`, an `abstract class … extends Injectable`. It gives every module class the DI container, a logger, and access to module metadata. `BlackListMain` ultimately builds on this base.
*   **Config interfaces** — `Core/src/Modules/Config/`. The aggregate base class `MikoPBX\Modules\Config\ConfigClass` extends the Asterisk generator base and implements the four module-facing interfaces:

    <pre class="language-php" data-title="Core/src/Modules/Config/ConfigClass.php"><code class="lang-php">abstract class ConfigClass extends AsteriskConfigClass implements
        SystemConfigInterface,
        RestAPIConfigInterface,
        WebUIConfigInterface,
        AsteriskConfigInterface
    {
        // The module hook applying priority
        protected int $priority = 10000;
    }
    </code></pre>

    `BlackListConf` extends `ConfigClass`; by overriding the interface methods it hooks into Asterisk config generation (`AsteriskConfigInterface`), the boot sequence (`SystemConfigInterface`), the REST layer (`RestAPIConfigInterface`), and the web UI (`WebUIConfigInterface`). The `$priority` property orders your module relative to others — `PBXConfModulesProvider` sorts enabled modules by it before invoking hooks.
* **`Setup/`** — install/uninstall lifecycle. `BlackListConf`'s installer extends `MikoPBX\Modules\Setup\PbxExtensionSetupBase` (which implements `PbxExtensionSetupInterface`) and provides `installModule()`, `installFiles()`, `installDB()`, `checkCompatibility()`, and `uninstallModule(bool $keepSettings = false)`.

{% hint style="success" %}
The smallest end-to-end modules that exercise this framework are shipped in the tree. See the working examples in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` (admin UI + model), `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/` (REST v3 controller), and `Extensions/EXAMPLES/AMI/ModuleExampleAmi/` (AMI integration).
{% endhint %}

### Service — the entry point

`MikoPBX\Service\` (`Core/src/Service/`) holds `Main` (`Core/src/Service/Main.php`, the application bootstrap / entry point) and `License` (`Core/src/Service/License.php`, license validation, exposed through the DI as the `license` service via `MarketPlaceProvider`).

## Storage: SQLite + Redis

MikoPBX deliberately keeps state in process-local stores so the appliance has no external database dependency.

**SQLite** holds all persistent configuration. There are several distinct database connections registered as DI services (`Core/src/Common/Providers/`):

| DI service             | Database               | Purpose                                                |
| ---------------------- | ---------------------- | ------------------------------------------------------ |
| `db`                   | `mikopbx.db`           | Main configuration — almost every model                |
| `dbCDR`                | `cdr.db`               | Call Detail Records (separate for performance)         |
| `dbRecordingStorage`   | `recording_storage.db` | Recording file location index                          |
| `{moduleId}_module_db` | per-module SQLite      | Module-private tables (`ModulesDBConnectionsProvider`) |

All connections share the PRAGMA tuning from `DatabaseProviderBase`: `journal_mode = WAL`, `busy_timeout = 5000`, `synchronous = NORMAL`, `cache_size = -10000`, `temp_store = MEMORY`.

**Redis** provides cache and inter-worker IPC, partitioned by logical DB:

| Redis DB | DI service       | Purpose                             |
| -------- | ---------------- | ----------------------------------- |
| 1        | `redis`          | Worker IPC / general                |
| 2        | `modelsMetadata` | Model schema/metadata cache         |
| 4        | `managedCache`   | Application cache (1h TTL)          |
| 5        | `session`        | Sessions (deprecated — auth is JWT) |

{% hint style="info" %}
Model reads are transparently cached through `ManagedCacheProvider` (`managedCache`, Redis DB 4); model metadata is read with the `Phalcon\Mvc\Model\MetaData\Redis` adapter (`modelsMetadata`, Redis DB 2). You generally do not touch the cache directly — saving a model invalidates the right keys for you.
{% endhint %}

## Core data-model graph

Every model extends `MikoPBX\Common\Models\ModelsBase` (`Core/src/Common/Models/ModelsBase.php`), which itself extends `Phalcon\Mvc\Model`. The base class enables snapshot tracking (`keepSnapshots(true)`), pulls in `RecordRepresentationTrait` for human-readable display, and supplies cache-key helpers. Each model maps to table `m_<ClassName>` on the `db` connection unless overridden (CDR and recording models are the exceptions noted above).

### Extensions — the polymorphic hub

`Extensions` is the centre of the graph. Every dialable object — a phone, a queue, a conference, an IVR — owns one `Extensions` row, and its `type` column discriminates what kind of entity it points to:

{% code title="Core/src/Common/Models/Extensions.php" %}
```php
public const string TYPE_SIP = 'SIP';
public const string TYPE_EXTERNAL = 'EXTERNAL';
public const string TYPE_QUEUE = 'QUEUE';
public const string TYPE_CONFERENCE = 'CONFERENCE';
public const string TYPE_DIALPLAN_APPLICATION = 'DIALPLAN APPLICATION';
public const string TYPE_IVR_MENU = 'IVR MENU';
public const string TYPE_MODULES = 'MODULES';
public const string TYPE_SYSTEM = 'SYSTEM';
public const string TYPE_PARKING = 'PARKING';
```
{% endcode %}

```
Users ──hasMany──> Extensions <──hasOne── Sip
                        │                   │
                        ├──> CallQueues     └──> SipHosts
                        ├──> ConferenceRooms
                        └──> IvrMenu ──hasMany──> IvrMenuActions

Providers ──hasMany──> IncomingRoutingTable <──belongsTo── Extensions
    └──hasMany──> OutgoingRoutingTable
```

Always branch on the constant, never on a string literal:

```php
$ext = Extensions::findFirstByNumber('100');

if ($ext !== null && $ext->type === Extensions::TYPE_SIP) {
    $sip = $ext->Sip;
}
```

Deleting an `Extensions` row cascades to its related records via `beforeDelete`, so you rarely delete child rows by hand.

### Functional clusters

The remaining \~40 models group into recognisable clusters (the classes themselves are the inventory — `Core/src/Common/Models/*.php`; the conventions they share are in `Core/src/Common/Models/AGENTS.md`):

* **Telephony** — `Users`, `Sip`, `Iax`, `Providers`, `SipHosts`, `CallQueues`, `CallQueueMembers`, `ConferenceRooms`, `IvrMenu`, `IvrMenuActions`, `ExternalPhones`, plus the routing tables `IncomingRoutingTable` / `OutgoingRoutingTable` and schedules `OutWorkTimes` / `OutWorkTimesRouts`.
* **Network & security** — `LanInterfaces`, `NetworkFilters`, `NetworkStaticRoutes`, `FirewallRules`, `Fail2BanRules`.
* **Auth & API** — `ApiKeys` (REST Bearer tokens; `generateApiKey()` mints a 64-char hex key), `UserPasskeys` (WebAuthn/FIDO2), `AsteriskManagerUsers` (AMI), `AsteriskRestUsers` (ARI).
* **System & config** — `PbxSettings` (key-value settings, with constants in `PbxSettingsConstants` and defaults in `PbxSettingsDefaultValuesTrait`), `Storage`, `StorageSettings`, `RecordingStorage`, `SoundFiles`, `Codecs`, `CustomFiles`, `DialplanApplications`, and `PbxExtensionModules` (the registry of installed modules; its `module_type` column is an open enum — `general`|`languagepack`|`security`|`cti`|`utility`|`call_feature`|`ai`|… — used for marketplace categorisation).
* **CDR** — `CallDetailRecords` (table `cdr_general`) and `CallDetailRecordsTmp` (table `cdr`) share `CallDetailRecordsBase` and live on the dedicated `dbCDR` connection.

{% hint style="warning" %}
Always check the result of `save()` and read `getMessages()` on failure — validation errors are returned, not thrown:

```php
if (!$model->save()) {
    foreach ($model->getMessages() as $message) {
        // handle $message->getMessage()
    }
}
```
{% endhint %}

A real module that defines its own model exactly like the core models do — extending `ModelsBase`, mapping to an `m_…` table — is in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`. Your `BlackListNumbers` model on `m_BlackListNumbers` follows the same shape. The full module-author walkthrough of models and relationships is in [module-developement/data-model.md](../module-developement/data-model.md).

## Where to go next

* [admin-interface.md](admin-interface.md) — the AdminCabinet MVC and how to add a settings page.
* [api/README.md](api/) — REST API v3 controllers, attributes, and the async queue.
* [module-developement/data-model.md](../module-developement/data-model.md) — the full model/relationship reference for module authors.
