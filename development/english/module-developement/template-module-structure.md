---
description: Clone the module template and prepare it for development.
---

# How to start

Every MikoPBX module follows the same skeleton: a `module.json` manifest, a PSR-4 autoloaded `composer.json`, and a handful of well-known directories that the Core discovers by convention. You never start from a blank directory. There are two supported ways to bootstrap a new module, and both produce the same layout.

Throughout this documentation we use a single running example — a fictional module called **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` backed by table `m_BlackListNumbers`, front-end asset `module-black-list-index.js`). Wherever a pattern is shown, you will also find a pointer to a **real** example module you can read in the repository.

## Two ways to bootstrap a module

### Path 1 — Clone the ModuleTemplate

`ModuleTemplate` is a complete, installable reference module that ships every common extension point already wired up. Cloning it gives you a guaranteed-correct starting layout.

{% tabs %}
{% tab title="Linux / macOS" %}
```bash
git clone https://github.com/mikopbx/ModuleTemplate.git ModuleBlackList
cd ModuleBlackList
rm -rf .git
```
{% endtab %}
{% endtabs %}

After cloning you rename the module identity (namespace, `moduleUniqueID`, classes, table names) to match your feature. The identity rules are described under [moduleUniqueID is the spine](template-module-structure.md#moduleuniqueid-is-the-spine) below.

### Path 2 — Generate with the `/mikopbx-module` skill

If you work with an AI coding agent (Claude Code, Codex, Cursor, Gemini CLI, …), the `/mikopbx-module` skill from [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills) generates a fully wired module from a natural-language description. It produces the same layout as the template, picks the integration "recipes" you actually need (UI, REST API, dialplan hooks, workers, AGI, firewall, …), and follows the naming conventions automatically.

```
/mikopbx-module Create a module that blocks incoming calls from a blacklist of numbers,
with a settings page and a REST API for managing the list.
```

See [Using the skill](../ai-assisted-development/using-the-skill.md) for the full workflow.

{% hint style="info" %}
Both paths converge on the structure described on this page. Read [Module anatomy](module-anatomy.md) afterwards for the complete, file-by-file map.
{% endhint %}

## The ModuleTemplate root layout

The template root contains everything the Core expects, plus empty **extension points** (directories with a single `.gitkeep`) that you fill in only if your module needs them.

```
ModuleTemplate/
├── module.json              # Manifest: ID, version, min_pbx_version, release settings
├── composer.json            # PSR-4 autoload + mikopbx/core dependency
├── README.md
├── App/                     # Web tier (admin cabinet)
│   ├── Module.php           # Phalcon micro-module bootstrap for the web app
│   ├── Controllers/         # Admin-cabinet controllers
│   ├── Forms/               # Phalcon\Forms\Form classes
│   ├── Views/               # *.volt templates (per controller)
│   └── Providers/           # Asset / menu providers (.gitkeep until used)
├── Lib/                     # Business logic
│   ├── TemplateConf.php     # Config class — the hook surface into the Core
│   ├── TemplateMain.php     # Plain business logic
│   ├── WorkerTemplateMain.php
│   └── WorkerTemplateAMI.php
├── Models/                  # Phalcon ORM models (one class = one m_* table)
│   └── ModuleTemplate.php
├── Setup/                   # Installer
│   └── PbxExtensionSetup.php
├── Messages/                # Translations: 28 locale files + languages.php registry
├── public/assets/           # Front-end assets
│   ├── js/src/              # ES6+ sources (transpiled with Babel)
│   ├── js/                  # Compiled JS served to the browser
│   ├── css/
│   └── img/
├── agi-bin/   .gitkeep      # AGI scripts (dialplan integration)
├── bin/       .gitkeep      # Worker / CLI binaries
└── db/        .gitkeep      # Optional pre-seeded SQLite data
```

The three `.gitkeep`-only directories — `agi-bin/`, `bin/`, and `db/` — are **extension points**. They keep the directory in version control so the path exists the moment you need it, without forcing every module to carry AGI scripts, workers, or seed data it does not use.

{% hint style="info" %}
You can confirm this layout against the shipped template at `Extensions/ModuleTemplate/`, and against a focused, real-world UI module at `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`.
{% endhint %}

## `moduleUniqueID` is the spine

A single value — `moduleUniqueID` in `module.json` — drives almost every other name in the module. Choose it once, in PascalCase, prefixed with `Module` (our example uses `ModuleBlackList`), and everything else follows by convention.

{% code title="module.json (identity field)" %}
```json
{
  "moduleUniqueID": "ModuleBlackList"
}
```
{% endcode %}

The Core derives the module's identity from this value. In particular, every module class lives under the namespace `Modules\{moduleUniqueID}\…`, and the Core resolves the running module's ID from exactly that namespace segment. See `PbxExtensionBase::__construct()` in `src/Modules/PbxExtensionBase.php`, which splits the class namespace and reads the second segment:

```php
// Core: src/Modules/PbxExtensionBase.php (simplified)
$partsOfNameSpace = explode('\\', $reflector->getNamespaceName());
if (count($partsOfNameSpace) === 3 && $partsOfNameSpace[0] === 'Modules') {
    $this->moduleUniqueId = $partsOfNameSpace[1];
    $this->moduleDir      = $modulesDir . '/' . $this->moduleUniqueId;
}
```

What `moduleUniqueID` controls:

| Concern                         | Derived value                   | Example (`ModuleBlackList`)                  |
| ------------------------------- | ------------------------------- | -------------------------------------------- |
| PHP namespace                   | `Modules\{ID}\…`                | `Modules\ModuleBlackList\Lib`                |
| On-disk module directory        | `{modulesDir}/{ID}`             | `…/Modules/ModuleBlackList`                  |
| Database table prefix           | `m_{Entity}` (per model)        | `m_BlackListNumbers`                         |
| URL slug (routes, assets, REST) | `module-{kebab-case}`           | `module-black-list`                          |
| Admin-cabinet route             | `/{slug}/{controller}/{action}` | `/module-black-list/module-black-list/index` |

The database table name is set per-model with `setSource()`. In the template's model `Models/ModuleTemplate.php` it is `$this->setSource('m_ModuleTemplate')`; for our example it would be `m_BlackListNumbers`.

The admin route is registered by the Core as `/{uncamelized-uniqueID}/:controller/:action/:params` — see `PbxExtensionUtils::registerEnabledModulesInRouter()` in `src/Modules/PbxExtensionUtils.php`, which builds the slug with `Text::uncamelize($moduleUniqueId, '-')`. Do not confuse this with `$this->view->currentPage`, which `BaseController` assembles as `"$module->uniqid/$controllerName/$actionName"` (PascalCase) purely to highlight the active menu entry — it is not a URL.

{% hint style="warning" %}
Rename **consistently**. When you change `moduleUniqueID`, you must also update the namespace in every PHP file, the directory name, the PSR-4 prefix in `composer.json`, the model `setSource()` calls, and the kebab-case asset / slug names. A mismatch between the namespace segment and the directory name will break module discovery.
{% endhint %}

For the full table of class, file, table, route, and translation-key conventions, see [Module anatomy](module-anatomy.md).

## Modernize the manifest for a new module

The shipped `Extensions/ModuleTemplate/module.json` and `composer.json` still target an older platform — `min_pbx_version: 2023.2.150` and PHP `^7.4 || ^8.0`. **Do not keep those values in a new module.** Update them to the current baseline.

### `module.json`

Set `min_pbx_version` to **`2025.1.1`** for new modules. This matches the current example modules — see `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/module.json`, which declares `"min_pbx_version": "2025.1.1"`.

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

`%ModuleVersion%` is a placeholder substituted by the release tooling at build time; leave it as-is in source. Every field is documented in detail on the [module.json reference](module-json.md) page.

### `composer.json`

Match the PHP and Core constraints to the current platform. MikoPBX Core 2025.1.1+ declares `"php": "^8.4"` in its own `composer.json`, so a module that depends on Core must require the same.

{% code title="composer.json" %}
```json
{
    "name": "mikopbx/moduleblacklist",
    "description": "ModuleBlackList",
    "type": "application",
    "license": "GPL-3.0-or-later",
    "require": {
        "php": "^8.4",
        "mikopbx/core": ">=2025.1.1"
    },
    "autoload": {
        "psr-4": {
            "Modules\\ModuleBlackList\\": "/"
        }
    },
    "config": {
        "sort-packages": true,
        "platform": {
            "php": "8.4"
        }
    }
}
```
{% endcode %}

{% hint style="danger" %}
The PSR-4 prefix **must** equal `Modules\{moduleUniqueID}\` and map to the module root (`/`). If the prefix does not match the namespace the Core derives from `moduleUniqueID`, autoloading and module discovery both fail.
{% endhint %}

## Install the `mikopbx/core` dependency

Pull in `mikopbx/core` and its dependencies so your IDE and static analysis can resolve MikoPBX class names, hooks, and ORM models.

{% hint style="info" %}
Read [Prepare IDE and system tools](../prepare-ide-tools/) first — it covers installing the PHP toolchain, Composer, and the IDE configuration the next steps assume.
{% endhint %}

{% tabs %}
{% tab title="Linux / macOS" %}
```bash
cd ModuleBlackList
composer install
```
{% endtab %}
{% endtabs %}

This populates `vendor/` with the Core sources. You exclude `vendor/` from both the module ZIP and version control — it is a development-time aid only, never shipped.

## Put the module under version control

{% tabs %}
{% tab title="Linux / macOS" %}
```bash
git init
echo "vendor/" > .gitignore
git add .
git commit -m 'Initial commit'
```
{% endtab %}
{% endtabs %}

If you have a remote repository, push to it:

{% tabs %}
{% tab title="Linux / macOS" %}
```bash
git remote add origin <url>
git push -u origin master
```
{% endtab %}
{% endtabs %}

## Try it on MikoPBX

Once the layout and `module.json` are in place you can package the module as a ZIP and install it through the admin cabinet (**Extension modules → Install module**). The PBX unpacks it, runs `Setup/PbxExtensionSetup.php`, creates the module's database tables from your `Models/`, and registers its menu and assets.

You do not need any custom code to get a first install working — the cloned template is installable as-is. Iterate from there: rename the identity, then add only the extension points your feature needs.

## Next steps

* [Module anatomy](module-anatomy.md) — the complete file-by-file map of a module.
* [module.json reference](module-json.md) — every manifest field explained.
* [Using the skill](../ai-assisted-development/using-the-skill.md) — generate and extend modules with the `/mikopbx-module` AI skill.
