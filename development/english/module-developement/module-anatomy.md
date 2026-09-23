---
description: >-
  The complete directory and file map of a MikoPBX module — every folder, the
  class it holds, and how the App / Lib / Worker tiers fit together.
---

# Module anatomy

A MikoPBX module is a self-contained Phalcon application that the Core mounts at runtime. It ships its own MVC web layer, its own background workers, its own isolated database, and its own translations. Once installed it lives under the Core's modules directory — `<mountpoint>/mikopbx/custom_modules/<ModuleUniqueID>/`, in practice `/storage/usbdisk1/mikopbx/custom_modules/<ModuleUniqueID>/` (the `core.modulesDir` setting, see `Core/src/Core/System/Directories.php`) — and is registered in the Core's dependency-injection container and router.

This page is the map. It walks the scaffold produced by the **ModuleTemplate** (`Extensions/ModuleTemplate/`) directory by directory, tells you which class lives where, and explains the three-tier split that keeps the web UI, the orchestration logic, and the long-running daemons cleanly separated.

Throughout the guide we thread a single fictional running example — a number-blacklisting module called **ModuleBlackList**:

| Concept              | ModuleTemplate (scaffold)   | ModuleBlackList (our example) |
| -------------------- | --------------------------- | ----------------------------- |
| Module unique ID     | `ModuleTemplate`            | `ModuleBlackList`             |
| Config / hooks class | `Lib/TemplateConf.php`      | `Lib/BlackListConf.php`       |
| Main logic class     | `Lib/TemplateMain.php`      | `Lib/BlackListMain.php`       |
| Model                | `Models/ModuleTemplate.php` | `Models/BlackListNumbers.php` |
| Database table       | `m_ModuleTemplate`          | `m_BlackListNumbers`          |
| Front-end script     | `module-template-modify.js` | `module-black-list-modify.js` |

Every pattern below is anchored to a real, working module you can read:

{% hint style="info" %}
**See the working examples in:**

* `Extensions/ModuleTemplate/` — the canonical scaffold every new module is cloned from.
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` — a fuller form-driven module with the same layout.

Read these alongside this page. Every file path, class name, and method shown here was taken from those sources.
{% endhint %}

## The top-level layout

A freshly scaffolded module has this shape (from `Extensions/ModuleTemplate/`):

```
ModuleBlackList/
├── module.json                 # Module manifest (ID, developer, min PBX version, release flags)
├── composer.json               # PSR-4 autoload + mikopbx/core dependency
├── App/                        # Phalcon MVC web layer (AdminCabinet integration)
│   ├── Module.php              # ModuleDefinitionInterface — DI entry point
│   ├── Controllers/            # Controllers extending AdminCabinet BaseController
│   ├── Forms/                  # Forms extending AdminCabinet BaseForm
│   ├── Views/                  # Volt templates (one folder per controller)
│   └── Providers/              # Optional ServiceProviderInterface providers (.gitkeep)
├── Lib/                        # Business logic + background tier
│   ├── BlackListConf.php       # ConfigClass — the hooks hub (4 interfaces)
│   ├── BlackListMain.php       # PbxExtensionBase — shared logic
│   ├── WorkerBlackListMain.php # Beanstalk-driven daemon
│   └── WorkerBlackListAMI.php  # Asterisk AMI-driven daemon
├── Models/                     # Phalcon models extending ModulesModelsBase
│   └── BlackListNumbers.php
├── Setup/
│   └── PbxExtensionSetup.php   # Installer hooks (extends PbxExtensionSetupBase)
├── Messages/                   # 28 locale files + languages.php registry
│   ├── en.php
│   ├── ru.php
│   └── languages.php
├── public/assets/              # Front-end assets served to the browser
│   ├── js/src/                 # ES6 source — compiled down to js/
│   ├── js/                     # Compiled, browser-ready scripts
│   ├── css/
│   └── img/logo.svg            # Module logo
├── agi-bin/                    # Optional AGI scripts (.gitkeep extension point)
├── bin/                        # Optional CLI binaries (.gitkeep extension point)
└── db/                         # Runtime home of the module's SQLite DB (module.db)
```

The `.gitkeep` files in `agi-bin/`, `bin/`, and `App/Providers/` are placeholders that keep otherwise-empty directories in version control. They mark **extension points**: the Core knows to look in these folders, so you drop files in when you need them and leave them empty otherwise. `db/` is different — it ships empty but is **not** optional, see [below](module-anatomy.md#extension-point-directories-agi-bin-bin-and-the-db-directory).

{% hint style="info" %}
For a step-by-step on cloning the scaffold and wiring up your IDE, see [How to start](template-module-structure.md).
{% endhint %}

## The three-tier split

Before the file-by-file tour, internalize the architecture. A module is split into three cooperating tiers, each with a distinct lifetime and a distinct execution context:

```
┌──────────────────────────────────────────────────────────────┐
│  App/  — WEB TIER (per-request, runs inside AdminCabinet)      │
│  Controllers, Forms, Views. Lives only for one HTTP request.   │
│  Talks to Models. Renders Volt. No long-running state.         │
└───────────────────────────┬──────────────────────────────────┘
                            │  reads/writes Models, calls Main
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  Lib/{Feature}Conf.php  — CONFIG / HOOKS HUB                   │
│  Extends ConfigClass. Implements 4 interfaces. The Core calls  │
│  its hook methods on system events (reload, enable, REST, menu)│
│  Lib/{Feature}Main.php  — SHARED LOGIC (PbxExtensionBase)      │
│  Reusable orchestration: start/stop workers, health checks.    │
└───────────────────────────┬──────────────────────────────────┘
                            │  spawns / restarts via Processes
                            ▼
