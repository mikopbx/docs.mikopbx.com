---
description: Recipes for building module admin forms and data tables.
---

# Forms

This chapter collects task-oriented recipes for building the **admin-cabinet UI** of a MikoPBX module: settings forms that read and write a model, server-paginated data grids backed by a REST API v3 endpoint, and surgical changes to forms that already ship with the core.

Every recipe is anchored to a real, working example module — `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` — and uses primary-source APIs from the core `BaseForm` class. Throughout the cookbook we thread a single fictional module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`), so the recipes read as one continuous story.

{% hint style="info" %}
**Asset names are per-action, not per-module.** The cabinet loads `module-<kebab-module-name>-<action>.js` and the matching `.css` — so ModuleBlackList's edit page uses `module-black-list-modify.js` and its list page uses `module-black-list-index.js`. There is no single `module-black-list.js`. The example module follows the same rule with `module-example-form-modify.js` / `module-example-form-index.js`.
{% endhint %}

## What a module form is made of

A module admin page is wired together from five collaborating pieces. The example module ships one of each, so use it as your skeleton:

| Piece              | Responsibility                                                                      | Real example                                                                                            |
| ------------------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Phalcon form class | Declares the fields and their types                                                 | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Forms/ModuleExampleFormForm.php`                |
| Controller         | Loads the model, builds dropdown options, renders the form, handles `save`/`delete` | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Controllers/ModuleExampleFormController.php`    |
| Volt view          | HTML template that prints the form                                                  | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/ModuleExampleForm/`                       |
| JavaScript module  | Client-side validation and AJAX submit/grid logic                                   | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/public/assets/js/src/module-example-form-modify.js` |
| Model              | The table the form persists to                                                      | `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Models/ModuleExampleForm.php`                       |

The form class is the heart of the page. It extends `MikoPBX\AdminCabinet\Forms\BaseForm`, which adds MikoPBX-specific element helpers on top of the standard `Phalcon\Forms\Form`:

{% code title="App/Forms/ModuleExampleFormForm.php" %}
```php
namespace Modules\ModuleExampleForm\App\Forms;

use MikoPBX\AdminCabinet\Forms\BaseForm;
use Phalcon\Forms\Element\Hidden;
use Phalcon\Forms\Element\Numeric;
use Phalcon\Forms\Element\Text;

class ModuleExampleFormForm extends BaseForm
{
    public function initialize($entity = null, $options = null): void
    {
        parent::initialize($entity, $options);

        // Hidden primary key — always present, never shown to the user
        $this->add(new Hidden('id', ['value' => $entity->id]));

        // Plain Phalcon elements work directly via add()
        $this->add(new Text('text_field'));
        $this->add(new Numeric('integer_field', ['maxlength' => 4, 'defaultValue' => 3]));

        // BaseForm helpers add MikoPBX-styled widgets
        $this->addTextArea('text_area_field', $entity->text_area_field ?? '', 90);
        $this->addCheckBox('checkbox_field', intval($entity->checkbox_field) === 1);
        $this->addSemanticUIDropdown(
            'select_field',
            $options['priorities'] ?? [],
            $entity->select_field ?? 'medium'
        );
    }
}
```
{% endcode %}

{% hint style="info" %}
**The three helpers `BaseForm` adds.** Confirmed against `MikoPBX/AdminCabinet/Forms/BaseForm.php`:

* `addTextArea(string $areaName, string $areaValue, int $areaWidth = 90, array $options = [])` — multi-line input with auto-height.
* `addCheckBox(string $fieldName, bool $checked, string $checkedValue = 'on')` — checkbox or toggle (the `toggle` CSS class is chosen in the Volt view).
* `addSemanticUIDropdown(string $name, array $options = [], $value = null, array $attributes = [])` — a Fomantic UI dropdown, fed either static options or rows from a model. Note it is declared **`protected`** (`BaseForm.php:114`), so you can only call it from inside your own form subclass, not from a controller.

For plain `Text`, `Password`, `Numeric`, and `Hidden` fields just call the inherited `$this->add(new Element(...))` — no helper needed.
{% endhint %}

## The three recipes

### 1. Create a module settings form

Build a single-record settings page (one row in the module table) from scratch: declare the form class, load the model in the controller, render it in a Volt view, and submit it over AJAX with client-side validation. This is the `base + ui` recipe combination described in the core skill's `reference/recipes.md`.

➡️ [Create a module settings form](create-module-form.md)

### 2. Create a server-paginated datatable

When a module manages **many** rows (blacklisted numbers, log entries, queued tasks) you render a Fomantic/DataTables grid whose pages are fetched from the server instead of loading every row into the browser. The grid calls a REST API v3 `getList` endpoint that returns a `PBXApiResult`. The list action in `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/GetListAction.php` is the canonical shape to copy:

{% code title="Lib/RestAPI/.../Actions/GetListAction.php" %}
```php
use MikoPBX\PBXCoreREST\Lib\PBXApiResult;

