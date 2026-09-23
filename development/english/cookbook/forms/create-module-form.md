---
description: 'Step-by-step: build a settings form for your module.'
---

# Create module form

This recipe walks through building a complete, persistable settings form for a MikoPBX module: a database model, a Phalcon form class, a controller that renders and saves the form, a Volt view, and a babel-compiled JavaScript controller that wires up validation and AJAX submission.

The narrative uses a fictional module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`, JS file `module-black-list-modify.js` — assets are named per **action**, not per module). Every pattern is anchored to the real, working example module that ships with MikoPBX:

{% hint style="success" %}
**Verbatim anchor:** `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`. The example module exercises **every** form element type covered here. When in doubt, read its files — they are the source of truth this page is built from.
{% endhint %}

Before you start, you should already have a module skeleton (see [module-interface-empty.md](../../module-developement/module-interface-empty.md)) and understand how MikoPBX models map to SQLite tables (see [data-model.md](../../module-developement/data-model.md)). For the full list of form recipes, see the [forms cookbook README](./).

## The five pieces

A settings form is made of five files that all reference each other through shared field names. Keeping the names identical across all layers is the single most important rule:

| Layer      | File (in `ModuleExampleForm`)                        | Responsibility                                                  |
| ---------- | ---------------------------------------------------- | --------------------------------------------------------------- |
| Model      | `Models/ModuleExampleForm.php`                       | Defines columns with `@Column`, maps to a `m_*` table.          |
| Form       | `App/Forms/ModuleExampleFormForm.php`                | Declares form elements whose names **match the model columns**. |
| Controller | `App/Controllers/ModuleExampleFormController.php`    | `modifyAction` builds the form, `saveAction` persists it.       |
| View       | `App/Views/ModuleExampleForm/modify.volt`            | Renders each element with `form.render('field')`.               |
| JavaScript | `public/assets/js/src/module-example-form-modify.js` | Validation rules + AJAX submit via the global `Form` object.    |

## Step 1 — Define the model

The model declares one property per column. Each property carries a Phalcon `@Column` annotation, the primary key is untyped, and the table name is set in `initialize()` via `setSource()`. Module models extend `ModulesModelsBase`.

For **ModuleBlackList** the model would be `BlackListNumbers` mapping to `m_BlackListNumbers`. The example module uses generic field names so it can demonstrate every element type:

{% code title="Models/ModuleExampleForm.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleExampleForm\Models;

use MikoPBX\Common\Models\Providers;
use MikoPBX\Modules\Models\ModulesModelsBase;
use Phalcon\Mvc\Model\Relation;

class ModuleExampleForm extends ModulesModelsBase
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
    public ?string $text_field = '';

    /**
     * @Column(type="string", nullable=true)
     */
    public ?string $text_area_field = '';

    /**
     * @Column(type="string", nullable=true)
     */
    public ?string $password_field = '';

    /**
     * @Column(type="integer", default="3", nullable=true)
     */
    public ?string $integer_field = '3';

    /**
     * @Column(type="integer", default="1", nullable=true)
     */
    public ?string $checkbox_field = '1';

    /**
     * @Column(type="integer", default="1", nullable=true)
     */
    public ?string $toggle_field = '1';

    /**
     * @Column(type="string", nullable=true)
     */
    public ?string $select_field = 'medium';

    /**
     * @Column(type="string", nullable=true)
     */
    public ?string $provider_field = '';

    /**
     * @Column(type="string", nullable=true)
     */
    public ?string $hidden_field = '';

    public function initialize(): void
    {
        $this->setSource('m_ModuleExampleForm');
        parent::initialize();
    }
}
```
{% endcode %}

{% hint style="info" %}
Phalcon/SQLite conventions: the primary key property is **untyped** (`public $id;`), string columns are `public ?string $name = '';`, and even integer/boolean columns are declared as `?string` and stored as the strings `'0'` / `'1'`. This matters in `saveAction` below, where checkbox values are normalized to `'1'` / `'0'`.
{% endhint %}