┌──────────────────────────────────────────────────────────────┐
│  Lib/Worker*.php  — DAEMON TIER (long-running PHP processes)   │
│  Extend WorkerBase. Self-exec bootstrap at file bottom.        │
│  Listen on Beanstalk queues or Asterisk AMI. Run forever.      │
└──────────────────────────────────────────────────────────────┘
```

* The **web tier** (`App/`) only exists during an HTTP request to the admin cabinet. It is stateless between requests.
* The **config tier** (`Lib/{Feature}Conf.php` + `Lib/{Feature}Main.php`) is the bridge. The `Conf` class is where the Core reaches _into_ your module via hooks; the `Main` class holds logic shared between the web tier and the daemons.
* The **daemon tier** (`Lib/Worker*.php`) is the only part that runs continuously. Workers are separate OS processes started and supervised by the Core's safe-script supervisor.

This separation is why `BlackListMain` exists as its own class rather than living inside the controller or the worker: both the web `saveAction()` (to restart services after a config change) and the worker `start()` need the same orchestration logic, so it is factored into a shared `PbxExtensionBase` subclass.

## `module.json` — the manifest

Every module starts with a manifest. A manifest for a module targeting the current baseline looks like this (the scaffold's `Extensions/ModuleTemplate/module.json` has the same shape):

{% code title="module.json" %}
```json
{
  "developer": "MIKO",
  "moduleUniqueID": "ModuleBlackList",
  "support_email": "help@miko.ru",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2025.1.1",
  "release_settings": {
    "publish_release": true,
    "changelog_enabled": true,
    "create_github_release": true
  }
}
```
{% endcode %}

* `moduleUniqueID` is the single most important value. It is the namespace segment (`Modules\ModuleBlackList\...`), the on-disk directory name, the DB service prefix, and the key the Core uses everywhere to identify your module.
* `version` is `%ModuleVersion%` in the repo — a placeholder that the build pipeline substitutes with the real release tag.
* `min_pbx_version` declares the minimum compatible Core. For a module targeting the current baseline this is `2025.1.1`.
* `release_settings` drives the automated release pipeline (changelog generation, GitHub release creation).

## `App/` — the Phalcon MVC web layer

`App/` is your module's slice of the AdminCabinet. It is a standard Phalcon MVC application, mounted by the Core under your module's route prefix.

### `App/Module.php` — the DI entry point

This is the first class the Core touches when it loads your web tier. It implements Phalcon's `ModuleDefinitionInterface` and its job is to register module-scoped services into the DI container — most importantly the dispatcher, which tells Phalcon where to find your controllers.

From `Extensions/ModuleTemplate/App/Module.php`:

{% code title="App/Module.php" %}
```php
<?php

