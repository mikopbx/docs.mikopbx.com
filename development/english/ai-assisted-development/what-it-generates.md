---
description: >-
  The files the /mikopbx-module skill produces, the post-generation checks it
  runs, and how to verify the output yourself.
---

# What it generates

When you approve a generation plan, the **`/mikopbx-module`** skill emits a complete module tree, then runs a fixed set of self-checks and prints a report. This page documents exactly what comes out, in what order, the code conventions it enforces, and — most importantly — how to re-verify everything by hand so you never have to take the skill's word for it.

The running example is the fictional spam-blocking module **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` backed by table `m_BlackListNumbers`, front-end script `module-black-list-index.js`). Every pattern below is anchored to a real, shipped module you can open and compare against.

## File generation order

The skill emits files in dependency order so that each file can reference the ones generated before it. A full "create" session with the `base`, `ui`, `rest-api`, `dialplan` and `agi` recipes selected produces the tree in this sequence:

1. **`module.json`** — module metadata (unique id, version, `min_pbx_version`, namespace).
2. **`README.md` and `README.ru.md`** — equivalent administrator documentation in English and Russian.
3. **`.github/workflows/build.yml`** — production build and publish automation.
4. **`Setup/PbxExtensionSetup.php`** — the installer (creates tables, registers the module).
5. **`Models/*.php`** — database models (e.g. `Models/BlackListNumbers.php`).
6. **`Lib/{Feature}Conf.php`** — the configuration class with dialplan / config hooks (e.g. `Lib/BlackListConf.php`).
7. **`Lib/{Feature}Main.php`** — business logic, where the recipe needs it (e.g. `Lib/BlackListMain.php`).
8. **`App/Controllers/`** — web controllers (`ui` recipe).
9. **`App/Forms/`** — Phalcon forms (`ui` recipe).
10. **`App/Views/`** — Volt templates (`ui` recipe).
11. **`App/Providers/`** — _optional_ per-module `AssetProvider` / `MenuProvider` helper classes (`ui` recipe). See the note below before you expect them.
12. **`public/assets/js/src/`** — ES6+ JavaScript source, one file per controller action (`ui` recipe).
13. **`public/assets/css/`** — CSS styles (`ui` recipe).
14. **`Lib/RestAPI/`** — REST API controllers and actions (`rest-api` recipe).
15. **`bin/`** — worker scripts (`workers` recipe).
16. **`agi-bin/`** — AGI scripts (`agi` recipe).
17. **`Messages/`** — translation files, generated via the `/translations` skill.

{% hint style="info" %}
**About `App/Providers/`.** These are a convenience pattern, not a requirement. The default — and what the canonical example `ModuleExampleForm` does — is to register assets _inline in the controller_ against the **core** provider constants:

```php
use MikoPBX\AdminCabinet\Providers\AssetProvider;

$this->assets->collection(AssetProvider::HEADER_CSS)->addCss(...);
$this->assets->collection(AssetProvider::FOOTER_JS)->addJs(...);
```

`ModuleExampleForm/App/Providers/` contains only a `.gitkeep`. Per-module provider classes pay off once a module has several pages and a long shared asset list; three shipped production modules use them — `ModuleLocalSpeechToText`, `ModuleRemoteSupport` and `ModuleAiSupervisor` (see `Extensions/ModuleLocalSpeechToText/App/Providers/AssetProvider.php` for `addCss()`/`addJs()` and `MenuProvider.php` for page-path constants). For a single-page module, skip them.
{% endhint %}

{% hint style="info" %}
The order is not cosmetic. `module.json` defines the namespace that every later file `use`s; the model exists before the `Conf`/`Main` classes that query it; the providers and JS come after the controller they wire up. If you generate files yourself, follow the same order to avoid forward references.
{% endhint %}

A representative tree for **ModuleBlackList**:

```
Extensions/ModuleBlackList/
├── module.json
├── README.md
├── README.ru.md
├── .github/workflows/build.yml
├── Setup/
│   └── PbxExtensionSetup.php
├── Models/
│   └── BlackListNumbers.php
├── Lib/
│   ├── BlackListConf.php
│   ├── BlackListMain.php
│   └── RestAPI/
│       └── Numbers/
│           ├── Controller.php
│           ├── Processor.php
│           ├── DataStructure.php
│           └── Actions/
│               └── GetListAction.php
├── App/
│   ├── Controllers/ModuleBlackListController.php
│   ├── Forms/ModuleBlackListForm.php
│   ├── Views/ModuleBlackList/index.volt
│   └── Providers/                 # optional — multi-page modules only (see note above)
│       ├── AssetProvider.php
│       └── MenuProvider.php
├── public/assets/
│   ├── js/src/module-black-list-index.js
│   └── css/module-black-list-index.css
├── agi-bin/
│   └── check-blacklist.php
└── Messages/
    ├── ru.php
    └── … (25 more languages)
```

The skill reads the published reference modules ([github.com/mikopbx](https://github.com/mikopbx), starting with [ModuleTemplate](https://github.com/mikopbx/ModuleTemplate)) before it writes each slice. The same slices are shown in the example modules used throughout this guide:

* Web UI page: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`
* REST API (current version): `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`
* AMI / background worker: `Extensions/EXAMPLES/AMI/ModuleExampleAmi/`

## Code conventions enforced

Every generated PHP file follows the same baseline.

### Strict types and no closing tag

Each PHP file opens with strict-types and never carries a closing `?>` tag:

{% code title="Lib/BlackListConf.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib;
```
{% endcode %}

### PHP 8.4 idioms

The skill uses modern PHP wherever it applies — typed properties, constructor property promotion, `match` expressions, named arguments and enums:

```php
public readonly string $moduleUniqueId;

public function __construct(
    private readonly string $moduleUniqueId,
) {}

$result = match ($action) {
    'check'  => $this->check(),
    'reload' => $this->reload(),
    default  => throw new \InvalidArgumentException("Unknown action: $action"),
};
```

### The Phalcon model exception

Model column properties are the one place the skill does **not** apply the typed-property rules above. They follow Phalcon's SQLite ORM convention instead: an untyped primary key, nullable string columns (integers are stored as strings in SQLite), and nullable int foreign keys:

{% code title="Models/BlackListNumbers.php" %}
```php
// Primary key — always untyped
public $id;

// String column — nullable, string default
public ?string $number = '';

// Integer-as-string column
public ?string $enabled = '0';

// Integer foreign key — nullable int
public ?int $userid = null;
```
{% endcode %}

This matches the core models in `Core/src/Common/Models/` — compare `Extensions.php`, `Sip.php` or `CallQueues.php`.

### The `Phalcon\Di\Di` import

The skill always imports the DI container by its full class name. The short form is a frequent mistake that breaks under Phalcon 5:

```php
// CORRECT
use Phalcon\Di\Di;

// WRONG — class not found
use Phalcon\Di;
```

## Post-generation checks the skill runs

After the last file is written, the skill runs seven checks and refuses to declare success silently if any fail. Paths below assume the module directory is `Extensions/ModuleBlackList`; substitute your own.

### 1. PHP syntax — `php -l` on every file

```bash
find Extensions/ModuleBlackList -name "*.php" -exec php -l {} \;
```

### 2. JavaScript transpilation via the `/babel-compiler` skill

If the `ui` recipe produced JavaScript, the skill transpiles the ES6+ source to ES5 with the Babel compiler from [MikoPBXUtils](https://github.com/mikopbx/MikoPBXUtils). The compiler is a Docker image you build once from a checkout of that repository — it is not published to a registry:

```bash
git clone https://github.com/mikopbx/MikoPBXUtils && cd MikoPBXUtils
docker build -t mikopbx-babel-compiler .
```

Each source file under `public/assets/js/src/` is compiled one-for-one into `public/assets/js/`, keeping its name. Run the container from the parent directory of your `Core` checkout (the directory that also contains `Extensions/`):

```bash
docker run --rm -v "$(pwd)":/workspace \
  mikopbx-babel-compiler \
  /workspace/Extensions/ModuleBlackList/public/assets/js/src/module-black-list-index.js \
  extension
```

produces `public/assets/js/module-black-list-index.js`.

{% hint style="info" %}
The entrypoint detects the target (`core` for admin-cabinet files, `extension` for module files) from the path itself; the trailing argument is accepted for clarity but does not change where the output goes. The Babel preset is fixed inside the image — do not override it. Output always lands in `public/assets/js/`; there is no `cache/` build directory in your checkout (`js/cache/<ModuleID>/…` paths you see in provider classes are the _runtime URL_ the admin cabinet serves module assets from, not a build target).
{% endhint %}

### 3. `module.json` validity

```bash
php -r "json_decode(file_get_contents('Extensions/ModuleBlackList/module.json'), true) ?: exit(1);"
```

### 4. Standalone translation catalogs

Every `Messages/<locale>.php` must `return` a literal array and nothing else — no `require`, `include`, variables, `array_keys`, `array_combine`, merges or runtime composition. MikoPBX loads and processes each catalog itself, so a computed catalog silently yields no keys:

```bash
! grep -RE '(require|include)(_once)?[[:space:]]*\(?[[:space:]]*(__DIR__|__FILE__|\$[A-Za-z_]|['"'"'"][^'"'"'"]*\.php)|\b(array_keys|array_combine)[[:space:]]*\(' \
  Extensions/ModuleBlackList/Messages/*.php
```

The pattern matches on the _argument_, so a translated string that merely contains the word "include" (for example "Include only internal calls") passes, while `array_merge(include 'base.php', …)` is caught.

### 5. README pair (production modules only)

```bash
test -f Extensions/ModuleBlackList/README.md
test -f Extensions/ModuleBlackList/README.ru.md
```

### 6. Publish workflow (production modules only)

```bash
test -f Extensions/ModuleBlackList/.github/workflows/build.yml
```

Checks 5 and 6 are skipped for example modules: they usually ship a single readme and are built by a shared workflow of the repository that hosts them.

### 7. REST/OpenAPI translations (when the `rest-api` recipe is present)

```bash
# run from the Core checkout root, or set MIKOPBX_CORE_DIR to it
php <skills-dir>/mikopbx-module/scripts/validate-rest-api-translations.php \
  Extensions/ModuleBlackList
```

The script ships with the skill. It checks that every module-owned REST key and every generated tag key (`rest_tag_ModuleBlackListNumbers`, operation summaries, parameter and schema descriptions) is defined in both `Messages/en.php` and `Messages/ru.php`. A raw identifier such as `rest_numbers_GetList` showing up in the OpenAPI UI is a failure, not an acceptable fallback.

### The report

When the checks finish, the skill prints a summary: the recipe set it applied, the full list of files it created, and the pass/fail result of each check, followed by suggested next steps:

```
Module: ModuleBlackList
Location: ModuleBlackList/
Recipes: base, ui, rest-api, dialplan, agi

Files created:
  Setup/PbxExtensionSetup.php
  Lib/BlackListConf.php
  …
  agi-bin/check-blacklist.php
  Messages/ru.php (+ 25 languages via /translations)
  module.json

Checks:
  PHP syntax: PASS (18/18 files)
  Babel: PASS
  module.json: VALID

Next steps:
  1. Review generated code
  2. Install module in MikoPBX
  3. Test functionality
  4. Customize business logic
```

## Generated AGI scripts: use the real Core API

{% hint style="warning" %}
**Check the class name in any generated AGI script.** Earlier revisions of the `agi` recipe emitted a client that **does not exist in the MikoPBX Core**:

```php
use AGI\AgiClient;          // ❌ no such class
$agi = new AgiClient();     // ❌
$agi->getVariable(...);     // ❌ no such method
$agi->setVariable(...);     // ❌ no such method
```

The real AGI client is `MikoPBX\Core\Asterisk\AGI` (see `Core/src/Core/Asterisk/AGI.php`) and its accessors are **snake\_case**: `get_variable()` and `set_variable()`. The skill templates (`templates/agi-recipe.md`, `reference/recipes.md`) have been corrected, but if a script comes back referencing `AgiClient`, replace it with the form below.
{% endhint %}

The correct, Core-verified pattern — confirmed against `Core/src/Core/Asterisk/AGI.php`:

{% code title="agi-bin/check-blacklist.php" %}
```php
#!/usr/bin/php
<?php

declare(strict_types=1);

use MikoPBX\Core\Asterisk\AGI;
use MikoPBX\Core\System\Util;

require_once 'Globals.php';

try {
    $agi    = new AGI();
    $number = $agi->request['agi_callerid'];

    // Read a channel variable (second arg true returns the value directly)
    $exten = $agi->get_variable('EXTEN', true);

    // Your lookup logic here, e.g. query BlackListNumbers …
    $blocked = '1';

    // Write a result variable back for the dialplan
    $agi->set_variable('BLACKLIST_RESULT', $blocked);
    $agi->noop("Blacklist check for $number -> $blocked");
} catch (\Throwable $e) {
    Util::sysLogMsg('ModuleBlackList', $e->getMessage(), LOG_ERR);
}
```
{% endcode %}

Confirmed `AGI` methods you can call (from `AGI.php`): `get_variable(string $variable, bool $getvalue = false)`, `set_variable(string $variable, string $value)`, `set_var(string $pVariable, string|int|float $pValue)`, `verbose()`, `answer()`, `noop()`, `exec()`, `exec_dial()`, `exec_goto()`, `stream_file()`, `getData()`, `wait_for_digit()`, `set_callerid()`, `getCallerIdName()`, `database_get()`, `databasePut()`. Incoming AGI parameters are read from the `$agi->request` array (for example `$agi->request['agi_callerid']`).

See the working example in `Extensions/ModuleCTIClientV5/agi-bin/set-caller-id.php`, which uses `new AGI()` and `$agi->set_variable('CALLERID(name)', $callerIDName)`. The recipe also points at two more real lookups worth reading: `Extensions/ModulePhoneBook/agi-bin/agi_phone_book.php` and `Extensions/ModuleTelegramProvider/agi-bin/saveSipHeadersInRedis.php`.

## Translations: the 26-language chain

The `Messages/` step does not call an AI translator inline — it delegates to the `/translations` skill, which enforces a strict Russian-first workflow across all 26 supported languages.

1. **Russian is the source of truth.** The skill writes `Messages/ru.php` first, with every key the generated controllers, forms and views reference, using the module prefix (`module_black_list_` for ModuleBlackList) and the `%placeholder%` format.
2. **The other 25 languages are derived from Russian** — never edited by hand. `/translations` processes them **one language and one file at a time**, translating only missing keys and preserving any existing ones.
3. **Key-count validation gates every step.** After each language is merged, its key count must match the Russian source _exactly_; on a mismatch the skill stops rather than ship an inconsistent file. Each merged file is also `php -l`-checked, and placeholder names must be identical to the Russian original.

### The three registration keys

Beyond the keys your own code references, every module catalog **must** carry three keys that only MikoPBX Core reads — nothing in your PHP, JavaScript or Volt will mention them, so they are easy to lose:

| Key                                  | What it labels                  |
| ------------------------------------ | ------------------------------- |
| `AdditionalMenuItem<ModuleUniqueID>` | the sidebar entry               |
| `Breadcrumb<ModuleUniqueID>`         | the module title and breadcrumb |
| `SubHeader<ModuleUniqueID>`          | the module description          |

For **ModuleBlackList** those are `AdditionalMenuItemModuleBlackList`, `BreadcrumbModuleBlackList` and `SubHeaderModuleBlackList`. They must be present and translated in **all 26 locales** — when one is missing, the module-management page renders the raw key name instead of a label.

### The standalone-array rule

Each `Messages/<locale>.php` returns a plain literal array. Never build a catalog at runtime (`require`, `include`, variables, `array_keys`, `array_combine`, array merges) — check 4 above enforces this.

Technical terms (SIP, IAX, AMI, PJSIP, RTP, CDR, IVR, DTMF, codec, trunk, extension, …) are left untranslated in every language. See [Module translations](../module-developement/translations.md) for the full key-naming, prefix and placeholder rules.

## The production repository contract

When you answer "production module" in the discovery dialog, the skill also lays down the release plumbing. Five requirements, straight from `SKILL.md`:

1. **A README pair.** `README.md` (English, the default) and `README.ru.md` with equivalent content — written for _administrators_: purpose first, then installation and use, security, privacy, troubleshooting and support. Contributor/build commands stay secondary.
2. **Copy a maintained module first.** Inspect a currently maintained production module published at [github.com/mikopbx](https://github.com/mikopbx) before adding automation, rather than inventing a workflow.
3. **`.github/workflows/build.yml`** built on the shared reusable workflow `mikopbx/.github-workflows/.github/workflows/extension-publish.yml@master`, triggered on `develop`, `master` and `workflow_dispatch`.
4. **Release flags in `module.json`** — `release_settings.publish_release`, `changelog_enabled` and `create_github_release`.
5. **Verify both branches before pushing either**: `develop` must produce a _prerelease_, `master` a production publication.

{% hint style="warning" %}
Steps 1, 3 and 4 are generated for you; step 5 is not. The skill cannot run your GitHub Actions — confirm the develop-prerelease / master-release behavior yourself before the first push to either branch.
{% endhint %}

## How to verify the output yourself

Treat the skill's report as a claim, not proof. Re-run every check independently before you install the module.

1.  **Re-lint every PHP file:**

    ```bash
    find Extensions/ModuleBlackList -name "*.php" -exec php -l {} \;
    ```
2.  **Re-transpile the JavaScript** and confirm the ES5 output exists and is non-trivial:

    ```bash
    # image built from MikoPBXUtils as in check 2 above; run from the parent of Core/
    docker run --rm -v "$(pwd)":/workspace \
      mikopbx-babel-compiler \
      /workspace/Extensions/ModuleBlackList/public/assets/js/src/module-black-list-index.js \
      extension
    ```
3.  **Re-validate `module.json`:**

    ```bash
    php -r "json_decode(file_get_contents('Extensions/ModuleBlackList/module.json'), true) ?: exit(1);"
    ```
4. **Run static analysis** beyond bare syntax — the Core checkout ships `phpcs` (PSR-12) and `phpstan`; point them at the module directory. Phalcon is a PHP extension, not a Composer package, so add `Core/vendor/phalcon/ide-stubs/src` to phpstan's `scanDirectories` or every Phalcon class shows up as "not found".
5. **Verify translation consistency** — confirm every language file has the same key count as `Messages/ru.php` (the `translations` skill ships consistency-check commands; a quick spot check is `php -r "echo count(include 'Extensions/ModuleBlackList/Messages/ru.php');"` compared against each language).
6. **Install and test live** — install the module into the development container or a test PBX, exercise the settings page, call the REST API with the `api-client` skill, check the rows with `sqlite-inspector`, and trigger the dialplan / AGI path while `log-analyzer` and `asterisk-validator` watch the logs. Runtime behavior is what matters, not static checks.

{% hint style="warning" %}
Static checks (`php -l`, Babel, JSON validation) prove the files _parse_. They do not prove the module _works_. Always finish with a live install and functional test before treating a generated module as done — and re-check any generated AGI script against the correct client class above.
{% endhint %}

## Related pages

* [Using the skill](using-the-skill.md) — the discovery-to-approval workflow that precedes generation.
* [Module recipes](../module-developement/recipes.md) — the full specification of each recipe and the files it contributes to the tree.
* [Module translations](../module-developement/translations.md) — the Russian-first, 26-language translation rules the `Messages/` step relies on.