The full example model also declares a `hasOne` relation to `Providers` and an optional `getDynamicRelations()` block — see the complete file in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Models/ModuleExampleForm.php` and the model recipe in [data-model.md](../../module-developement/data-model.md).

## Step 2 — Build the form class

The form extends `MikoPBX\AdminCabinet\Forms\BaseForm` and overrides `initialize($entity = null, $options = null)`. **Always call `parent::initialize($entity, $options)` first.**

`BaseForm::initialize()` (`Core/src/AdminCabinet/Forms/BaseForm.php:37-46`) is six lines long and does exactly two things: it substitutes a `stdClass` when `$entity` is `null` (so `$entity->field` never fatals on a fresh record), and it fires the `WebUIConfigInterface::ON_BEFORE_FORM_INITIALIZE` hook across every enabled module via `PBXConfModulesProvider::hookModulesMethod()`. It does **not** set up a CSRF token or any other shared configuration.

That second point is the reason the call is mandatory rather than merely polite: your `parent::initialize()` call is the _only_ thing that lets other modules extend your form (see [add a field to an existing form](add-field-into-existing-form.md)). Skip it and the extension point silently disappears.

`BaseForm` provides three convenience helpers on top of the plain Phalcon elements:

| Helper                  | Signature (from `Core/src/AdminCabinet/Forms/BaseForm.php`)                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| `addTextArea`           | `addTextArea(string $areaName, string $areaValue, int $areaWidth = 90, array $options = []): void`      |
| `addCheckBox`           | `addCheckBox(string $fieldName, bool $checked, string $checkedValue = 'on'): void`                      |
| `addSemanticUIDropdown` | `addSemanticUIDropdown(string $name, array $options = [], $value = null, array $attributes = []): void` |

Plain Phalcon elements — `Text`, `Password`, `Numeric`, `Hidden` — are added with the standard `$this->add(new ...)`. Each element name **must equal a model column**.

{% code title="App/Forms/ModuleExampleFormForm.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleExampleForm\App\Forms;

use MikoPBX\AdminCabinet\Forms\BaseForm;
use Phalcon\Forms\Element\Hidden;
use Phalcon\Forms\Element\Numeric;
use Phalcon\Forms\Element\Password;
use Phalcon\Forms\Element\Text;

class ModuleExampleFormForm extends BaseForm
{
    public function initialize($entity = null, $options = null): void
    {
        parent::initialize($entity, $options);

        // Hidden primary key — always present, never shown to the user
        $this->add(new Hidden('id', ['value' => $entity->id]));

        // Text input
        $this->add(new Text('text_field'));

        // TextArea: name, value, width, extra attributes
        $this->addTextArea(
            'text_area_field',
            $entity->text_area_field ?? '',
            90,
            ['placeholder' => "Line 1 of placeholder\nLine 2 shows auto-height"]
        );

        // Password input
        $this->add(new Password('password_field'));

        // Numeric input with maxlength and inline width
        $this->add(new Numeric('integer_field', [
            'maxlength'    => 4,
            'style'        => 'width: 100px;',
            'defaultValue' => 3,
        ]));

        // Checkbox — second arg is the boolean "checked" state
        $this->addCheckBox('checkbox_field', intval($entity->checkbox_field) === 1);

        // Toggle — same PHP element as a checkbox; the toggle look comes
        // from the Volt CSS class (see Step 4)
        $this->addCheckBox('toggle_field', intval($entity->toggle_field) === 1);

        // Dropdown with STATIC options supplied by the controller
        $this->addSemanticUIDropdown(
            'select_field',
            $options['priorities'] ?? [],
            $entity->select_field ?? 'medium',
            ['placeholder' => $options['selectPlaceholder'] ?? '']
        );

        // Dropdown populated from the DATABASE (built in the controller)
        $this->addSemanticUIDropdown(
            'provider_field',
            $options['providers'] ?? [],
            $entity->provider_field ?? '',
            ['placeholder' => $options['providerPlaceholder'] ?? '']
        );

        // Hidden field whose value is set later by JavaScript
        $this->add(new Hidden('hidden_field', ['value' => $entity->hidden_field ?? '']));
    }
}
```
{% endcode %}

{% hint style="info" %}
**Checkbox vs. Toggle.** Both are created with `addCheckBox()` — there is no separate toggle element. The visual difference is purely a CSS class in the Volt template: `<div class="ui checkbox">` versus `<div class="ui toggle checkbox">`.
{% endhint %}