namespace Modules\ModuleBlackList\App;

use Phalcon\Di\DiInterface;
use Phalcon\Events\Manager;
use Phalcon\Mvc\Dispatcher;
use Phalcon\Mvc\ModuleDefinitionInterface;

class Module implements ModuleDefinitionInterface
{
    /**
     * Registers an autoloader related to the module
     */
    public function registerAutoloaders(DiInterface $container = null)
    {
    }

    /**
     * Registers services related to the module
     */
    public function registerServices(DiInterface $container)
    {
        $container->set('dispatcher', function () {
            $dispatcher = new Dispatcher();

            $eventManager = new Manager();

            $dispatcher->setEventsManager($eventManager);
            $dispatcher->setDefaultNamespace('Modules\ModuleBlackList\App\Controllers\\');
            return $dispatcher;
        });
    }
}
```
{% endcode %}

The critical line is `setDefaultNamespace(...)` — change `ModuleTemplate` to your module's unique ID and the dispatcher resolves controller class names against the right namespace. `registerAutoloaders()` is typically left empty because Composer's PSR-4 autoloader (declared in `composer.json`) already covers the module.

### `App/Controllers/` — request handlers

Controllers extend `MikoPBX\AdminCabinet\Controllers\BaseController`. They handle the standard CRUD actions of an admin page (`indexAction`, `modifyAction`, `saveAction`, `deleteAction`), attach the module's compiled JS/CSS to the page, and pick the Volt view to render.

From `Extensions/ModuleTemplate/App/Controllers/ModuleTemplateController.php`:

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
<?php

namespace Modules\ModuleBlackList\App\Controllers;

use MikoPBX\AdminCabinet\Controllers\BaseController;
use MikoPBX\AdminCabinet\Providers\AssetProvider;
use MikoPBX\Modules\PbxExtensionUtils;
use Modules\ModuleBlackList\App\Forms\ModuleBlackListForm;
use Modules\ModuleBlackList\Models\BlackListNumbers;

class ModuleBlackListController extends BaseController
{
    private string $moduleUniqueID = 'ModuleBlackList';
    private string $moduleDir;

    public function initialize(): void
    {
        $this->moduleDir = PbxExtensionUtils::getModuleDir($this->moduleUniqueID);
        $this->view->logoImagePath = $this->url->get() . 'assets/img/cache/' . $this->moduleUniqueID . '/logo.svg';
        $this->view->submitMode = null;
        parent::initialize();
    }

    public function modifyAction(): void
    {
        $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
        $footerCollectionJS
            ->addJs('js/pbx/main/form.js', true)
            ->addJs('js/cache/' . $this->moduleUniqueID . '/module-black-list-modify.js', true);

        $settings = BlackListNumbers::findFirst();
        if ($settings === null) {
            $settings = new BlackListNumbers();
        }

        $this->view->form = new ModuleBlackListForm($settings);
        $this->view->pick('Modules/' . $this->moduleUniqueID . '/ModuleBlackList/modify');
    }
}
```
{% endcode %}

Note the conventions verified in the scaffold:

