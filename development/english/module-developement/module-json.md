---
description: >-
  Field reference for the module.json manifest, version constraints, and how
  dependencies actually work in MikoPBX.
---

# module.json reference

Every MikoPBX module ships a single `module.json` file at its root. This manifest is the authoritative source of identity (unique ID, developer, version), the only place where the author declares a PBX compatibility constraint, and the carrier of optional metadata for commercial licensing, language packs, and the release pipeline.

The PBX reads `module.json` at install time and on every construction of the module's setup object. See `MikoPBX\Modules\Setup\PbxExtensionSetupBase::__construct()` in `Core/src/Modules/Setup/PbxExtensionSetupBase.php`, which `json_decode`s the file and maps its fields onto setup properties.

{% hint style="info" %}
The field list below was verified against **all 60 real `module.json` manifests** in the production `Extensions/` tree. Every field documented here appears in at least one shipping module, and the parsing of each is traced to Core PHP. Fields that do **not** exist (`dependencies`, `max_pbx_version`) are called out explicitly in [Dependencies are procedural, not declarative](module-json.md#dependencies-are-procedural-not-declarative).
{% endhint %}

Running example throughout the docs: a fictional module **ModuleBlackList** (`moduleUniqueID: "ModuleBlackList"`). For a minimal real manifest see `Extensions/ModuleTemplate/module.json`; for a commercial one see `Extensions/ModuleAmoCrm/module.json`; for a language pack see `Extensions/LanguagePacks/ModuleDutchLanguagePack/module.json`.

## Minimal manifest

Five fields are present in 100% of manifests and are effectively required: `developer`, `moduleUniqueID`, `support_email`, `version`, and `min_pbx_version`. The shipping `ModuleTemplate` manifest is the canonical minimal example.

{% code title="Extensions/ModuleTemplate/module.json" %}
```json
{
  "developer": "MIKO",
  "moduleUniqueID": "ModuleTemplate",
  "support_email": "help@miko.ru",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2023.2.150",
  "release_settings": {
    "publish_release": true,
    "changelog_enabled": true,
    "create_github_release": true
  }
}
```
{% endcode %}

For ModuleBlackList, the equivalent minimal manifest targeting the current baseline:

{% code title="Modules/ModuleBlackList/module.json" %}
```json
{
  "developer": "Your Company",
  "moduleUniqueID": "ModuleBlackList",
  "support_email": "support@example.com",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2025.1.1"
}
```
{% endcode %}

{% hint style="warning" %}
`release_settings` is present in the large majority of manifests but is **not** parsed by the PBX at all — it is consumed only by the release/build tooling. A module installs and runs fine without it. See [release\_settings](module-json.md#release_settings).
{% endhint %}

## Core fields

### developer

`string`, required. Human-readable author/vendor name. Mapped to `PbxExtensionSetupBase::$developer` and written into the `developer` column of the `m_PbxExtensionModules` table during registration. Exposed unchanged in the v3 REST API (see the `developer` field in `Core/src/PBXCoreREST/Lib/Modules/DataStructure.php`).

### moduleUniqueID

`string`, required. The module's globally unique identifier. This is the single most important field: it is the directory name under the modules root, the PHP namespace segment (`Modules\ModuleBlackList\...`), the value of the `uniqid` column in `m_PbxExtensionModules`, and the key used for every lookup (`PbxExtensionModules::findFirstByUniqid()`).

Conventions observed across every production manifest:

* Always starts with the prefix `Module`.
* PascalCase, no spaces or punctuation.
* Matches the module's root directory name exactly.

For ModuleBlackList the directory is `Modules/ModuleBlackList/` and the value is `"ModuleBlackList"`.

{% hint style="danger" %}
`moduleUniqueID` is permanent. Changing it after release orphans the installed copy (the PBX keys everything off the old `uniqid`) and breaks marketplace identity. Choose it once.
{% endhint %}

### support\_email

`string`, required. Contact address for the module's author. Mapped to `PbxExtensionSetupBase::$support_email`. Used for support links in the web UI and the release pipeline; not otherwise interpreted by the PBX.

### version

`string`, required. The module's own version. In source control this is the literal placeholder `"%ModuleVersion%"` (the value in `ModuleTemplate`, `ModuleAmoCrm`, and every language pack). The build/release pipeline substitutes the real semantic version (for example `1.2.0`) into the packaged `module.json` before the ZIP is produced.

At runtime the parsed value is stored in `PbxExtensionSetupBase::$version` and written to the `version` column of `m_PbxExtensionModules`; the REST API surfaces it via the `version` field in `DataStructure.php`.

{% hint style="info" %}
Keep `"%ModuleVersion%"` verbatim in the repository. Do not hand-edit it to a number — the pipeline owns that substitution, and a hard-coded version will be overwritten or will desynchronise from your release tag.
{% endhint %}

### min\_pbx\_version

`string`, required. **The only author-declared compatibility constraint in the entire manifest.** It states the minimum MikoPBX version the module is allowed to install on.

It is mapped to `PbxExtensionSetupBase::$min_pbx_version` and enforced by `PbxExtensionSetupBase::checkCompatibility()`, which `installModule()` calls first — before license activation, before file installation, before DB migration. If the gate fails, installation aborts.

{% code title="Core/src/Modules/Setup/PbxExtensionSetupBase.php" %}
```php
public function checkCompatibility(): bool
{
    // Get the current PBX version from the settings.
    $currentVersionPBX = PbxSettings::getValueByKey(PbxSettings::PBX_VERSION);

    // Remove any '-dev' suffix from the version.
    $currentVersionPBX = str_replace('-dev', '', $currentVersionPBX);
    if (version_compare($currentVersionPBX, $this->min_pbx_version) < 0) {
        $this->messages[] = $this->translation->_(
            "ext_ModuleDependsHigherVersion",
            ['version' => $this->min_pbx_version]
        );
        return false;
    }

    // The current PBX version is compatible.
    return true;
}
```
{% endcode %}

Behaviour to note:

* The comparison is a standard PHP `version_compare(current, min) < 0`. The check is **one sided** — there is no upper bound (see [Dependencies are procedural, not declarative](module-json.md#dependencies-are-procedural-not-declarative)).
* A trailing `-dev` suffix on the running PBX version is stripped before comparison, so a developer build compares like its release counterpart.
* **Asymmetric fallback.** `min_pbx_version` is only meaningful if it is present. If the field is missing or empty, `PbxExtensionSetupBase::__construct()` assigns `''` (`$module_settings['min_pbx_version'] ?? ''`), and `version_compare(anything, '')` never returns `< 0` — so the compatibility gate effectively passes for any PBX version. Note that the property's class-level default is the string `'2024.2.3'`, but the constructor overwrites it with whatever the JSON contains (including the empty string). Always declare `min_pbx_version` explicitly; never rely on the default.

{% hint style="warning" %}
For a new module targeting this documentation's baseline, set `"min_pbx_version": "2025.1.1"`. The older values you will see in legacy manifests (`2022.2.1` in `ModuleAmoCrm`, `2023.2.150` in `ModuleTemplate`) reflect when those modules were first released — they are not a recommendation for new code.
{% endhint %}

### module\_type

`string`, optional, defaults to `"general"`. Categorises the module for the UI and marketplace. Parsed in `PbxExtensionSetupBase::__construct()` and stored both on the setup object (`public string $module_type`) and in the `module_type` column of `m_PbxExtensionModules` (default `'general'`, see `PbxExtensionModules.php`).

The values documented by Core (the doc-comment on `PbxExtensionSetupBase::$module_type`) are:

| Value          | Meaning                                 |
| -------------- | --------------------------------------- |
| `general`      | Default. Ordinary feature module.       |
| `languagepack` | Web-interface translation pack.         |
| `security`     | Security / firewall / hardening module. |
| `cti`          | CTI / telephony-integration module.     |
| `utility`      | Utility / tooling module.               |
| `call_feature` | Call-handling feature.                  |
| `ai`           | AI-related module.                      |

{% hint style="info" %}
The list is open-ended in code (the doc-comment ends with `...`), and the field is parsed as a free `string`. Across the production manifests only `languagepack` is set explicitly (in all 24 language packs); every other module omits the field and inherits `general`. Use a value from the table above so the UI categorises your module correctly.
{% endhint %}

ModuleBlackList is a call-filtering feature, so a reasonable choice is `"module_type": "call_feature"`.

## Commercial fields

These two fields appear together in a minority of manifests — currently 14, all commercial MIKO modules, e.g. `ModuleAmoCrm`). Omit both for a free/open module.

### lic\_product\_id

`integer`, optional. The MIKO licensing product identifier. Parsed in `PbxExtensionSetupBase::__construct()` into `public $lic_product_id`; defaults to `0` when absent. A non-zero value makes the module commercial: `activateLicense()` requires a valid PBX license key when `lic_product_id > 0`.

### lic\_feature\_id

`integer`, optional. The MIKO licensing feature identifier. Parsed into `public $lic_feature_id`; defaults to `0` when absent. When `lic_feature_id > 0`, `PbxExtensionState::enableModule()` calls `License::featureAvailable()` and refuses to enable the module (recording `disableReason = DISABLED_BY_LICENSE`) if the feature is not captured.

{% code title="Extensions/ModuleAmoCrm/module.json" %}
```json
{
  "developer": "MIKO",
  "moduleUniqueID": "ModuleAmoCrm",
  "support_email": "help@miko.ru",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2022.2.1",
  "lic_product_id": 81,
  "lic_feature_id": 45,
  "release_settings": {
    "publish_release": true,
    "changelog_enabled": true,
    "create_github_release": true
  },
  "wiki_links": {
    "ru": {"https://wiki.mikopbx.com/module-amo-crm": "https://docs.mikopbx.com/mikopbx/modules/miko/amocrm"},
    "en": {"https://wiki.mikopbx.com/module-amo-crm": "https://docs.mikopbx.com/mikopbx/modules/miko/amocrm"}
  }
}
```
{% endcode %}

See [Marketplace & licensing](../marketplace/licensing.md) for how these IDs are issued and how the activation/feature-capture flow works end to end.

## Documentation field

### wiki\_links

`object`, optional. Present in a large minority of manifests (currently 23). A map of documentation URLs keyed by language code (`ru`, `en`). Parsed in `PbxExtensionSetupBase::__construct()` into `public array $wiki_links` (only if the JSON value is an array/object). Surfaced as help links in the web UI. The inner shape is a map of `old-url => new-url` per language, as seen in `ModuleAmoCrm` above.

## Language-pack fields

Language packs (`module_type: "languagepack"`) carry two extra fields. Both appear in all 24 language-pack manifests and nowhere else.

### language\_code

`string`. The locale this pack provides, e.g. `"nl-nl"`, `"pt-br"`. Identifies the translation target inside the PBX. This top-level field **is read by Core at runtime** — not by `PbxExtensionSetupBase::__construct()`, but by `PbxExtensionUtils::getLanguagePackCode()` (in `Core/src/Modules/PbxExtensionUtils.php`), which returns this value (falling back to the single sub-directory name under `sounds/` when the field is absent).

### translation\_sync

`object`. Configuration for the tooling that syncs translation strings from the Core repository. **Consumed by the release/translation tooling, not by the PBX runtime** — there is no reference to it in `PbxExtensionSetupBase` or `PbxExtensionState`. Observed shape (from `ModuleDutchLanguagePack`):

{% code title="Extensions/LanguagePacks/ModuleDutchLanguagePack/module.json" %}
```json
{
  "developer": "MIKO",
  "moduleUniqueID": "ModuleDutchLanguagePack",
  "support_email": "help@miko.ru",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2026.2.20",
  "module_type": "languagepack",
  "language_code": "nl-nl",
  "translation_sync": {
    "enabled": true,
    "source_repo": "mikopbx/Core",
    "source_branch": "develop",
    "language_code": "nl",
    "exclude_files": ["ModuleDutchLanguagePack.php"]
  },
  "release_settings": {
    "publish_release": true,
    "changelog_enabled": true,
    "create_github_release": true
  }
}
```
{% endcode %}

`translation_sync` keys observed across all 24 packs: `enabled` (`bool`), `source_repo` (`string`), `source_branch` (`string`), `language_code` (`string`, the short code used by the sync, distinct from the top-level `language_code`), and `exclude_files` (array of file names to skip).

## release\_settings

`object`, optional. Present in most manifests (currently 49), always with the identical shape:

```json
"release_settings": {
  "publish_release": true,
  "changelog_enabled": true,
  "create_github_release": true
}
```

This block is **build-pipeline metadata only**. No Core class reads it; the PBX neither parses nor validates it. It tells the release tooling whether to publish, generate a changelog, and cut a GitHub release. A module without it installs and runs identically. See [Building & packaging a module](module-installer.md) for the release flow.

## Dependencies are procedural, not declarative

This is the most common point of confusion, so it is stated plainly:

{% hint style="danger" %}
**There is no `dependencies` array and no `max_pbx_version` field in any `module.json`.** Verified across all 60 production manifests — neither key appears anywhere. MikoPBX has **no declarative dependency graph** for modules.
{% endhint %}

### How inter-module relationships are actually enforced

Module-to-module dependencies are handled **procedurally at runtime**, through two mechanisms, never through a declared graph:

1.  **Lifecycle hooks on the module's config class.** `PbxExtensionState::enableModule()` and `disableModule()` invoke optional hooks on your `BlackListConf` config class when they exist (checked with `method_exists`). The hook names are defined as constants in `Core/src/Modules/Config/SystemConfigInterface.php`:

    * `onBeforeModuleEnable` (`SystemConfigInterface::ON_BEFORE_MODULE_ENABLE`)
    * `onAfterModuleEnable` (`SystemConfigInterface::ON_AFTER_MODULE_ENABLE`)
    * `onBeforeModuleDisable` (`SystemConfigInterface::ON_BEFORE_MODULE_DISABLE`)
    * `onAfterModuleDisable` (`SystemConfigInterface::ON_AFTER_MODULE_DISABLE`)

    If module A needs module B, A's `onBeforeModuleEnable` is where it checks B's presence/state and refuses or adjusts. There is no manifest field that does this for you.
2. **Model relations + delete guards.** `PbxExtensionState::makeBeforeDisableTest()` walks every model under `Modules/<ModuleUniqueID>/Models/`, and for models that declare Phalcon relations it runs `beforeDelete()` against existing records. If another entity still references the module's data, the delete is blocked and disable fails. This is how the PBX prevents, for example, disabling a module whose records are still referenced by the `Extensions` table — entirely at runtime, via the ORM, with no declared dependency.

### What about max\_pbx\_version?

`max_pbx_version` exists **only as marketplace/release-server metadata**. You can see it in `Core/src/PBXCoreREST/Lib/Modules/DataStructure.php`: it appears in the API data structure built from **repository** data (`createFromRepositoryData()`, the available-modules listing) and in the read-only OpenAPI `detail` schema. It is a `readOnly` response field populated by the release server — **not** something an author writes into `module.json`, and **not** something `checkCompatibility()` consults. The compatibility gate enforced on-device is purely the lower bound (`min_pbx_version`).

{% hint style="info" %}
Practical takeaway: declare a single honest `min_pbx_version`, and implement any cross-module requirements in your config class hooks and model relations. Do not look for (or invent) a `dependencies` or `max_pbx_version` field — they will be ignored.
{% endhint %}

## Field summary

| Field                 | Type    | Required | Parsed by PBX | Where it goes / who reads it                                               |
| --------------------- | ------- | -------- | ------------- | -------------------------------------------------------------------------- |
| `developer`           | string  | yes      | yes           | `PbxExtensionSetupBase::$developer`, `m_PbxExtensionModules`               |
| `moduleUniqueID`      | string  | yes      | yes           | directory, namespace, `uniqid` column, all lookups                         |
| `support_email`       | string  | yes      | yes           | `PbxExtensionSetupBase::$support_email`                                    |
| `version`             | string  | yes      | yes           | `$version`; `%ModuleVersion%` placeholder filled by pipeline               |
| `min_pbx_version`     | string  | yes      | yes           | `checkCompatibility()` lower-bound gate                                    |
| `module_type`         | string  | no       | yes           | `$module_type` / `module_type` column (default `general`)                  |
| `lic_product_id`      | integer | no       | yes           | `$lic_product_id`; gates `activateLicense()`                               |
| `lic_feature_id`      | integer | no       | yes           | `$lic_feature_id`; gates `enableModule()` feature capture                  |
| `wiki_links`          | object  | no       | yes           | `$wiki_links`; UI help links                                               |
| `language_code`       | string  | no\*     | yes           | `PbxExtensionUtils::getLanguagePackCode()` (\*required for language packs) |
| `translation_sync`    | object  | no\*     | no            | translation tooling (\*language packs only)                                |
| `release_settings`    | object  | no       | no            | release/build pipeline only                                                |
| ~~`dependencies`~~    | —       | —        | —             | **does not exist**; relationships are procedural                           |
| ~~`max_pbx_version`~~ | —       | —        | —             | **not an author field**; marketplace metadata only                         |

## Related pages

* [Building & packaging a module](module-installer.md) — `installModule()`, `checkCompatibility()`, and the release pipeline that fills `%ModuleVersion%`.
* [Template module structure](template-module-structure.md) — where `module.json` sits and the config/main/model classes it ties together.
* [Marketplace & licensing](../marketplace/licensing.md) — `lic_product_id` / `lic_feature_id` and the activation flow.