The static-vs-DB dropdown distinction is just the contents of the `$options` array the form receives. Both calls are identical; only the supplied option list differs. The full annotated form is in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Forms/ModuleExampleFormForm.php`.

## Step 3 — Wire up the controller

The controller extends `MikoPBX\AdminCabinet\Controllers\BaseController`. Three methods matter for a form: `initialize()`, `modifyAction()` (render) and `saveAction()` (persist). `BaseController` supplies the protected helpers `saveEntity(mixed $entity, string $reloadPath = ''): bool` and `deleteEntity(mixed $entity, string $reloadPath = ''): bool`.

### modifyAction — render the form

`modifyAction` queues the required JavaScript assets, loads (or creates) the settings record, builds the dropdown option lists, and assigns the form to the view.

{% hint style="warning" %}
Always queue **`js/pbx/main/form.js` first**, then your compiled module script. `form.js` defines the global `Form` object your JS depends on (Step 5). Asset collection constants live on `MikoPBX\AdminCabinet\Providers\AssetProvider` (`HEADER_CSS`, `FOOTER_JS`).
{% endhint %}

{% code title="App/Controllers/ModuleExampleFormController.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleExampleForm\App\Controllers;

use MikoPBX\AdminCabinet\Controllers\BaseController;
use MikoPBX\AdminCabinet\Providers\AssetProvider;
use MikoPBX\Common\Models\Providers;
use Modules\ModuleExampleForm\App\Forms\ModuleExampleFormForm;
use Modules\ModuleExampleForm\Models\ModuleExampleForm;

class ModuleExampleFormController extends BaseController
{
    private string $moduleUniqueID = 'ModuleExampleForm';

    public function initialize(): void
    {
        $this->view->logoImagePath =
            $this->url->get() . 'assets/img/cache/' . $this->moduleUniqueID . '/logo.svg';
        $this->view->submitMode = null;
        parent::initialize();
    }

    public function modifyAction(): void
    {
        // 1) Queue assets: form.js MUST be added before the module script
        $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
        $footerCollectionJS
            ->addJs('js/pbx/main/form.js', true)
            ->addJs('js/cache/' . $this->moduleUniqueID . '/module-example-form-modify.js', true);

        // 2) Load the single settings row (or create a fresh, empty one)
        $settings = ModuleExampleForm::findFirst();
        if ($settings === null) {
            $settings = new ModuleExampleForm();
        }

        // 3) Build the option lists for the dropdowns
        $options = $this->buildFormOptions();

        // 4) Hand the form to the view
        $this->view->form = new ModuleExampleFormForm($settings, $options);
        $this->view->pick('Modules/' . $this->moduleUniqueID . '/ModuleExampleForm/modify');
    }

    private function buildFormOptions(): array
    {
        $options = [];

        // STATIC dropdown options — value => translated display text
        $options['priorities'] = [
            'low'      => $this->translation->_('module_template_PriorityLow'),
            'medium'   => $this->translation->_('module_template_PriorityMedium'),
            'high'     => $this->translation->_('module_template_PriorityHigh'),
            'critical' => $this->translation->_('module_template_PriorityCritical'),
        ];
        $options['selectPlaceholder'] = $this->translation->_('module_template_SelectPlaceholder');

        // DB-backed dropdown — value => label pulled from a model query
        $providersList = [];
        foreach (Providers::find() as $provider) {
            $providersList[$provider->uniqid] = $provider->getRepresent();
        }
        $options['providers'] = $providersList;
        $options['providerPlaceholder'] = $this->translation->_('module_template_ProviderPlaceholder');

        return $options;
    }
}
```
{% endcode %}

### saveAction — persist the posted data

`saveAction` reads the POST, loads the record by `id` (or creates one), then **iterates the model's own properties** and copies matching POST values onto them. This loop is the recommended pattern: it only touches real columns and handles the checkbox/toggle special case in one place.

{% hint style="danger" %}
Unchecked checkboxes and toggles are **not posted by the browser at all**. You must default them explicitly, otherwise a turned-off toggle would keep its old value. The loop below sets `'1'` when the field arrives as `'on'`, and `'0'` when the key is absent.
{% endhint %}

{% code title="App/Controllers/ModuleExampleFormController.php (continued)" %}
```php
    public function saveAction(): void
    {
        if (!$this->request->isPost()) {
            return;
        }
        $data   = $this->request->getPost();
        $record = ModuleExampleForm::findFirstById($data['id'] ?? null);
        if ($record === null) {
            $record = new ModuleExampleForm();
        }

        foreach ($record as $key => $value) {
            switch ($key) {
                case 'id':
                    break;
                case 'checkbox_field':
                case 'toggle_field':
                    // Fomantic checkboxes post 'on' when checked, nothing when not
                    if (array_key_exists($key, $data)) {
                        $record->$key = ($data[$key] === 'on') ? '1' : '0';
                    } else {
                        $record->$key = '0';
                    }
                    break;
                default:
                    if (array_key_exists($key, $data)) {
                        $record->$key = $data[$key];
                    } else {
                        $record->$key = '';
                    }
            }
        }

        // saveEntity() runs $record->save(), flashes errors, and (for AJAX
        // requests) sets $this->view->success used by the global Form object.
        $this->saveEntity($record);
    }

    public function deleteAction(string $recordId): void
    {
        $record = ModuleExampleForm::findFirstById($recordId);
        if ($record !== null) {
            $this->deleteEntity($record, 'module-example-form/module-example-form/index');
        }
    }
```
{% endcode %}