* Assets are added through `$this->assets->collection(AssetProvider::HEADER_CSS)` / `AssetProvider::FOOTER_JS` and reference the **compiled** asset under `js/cache/<ModuleUniqueID>/...` (the Core symlinks/caches your `public/assets/` into the cabinet's served path).
* `$this->view->pick('Modules/<ModuleUniqueID>/<Controller>/<view>')` selects the Volt template.
* Persistence uses the inherited helpers `saveEntity($record)` and `deleteEntity($record, $redirectUrl)` from `BaseController`.

The scaffold also ships a second controller, `AdditionalPageController.php`, demonstrating a module page that is not the main settings form.

### `App/Forms/` — form definitions

Forms extend `MikoPBX\AdminCabinet\Forms\BaseForm` and declare the fields rendered on the modify page using standard Phalcon form elements. From `Extensions/ModuleTemplate/App/Forms/ModuleTemplateForm.php`:

{% code title="App/Forms/ModuleBlackListForm.php" %}
```php
<?php

namespace Modules\ModuleBlackList\App\Forms;

use MikoPBX\AdminCabinet\Forms\BaseForm;
use Phalcon\Forms\Element\Text;
use Phalcon\Forms\Element\Hidden;

class ModuleBlackListForm extends BaseForm
{
    public function initialize($entity = null, $options = null): void
    {
        parent::initialize($entity, $options);

        $this->add(new Hidden('id', ['value' => $entity->id]));
        $this->add(new Text('number'));
        $this->addCheckBox('enabled', intval($entity->enabled) === 1);
    }
}
```
{% endcode %}

`BaseForm` provides convenience helpers such as `addTextArea(...)` and `addCheckBox(...)` on top of Phalcon's element classes (`Text`, `Numeric`, `Password`, `Check`, `Select`, `Hidden`).

{% hint style="warning" %}
Always call `parent::initialize($entity, $options)` **first**. `BaseForm::initialize()` is the only place that fires the `WebUIConfigInterface::ON_BEFORE_FORM_INITIALIZE` hook (`Core/src/AdminCabinet/Forms/BaseForm.php`), which is how other modules inject or disable fields on your form. Skip it and that hook silently never runs. The scaffold's own `ModuleTemplateForm.php` predates this rule — follow `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Forms/ModuleExampleFormForm.php` instead.
{% endhint %}

### `App/Views/` — Volt templates

Views are Phalcon **Volt** templates (`.volt`), organised one folder per controller. The scaffold has `App/Views/ModuleTemplate/index.volt`, `App/Views/ModuleTemplate/modify.volt`, and `App/Views/AdditionalPage/index.volt`. The folder name matches the controller name and is what `$this->view->pick('Modules/<ID>/<Controller>/<view>')` resolves against.

### `App/Providers/` — optional service providers

`App/Providers/` ships only a `.gitkeep` in the scaffold. It is an extension point for classes implementing Phalcon's `ServiceProviderInterface` when your module needs to register additional DI services beyond the dispatcher set up in `Module.php`. Most modules leave it empty.

## `Lib/` — logic and background tier

`Lib/` is the heart of the module. It contains the config/hooks hub, the shared logic class, and the worker daemons.

### `Lib/{Feature}Conf.php` — the ConfigClass hooks hub

This is the single most important class in the module. `BlackListConf` extends `MikoPBX\Modules\Config\ConfigClass`. In the Core, `ConfigClass` is declared as:

{% code title="Core/src/Modules/Config/ConfigClass.php" %}
```php
abstract class ConfigClass extends AsteriskConfigClass implements
    SystemConfigInterface,
    RestAPIConfigInterface,
    WebUIConfigInterface,
    AsteriskConfigInterface
```
{% endcode %}

So a single `Conf` class is the hub for **four configuration interfaces**:

* `SystemConfigInterface` — workers, cron, nginx locations, fail2ban, firewall, enable/disable hooks.
* `RestAPIConfigInterface` — REST callbacks, additional routes, pre/post-route hooks.
* `WebUIConfigInterface` — auth, ACL, header menu, routes, asset injection, form hooks.
* `AsteriskConfigInterface` — generating Asterisk dialplan / config fragments. (`ConfigClass` _extends_ `AsteriskConfigClass`, which is where the Asterisk-config default behaviour lives, and also re-declares the interface so the contract is explicit.)

`ConfigClass` provides safe default stubs for every interface method, so your module **overrides only the hooks it cares about**. The Core scans every enabled module's `Conf` class and calls the relevant hook on system events.

From `Extensions/ModuleTemplate/Lib/TemplateConf.php`, three representative hooks:

{% code title="Lib/BlackListConf.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Common\Models\PbxSettings;
use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;
use MikoPBX\Modules\Config\ConfigClass;
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;

class BlackListConf extends ConfigClass
{
    /**
     * React to changes in the main MikoPBX database.
     */
    public function modelsEventChangeData($data): void
    {
        if (
            $data['model'] === PbxSettings::class
            && $data['recordId'] === 'PBXLanguage'
        ) {
            $main = new BlackListMain();
            $main->startAllServices(true);
        }
    }

    /**
     * Declare the workers the safe-script supervisor must keep alive.
     */
    public function getModuleWorkers(): array
    {
        return [
            [
                'type'   => WorkerSafeScriptsCore::CHECK_BY_BEANSTALK,
                'worker' => WorkerBlackListMain::class,
            ],
            [
                'type'   => WorkerSafeScriptsCore::CHECK_BY_AMI,
                'worker' => WorkerBlackListAMI::class,
            ],
        ];
    }

    /**
     * Handle REST API calls routed to this module under root rights.
     */
    public function moduleRestAPICallback(array $request): PBXApiResult
    {
        $res = new PBXApiResult();
        $res->processor = __METHOD__;
        switch (strtoupper($request['action'])) {
            case 'CHECK':
                $res = (new BlackListMain())->checkModuleWorkProperly();
                break;
            case 'RELOAD':
                (new BlackListMain())->startAllServices(true);
                $res->success = true;
                break;
            default:
                $res->success = false;
                $res->messages['error'][] = 'API action not found in ' . __METHOD__;
        }
        return $res;
    }

    /**
     * Add this module's entry to the AdminCabinet header menu.
     */
    public function onBeforeHeaderMenuShow(array &$menuItems): void
    {
        $menuItems['module_black_list_AdditionalMenuItem'] = [
            'caption'   => 'module_black_list_AdditionalMenuItem',
            'iconclass' => '',
            'submenu'   => [
                '/module-black-list/additional-page' => [
                    'caption'   => 'module_black_list_AdditionalSubMenuItem',
                    'iconclass' => 'gear',
                    'action'    => 'index',
                    'param'     => '',
                    'style'     => '',
                ],
            ],
        ];
    }
}
```
{% endcode %}

{% hint style="info" %}
The full hook catalog — every method on all four interfaces, their signatures, and when the Core calls them — is documented in [The module configuration class](module-class.md). The constants themselves are declared in `Core/src/Modules/Config/*Interface.php`; `Core/src/Modules/AGENTS.md` explains how the Core dispatches them.
{% endhint %}

### `Lib/{Feature}Main.php` — shared logic

`BlackListMain` extends `MikoPBX\Modules\PbxExtensionBase`. `PbxExtensionBase` is an `Injectable` base that hands you a module-scoped `$this->logger` and `$this->moduleUniqueId`. This class holds logic shared between the web tier and the daemons — typically worker orchestration and health checks.

From `Extensions/ModuleTemplate/Lib/TemplateMain.php`:

{% code title="Lib/BlackListMain.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Core\System\Processes;
use MikoPBX\Core\Workers\Cron\WorkerSafeScriptsCore;
use MikoPBX\Modules\PbxExtensionBase;
use MikoPBX\Modules\PbxExtensionUtils;
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;

class BlackListMain extends PbxExtensionBase
{
    public function checkModuleWorkProperly(): PBXApiResult
    {
        $res = new PBXApiResult();
        $res->processor = __METHOD__;
        $res->success = true;
        return $res;
    }

    public function startAllServices(bool $restart = false): void
    {
        if (!PbxExtensionUtils::isEnabled($this->moduleUniqueId)) {
            return;
        }
        $workersToRestart = (new BlackListConf())->getModuleWorkers();

        if ($restart) {
            foreach ($workersToRestart as $moduleWorker) {
                Processes::processPHPWorker($moduleWorker['worker']);
            }
        } else {
            $safeScript = new WorkerSafeScriptsCore();
            foreach ($workersToRestart as $moduleWorker) {
                if ($moduleWorker['type'] === WorkerSafeScriptsCore::CHECK_BY_AMI) {
                    $safeScript->checkWorkerAMI($moduleWorker['worker']);
                } else {
                    $safeScript->checkWorkerBeanstalk($moduleWorker['worker']);
                }
            }
        }
    }
}
```
{% endcode %}

Two things to internalize here:

* `startAllServices()` is the join point between the config tier and the daemon tier: it reads the worker list from `BlackListConf::getModuleWorkers()` and either force-restarts each worker (`Processes::processPHPWorker(...)`) or asks the supervisor to ensure it is alive (`checkWorkerAMI` / `checkWorkerBeanstalk`).
* `PbxExtensionUtils::isEnabled($this->moduleUniqueId)` is the gate — disabled modules never start their workers.

### `Lib/Worker*.php` — the daemons

Workers are long-running PHP processes. They extend `MikoPBX\Core\Workers\WorkerBase` and are started and kept alive by the Core's safe-script supervisor (`WorkerSafeScriptsCore`). The scaffold ships two, illustrating the two supervision modes:

* **`WorkerTemplateMain.php`** — a **Beanstalk** worker. It subscribes to a Beanstalk queue and processes messages pushed to it. Supervised via `CHECK_BY_BEANSTALK`.
* **`WorkerTemplateAMI.php`** — an **AMI** worker. It connects to the Asterisk Manager Interface and reacts to telephony `UserEvent`s. Supervised via `CHECK_BY_AMI`.

Every worker file ends with a **self-exec bootstrap** block — the code that lets the file be launched directly as a PHP CLI process. From `Extensions/ModuleTemplate/Lib/WorkerTemplateMain.php`:

{% code title="Lib/WorkerBlackListMain.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\Common\Handlers\CriticalErrorsHandler;
use MikoPBX\Core\System\BeanstalkClient;
use MikoPBX\Core\Workers\WorkerBase;

require_once 'Globals.php';

class WorkerBlackListMain extends WorkerBase
{
    protected BlackListMain $blackListMain;

    public function start(array $argv): void
    {
        $this->blackListMain = new BlackListMain();
        $client = new BeanstalkClient(self::class);
        $client->subscribe($this->makePingTubeName(self::class), [$this, 'pingCallBack']);
        $client->subscribe(self::class, [$this, 'beanstalkCallback']);
        while (true) {
            $client->wait();
        }
    }

    public function beanstalkCallback(BeanstalkClient $message): void
    {
        $received = json_decode($message->getBody(), true);
        $this->blackListMain->processBeanstalkMessage($received);
    }
}

// ----- self-exec bootstrap -----
$workerClassname = WorkerBlackListMain::class;
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

The bootstrap block is verified in both scaffold workers. Note:

* `require_once 'Globals.php';` pulls in the Core's autoloader/environment so the worker can run standalone.
* The `if (isset($argv) && count($argv) > 1)` guard means the bootstrap only fires when the file is executed directly (the supervisor passes arguments), not when it is merely autoloaded as a class.
* `cli_set_process_title($workerClassname)` names the process so the supervisor and `ps` can find it.
* `$this->makePingTubeName(self::class)` + `pingCallBack` form the health-check channel the supervisor uses to confirm the worker is alive.

The AMI variant (`WorkerTemplateAMI.php`) follows the identical bootstrap pattern but its `start()` connects to Asterisk via `Util::getAstManager()`, installs event filters, and loops on `waitUserEvent(true)`.

## `Models/` — the data layer

Models extend `MikoPBX\Modules\Models\ModulesModelsBase`, which automatically connects the model to your module's **isolated database** based on the namespace. You never share tables with the Core's main database. The database file is `<moduleDir>/db/module.db`; the DI connection service that points at it is named `<moduleUniqueId>_module_db` (`ModulesModelsBase::getConnectionServiceName()`).

Fields are declared as public properties annotated with Phalcon `@Column` doc-block annotations, and the physical table name is bound in `initialize()` via `setSource('m_<Entity>')`. From `Extensions/ModuleTemplate/Models/ModuleTemplate.php`:

{% code title="Models/BlackListNumbers.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Models;

use MikoPBX\Modules\Models\ModulesModelsBase;

class BlackListNumbers extends ModulesModelsBase
{
    /**
     * @Primary
     * @Identity
     * @Column(type="integer", nullable=false)
     */
    public $id;

    /**
     * @Column(type="string", nullable=true)
     */
    public $number;

    /**
     * @Column(type="integer", default="1", nullable=true)
     */
    public $enabled;

    public function initialize(): void
    {
        $this->setSource('m_BlackListNumbers');
        parent::initialize();
    }
}
```
{% endcode %}

{% hint style="warning" %}
Always call `parent::initialize()` at the **end** of your own `initialize()`. `ModulesModelsBase::initialize()` is what wires up the module-specific database connection; skipping it leaves the model unconnected.
{% endhint %}

The table-naming convention is `m_<EntityName>` — the `m_` prefix marks module tables. The scaffold model also demonstrates **relations** to Core models (`hasOne` to `Providers`) and the `getDynamicRelations()` hook for declaring relations on a Core model from the module side. Those are covered in detail in [The module data model](data-model.md).

## `Setup/PbxExtensionSetup.php` — the installer

`Setup/PbxExtensionSetup.php` extends `MikoPBX\Modules\Setup\PbxExtensionSetupBase` and drives install / enable / disable / uninstall. In the scaffold it is **deliberately empty**:

{% code title="Setup/PbxExtensionSetup.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Setup;

use MikoPBX\Modules\Setup\PbxExtensionSetupBase;

class PbxExtensionSetup extends PbxExtensionSetupBase
{
}
```
{% endcode %}

The base class already implements the full lifecycle — installing files, creating the module database from your models, registering the module from `module.json`, fixing file rights, and the enable/disable firewall and sound steps. You only override a method here when you need custom install-time behaviour (seeding data, custom DB migrations, extra cleanup on uninstall).

{% hint style="info" %}
The complete install/enable/disable/uninstall sequence and which methods you can override is documented in [The module installer](module-installer.md).
{% endhint %}

## `Messages/` — translations

A module is fully localised. `Messages/` holds **28 locale files** — one PHP file per supported language (`en.php`, `ru.php`, `de.php`, `fr.php`, `zh_Hans.php`, `pt_BR.php`, and so on) — plus a `languages.php` registry that lists every translation key the module uses.

Each locale file returns an associative array mapping a translation key to its localised string. `languages.php` from `Extensions/ModuleTemplate/Messages/languages.php` is the master key list (values empty, to be filled per locale):

{% code title="Messages/languages.php" %}
```php
<?php
return [
    'repModuleBlackList'                 => '',
    'BreadcrumbModuleBlackList'          => '',
    'SubHeaderModuleBlackList'           => '',
    'module_black_list_AddNewRecord'     => '',
    'module_black_list_AdditionalMenuItem'    => '',
    'module_black_list_AdditionalSubMenuItem' => '',
    // ... one entry per translatable string
];
```
{% endcode %}

The keys you reference from PHP (`onBeforeHeaderMenuShow` captions, validation messages) and from Volt templates resolve against these files. The translation keys themselves — not human text — are what you write in code; the Core's translation layer swaps in the active language at render time.

## `public/assets/` — front-end resources

Everything the browser loads lives here:

```
public/assets/
├── js/src/                       # ES6 source you edit
│   ├── module-black-list-index.js
│   └── module-black-list-modify.js
├── js/                           # Compiled, browser-ready output
│   ├── module-black-list-index.js
│   └── module-black-list-modify.js
├── css/
│   └── module-black-list-index.css
└── img/
    └── logo.svg                  # Module logo shown in the cabinet
```

* `js/src/` holds the **ES6 source** you author. It is compiled (Babel, airbnb preset) down into `js/`, which holds the **browser-ready** scripts the controller actually loads (`->addJs('js/cache/<ModuleUniqueID>/module-black-list-modify.js')`).
* `img/logo.svg` is the module logo. The controller exposes it to views as `assets/img/cache/<ModuleUniqueID>/logo.svg`.

The Core caches/symlinks `public/assets/` into the cabinet's served `cache/<ModuleUniqueID>/` path on enable, which is why the controller paths contain `cache/<ModuleUniqueID>/`.

## Extension-point directories: `agi-bin/`, `bin/`, and the `db/` directory

All three ship containing only a `.gitkeep`, but they do not play the same role.

Two are genuine **extension points** — recognised locations the Core knows to look in, which you fill only if your feature needs them:

* **`agi-bin/`** — AGI scripts (Asterisk Gateway Interface). When your module needs Asterisk to call into PHP during a call, drop the AGI script here; the Core symlinks it into the AGI path on enable (`PbxExtensionUtils::createAgiBinSymlinks()`).
* **`bin/`** — module CLI binaries or helper executables.

Leave those two as empty `.gitkeep` directories until you actually need them.

**`db/` is not optional.** It is the runtime home of your module's SQLite database: `ModulesDBConnectionsProvider` points the module's DB connection at `<moduleDir>/db/module.db` and creates the file there on first use (`Core/src/Common/Providers/ModulesDBConnectionsProvider.php`). It is also the directory the installer preserves — `unInstallFiles($keepSettings = true)` copies `db/` to `<modulesDir>/Backup/<ModuleUniqueID>` and `installFiles()` copies it back on the next install, which is what makes an upgrade keep user data. Ship it empty (that is what the `.gitkeep` is for) and let the tables be generated from your models, but never delete the directory.

## How it all connects at runtime

Putting the tiers together, here is the lifecycle of a single config change in **ModuleBlackList**:

1. An admin opens `/module-black-list/modify`. The Core routes to `ModuleBlackListController::modifyAction()` (web tier), which builds a `ModuleBlackListForm` from a `BlackListNumbers` model and renders the Volt view.
2. The admin saves. `saveAction()` writes the `BlackListNumbers` record to the module's isolated DB, then calls `(new BlackListMain())->startAllServices(true)` to apply the change.
3. `BlackListMain::startAllServices()` reads the worker list from `BlackListConf::getModuleWorkers()` and restarts the daemons via `Processes::processPHPWorker(...)`.
4. The Core's `WorkerSafeScriptsCore` supervisor keeps `WorkerBlackListMain` and `WorkerBlackListAMI` alive, pinging them on their health tubes.
5. Independently, when the system reloads or a relevant Core record changes, the Core calls `BlackListConf::modelsEventChangeData()` and other hooks — the config tier reacting to system events.

That loop — web tier writes config → Main orchestrates → daemons run, all coordinated through the Conf hooks hub — is the shape of essentially every MikoPBX module.

## Where to go next

* [How to start](template-module-structure.md) — clone the scaffold and rename it for your module.
* [The module configuration class](module-class.md) — the full hook catalog of `ConfigClass`.
* [The module data model](data-model.md) — models, columns, relations, and the isolated DB.
* [The module installer](module-installer.md) — the install/enable/disable/uninstall lifecycle.
