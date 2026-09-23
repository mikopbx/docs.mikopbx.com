---
description: 'Module localization: Messages files and the 29-language translation workflow.'
---

# Translations

Every string a MikoPBX module shows to a user — page titles, field labels, validation prompts, REST endpoint descriptions — is a **translation key**, not a hard-coded string. The module ships a `Messages/` folder with one PHP file per locale; the Core merges those files into a single global translation array at request time and resolves keys on the server (Volt, models) and in the browser (JavaScript).

Throughout this page the running example is **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_ModuleBlackList_BlackListNumbers`, front-end assets named per action — `module-black-list-index.js`, `module-black-list-modify.js`). Its translation keys use the prefix `module_black_list_`.

{% hint style="info" %}
The single source of truth is **Russian** (`ru.php`). Developers add and edit keys **only** in `ru.php`; every other locale is produced from it by the `/translations` workflow (see below). This is enforced by the MikoPBX `translations` skill from [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills).
{% endhint %}

## The `Messages/` directory

A feature module keeps its translations in a flat directory — **one file per locale**, each returning a `key => translation` array:

```
Extensions/ModuleBlackList/Messages/
├── ru.php          ⭐ source of truth — edit ONLY this
├── en.php
├── de.php
├── es.php
├── ...
├── zh_Hans.php
└── languages.php   key scaffold (see below) — not a runtime locale
```

This flat layout is the standard structure for an ordinary feature module. Every shipped example uses it:

* `Extensions/ModuleTemplate/Messages/` — the canonical starter set.
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Messages/` — a web-interface module (ships `ru.php` + `en.php`).
* `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Messages/` — a REST module (ships `ru.php` + `en.php`).
* `Extensions/ModuleUsersGroups/Messages/` — a full production module with the complete locale set.

{% hint style="info" %}
The Core loader also supports a **directory-per-locale** form (`Messages/<lang>/Feature.php`, `Messages/<lang>/Common.php`, …) used by Language Pack modules that split thousands of keys across multiple files. For a normal feature module the flat `Messages/<lang>.php` form is what you want. The loader tries the directory form first and falls back to the flat file — see `MessagesProvider::loadModuleTranslations()` in `Core/src/Common/Providers/MessagesProvider.php`.
{% endhint %}

### Anatomy of a locale file

A locale file is a plain PHP file that `return`s an associative array. Here is `ru.php` for ModuleBlackList — the source of truth you edit by hand:

{% code title="Extensions/ModuleBlackList/Messages/ru.php" %}
```php
<?php
return [
    // Model representation label (see "Model keys" below) — keyed on the
    // module UniqueID, %represent% is filled with the linked module name
    'repModuleBlackList'                    => 'Черный список - %represent%',

    // Model title used by getRepresent() — mo_<UniqueID>, single "Module"
    'mo_ModuleBlackList'                    => 'Черный список',

    // Page chrome. BreadcrumbModuleBlackList is ALSO the left-menu caption:
    // PbxExtensionSetupBase::addToSidebar() stores "Breadcrumb$moduleUniqueID"
    // as the menu item caption.
    'BreadcrumbModuleBlackList'             => 'Черный список номеров',
    'SubHeaderModuleBlackList'              => 'Блокировка входящих вызовов по номеру',

    // UI labels — prefix module_black_list_
    'module_black_list_AddNewRecord'        => 'Добавить номер',
    'module_black_list_NumberFieldLabel'    => 'Телефонный номер',
    'module_black_list_DescriptionLabel'    => 'Комментарий',
    'module_black_list_EnabledLabel'        => 'Включить блокировку',

    // Validation prompts (read from JavaScript)
    'module_black_list_ValidateNumberEmpty' => 'Укажите номер для блокировки',
    'module_black_list_ValidateNumberBad'   => 'Номер содержит недопустимые символы',
];
```
{% endcode %}

The matching `en.php` has identical keys, with the values translated:

