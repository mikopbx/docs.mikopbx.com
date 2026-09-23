---
description: >-
  The PbxExtensionSetup lifecycle: install, setup and uninstall an extension
  module.
---

# Module installer class

Every MikoPBX module ships a setup class at `Setup/PbxExtensionSetup.php`. MikoPBX calls this class when the module is uploaded (install/upgrade) and when it is removed. The class implements the full install/uninstall lifecycle: compatibility checks, license activation, file copying and symlinking, database table creation, module registration and rollback.

The setup class extends the abstract base [`PbxExtensionSetupBase`](https://github.com/mikopbx/Core/blob/master/src/Modules/Setup/PbxExtensionSetupBase.php) (`MikoPBX\Modules\Setup\PbxExtensionSetupBase`), which implements the `MikoPBX\Modules\Setup\PbxExtensionSetupInterface` contract. The base is a **template-method** implementation: it provides a working default for every step, so a module's `Setup/PbxExtensionSetup.php` only overrides the methods it actually needs to customize.

{% hint style="info" %}
The minimal subclass is literally empty — it inherits every step from the base. See the working example in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Setup/PbxExtensionSetup.php`, whose entire body is `class PbxExtensionSetup extends PbxExtensionSetupBase {}`.
{% endhint %}

Throughout this page the running example is the fictional **ModuleBlackList** module (unique ID `ModuleBlackList`, model `BlackListNumbers`, table `m_BlackListNumbers`). Each pattern is anchored to a **real** example module by repo-relative path so you can read the source yourself.

## How MikoPBX invokes the setup class

There are two ways to install a module:

* from a locally uploaded ZIP archive (recommended during development),
* from the MIKO modules repository (the Marketplace).

In both cases MikoPBX unzips the module files into the modules directory and then instantiates the module's own `PbxExtensionSetup` class and calls `installModule()`:

{% code title="How the Core calls your setup class" %}
```php
$pbxExtensionSetupClass = "\\Modules\\{$moduleUniqueID}\\Setup\\PbxExtensionSetup";
$setup = new $pbxExtensionSetupClass($moduleUniqueID);
$result = $setup->installModule();
if (!$result) {
    // The UI shows the strings collected in $setup->getMessages()
    $errors = $setup->getMessages();
}
```
{% endcode %}

`installModule()` returns `bool`. On failure it stops at the first failing step and records a human-readable message; the REST layer surfaces those messages (`getMessages()`) to the user.

## Constructor and inherited properties

The base constructor (`__construct(string $moduleUniqueID)`) wires up the dependency-injected services and reads the module's [`module.json`](module-json.md), populating these `protected` properties for every step to use:

{% code title="Properties populated from module.json and the DI container" %}
```php
protected string $moduleUniqueID;        // unique ID, e.g. 'ModuleBlackList'
protected $version;                      // 'version' from module.json
protected string $min_pbx_version = '2024.2.3'; // 'min_pbx_version' from module.json
protected $developer;                    // 'developer'
protected $support_email;                // 'support_email'
protected $db;                           // Phalcon SQLite adapter (MainDatabaseProvider)
protected string $moduleDir;             // absolute path to the module folder
protected $config;                       // Phalcon\Config\Config (ConfigProvider)
protected array $messages;               // error / verbose messages
protected $license;                      // MarketPlaceProvider license worker
public $lic_product_id;                  // 'lic_product_id' from module.json (0 if absent)
public $lic_feature_id;                  // 'lic_feature_id' from module.json (0 if absent)
public array $wiki_links = [];           // 'wiki_links' from module.json
public string $module_type = 'general';  // 'module_type' from module.json
```
{% endcode %}

`PbxExtensionSetupBase` extends `Phalcon\Di\Injectable`, so it also resolves the `license`, `translation` and `config` services from the DI container via magic properties.

{% hint style="warning" %}
You usually should **not** override the constructor. If you must add a property, call `parent::__construct($moduleUniqueID)` first so the inherited properties are still initialized:

```php
class PbxExtensionSetup extends PbxExtensionSetupBase
{
    public function __construct(string $moduleUniqueID)
    {
        parent::__construct($moduleUniqueID);
        // ... your extra initialization ...
    }
}
```
{% endhint %}

## The install pipeline: `installModule()`

`installModule()` is the orchestrator. It is implemented once in the base and you normally **do not** override it — you override the individual steps it calls. The verified call order is:

```
checkCompatibility() → activateLicense() → installFiles() → installDB() → fixFilesRights()
```

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (installModule)" %}
```php
public function installModule(): bool
{
    try {
        if (!$this->checkCompatibility()) {
            return false;
        }
        if (!$this->activateLicense()) {
            $this->messages[] = $this->translation->_("ext_ErrorOnLicenseActivation");
            return false;
        }
        if (!$this->installFiles()) {
            $this->messages[] = $this->translation->_("ext_ErrorOnInstallFiles");
            return false;
        }
        if (!$this->installDB()) {
            $this->messages[] = $this->translation->_("ext_ErrorOnInstallDB");
            return false;
        }
        if (!$this->fixFilesRights()) {
            $this->messages[] = $this->translation->_("ext_ErrorOnAppliesFilesRights");
            return false;
        }

        // Recreate version hash for JS files and translations
        PBXConfModulesProvider::getVersionsHash(true);
    } catch (Throwable $exception) {
        $this->messages[] = CriticalErrorsHandler::handleExceptionWithSyslog($exception);
        return false;
    }

    return true;
}
```
{% endcode %}

After all steps succeed it calls `PBXConfModulesProvider::getVersionsHash(true)` to rebuild the cache-busting hash for module JS files and translations. Each step is described below.

### checkCompatibility()

Compares the running PBX version against the module's `min_pbx_version` (from [`module.json`](module-json.md)). If the PBX is older, it records a message and returns `false`, aborting the install. The base reads the current version with the `PbxSettings::PBX_VERSION` constant and strips any `-dev` suffix before comparing:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (checkCompatibility)" %}
```php
public function checkCompatibility(): bool
{
    $currentVersionPBX = PbxSettings::getValueByKey(PbxSettings::PBX_VERSION);
    $currentVersionPBX = str_replace('-dev', '', $currentVersionPBX);
    if (version_compare($currentVersionPBX, $this->min_pbx_version) < 0) {
        $this->messages[] = $this->translation->_(
            "ext_ModuleDependsHigherVersion",
            ['version' => $this->min_pbx_version]
        );
        return false;
    }
    return true;
}
```
{% endcode %}

You rarely override this. Override it only if ModuleBlackList needs an _additional_ check (for example, refusing to install when a conflicting module is enabled). Call `parent::checkCompatibility()` first so the version check still runs.

### activateLicense()

Used only by **commercial** modules. The base activates a license **only** when `lic_product_id > 0` (i.e. the module declares a `lic_product_id` in `module.json`). For free modules it is a no-op that returns `true`:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (activateLicense)" %}
```php
public function activateLicense(): bool
{
    if ($this->lic_product_id > 0) {
        $lic = PbxSettings::getValueByKey(PbxSettings::PBX_LICENSE);
        if (empty($lic)) {
            $this->messages[] = $this->translation->_("ext_EmptyLicenseKey");
            return false;
        }
        // Get a trial license for this module
        $this->license->addtrial($this->lic_product_id);
    }
    return true;
}
```
{% endcode %}

ModuleBlackList is a free module, so it leaves this step alone. For commercial modules see [Licensing](../marketplace/licensing.md).

### installFiles()

Copies files and creates the symlinks the web UI and the dialplan need. The base default performs four jobs and you only override it if you have extra files to place. Internally it delegates to three **static helpers** on `MikoPBX\Modules\PbxExtensionUtils` (`src/Modules/PbxExtensionUtils.php`):

| Helper                                         | What it links         | Source → target                                                                              |
| ---------------------------------------------- | --------------------- | -------------------------------------------------------------------------------------------- |
| `PbxExtensionUtils::createAssetsSymlinks($id)` | JS / CSS / IMG assets | `<module>/public/assets/{js,css,img}` → `sites/admin-cabinet/assets/{js,css,img}/cache/<id>` |
| `PbxExtensionUtils::createViewSymlinks($id)`   | Volt view templates   | `<module>/App/Views` → `src/AdminCabinet/Views/Modules/<id>`                                 |
| `PbxExtensionUtils::createAgiBinSymlinks($id)` | AGI dialplan scripts  | `<module>/agi-bin/*.php` → the Asterisk `agi-bin` directory                                  |

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (installFiles)" %}
```php
public function installFiles(): bool
{
    // Cache links for JS, CSS, IMG folders
    PbxExtensionUtils::createAssetsSymlinks($this->moduleUniqueID);

    // Links for the module view templates
    PbxExtensionUtils::createViewSymlinks($this->moduleUniqueID);

    // Links for agi-bin scripts
    PbxExtensionUtils::createAgiBinSymlinks($this->moduleUniqueID);

    // Restore the module DB saved by a previous uninstall with $keepSettings=true
    $modulesDir = $this->config->path('core.modulesDir');
    $backupPath = "$modulesDir/Backup/$this->moduleUniqueID";
    if (is_dir($backupPath)) {
        $cp = Util::which('cp');
        Processes::mwExec("$cp -r $backupPath/db/* $this->moduleDir/db/");
    }

    // ... clears the Volt template cache ...
    return true;
}
```
{% endcode %}

{% hint style="info" %}
**Settings preservation #1 — restore.** When a previous uninstall ran with `$keepSettings = true`, the module's `db` folder was copied to `<modulesDir>/Backup/<moduleUniqueID>`. `installFiles()` copies it back, so a reinstall/upgrade keeps the user's data. This is distinct from the legacy migration in `transferOldSettings()` described below.

Sound files are intentionally **not** installed here. They are managed by the module's enable/disable hooks (`onAfterModuleEnable` / `onAfterModuleDisable`) — see [Module class](module-class.md).
{% endhint %}

Override `installFiles()` only to place extra files (for example, ModuleBlackList copying a firewall ruleset into a system path). Always call `parent::installFiles()` so the symlinks above are still created.

### installDB()

Builds the module's database structure, then registers the module and (optionally) adds a sidebar item. This is the step modules customize most. The base default chains three calls:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (installDB)" %}
```php
public function installDB(): bool
{
    $result = $this->createSettingsTableByModelsAnnotations();

    if ($result) {
        $result = $this->registerNewModule();
    }

    if ($result) {
        $result = $this->addToSidebar();
    }
    return $result;
}
```
{% endcode %}

#### createSettingsTableByModelsAnnotations()

You do **not** ship a prebuilt SQLite file. Instead you describe each table as a Phalcon model with [column annotations](data-model.md), and this helper creates or **alters** the matching tables — both on first install and on upgrade. It registers the module's DB connection, iterates every file in `<module>/Models/*.php`, and runs each through `UpdateDatabase::createUpdateDbTableByAnnotations()`:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (createSettingsTableByModelsAnnotations)" %}
```php
public function createSettingsTableByModelsAnnotations(): bool
{
    ModulesDBConnectionsProvider::recreateModulesDBConnections();

    $results   = glob($this->moduleDir . '/Models/*.php', GLOB_NOSORT);
    $dbUpgrade = new UpdateDatabase();
    foreach ($results as $file) {
        $className        = pathinfo($file)['filename'];
        $moduleModelClass = "Modules\\{$this->moduleUniqueID}\\Models\\$className";
        if (!$dbUpgrade->createUpdateDbTableByAnnotations($moduleModelClass)) {
            return false;
        }
    }
    ModulesDBConnectionsProvider::recreateModulesDBConnections();
    return true;
}
```
{% endcode %}

For ModuleBlackList, the model `BlackListNumbers` maps to the table `m_BlackListNumbers`:

{% code title="Modules/ModuleBlackList/Models/BlackListNumbers.php" %}
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
     * Blocked phone number.
     *
     * @Column(type="string", nullable=true)
     */
    public $number;

    public function initialize(): void
    {
        $this->setSource('m_BlackListNumbers');
        parent::initialize();
    }
}
```
{% endcode %}

See [Data model](data-model.md) for the full annotation reference. After `createSettingsTableByModelsAnnotations()` succeeds you can seed defaults through the model:

```php
public function installDB(): bool
{
    $result = $this->createSettingsTableByModelsAnnotations();
    if ($result) {
        $number = BlackListNumbers::findFirst();
        if ($number === null) {
            $number = new BlackListNumbers();
            $number->number = '0000000000';
            $result = $number->save();
        }
    }
    if ($result) {
        $result = $this->registerNewModule();
    }
    return $result;
}
```

#### registerNewModule()

Adds (or updates) a row in the `PbxExtensionModules` table from `module.json` data. The new record is created **disabled** (`disabled = '1'`); the user enables the module later. Name, description and wiki links are taken from translations and the parsed `module.json`:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (registerNewModule)" %}
```php
public function registerNewModule(): bool
{
    $module = PbxExtensionModules::findFirstByUniqid($this->moduleUniqueID);
    if (!$module) {
        $module           = new PbxExtensionModules();
        $module->name     = $this->translation->_("Breadcrumb$this->moduleUniqueID");
        $module->disabled = '1';
    }
    $module->uniqid        = $this->moduleUniqueID;
    $module->developer     = $this->developer;
    $module->version       = $this->version;
    $module->description   = $this->translation->_("SubHeader$this->moduleUniqueID");
    $module->support_email = $this->support_email;
    $module->module_type   = $this->module_type;
    // ... wiki_links encoded as JSON ...
    return $module->save();
}
```
{% endcode %}

#### addToSidebar()

Adds a left-menu entry by writing an `AdditionalMenuItem<moduleUniqueID>` row into `PbxSettings`. The base default uses the `puzzle` icon and the `modules` group:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (addToSidebar)" %}
```php
public function addToSidebar(): bool
{
    $menuSettingsKey = "AdditionalMenuItem$this->moduleUniqueID";
    $menuSettings    = PbxSettings::findFirstByKey($menuSettingsKey);
    if ($menuSettings === null) {
        $menuSettings      = new PbxSettings();
        $menuSettings->key = $menuSettingsKey;
    }
    $value = [
        'uniqid'        => $this->moduleUniqueID,
        'group'         => 'modules',
        'iconClass'     => 'puzzle',
        'caption'       => "Breadcrumb$this->moduleUniqueID",
        'showAtSidebar' => true,
    ];
    $menuSettings->value = json_encode($value);
    return $menuSettings->save();
}
```
{% endcode %}

Override `addToSidebar()` to change the icon, menu group, or to add an explicit `href`. The working override in `Extensions/ModuleBackup/Setup/PbxExtensionSetup.php` places the item in the `maintenance` group with a `history` icon and a custom link. Modernized to the current `Text` helper, an equivalent ModuleBlackList override looks like:

{% code title="Modules/ModuleBlackList/Setup/PbxExtensionSetup.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Setup;

use MikoPBX\Common\Library\Text;
use MikoPBX\Common\Models\PbxSettings;
use MikoPBX\Modules\Setup\PbxExtensionSetupBase;

class PbxExtensionSetup extends PbxExtensionSetupBase
{
    public function installDB(): bool
    {
        $result = $this->createSettingsTableByModelsAnnotations();
        if ($result) {
            $result = $this->registerNewModule();
        }
        if ($result) {
            $result = $this->addToSidebar();
        }
        return $result;
    }

    public function addToSidebar(): bool
    {
        $menuSettingsKey = "AdditionalMenuItem$this->moduleUniqueID";
        $controllerName  = Text::uncamelize($this->moduleUniqueID, '-');
        $menuSettings    = PbxSettings::findFirstByKey($menuSettingsKey);
        if ($menuSettings === null) {
            $menuSettings      = new PbxSettings();
            $menuSettings->key = $menuSettingsKey;
        }
        $value = [
            'uniqid'        => $this->moduleUniqueID,
            'href'          => "/admin-cabinet/$controllerName",
            'group'         => 'security',
            'iconClass'     => 'ban',
            'caption'       => "Breadcrumb$this->moduleUniqueID",
            'showAtSidebar' => true,
        ];
        $menuSettings->value = json_encode($value);
        return $menuSettings->save();
    }
}
```
{% endcode %}

