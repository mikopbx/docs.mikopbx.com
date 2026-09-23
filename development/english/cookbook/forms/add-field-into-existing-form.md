---
description: Inject a custom field into a core MikoPBX form from your module.
---

# Add field into existing form

A module often needs to attach extra data to an existing **core** entity — an extension, a provider, a queue — without forking the Core. MikoPBX exposes two Web-UI hooks that together let a module surgically extend a form it does not own:

1. **`onBeforeFormInitialize(Form $form, $entity, $options)`** — add Phalcon form elements (inputs, selects, checkboxes) to a core form _before_ it is built.
2. **`onVoltBlockCompile(string $controller, string $blockName, View $view)`** — inject a partial Volt template into a named block of a core view, so those new elements actually render on the page.

To **persist** the value you add a model in your module and relate it to the core model with `getDynamicRelations()`.

Throughout this page we extend the core **Extensions** edit form with a new field from a fictional module **ModuleBlackList** (config class `BlackListConf`, model `BlackListNumbers`, table `m_BlackListNumbers`). The real, working reference for this exact pattern is **ModuleUsersUI**, which uses both hooks to add an access group selector to the same Extensions form — see `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`.

{% hint style="info" %}
The two Web-UI hooks live on the module config class (`BlackListConf extends ConfigClass`). The base implementations are no-ops in `Core/src/Modules/Config/ConfigClass.php` (namespace `MikoPBX\Modules\Config`); you only override the ones you need. The third piece — `getDynamicRelations()` — is **not** a config-class hook: it is a static method you declare on your own **model** class (see Step 3). For the full hook catalogue see [hooks-reference.md](../../module-developement/hooks-reference.md).
{% endhint %}

## How the hooks fire

Both hooks are dispatched by the Core, so you never call them yourself.

`onBeforeFormInitialize` is invoked from `BaseForm::initialize()` for **every** admin form:

{% code title="Core/src/AdminCabinet/Forms/BaseForm.php" %}
```php
public function initialize($entity = null, $options = null): void
{
    if ($entity === null) {
        $entity = new stdClass();
    }
    PBXConfModulesProvider::hookModulesMethod(
        WebUIConfigInterface::ON_BEFORE_FORM_INITIALIZE,
        [$this, $entity, $options]
    );
}
```
{% endcode %}

{% hint style="warning" %}
The hook runs on **all** forms, so your first line must be a type guard. In MikoPBX 2025.1.1+ the form is frequently initialized with `$entity` as an empty `stdClass` (form data is loaded separately over the REST API), so never assume `$entity` is a populated model instance. Note also that `$options` is **untyped** in the signature (`onBeforeFormInitialize(Form $form, $entity, $options)`).
{% endhint %}

`onVoltBlockCompile` is invoked from the Volt `hookVoltBlock(...)` function while a template compiles. Core views declare named injection points like this:

{% code title="Core/src/AdminCabinet/Views/Extensions/modify.volt" %}
```twig
{{ partial("PbxExtensionModules/hookVoltBlock",
    ['arrayOfPartials': hookVoltBlock('TabularMenu')]) }}
...
{{ partial("PbxExtensionModules/hookVoltBlock",
    ['arrayOfPartials': hookVoltBlock('AdditionalTab')]) }}
```
{% endcode %}

For each `hookVoltBlock` call the Core gathers a partial path from every module (`Core/src/AdminCabinet/Providers/VoltProvider.php`), passing your config class the controller name (`Extensions`) and the block name (`TabularMenu` / `AdditionalTab`). The Extensions edit form exposes both `TabularMenu` (the tab strip) and `AdditionalTab` (the tab bodies) — that is the pair we target.

## Step 1 — Add the form element

Override `onBeforeFormInitialize` in your config class. Guard on the exact core form class, then add Phalcon form elements.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Lib;

use MikoPBX\AdminCabinet\Forms\ExtensionEditForm;
use MikoPBX\Modules\Config\ConfigClass;
use Modules\ModuleBlackList\Models\BlackListNumbers;
use Phalcon\Forms\Element\Check;
use Phalcon\Forms\Form;
use Phalcon\Mvc\View;

