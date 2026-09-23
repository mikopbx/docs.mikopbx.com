---
description: >-
  Building the admin-cabinet UI: controllers, forms, Volt views, providers and
  JS/CSS.
---

# Module interface

A module's web interface lives inside the MikoPBX admin cabinet. It is a small, self-contained Phalcon MVC application that the core mounts as a module: your controllers extend the core `BaseController`, your forms extend the core `BaseForm`, and your views are rendered by the [Volt template engine](https://docs.phalcon.io/5.0/en/volt) using the global Fomantic UI / jQuery toolkit shared with the rest of the cabinet.

This page walks the full UI recipe through a running example module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`, table `m_BlackListNumbers`, JS file `module-black-list-index.js`). For every pattern there is a pointer to a real, working module you can read and copy:

* **`Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`** — the canonical single-form example. Read this first.
* **`Extensions/ModuleUsersUI/`** — a larger production module with several controllers, custom providers and a sidebar item.

{% hint style="info" %}
File layout assumed throughout this page. The module unique ID is the folder name (`ModuleBlackList`); the controller/route slug is the kebab-case form (`module-black-list`).

```
Modules/ModuleBlackList/
└── App
    ├── Module.php
    ├── Controllers
    │   └── ModuleBlackListController.php
    ├── Forms
    │   └── ModuleBlackListForm.php
    ├── Views
    │   └── ModuleBlackList
    │       ├── index.volt
    │       └── modify.volt
    └── Providers          # usually empty — see Providers below
        ├── ViewProvider.php   # optional
        └── VoltProvider.php   # optional
public/assets/js/src/
    └── module-black-list-modify.js
```
{% endhint %}

## The controller

A module controller extends `MikoPBX\AdminCabinet\Controllers\BaseController`. The base class wires the cabinet layout, the `$this->view`, `$this->assets`, `$this->request`, `$this->translation` services, and provides the `saveEntity()` / `deleteEntity()` helpers that persist a model and return a JSON/redirect response in the format the cabinet expects.

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\App\Controllers;

use MikoPBX\AdminCabinet\Controllers\BaseController;
use MikoPBX\AdminCabinet\Providers\AssetProvider;
use Modules\ModuleBlackList\App\Forms\ModuleBlackListForm;
use Modules\ModuleBlackList\Models\BlackListNumbers;

class ModuleBlackListController extends BaseController
{
    private string $moduleUniqueID = 'ModuleBlackList';

    public function initialize(): void
    {
        $this->view->logoImagePath = $this->url->get() . 'assets/img/cache/' . $this->moduleUniqueID . '/logo.svg';
        $this->view->submitMode = null;
        parent::initialize();
    }
}
```
{% endcode %}

`initialize()` is called by Phalcon before every action. Always call `parent::initialize()` last — it finalises the cabinet layout based on the properties you set (`logoImagePath`, `submitMode`).

{% hint style="info" %}
**Two controllers or one?** A module with a single page extends `BaseController` directly — see `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Controllers/ModuleExampleFormController.php`. A module with several pages usually adds one shared intermediate base class so all controllers inherit common `initialize()` logic and helper queries. See `Extensions/ModuleUsersUI/App/Controllers/ModuleUsersUIBaseController.php` (which `extends BaseController`); every page controller there — `ModuleUsersUIController`, `AccessGroupsController`, etc. — `extends ModuleUsersUIBaseController`. Use the pattern `Module{Feature}Controller extends {Feature}BaseController extends BaseController` only when you actually have shared per-controller logic to hoist.
{% endhint %}

### Actions

The cabinet routes module URLs as `{module-slug}/{controller-slug}/{action}`. For `ModuleBlackList` the controller slug is `module-black-list`, so the working URLs are `module-black-list/module-black-list/index`, `.../modify`, `.../save`, `.../delete`.

#### indexAction — the landing page