{% code title="Extensions/ModuleBlackList/Messages/en.php" %}
```php
<?php
return [
    'repModuleBlackList'                    => 'Black list - %represent%',
    'mo_ModuleBlackList'                    => 'Black list',
    'BreadcrumbModuleBlackList'             => 'Black list of numbers',
    'SubHeaderModuleBlackList'              => 'Block inbound calls by number',
    'module_black_list_AddNewRecord'        => 'Add number',
    'module_black_list_NumberFieldLabel'    => 'Phone number',
    'module_black_list_DescriptionLabel'    => 'Comment',
    'module_black_list_EnabledLabel'        => 'Enable blocking',
    'module_black_list_ValidateNumberEmpty' => 'Enter the number to block',
    'module_black_list_ValidateNumberBad'   => 'The number contains invalid characters',
];
```
{% endcode %}

See the real, copy-ready pair in `Extensions/ModuleTemplate/Messages/ru.php` and `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Messages/en.php`.

### `languages.php` is a key scaffold, not a locale

`Messages/languages.php` looks like a locale file, but it is **not loaded at runtime**. The loader (`MessagesProvider::loadModuleTranslations()`) only reads `Messages/<langcode>.php` for codes the Core actually knows about, and `languages` is not a language code — so it is silently ignored when building the translation array.

What it actually is: a developer-facing **template** listing every translation key the module defines, with empty string values:

{% code title="Extensions/ModuleTemplate/Messages/languages.php" %}
```php
<?php
return [
    'repModuleTemplate' => '',
    'mo_ModuleModuleTemplate' => '',
    'BreadcrumbModuleTemplate' => '',
    'SubHeaderModuleTemplate' => '',
    'module_template_AddNewRecord' => '',
    // ... one empty entry per key
];
```
{% endcode %}

It serves as a quick "what keys exist" index and a scaffold to copy when seeding a new locale. It does not affect what the user sees.

## Where keys are referenced

The merged translation array is exposed in three places. Use the same key string in all of them.

### 1. Volt templates — `t._('key')`

In `.volt` views, translate with the `t` (translation) service:

```twig
<div class="ui dividing header">{{ t._('BreadcrumbModuleBlackList') }}</div>
{{ t._('module_black_list_AddNewRecord') }}
```

Real usage: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/AdditionalPage/index.volt`.

### 2. JavaScript — `globalTranslate.key`

The Core injects the active locale's translation array into the page as the global `globalTranslate` object. Reference any key as a property:

```javascript
/* global globalTranslate, Form */

const validateRules = {
    number: {
        identifier: 'number',
        rules: [{
            type: 'empty',
            prompt: globalTranslate.module_black_list_ValidateNumberEmpty,
        }],
    },
};
```

Real usage: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/public/assets/js/module-example-form-modify.js`.

{% hint style="warning" %}
`globalTranslate` is a JavaScript object: `globalTranslate.someKey` resolves to the string, and a **missing key is `undefined`**, not the key name. Declare it in the file's `/* global ... */` comment so the linter does not flag it.
{% endhint %}

### 3. Models — `rep…` representation keys

A module model's human-readable label (shown in lists, deletion warnings, related-record links) comes from a `rep<ModuleUniqueID>` key resolved through the model's `getRepresent()` method. Module models extend `ModulesModelsBase`, which overrides `getRepresent()` in `Core/src/Modules/Models/ModulesModelsBase.php`. When called with a link (`getRepresent(true)`), it resolves:

```php
$this->t('rep' . $this->moduleUniqueId, [
    'represent' => "<a href='$link'>$name</a>",
]);
```

So for ModuleBlackList (UniqueID `ModuleBlackList`) the key is **`repModuleBlackList`** — keyed on the module's UniqueID, not on the model or any field. The `%represent%` placeholder is filled with the linked module name (an `<a>` tag), producing e.g. `Black list - <a href="…">Black list</a>`. Without a link, `getRepresent()` returns the `mo_<ModuleUniqueID>` title instead and the `rep…` key is not used.

{% hint style="info" %}
The Core trait `Core/src/Common/Models/Traits/RecordRepresentationTrait.php` holds the `getRepresent()` switch for **built-in** Core models only; module models get their representation from the `ModulesModelsBase` override described above.
{% endhint %}