`Text::uncamelize()` (`MikoPBX\Common\Library\Text`) converts `ModuleBlackList` to the `module-black-list` route segment.

{% hint style="info" %}
**Legacy migration with `transferOldSettings()`.** `transferOldSettings()` is **not** a base method — it is a migration helper a module defines in **its own subclass** to move data out of legacy system-database tables into the module's own database. The working example is `Extensions/ModuleTelegramNotify/Setup/PbxExtensionSetup.php`, which overrides `installDB()` to call `createSettingsTableByModelsAnnotations()`, `registerNewModule()`, and then its own `transferOldSettings()`. It reads old `m_ModuleTelegramNotify*` rows via `$this->db`, copies matching fields into the new models, saves, and drops the legacy table. Implement this only when you are migrating from an older schema; new modules do not need it. This is separate from the backup/restore handled by `installFiles()`.
{% endhint %}

### fixFilesRights()

The final install step. It applies the standard `www` ownership to the whole module folder and adds the executable bit to anything under `agi-bin/` and `bin/`:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (fixFilesRights)" %}
```php
public function fixFilesRights(): bool
{
    Util::addRegularWWWRights($this->moduleDir);
    $dirs = [
        "{$this->moduleDir}/agi-bin",
        "{$this->moduleDir}/bin",
    ];
    foreach ($dirs as $dir) {
        if (file_exists($dir) && is_dir($dir)) {
            Util::addExecutableRights($dir);
        }
    }
    return true;
}
```
{% endcode %}