class BlackListConf extends ConfigClass
{
    /**
     * Add a "block this number" checkbox to the core Extensions edit form.
     *
     * @param Form  $form    The core form instance being initialized.
     * @param mixed $entity  The form entity (may be an empty stdClass).
     * @param mixed $options The form options (untyped).
     */
    public function onBeforeFormInitialize(Form $form, $entity, $options): void
    {
        // Only touch the form we want — the hook fires for every admin form.
        if (!is_a($form, ExtensionEditForm::class)) {
            return;
        }

        // $entity may be an empty stdClass in 2025.1.1+, so read defensively.
        $userId  = is_object($entity) ? ($entity->user_id ?? null) : null;
        $blocked = false;
        if (!empty($userId)) {
            $record = BlackListNumbers::findFirst([
                'conditions' => 'user_id = :user_id:',
                'bind'       => ['user_id' => $userId],
            ]);
            $blocked = $record !== null && $record->blocked === '1';
        }

        $checkAttributes = ['value' => 'on'];
        if ($blocked) {
            $checkAttributes['checked'] = 'checked';
        }
        $form->add(new Check('module_black_list_blocked', $checkAttributes));
    }
}
```
{% endcode %}

{% hint style="info" %}
Use a unique, prefixed field name (here `module_black_list_blocked`). It must not collide with any core field name on the form, and the prefix makes it easy to recognise your fields in the submitted POST data later. ModuleUsersUI follows the same convention with a `module_users_ui_` prefix.
{% endhint %}

{% hint style="warning" %}
**Reading `$entity->user_id` is not enough on its own.** Because the form is now initialized without an entity, the snippet above renders an unchecked box on every real page load. `ModuleUsersUI` solves this rather than tolerating it: when `$entity` is not an object or `user_id` is empty, it recovers the id from the dispatcher parameters and rebuilds a stand-in entity —

```php
if (!is_object($entity) || empty($entity->user_id)) {
    $userId = $this->resolveUserIdFromDispatcher();
    $entity = (object)['user_id' => $userId];
}
```

— see `Extensions/ModuleUsersUI/Lib/UsersUIConf.php:175-188` and its `resolveUserIdFromDispatcher()` helper just below. Copy that shape if your field has to show a stored value on first render. Note the helper also has to cope with the URL parameter meaning different things across versions (in 2025.1.1 the modify-route parameter _is_ the `user_id`; in older builds it is the extension id and needs a lookup).
{% endhint %}

ModuleUsersUI builds a richer set of elements (Text, Password, Check, Hidden, Select) in a dedicated helper and adds them all to the same form — see the verified reference in `Extensions/ModuleUsersUI/App/Forms/ExtensionEditAdditionalForm.php` (`prepareAdditionalFields()`), called from `onBeforeFormInitialize` in `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`.

## Step 2 — Render the field with a Volt partial

Adding the element to the form object is not enough — nothing renders it until you return a partial template for the relevant block. Override `onVoltBlockCompile` and match on the `"$controller:$blockName"` pair.

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
/**
 * Inject our partials into the core Extensions edit view.
 *
 * @return string Partial path WITHOUT extension, relative to the modules root.
 */
public function onVoltBlockCompile(string $controller, string $blockName, View $view): string
{
    switch ("$controller:$blockName") {
        case 'Extensions:TabularMenu':
            // Adds the tab label into the Extensions tab strip.
            return 'Modules/ModuleBlackList/Extensions/tabularmenu';
        case 'Extensions:AdditionalTab':
            // Adds the tab body that renders our form field.
            return 'Modules/ModuleBlackList/Extensions/additionaltab';
        default:
            return '';
    }
}
```
{% endcode %}

{% hint style="warning" %}
Return the partial path **without** the `.volt` extension, and return an empty string for blocks you do not handle. The path is resolved against the modules views root, so the string `Modules/ModuleBlackList/Extensions/additionaltab` maps to the physical file `<moduleDir>/App/Views/Extensions/additionaltab.volt`. ModuleUsersUI returns `"Modules/ModuleUsersUI/Extensions/tabularmenu"` for the file at `Extensions/ModuleUsersUI/App/Views/Extensions/tabularmenu.volt` — the same mapping.
{% endhint %}

Now create the two partials. The tab label:

{% code title="Extensions/ModuleBlackList/App/Views/Extensions/tabularmenu.volt" %}
```twig
<a class="item" data-tab="blackList">{{ t._('module_blacklist_TabName') }}</a>
```
{% endcode %}

And the tab body that renders the element you added in Step 1. Reference the field by the exact name you registered:

{% code title="Extensions/ModuleBlackList/App/Views/Extensions/additionaltab.volt" %}
```twig
<div class="ui bottom attached tab segment" data-tab="blackList">
    <div class="field">
        <div class="ui toggle checkbox">
            {{ form.render('module_black_list_blocked') }}
            <label for="module_black_list_blocked">
                {{ t._('module_blacklist_BlockThisNumber') }}
            </label>
        </div>
    </div>
</div>
```
{% endcode %}

Compare with the working originals: `Extensions/ModuleUsersUI/App/Views/Extensions/tabularmenu.volt` and `.../additionaltab.volt`, which render `module_users_ui_*` fields with `form.render(...)`.