{% hint style="info" %}
`saveEntity()` already handles both AJAX and non-AJAX cases: on an AJAX POST it sets `$this->view->success` (which the global `Form` object reads), and on a normal POST it flashes a success message and optionally forwards to `$reloadPath`. You do **not** write a JSON response yourself.
{% endhint %}

The complete controller (including `indexAction`) is in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Controllers/ModuleExampleFormController.php`.

## Step 4 — Render the Volt view

`modify.volt` is a plain `<form>` whose `action` points at the `save` route. Each field is emitted with `{{ form.render('field_name') }}`, wrapped in Fomantic UI markup, with labels translated via `{{ t._('key') }}`. The submit button is a shared partial.

Key rendering rules:

* The hidden `id` and `hidden_field` are rendered at the top, outside any visible layout.
* Checkboxes use `<div class="ui checkbox">`; toggles use `<div class="ui toggle checkbox">` — both wrap `form.render(...)` plus a `<label>`.
* Close with `partial("partials/submitbutton", ['indexurl': '<index route>'])`, which renders the standard Save / Back buttons used across the admin cabinet (`Core/src/AdminCabinet/Views/partials/submitbutton.volt`).

{% code title="App/Views/ModuleExampleForm/modify.volt" %}
```html
<form method="post" action="module-example-form/module-example-form/save" role="form"
      class="ui large form" id="module-example-form-form">

    {{ form.render('id') }}
    {{ form.render('hidden_field') }}

    {# Text field with an info-tooltip icon #}
    <div class="ten wide field disability">
        <label>{{ t._('module_template_TextFieldLabel') }}
            <i class="small info circle icon popup"
               data-content="{{ t._('module_template_TextFieldInfo') }}"
               data-variation="wide"></i>
        </label>
        {{ form.render('text_field') }}
    </div>

    {# TextArea #}
    <div class="ten wide field disability">
        <label>{{ t._('module_template_TextAreaFieldLabel') }}</label>
        {{ form.render('text_area_field') }}
    </div>

    {# Password #}
    <div class="ten wide field disability">
        <label>{{ t._('module_template_PasswordFieldLabel') }}</label>
        {{ form.render('password_field') }}
    </div>

    {# Numeric (narrow) #}
    <div class="four wide field disability">
        <label>{{ t._('module_template_IntegerFieldLabel') }}</label>
        {{ form.render('integer_field') }}
    </div>

    {# Standard checkbox #}
    <div class="field disability">
        <div class="ui segment">
            <div class="ui checkbox">
                {{ form.render('checkbox_field') }}
                <label>{{ t._('module_template_CheckBoxFieldLabel') }}</label>
            </div>
        </div>
    </div>

    {# Toggle switch — same element, "toggle" CSS class #}
    <div class="field disability">
        <div class="ui segment">
            <div class="ui toggle checkbox">
                {{ form.render('toggle_field') }}
                <label>{{ t._('module_template_ToggleFieldLabel') }}</label>
            </div>
        </div>
    </div>

    {# Dropdown — static options #}
    <div class="ten wide field disability">
        <label>{{ t._('module_template_SelectFieldLabel') }}</label>
        {{ form.render('select_field') }}
    </div>

    {# Dropdown — DB-backed options #}
    <div class="ten wide field disability">
        <label>{{ t._('module_template_ProviderFieldLabel') }}</label>
        {{ form.render('provider_field') }}
    </div>

    {{ partial("partials/submitbutton", ['indexurl': 'module-example-form/module-example-form/index']) }}
</form>
```
{% endcode %}

{% hint style="info" %}
The real example wraps these fields in a tabbed menu and an accordion. Those are optional layout niceties — the load-bearing parts are the `form.render(...)` calls, the matching field names, and the `submitbutton` partial. See the full template in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/ModuleExampleForm/modify.volt`.
{% endhint %}

## Step 5 — JavaScript: validation and AJAX submit

The client script uses the global `Form` object (provided by `js/pbx/main/form.js`). You configure it, then call `Form.initialize()`. The standard contract is:

* `Form.$formObj` — jQuery handle of your `<form>`.
* `Form.url` — absolute save URL (`globalRootUrl` + your save route).
* `Form.validateRules` — Fomantic UI form-validation rule set.
* `Form.cbBeforeSendForm(settings)` — return the (possibly modified) AJAX settings; this is where you assemble `settings.data`.
* `Form.cbAfterSendForm()` — runs after a successful save.

For **ModuleBlackList** this file would be `module-black-list-modify.js` (the list page gets its own `module-black-list-index.js`); the example uses `module-example-form-modify.js`:

{% code title="public/assets/js/src/module-example-form-modify.js" %}
```javascript
/* global globalRootUrl, globalTranslate, Form */

const ModuleExampleFormModify = {

    $formObj: $('#module-example-form-form'),
    $checkBoxes: $('#module-example-form-form .ui.checkbox'),
    $dropDowns: $('#module-example-form-form .ui.dropdown'),

    // Fomantic UI validation rules. identifier === the field/element name.
    validateRules: {
        textField: {
            identifier: 'text_field',
            rules: [
                {
                    type: 'empty',
                    prompt: globalTranslate.module_template_ValidateValueIsEmpty,
                },
                {
                    type: 'minLength[3]',
                    prompt: globalTranslate.module_template_ValidateMinLength,
                },
            ],
        },
        integerField: {
            identifier: 'integer_field',
            rules: [
                {
                    type: 'integer[1..9999]',
                    prompt: globalTranslate.module_template_ValidateIntegerRange,
                },
            ],
        },
    },

    initialize() {
        // Initialize Fomantic components
        ModuleExampleFormModify.$checkBoxes.checkbox();
        ModuleExampleFormModify.$dropDowns.dropdown();

        // Example of populating a Hidden field from JS
        const now = new Date().toISOString();
        $('#hidden_field').val(now);

        ModuleExampleFormModify.initializeForm();
    },

    // Collect all form values right before the AJAX request goes out
    cbBeforeSendForm(settings) {
        const result = settings;
        result.data = ModuleExampleFormModify.$formObj.form('get values');
        return result;
    },

    // Runs after the server confirms a successful save
    cbAfterSendForm() {
    },

    initializeForm() {
        Form.$formObj = ModuleExampleFormModify.$formObj;
        Form.url = `${globalRootUrl}module-example-form/module-example-form/save`;
        Form.validateRules = ModuleExampleFormModify.validateRules;
        Form.cbBeforeSendForm = ModuleExampleFormModify.cbBeforeSendForm;
        Form.cbAfterSendForm = ModuleExampleFormModify.cbAfterSendForm;
        Form.initialize();
    },
};

$(document).ready(() => {
    ModuleExampleFormModify.initialize();
});
```
{% endcode %}

{% hint style="warning" %}
**Compile the source.** Files under `public/assets/js/src/` are written in modern ES and must be babel-compiled into `public/assets/js/` — the parent directory, not a `cache` subfolder. The `js/cache/<moduleUniqueID>` path you see in `modifyAction` is a symlink the installer points straight at `<moduleDir>/public/assets/js` (`PbxExtensionUtils::createAssetsSymlinks()`, `Core/src/Modules/PbxExtensionUtils.php:105-114`), so the compiled file lands exactly where the URL resolves. MikoPBX never loads the `src/` file directly. Recompile after every edit.
{% endhint %}

The full client controller (with TextArea/Password rules, accordion, tab, and popup initialization) is in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/public/assets/js/src/module-example-form-modify.js`.

## How the pieces connect

```
Model column  ──►  Form element name  ──►  form.render('name') in Volt
     ▲                                              │
     │                                              ▼
saveAction loop  ◄──  POST data  ◄──  Form.cbBeforeSendForm → AJAX → /save
```

1. **Render:** the browser opens `modifyAction`, which queues `form.js`, builds the form from the loaded model row, and renders `modify.volt`.
2. **Submit:** the JS calls `Form.initialize()`; on submit, Fomantic validates with `validateRules`, `cbBeforeSendForm` gathers values, and an AJAX POST hits `/save`.
3. **Persist:** `saveAction` maps POST values onto the model (normalizing checkboxes/toggles) and calls `saveEntity()`, which saves and reports success back to the `Form` object; `cbAfterSendForm` then runs in the browser.

## Checklist

* [ ] Model column names, form element names, `form.render()` names, and `validateRules` identifiers are **all identical**.
* [ ] `parent::initialize($entity, $options)` is the first line of the form's `initialize()`.
* [ ] `modifyAction` queues `js/pbx/main/form.js` **before** your module script.
* [ ] `saveAction` defaults missing checkbox/toggle fields to `'0'`.
* [ ] The JS `src/` file is babel-compiled before testing.
* [ ] The view ends with the `partials/submitbutton` partial.

## Related pages

* [Forms cookbook](./)
* [Empty module interface](../../module-developement/module-interface-empty.md)
* [Data model](../../module-developement/data-model.md)