The index action loads page assets and picks the index view. For a settings module the index is often just a link to the edit form; for a list module it hosts a DataTable (see [Create a DataTable](../cookbook/forms/create-datatable.md)).

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
public function indexAction(): void
{
    $headerCollectionCSS = $this->assets->collection(AssetProvider::HEADER_CSS);
    $headerCollectionCSS->addCss('css/cache/' . $this->moduleUniqueID . '/module-black-list-index.css', true);

    $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
    $footerCollectionJS->addJs('js/cache/' . $this->moduleUniqueID . '/module-black-list-index.js', true);

    $this->view->pick('Modules/' . $this->moduleUniqueID . '/ModuleBlackList/index');
}
```
{% endcode %}

* `$this->view->pick('Modules/<UniqueID>/<Controller>/<view>')` selects the Volt template **without** its `.volt` extension. The `Modules/<UniqueID>/` prefix is what makes the cabinet resolve the template inside your module directory rather than the core views.
* Assets are attached, not echoed, through `$this->assets->collection(...)`. The constants live on `MikoPBX\AdminCabinet\Providers\AssetProvider`:
  * `AssetProvider::HEADER_CSS` — CSS injected in `<head>`.
  * `AssetProvider::FOOTER_JS` — JS injected before `</body>` (this is where module scripts go).
  * also available: `HEADER_JS`, `HEADER_PBX_JS`, `SEMANTIC_UI_CSS`, `SEMANTIC_UI_JS`, `FOOTER_PBX_JS`.
* Reference the **compiled** asset under `js/cache/<UniqueID>/...` / `css/cache/<UniqueID>/...`. You author the source in `public/assets/js/src/` and Babel compiles it one directory up into `public/assets/js/` — never into a `cache/` folder, which is a symlink the installer creates (see [JavaScript](module-interface-empty.md#javascript)). The second argument `true` marks the path as local (relative to the module asset root).

{% hint style="info" %}
`AssetProvider` is a **core** service you _consume_ via `$this->assets`. You do not register it from your module. The providers you register yourself are the view and Volt providers — see [Providers](module-interface-empty.md#providers).
{% endhint %}

#### modifyAction — the edit form

`modifyAction()` loads the cabinet form engine (`js/pbx/main/form.js`), instantiates your form against the model record, and picks the `modify` view.

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
public function modifyAction(): void
{
    $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
    $footerCollectionJS
        ->addJs('js/pbx/main/form.js', true)
        ->addJs('js/cache/' . $this->moduleUniqueID . '/module-black-list-modify.js', true);

    $record = BlackListNumbers::findFirst();
    if ($record === null) {
        $record = new BlackListNumbers();
    }

    $options = $this->buildFormOptions();

    $this->view->form = new ModuleBlackListForm($record, $options);
    $this->view->pick('Modules/' . $this->moduleUniqueID . '/ModuleBlackList/modify');
}
```
{% endcode %}

The form is exposed to Volt as `$this->view->form`, where it becomes the `form` variable used by `{{ form.render('field') }}`. The `$options` array carries data the form cannot compute on its own — typically dropdown contents built from the database:

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
private function buildFormOptions(): array
{
    $options = [];

    // Static dropdown options
    $options['priorities'] = [
        'low'      => $this->translation->_('module_black_list_PriorityLow'),
        'medium'   => $this->translation->_('module_black_list_PriorityMedium'),
        'high'     => $this->translation->_('module_black_list_PriorityHigh'),
    ];
    $options['selectPlaceholder'] = $this->translation->_('module_black_list_SelectPlaceholder');

    // DB-backed dropdown — providers list
    $providersList = [];
    foreach (\MikoPBX\Common\Models\Providers::find() as $provider) {
        $providersList[$provider->uniqid] = $provider->getRepresent();
    }
    $options['providers'] = $providersList;
    $options['providerPlaceholder'] = $this->translation->_('module_black_list_ProviderPlaceholder');

    return $options;
}
```
{% endcode %}

See the working version in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Controllers/ModuleExampleFormController.php` (`buildFormOptions()`).