class GetListAction
{
    public static function main(array $data): PBXApiResult
    {
        $result = new PBXApiResult();
        $result->success = true;
        // Real implementation queries the model and applies limit/offset from $data:
        //   $result->data = BlackListNumbers::find([...])->toArray();
        $result->data = [
            ['id' => 1, 'title' => 'Example Task 1', 'status' => 'pending'],
        ];
        return $result;
    }
}
```
{% endcode %}

➡️ [Create a server-paginated datatable](create-datatable.md)

### 3. Add a field to an existing form

Sometimes you do not own the form — you want to inject one extra field into a **core** page (for example, add a setting to the SIP extension form) without patching core files. MikoPBX exposes a WebUI hook, `WebUIConfigInterface::ON_BEFORE_FORM_INITIALIZE`, which `BaseForm::initialize()` dispatches to every enabled module via `PBXConfModulesProvider::hookModulesMethod()`. Your module implements the hook, receives the live form plus its entity, and calls `$form->add(...)`.

➡️ [Add a field to an existing form](add-field-into-existing-form.md)

## Compiling the JavaScript

The browser-side code lives as ES6+ source under `public/assets/js/src/` (for the example: `module-example-form-modify.js` and `module-example-form-index.js`). MikoPBX serves the **transpiled** copy from `js/cache/<moduleUniqueID>/...`, which the controller registers on the footer asset collection:

{% code title="App/Controllers/ModuleExampleFormController.php" %}
```php
$footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
$footerCollectionJS
    ->addJs('js/pbx/main/form.js', true)
    ->addJs('js/cache/' . $this->moduleUniqueID . '/module-example-form-modify.js', true);
```
{% endcode %}

`js/cache/<moduleUniqueID>` is **not** a directory the build writes into — it is a symlink the installer creates pointing at your module's `public/assets/js` (`PbxExtensionUtils::createAssetsSymlinks()`, `Core/src/Modules/PbxExtensionUtils.php:105-114`; the same is done for `css` and `img`). So the URL `js/cache/ModuleBlackList/module-black-list-modify.js` resolves to `<moduleDir>/public/assets/js/module-black-list-modify.js`.

That means Babel's output directory is `public/assets/js` — the **parent** of `src/`, not a `cache` subfolder:

{% code title="bash" %}
```bash
babel public/assets/js/src/module-black-list-modify.js \
  --out-dir public/assets/js \
  --source-maps inline \
  --presets airbnb
```
{% endcode %}

{% hint style="warning" %}
Editing the `src/` file alone changes nothing in the running cabinet — the page loads the compiled file from `js/cache/`. Re-run the Babel step (or your build task) after every JavaScript edit, and clear the browser cache if the new code does not appear.
{% endhint %}

## Where to go next

* Start from an empty interface scaffold: [Module interface (empty)](../../module-developement/module-interface-empty.md).
* Then follow the recipes above in order — most modules implement recipe 1, add recipe 2 when they manage a list, and reach for recipe 3 only to extend a core page.