{% hint style="danger" %}
Use the placeholder spelled **`%represent%`** — that is the name the Core passes (`['represent' => …]`). Some shipped starter files (`ModuleTemplate`, `ModuleUsersGroups`) contain the misspelling `%repesent%` and even leftover demo text like "Module amoCRM"; do **not** copy those into your module. Match the placeholder name the Core supplies, or the substitution will not happen.

The same starter files also ship the model-title key with a doubled prefix — `mo_ModuleModuleTemplate`, `mo_ModuleModuleUsersGroups`. The Core reads `mo_` **+ moduleUniqueID** (`ModulesModelsBase::getRepresent()`, line 75), i.e. `mo_ModuleTemplate` / `mo_ModuleUsersGroups`, so the doubled keys are dead weight and the key that is actually looked up is missing. Use the single form.
{% endhint %}

## Key naming conventions

| Key shape                        | Used for                                                                               | Example                                 |
| -------------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------- |
| `module_{feature}_{Name}`        | UI labels, buttons, validation prompts                                                 | `module_black_list_AddNewRecord`        |
| `mo_{ModuleUniqueID}`            | Model title used by `getRepresent()`                                                   | `mo_ModuleBlackList`                    |
| `Breadcrumb{Page}`               | Breadcrumb / page heading — and the left-menu caption for `Breadcrumb{ModuleUniqueID}` | `BreadcrumbModuleBlackList`             |
| `SubHeader{Page}`                | Sub-header descriptive text                                                            | `SubHeaderModuleBlackList`              |
| `rep{ModuleUniqueID}`            | Model representation (with `%represent%`)                                              | `repModuleBlackList`                    |
| `module_{feature}_Validate{...}` | Validation messages read from JS                                                       | `module_black_list_ValidateNumberEmpty` |

The prefix convention `module_{feature}_` keeps a module's keys namespaced so they never collide with the Core's keys or another module's. For ModuleBlackList the feature prefix is `module_black_list_`. REST-API modules add their own families (`rest_`, `rest_tag_`, `rest_param_`, `rest_schema_`, …) — see the full set in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Messages/en.php`.

## The `/translations` workflow (26 languages)

MikoPBX ships interface translations for a broad set of languages. The authoritative **active** set is the constant `LanguageProvider::AVAILABLE_LANGUAGES` in `Core/src/Common/Providers/LanguageProvider.php` — currently these 26 codes:

```
en  ru  de  es  el  fr  pt  pt_BR  uk  ka  it  da  nl  pl  sv
cs  tr  ja  vi  az  ro  th  hu  fi  hr  zh_Hans
```

The starter `ModuleTemplate/Messages/` directory ships **28** locale files (the 26 above plus `he` and `fa`, which are harmless extras kept for historical reasons) alongside the `languages.php` scaffold. The `translations` skill works on the same 26 active codes and ignores the two extras — for what is actually rendered, trust `AVAILABLE_LANGUAGES`. New locales are added to a module by adding the matching `Messages/<code>.php` file for any code present in that constant.

The end-to-end process is owned by the `/translations` skill (`skills/translations/SKILL.md` in [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills)):

1. **Collect keys in `ru.php` first.** Add or change every key in Russian only, with the correct `module_{feature}_` prefix. Russian is the baseline against which all other files are validated.
2. **Translate file-by-file, language-by-language.** Complete one locale fully (analyse → translate the missing keys → merge → validate) before starting the next. Never edit several files at once.
3. **Preserve existing translations.** Only the _missing_ keys are translated; correct existing values are left untouched.
4. **Validate key count and structure after every locale.** Each locale file must have the **exact same key count, the same keys, and the same array structure** as `ru.php`. If Russian has 47 keys, every other locale must have exactly 47. A mismatch stops the process.
5. **Keep `%placeholder%` names identical.** Placeholders use the `%name%` form (e.g. `%represent%`, `%length%`); the placeholder names must match the Russian source verbatim in every locale.
6. **Do not translate technical terms.** Keep `SIP`, `IAX`, `AMI`, `PJSIP`, `NAT`, `STUN`, `TURN`, `RTP`, `CDR`, `IVR`, `DID`, `CID`, `DTMF`, `codec`, `trunk`, `extension`, `IP`, `DNS`, `VPN` unchanged in every language.
7. **Escape quotes for PHP.** A single quote inside a single-quoted value must be escaped: `'He said: "Don\'t"'`.