Override this only if ModuleBlackList ships extra executables outside `agi-bin/` and `bin/`. Keep your folder layout aligned with the example modules so the defaults work unchanged.

## The uninstall pipeline: `uninstallModule()`

When a module is deleted (or upgraded, which deletes then reinstalls), MikoPBX calls:

{% code title="How the Core uninstalls your module" %}
```php
$moduleClass = "\\Modules\\{$moduleUniqueID}\\Setup\\PbxExtensionSetup";
$setup = new $moduleClass($moduleUniqueID);
$setup->uninstallModule($keepSettings);   // $keepSettings === true on upgrade
```
{% endcode %}

`uninstallModule()` runs `unInstallDB()` then `unInstallFiles()`, passing the `$keepSettings` flag through, and finally rebuilds the version hash:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (uninstallModule)" %}
```php
public function uninstallModule(bool $keepSettings = false): bool
{
    $result = true;
    try {
        if (!$this->unInstallDB($keepSettings)) {
            $this->messages[] = $this->translation->_("ext_UninstallDBError");
            $result = false;
        }
        if ($result && !$this->unInstallFiles($keepSettings)) {
            $this->messages[] = $this->translation->_("ext_UnInstallFiles");
            $result = false;
        }
        PBXConfModulesProvider::getVersionsHash(true);
    } catch (Throwable $exception) {
        $result = false;
        $this->messages[] = $exception->getMessage();
    }
    return $result;
}
```
{% endcode %}