{% hint style="info" %}
A Semantic-UI toggle checkbox needs `$('.ui.checkbox').checkbox()` to initialise. Because you are extending a page owned by the **core** controller, you cannot add the asset from your own `indexAction`/`modifyAction` — use the `onAfterAssetsPrepared(Manager $assets, Dispatcher $dispatcher)` hook (`WebUIConfigInterface::ON_AFTER_ASSETS_PREPARED`, `Core/src/Modules/Config/WebUIConfigInterface.php:49,111`), which hands you the asset manager plus the dispatcher so you can check which controller/action is being rendered before attaching anything. Name the file for the action you are extending — `module-black-list-extensions-modify.js` — since asset names are per-action, not per-module. Asset loading is its own topic and is otherwise out of scope for this page.
{% endhint %}

## Step 3 — Persist the value with a model relation

The field now renders and submits, but you still need somewhere to store it. Create a module model backed by your own table and relate it to the **core** model with a static `getDynamicRelations()` method.

When a core model is initialized (`ModelsBase::initialize()` → `addExtensionModulesRelations()`), the Core scans each enabled module's `Models/` directory and, when a model class declares `getDynamicRelations()`, calls it so the module can attach relationships to the core model being loaded:

{% code title="Core/src/Common/Models/ModelsBase.php" %}
```php
if (
    class_exists($moduleModelClass)
    && method_exists($moduleModelClass, 'getDynamicRelations')
) {
    $moduleModelClass::getDynamicRelations($this);
}
```
{% endcode %}

Declare the relation from your model toward the core `Users` model (the entity behind an extension):

{% code title="Extensions/ModuleBlackList/Models/BlackListNumbers.php" %}
```php
<?php

namespace Modules\ModuleBlackList\Models;

use MikoPBX\Common\Models\ModelsBase;
use MikoPBX\Common\Models\Users;
use Phalcon\Mvc\Model\Relation;

class BlackListNumbers extends ModelsBase
{
    public $id;
    public $user_id;
    public $blocked;

    public function initialize(): void
    {
        $this->setSource('m_BlackListNumbers');
        parent::initialize();
    }

    /**
     * Cross-model relation: link this module table to the core Users model.
     * Called by ModelsBase when a core model is initialized.
     *
     * @param mixed $calledModelObject
     */
    public static function getDynamicRelations(&$calledModelObject): void
    {
        if (is_a($calledModelObject, Users::class)) {
            $calledModelObject->hasMany(
                'id',
                __CLASS__,
                'user_id',
                [
                    'alias'      => 'ModuleBlackListNumbers',
                    'foreignKey' => [
                        'allowNulls' => 0,
                        'message'    => __CLASS__,
                        'action'     => Relation::ACTION_CASCADE,
                    ],
                ]
            );
        }
    }
}
```
{% endcode %}

{% hint style="warning" %}
`getDynamicRelations()` declares a **Phalcon relationship** (alias, foreign key, cascade/restrict behaviour) — it ties your record to the core record so deletes cascade and referential integrity holds. It does **not** read the submitted form value. Capturing the value the user typed into `module_black_list_blocked` and writing it to `BlackListNumbers` is a **separate** save step (intercepting the save in your own controller/REST handler). The relation alone does not auto-persist the field.
{% endhint %}

Real, verified examples of `getDynamicRelations()`:

* `Extensions/ModuleAutoprovision/Models/ModuleAutoprovisionUsers.php` — a live `hasMany` relation to `Users` (the pattern reproduced above).
* `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Models/ModuleExampleForm.php` — a documented `getDynamicRelations()` stub showing a `belongsTo` to a core `Providers` model.

## Putting it together

| Goal                     | Hook / method                                                           | Where it fires            |
| ------------------------ | ----------------------------------------------------------------------- | ------------------------- |
| Add the form element     | `onBeforeFormInitialize(Form $form, $entity, $options)`                 | `BaseForm::initialize()`  |
| Render the element       | `onVoltBlockCompile(string $controller, string $blockName, View $view)` | Volt `hookVoltBlock(...)` |
| Relate to the core model | `getDynamicRelations(&$calledModelObject)` (static, on your model)      | `ModelsBase`              |

{% hint style="success" %}
**Verify your work:** open the core form in the admin UI. If the tab/label appears but the field is missing, your `onVoltBlockCompile` partial path or block name is wrong. If the element is in the form object but nothing shows, the Volt partial is not rendering it (check the `form.render(...)` field name). If the field shows but your data does not survive a reload, the relation is fine but the **save step** is missing.
{% endhint %}

## See also

* [Forms cookbook overview](./)
* [Hooks reference](../../module-developement/hooks-reference.md)
* [Module interface (empty skeleton)](../../module-developement/module-interface-empty.md)