{% hint style="info" %}
**Validation commands.** Confirm a locale matches the Russian source:

```bash
# Key count for one file
php -r "echo count(include 'Extensions/ModuleBlackList/Messages/ru.php');"

# Compare counts across locales
for lang in en de es fr; do
  echo "$lang: $(php -r "echo count(include 'Extensions/ModuleBlackList/Messages/$lang.php');")"
done

# Syntax check
php -l Extensions/ModuleBlackList/Messages/ru.php
```
{% endhint %}

### Common mistakes

```php
// ❌ Inconsistent placeholder name (ru has %represent%)
'repModuleBlackList' => 'Black list - %value%',
// ✅ Same placeholder name in every locale
'repModuleBlackList' => 'Black list - %represent%',

// ❌ Translated technical term
'module_black_list_SipHint' => 'Configuración del proveedor СИП',
// ✅ SIP stays SIP
'module_black_list_SipHint' => 'Configuración del proveedor SIP',

// ❌ Unescaped quote breaks PHP
'module_black_list_Msg' => 'Don't block this',
// ✅ Escaped
'module_black_list_Msg' => 'Don\'t block this',
```

## How the Core assembles translations

You do not call any loader yourself — `MessagesProvider` in `Core/src/Common/Providers/MessagesProvider.php` does it on each request:

1. Loads the Core's English strings as a base.
2. Merges the Core's strings for the active language (if not English).
3. Merges **English** strings from every module's `Messages/` (so an untranslated key always has _some_ value).
4. Merges the active language's strings from every module's `Messages/`.
5. Merges the static language-name array from `LanguageProvider` (the `ex_English`, `ex_Russian`, … keys).
6. Caches the result (`ManagedCache`, key `LocalisationArray:<versionHash>:<lang>`), but only for web requests — under CLI (`php_sapi_name() === 'cli'`) no cache key is built and the array is rebuilt on every run.

Because step 3 always loads module English first, **`en.php` is your safety net**: any key missing from a locale falls back to English rather than showing a raw key. Keep `en.php` complete.

{% hint style="warning" %}
Translations are **cached**. After editing `Messages/*.php` you must clear the cache before the change appears, then hard-refresh the browser (`Ctrl+Shift+R` / `Cmd+Shift+R`).

The managed cache lives in **Redis database 4** (`ManagedCacheProvider::DATABASE_INDEX`), not database 0, so you must pass `-n 4` — a bare `redis-cli FLUSHDB` targets database 0 and clears nothing:

```bash
# Drop just the translation arrays
redis-cli -n 4 --scan --pattern '_PH_MANAGED_CACHE:*LocalisationArray*' | \
  xargs -r redis-cli -n 4 DEL

# …or flush the whole managed cache
redis-cli -n 4 FLUSHDB
```

The cached array is also keyed on the module version hash, so bumping the module version invalidates it. Restarting `php-fpm` (`monit restart php-fpm`) does **not** clear the cache — the entries live in Redis, not in the PHP process.
{% endhint %}

## Checklist before shipping

* [ ] All new/changed keys exist in `ru.php` with the `module_{feature}_` prefix.
* [ ] Every locale file has the **same key count and keys** as `ru.php`.
* [ ] Placeholder names (`%represent%`, …) match `ru.php` in every locale.
* [ ] Technical terms (SIP, IAX, PJSIP, CDR, IVR, …) are left untranslated.
* [ ] `en.php` is complete (it is the fallback for missing keys).
* [ ] Every file passes `php -l`.
* [ ] Cache cleared and the result verified in the UI.

## Related pages

* [Module interface (Messages folder)](module-interface-empty.md) — where the `Messages/` directory sits in the overall module layout.
* [What the AI generator produces](../ai-assisted-development/what-it-generates.md) — the generator chains the `/translations` skill as its **final** step, emitting `ru.php` first and then the full locale set automatically.