### unInstallDB($keepSettings)

By default it just unregisters the module from `PbxExtensionModules` via `unregisterModule()`:

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (unInstallDB / unregisterModule)" %}
```php
public function unInstallDB(bool $keepSettings = false): bool
{
    return $this->unregisterModule();
}

public function unregisterModule(): bool
{
    $result = true;
    $module = PbxExtensionModules::findFirstByUniqid($this->moduleUniqueID);
    if ($module !== null) {
        $result = $module->delete();
    }
    return $result;
}
```
{% endcode %}

Override `unInstallDB()` to remove references to your module from **system** tables before calling `parent::unInstallDB($keepSettings)` (for example, ModuleBlackList deleting firewall rules it created). Use `$keepSettings` to decide whether to also clear stored user data.

### unInstallFiles($keepSettings)

Deletes the installed files, folders and asset symlinks. When `$keepSettings = true` it first copies the module's `db` folder to `<modulesDir>/Backup/<moduleUniqueID>` so a later reinstall can restore it (the restore happens in `installFiles()`, see above):

{% code title="src/Modules/Setup/PbxExtensionSetupBase.php (unInstallFiles, abridged)" %}
```php
public function unInstallFiles(bool $keepSettings = false): bool
{
    $cp = Util::which('cp');
    $rm = Util::which('rm');
    $modulesDir = $this->config->path('core.modulesDir');
    $backupPath = "$modulesDir/Backup/$this->moduleUniqueID";
    Processes::mwExec("$rm -rf {$backupPath}");
    if ($keepSettings) {
        Util::mwMkdir($backupPath);
        Processes::mwExec("$cp -r {$this->moduleDir}/db {$backupPath}/");
    }
    Processes::mwExec("$rm -rf {$this->moduleDir}");

    // Remove IMG / CSS / JS asset cache symlinks (omitted here)
    return true;
}
```
{% endcode %}