#### saveAction — persisting the form

`saveAction()` reads the POST, maps it onto the model, and delegates to `saveEntity()`. The critical detail is **checkbox/toggle handling**: an unchecked Fomantic UI checkbox is simply absent from the POST, and a checked one arrives as the string `'on'`. You must normalise both to `'1'` / `'0'` before saving.

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
public function saveAction(): void
{
    if (!$this->request->isPost()) {
        return;
    }
    $data = $this->request->getPost();

    $record = BlackListNumbers::findFirstById($data['id'] ?? null);
    if ($record === null) {
        $record = new BlackListNumbers();
    }

    foreach ($record as $key => $value) {
        switch ($key) {
            case 'id':
                break;
            case 'enabled':         // checkbox/toggle columns
            case 'block_calls':
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

    $this->saveEntity($record);
}
```
{% endcode %}

`saveEntity(mixed $entity, string $reloadPath = ''): bool` is provided by `BaseController`. It saves the model and, on an AJAX request, populates the view variables the cabinet form engine reads back (`$this->view->success` and, when `$reloadPath` is set, `$this->view->reload`); on validation failure it pushes the model's messages to `$this->flash`. It returns `true`/`false` for the save result. Iterating `foreach ($record as $key => $value)` over the model copies only columns that actually exist on the entity — fields posted by JS that are not columns are ignored.

#### deleteAction — removing a record

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
public function deleteAction(string $recordId): void
{
    $record = BlackListNumbers::findFirstById($recordId);
    if ($record !== null) {
        $this->deleteEntity($record, 'module-black-list/module-black-list/index');
    }
}
```
{% endcode %}

`deleteEntity(mixed $entity, string $reloadPath = ''): bool` deletes the record and, like `saveEntity()`, sets the AJAX reload target (`$this->view->reload = $reloadPath`, a `{module-slug}/{controller-slug}/{action}` path) so the cabinet navigates there after deletion. See `deleteAction()` in `ModuleExampleFormController.php`.

## The form

A module form extends `MikoPBX\AdminCabinet\Forms\BaseForm` and builds its elements in `initialize($entity = null, $options = null)`. Always call `parent::initialize($entity, $options)` first — it registers the cabinet-wide behaviour shared by every form. The `$entity` is the model passed from the controller; `$options` is the array you built in `buildFormOptions()`.

`BaseForm` mixes two kinds of elements:

* **Plain Phalcon 5 elements** added with `$this->add(new Text(...))` — `Text`, `Password`, `Numeric`, `Hidden`, `TextArea`, etc., from the `Phalcon\Forms\Element\*` namespace.
* **Cabinet helper methods** that wrap Fomantic UI widgets so they render and behave consistently:
  * `addTextArea(string $areaName, string $areaValue, int $areaWidth = 90, array $options = []): void`
  * `addCheckBox(string $fieldName, bool $checked, string $checkedValue = 'on'): void`
  * `addSemanticUIDropdown(string $name, array $options = [], $value = null, array $attributes = []): void` (`protected`)

{% code title="App/Forms/ModuleBlackListForm.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\App\Forms;

use MikoPBX\AdminCabinet\Forms\BaseForm;
use Phalcon\Forms\Element\Hidden;
use Phalcon\Forms\Element\Numeric;
use Phalcon\Forms\Element\Password;
use Phalcon\Forms\Element\Text;

class ModuleBlackListForm extends BaseForm
{
    public function initialize($entity = null, $options = null): void
    {
        parent::initialize($entity, $options);

        // Hidden primary key — always present, never shown
        $this->add(new Hidden('id', ['value' => $entity->id]));

        // Text input
        $this->add(new Text('number'));

        // TextArea: name, value, width, attributes
        $this->addTextArea(
            'comment',
            $entity->comment ?? '',
            90,
            ['placeholder' => "Reason for blocking"]
        );

        // Password input
        $this->add(new Password('api_token'));

        // Numeric input with attributes
        $this->add(new Numeric('priority', [
            'maxlength'    => 4,
            'style'        => 'width: 100px;',
            'defaultValue' => 3,
        ]));

        // Checkbox — rendered as <div class="ui checkbox"> in Volt
        $this->addCheckBox('enabled', intval($entity->enabled) === 1);

        // Toggle — same PHP element, rendered as <div class="ui toggle checkbox"> in Volt
        $this->addCheckBox('block_calls', intval($entity->block_calls) === 1);

        // SemanticUIDropdown with STATIC options
        $this->addSemanticUIDropdown(
            'priority_level',
            $options['priorities'] ?? [],
            $entity->priority_level ?? 'medium',
            ['placeholder' => $options['selectPlaceholder'] ?? '']
        );

        // SemanticUIDropdown populated from the DATABASE
        $this->addSemanticUIDropdown(
            'provider_field',
            $options['providers'] ?? [],
            $entity->provider_field ?? '',
            ['placeholder' => $options['providerPlaceholder'] ?? '']
        );

        // Hidden field whose value is set by JavaScript
        $this->add(new Hidden('last_sync', ['value' => $entity->last_sync ?? '']));
    }
}
```
{% endcode %}

Every element type above is demonstrated in the canonical example `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Forms/ModuleExampleFormForm.php`.

{% hint style="info" %}
**Checkbox vs Toggle** are the _same_ PHP element (`addCheckBox`). The only difference is the wrapper CSS class in the Volt view: `<div class="ui checkbox">` versus `<div class="ui toggle checkbox">`. Both submit `'on'` when checked — which is exactly why `saveAction()` normalises `'on' → '1'`.
{% endhint %}

For a deeper walkthrough of form construction and validation see [Forms overview](../cookbook/forms/) and [Create a module form](../cookbook/forms/create-module-form.md).

## Volt views

Views are Volt templates living under `App/Views/<UniqueID>/`. Use the translation function `t._('key')` for every user-facing string (see [Translations](translations.md)).

### index.volt

For a settings module the index can be a single button to the edit form:

{% code title="App/Views/ModuleBlackList/index.volt" %}
```twig
{{ link_to(
    "module-black-list/module-black-list/modify",
    '<i class="gear icon"></i>  ' ~ t._('module_black_list_ChangeRecord'),
    "class": "ui blue button", "id": "change-record"
) }}
```
{% endcode %}

This mirrors the real `index.volt` in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/ModuleExampleForm/index.volt`. For a list page that hosts a DataTable instead, follow [Create a DataTable](../cookbook/forms/create-datatable.md).

### modify.volt

The edit view opens a `<form>` whose `action` points at your `save` action and whose `id` matches the selector your JS uses. Render each element with `{{ form.render('field') }}`, wrapping it in the Fomantic UI field markup. End with the shared submit-button partial.

{% code title="App/Views/ModuleBlackList/modify.volt" %}
```twig
<form method="post" action="module-black-list/module-black-list/save" role="form"
      class="ui large form" id="module-black-list-form">

    {{ form.render('id') }}
    {{ form.render('last_sync') }}

    {# --- Text field with info tooltip --- #}
    <div class="ten wide field disability">
        <label>{{ t._('module_black_list_NumberLabel') }}
            <i class="small info circle icon popup"
               data-content="{{ t._('module_black_list_NumberInfo') }}"
               data-variation="wide"></i>
        </label>
        {{ form.render('number') }}
    </div>

    {# --- TextArea --- #}
    <div class="ten wide field disability">
        <label>{{ t._('module_black_list_CommentLabel') }}</label>
        {{ form.render('comment') }}
    </div>

    {# --- Checkbox: <div class="ui checkbox"> --- #}
    <div class="field disability">
        <div class="ui segment">
            <div class="ui checkbox">
                {{ form.render('enabled') }}
                <label>{{ t._('module_black_list_EnabledLabel') }}</label>
            </div>
        </div>
    </div>

    {# --- Toggle: <div class="ui toggle checkbox"> --- #}
    <div class="field disability">
        <div class="ui segment">
            <div class="ui toggle checkbox">
                {{ form.render('block_calls') }}
                <label>{{ t._('module_black_list_BlockCallsLabel') }}</label>
            </div>
        </div>
    </div>

    {# --- SemanticUIDropdown --- #}
    <div class="ten wide field disability">
        <label>{{ t._('module_black_list_PriorityLabel') }}</label>
        {{ form.render('priority_level') }}
    </div>

    {{ partial("partials/submitbutton", ['indexurl': 'module-black-list/module-black-list/index']) }}
</form>
```
{% endcode %}

Key points, all verified against `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/ModuleExampleForm/modify.volt`:

* `{{ form.render('field') }}` outputs the element; you supply the surrounding `<div class="... field">` and `<label>`.
* `{{ form.getValue('field') }}` reads the current value of a field for display.
* `{{ partial("partials/submitbutton", ['indexurl': '...']) }}` renders the standard Save / Back button row. `indexurl` is the path the Back button returns to.
* Tabs (`<div class="ui top attached tabular menu">` + `data-tab` segments), accordions (`<div class="ui accordion field">`) and popup icons (`<i class="... icon popup" data-content="...">`) are plain Fomantic UI markup, initialised from JS.

## JavaScript

Module JS is authored in `public/assets/js/src/` as ES6 and compiled with Babel into `public/assets/js/` — one directory up from the sources. The `js/cache/<UniqueID>/` path the controller attaches is the **served** path: on install `PbxExtensionUtils::createAssetsSymlinks()` symlinks your `public/assets/js` directory to `sites/admin-cabinet/assets/js/cache/<UniqueID>`. You never create a `cache/` directory inside your module. The cabinet exposes globals you build against: `Form` (the form submission/validation engine loaded via `js/pbx/main/form.js`), `globalRootUrl`, `globalTranslate` (your translation keys), and `PbxApi` (the REST client for core endpoints).

A modify script wires Fomantic UI components, then hands the form to the global `Form` object:

{% code title="public/assets/js/src/module-black-list-modify.js" %}
```javascript
/* global globalRootUrl, globalTranslate, Form */

const ModuleBlackListModify = {
    $formObj: $('#module-black-list-form'),
    $checkBoxes: $('#module-black-list-form .ui.checkbox'),
    $dropDowns: $('#module-black-list-form .ui.dropdown'),

    validateRules: {
        number: {
            identifier: 'number',
            rules: [
                {
                    type: 'empty',
                    prompt: globalTranslate.module_black_list_ValidateValueIsEmpty,
                },
            ],
        },
    },

    initialize() {
        ModuleBlackListModify.$checkBoxes.checkbox();
        ModuleBlackListModify.$dropDowns.dropdown();
        $('#module-black-list-form i[data-content]').popup();
        ModuleBlackListModify.initializeForm();
    },

    cbBeforeSendForm(settings) {
        const result = settings;
        result.data = ModuleBlackListModify.$formObj.form('get values');
        return result;
    },

    cbAfterSendForm() {
        // runs after a successful save
    },

    initializeForm() {
        Form.$formObj = ModuleBlackListModify.$formObj;
        Form.url = `${globalRootUrl}module-black-list/module-black-list/save`;
        Form.validateRules = ModuleBlackListModify.validateRules;
        Form.cbBeforeSendForm = ModuleBlackListModify.cbBeforeSendForm;
        Form.cbAfterSendForm = ModuleBlackListModify.cbAfterSendForm;
        Form.initialize();
    },
};

$(document).ready(() => {
    ModuleBlackListModify.initialize();
});
```
{% endcode %}

The contract with the global `Form` object (verified in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/public/assets/js/src/module-example-form-modify.js`):

* `Form.$formObj` — the jQuery form element.
* `Form.url` — the absolute save URL (`globalRootUrl` + slug path).
* `Form.validateRules` — Fomantic UI form validation rules (see [the Fomantic UI form behaviour docs](https://fomantic-ui.com/behaviors/form.html)).
* `Form.cbBeforeSendForm(settings)` — return the AJAX `settings` with `result.data` set; the canonical pattern collects values with `$formObj.form('get values')`.
* `Form.cbAfterSendForm()` — post-save callback.
* `Form.initialize()` — binds everything and takes over submission.

For calls to core services (status, restart, reading PBX state, etc.) use the global `PbxApi` client rather than hand-rolling fetches; it knows the cabinet's CSRF and endpoint conventions.

{% hint style="info" %}
Compile JS sources before packaging. The cabinet only ever serves the compiled file from `public/assets/js/` (as `js/cache/<UniqueID>/...`); the `src/` file is never loaded directly.
{% endhint %}

## Providers

{% hint style="warning" %}
**Most modules need no providers at all.** Volt views resolve without any DI wiring: on install `PbxExtensionUtils::createViewSymlinks()` symlinks your `App/Views` directory into `src/AdminCabinet/Views/Modules/<UniqueID>`, which is exactly what `$this->view->pick('Modules/<UniqueID>/...')` resolves against. The canonical example `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` has an **empty** `App/Providers/` directory and registers nothing but the dispatcher — and its views render fine.

Register the two providers below only when you need to take over view resolution or extend the Volt compiler (custom Volt functions, a module-specific cache directory). `Extensions/ModuleUsersUI/` does this because it adds an `isAllowed` Volt helper for ACL-driven rendering.
{% endhint %}

To take that control, a module registers two service providers in its `App/Module.php`. Each provider implements `Phalcon\Di\ServiceProviderInterface`.

### ViewProvider

Points the `view` service at the module's own `App/Views` directory and registers the `.volt` engine.

{% code title="App/Providers/ViewProvider.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\App\Providers;

use Phalcon\Di\DiInterface;
use Phalcon\Di\ServiceProviderInterface;
use Phalcon\Mvc\View;
use function MikoPBX\Common\Config\appPath;

class ViewProvider implements ServiceProviderInterface
{
    public const SERVICE_NAME = 'view';

    public function register(DiInterface $di): void
    {
        $di->setShared(
            self::SERVICE_NAME,
            function () {
                $viewsDir = appPath('App/Views');
                $view = new View();
                $view->setViewsDir($viewsDir);
                $view->registerEngines(['.volt' => 'volt']);
                return $view;
            }
        );
    }
}
```
{% endcode %}

This is the verbatim pattern from `Extensions/ModuleUsersUI/App/Providers/ViewProvider.php`.

### VoltProvider

Configures the Volt compiler — cache directory, debug-mode recompilation, and any custom Volt functions your views need.

{% code title="App/Providers/VoltProvider.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\App\Providers;

use Phalcon\Di\DiInterface;
use Phalcon\Di\ServiceProviderInterface;
use Phalcon\Mvc\View\Engine\Volt as VoltEngine;

class VoltProvider implements ServiceProviderInterface
{
    public const SERVICE_NAME = 'volt';

    public function register(DiInterface $di): void
    {
        $view      = $di->getShared('view');
        $appConfig = $di->getShared('config')->adminApplication;
        $di->setShared(
            self::SERVICE_NAME,
            function () use ($view, $di, $appConfig) {
                $volt = new VoltEngine($view, $di);
                $volt->setOptions(['path' => $appConfig->voltCacheDir . '/']);

                $compiler = $volt->getCompiler();
                $compiler->addFunction('in_array', 'in_array');

                if ($appConfig->debugMode === true) {
                    $volt->setOptions(['compileAlways' => true]);
                }
                return $volt;
            }
        );
    }
}
```
{% endcode %}

See the full production version (including the `isAllowed` Volt helper) in `Extensions/ModuleUsersUI/App/Providers/VoltProvider.php`.

### Registering providers in App/Module.php

`App/Module.php` implements `Phalcon\Mvc\ModuleDefinitionInterface`. Setting the dispatcher's default namespace to your controllers is the **only** mandatory part; add the `register()` calls just if you actually wrote the providers above:

{% code title="App/Module.php" %}
```php
<?php

namespace Modules\ModuleBlackList\App;

use Modules\ModuleBlackList\App\Providers\ViewProvider;
use Modules\ModuleBlackList\App\Providers\VoltProvider;
use Phalcon\Di\DiInterface;
use Phalcon\Events\Manager;
use Phalcon\Mvc\Dispatcher;
use Phalcon\Mvc\ModuleDefinitionInterface;

class Module implements ModuleDefinitionInterface
{
    public function registerAutoloaders(DiInterface $container = null)
    {
    }

    public function registerServices(DiInterface $container)
    {
        $container->register(new ViewProvider());
        $container->register(new VoltProvider());

        $container->set('dispatcher', function () {
            $dispatcher = new Dispatcher();
            $dispatcher->setEventsManager(new Manager());
            $dispatcher->setDefaultNamespace('Modules\ModuleBlackList\App\Controllers\\');
            return $dispatcher;
        });
    }
}
```
{% endcode %}

Verified against `Extensions/ModuleUsersUI/App/Module.php`.

{% hint style="warning" %}
There is no separate "MenuProvider" service. The sidebar/menu integration is done through the module config-class hook or the installer helper described next — not through a registered DI provider.
{% endhint %}

## Sidebar and menu integration

There are two distinct mechanisms, both confirmed in real modules.

### onBeforeHeaderMenuShow — runtime header menu

Your module **config class** (`BlackListConf`, which extends the core `ConfigClass`) may implement `onBeforeHeaderMenuShow(array &$menuItems): void`. The constant for this hook name is `MikoPBX\Modules\Config\WebUIConfigInterface::ON_BEFORE_HEADER_MENU_SHOW`. You mutate the `$menuItems` array by reference to add a top-menu entry with an optional submenu:

{% code title="Lib/BlackListConf.php" %}
```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $menuItems['module_black_list_MenuGroup'] = [
        'caption'   => 'module_black_list_MenuGroup',
        'iconclass' => '',
        'submenu'   => [
            '/module-black-list/module-black-list/index' => [
                'caption'   => 'module_black_list_MenuItem',
                'iconclass' => 'ban',
                'action'    => 'index',
                'param'     => '',
                'style'     => '',
            ],
        ],
    ];
}
```
{% endcode %}

Confirmed in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php` (`onBeforeHeaderMenuShow`). The same hook is used in `Extensions/ModuleUsersUI/Lib/UsersUIConf.php` and `Extensions/ModuleTemplate/Lib/TemplateConf.php`.

### addToSidebar — persisted left sidebar item

To place a persistent item in the left sidebar, your installer (a class extending `PbxExtensionSetupBase`) overrides / calls `addToSidebar(): bool`. The base implementation stores a `PbxSettings` record keyed `AdditionalMenuItem<UniqueID>` whose JSON value describes the entry:

```php
$value = [
    'uniqid'        => $this->moduleUniqueID,   // 'ModuleBlackList'
    'group'         => 'modules',
    'iconClass'     => 'puzzle',
    'caption'       => "Breadcrumb$this->moduleUniqueID",
    'showAtSidebar' => true,
];
```

This is the exact shape written by `addToSidebar()` in `Core/src/Modules/Setup/PbxExtensionSetupBase.php`. It is invoked from the module install flow; override it in your setup class to customise `iconClass`, `caption` or `group`.

## See also

* [Forms overview](../cookbook/forms/)
* [Create a module form](../cookbook/forms/create-module-form.md)
* [Create a DataTable](../cookbook/forms/create-datatable.md)
* [Translations](translations.md)
* [Admin interface](../internal-structure/admin-interface.md)