If your module runs background binaries you must stop them **before** the files are removed, and clean up any temporary or log files you created. Override `unInstallFiles()`, do your cleanup, then delegate to the parent:

{% code title="Modules/ModuleBlackList/Setup/PbxExtensionSetup.php (cleanup before delete)" %}
```php
use MikoPBX\Core\System\Processes;
use MikoPBX\Core\System\Util;

public function unInstallFiles(bool $keepSettings = false): bool
{
    // Stop the module's worker before deleting its files
    Processes::killByName('WorkerBlackList');

    // Remove the module log directory
    $rm     = Util::which('rm');
    $logDir = $this->config->path('core.logsDir') . '/' . $this->moduleUniqueID;
    Processes::mwExec("$rm -rf $logDir");

    return parent::unInstallFiles($keepSettings);
}
```
{% endcode %}

## Choosing what to override — summary

| Method             | Override when ModuleBlackList needs to…                                        | Real example                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| (nothing)          | use the defaults for everything                                                | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Setup/PbxExtensionSetup.php`                                     |
| `installDB()`      | seed default rows, register, add a custom sidebar item, run a legacy migration | `Extensions/ModuleTelegramNotify/Setup/PbxExtensionSetup.php`, `Extensions/ModuleBackup/Setup/PbxExtensionSetup.php` |
| `addToSidebar()`   | change menu group / icon / href                                                | `Extensions/ModuleBackup/Setup/PbxExtensionSetup.php`                                                                |
| `installFiles()`   | copy extra files outside the standard folders                                  | base default suffices for most modules                                                                               |
| `fixFilesRights()` | mark extra executables                                                         | base default suffices for most modules                                                                               |
| `unInstallDB()`    | clean references from system tables                                            | extend `parent::unInstallDB()`                                                                                       |
| `unInstallFiles()` | stop workers, delete logs/temp files                                           | extend `parent::unInstallFiles()`                                                                                    |

{% hint style="success" %}
Override the smallest possible surface. The base implements every step, so the recommended pattern is to override one method, do your specific work, and call `parent::<method>()` to keep the standard behavior. The error strings you set via `$this->messages[]` are shown to the user, so make them actionable.
{% endhint %}

## Related pages

* [module.json reference](module-json.md) — every field consumed by the constructor.
* [Data model](data-model.md) — model column annotations used by `createSettingsTableByModelsAnnotations()`.
* [Module class](module-class.md) — the enable/disable hooks (`onAfterModuleEnable` / `onAfterModuleDisable`) that manage sound files and runtime state.
