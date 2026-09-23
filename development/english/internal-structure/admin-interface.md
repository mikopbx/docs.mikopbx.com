---
description: Architecture of the MikoPBX admin-cabinet web application.
---

# Admin interface

The admin cabinet (the web UI you reach at `https://<pbx>/admin-cabinet/`) is a self-contained Phalcon 5 MVC application that lives entirely under `Core/src/AdminCabinet/`. It is registered with the front controller as a **Phalcon module** and follows a strict layered layout: a thin `Module.php` bootstrap, a DI registration file, one controller per resource, one form per resource, Volt views, and a set of providers/plugins that wire authentication, routing, and asset bundling together.

This page maps that architecture so you understand the machine your module plugs into. The hooks your module uses to extend the UI are summarised at the end and covered in full on [Module web interface](../module-developement/module-interface-empty.md). For the broader picture of how the admin cabinet sits next to the other MikoPBX applications, see [Core](core.md).

{% hint style="info" %}
Throughout the docs we thread one fictional module, **ModuleBlackList** (config class `BlackListConf`, model `BlackListNumbers`, table `m_BlackListNumbers`, JS files named per action — `module-black-list-index.js`, `module-black-list-modify.js`). On this page `BlackListConf` is the class that implements the UI extension hooks. Every pattern is also anchored to a real, shipping module under `Extensions/`.
{% endhint %}

## The module bootstrap

`AdminCabinet/Module.php` implements `Phalcon\Mvc\ModuleDefinitionInterface`. It does almost nothing itself — it delegates all service registration to `RegisterDIServices`:

{% code title="Core/src/AdminCabinet/Module.php" %}
```php
namespace MikoPBX\AdminCabinet;

use MikoPBX\AdminCabinet\Config\RegisterDIServices;
use Phalcon\Di\DiInterface;
use Phalcon\Mvc\ModuleDefinitionInterface;

class Module implements ModuleDefinitionInterface
{
    public function registerAutoloaders(?DiInterface $container = null)
    {
    }

    public function registerServices(DiInterface $container): void
    {
        RegisterDIServices::init($container);
    }
}
```
{% endcode %}

The two interface methods are the only contract a Phalcon module must satisfy: `registerAutoloaders()` (empty here — autoloading is handled globally by Composer) and `registerServices()`, which receives the shared DI container and populates it with everything the admin cabinet needs.

## DI registration: admin-local + shared Common providers

`AdminCabinet/Config/RegisterDIServices::init()` builds an ordered list of service-provider classes and registers each one. Every provider exposes a `SERVICE_NAME` constant and a `register(DiInterface)` method; the loop first removes any previously registered service with that name (so re-entering the application cleanly replaces stale services) and then registers fresh:

{% code title="Core/src/AdminCabinet/Config/RegisterDIServices.php" %}
```php
foreach ($adminCabinetProviders as $provider) {
    // Delete previous provider
    $di->remove($provider::SERVICE_NAME);
    (new $provider())->register($di);
}

// Set the library name
$di->getShared(RegistryProvider::SERVICE_NAME)->libraryName = 'admin-cabinet';
```
{% endcode %}

The provider list mixes two groups:

* **AdminCabinet-local providers** (`MikoPBX\AdminCabinet\Providers\*`) — the web layer: `DispatcherProvider`, `ViewProvider`, `VoltProvider`, `FlashProvider`, `ElementsProvider`, `AssetProvider`, `SecurityPluginProvider`.
* **Shared Common providers** (`MikoPBX\Common\Providers\*`) — infrastructure reused by every MikoPBX application: loggers, the managed cache and Redis client, the database connections (`MainDatabaseProvider`, `ModulesDBConnectionsProvider`, `CDRDatabaseProvider`, `RecordingStorageDatabaseProvider`), the web router/URL providers (`RouterProvider`, `UrlProvider`), sessions, ACL (`AclProvider`), JWT (`JwtProvider`), translations, the marketplace/license provider, `PBXConfModulesProvider` (the module hook bus), `RegistryProvider`, `CryptProvider`, the REST client, the event bus, and the WAF provider.

The last line tags this DI instance with `libraryName = 'admin-cabinet'`, which several providers and the localization cache key off.

{% hint style="info" %}
`PBXConfModulesProvider` is the bridge between this application and your module. The admin cabinet calls `PBXConfModulesProvider::hookModulesMethod(<hookConstant>, [...])` at well-defined points, and every installed module's config class that defines that method gets invoked. This is the mechanism behind the UI extension hooks described at the end of this page.
{% endhint %}

## Layer 1 — Controllers

`AdminCabinet/Controllers/` holds **33 PHP files**: one controller per resource (`ExtensionsController`, `ProvidersController`, `CallQueuesController`, `GeneralSettingsController`, `PbxExtensionModulesController`, …) plus the shared `BaseController` they all extend, and the special `SessionController` (login) and `ErrorsController` (401/404/500 pages).

### BaseController

`BaseController extends Phalcon\Mvc\Controller` and centralises everything a resource controller needs. The members you will rely on:

```php
class BaseController extends Controller
{
    protected string $actionName;               // current action
    protected string $controllerName;           // CamelCase, e.g. Extensions
    protected string $controllerClass;          // FQCN of the dispatched controller
    protected string $controllerNameUnCamelized; // kebab-case, e.g. extensions
    protected bool   $isExternalModuleController; // true when namespace starts with "Modules"
}
```

**`initialize()`** runs on every request. It reads the dispatched controller/action from the dispatcher, detects whether the request belongs to a module (`str_starts_with($this->dispatcher->getNamespaceName(), 'Modules')`), and — for non-AJAX requests — calls `prepareView()`.

**`prepareView()`** populates the shared view variables: the page timezone (from `PbxSettings`), the license string, the support URL, the page `<title>` (built from `Breadcrumb*` translation keys), the translation helper `t`, debug mode, the logo URL/href, the current language, the PBX version/name, and the layout template. Layout selection is explicit:

* `SessionController` — no `templateAfter` (the login page renders itself).
* `ErrorsController` — a self-contained `error` layout, rendered only to `View::LEVEL_AFTER_TEMPLATE` so the heavy `index.volt` (and its polling workers) is never loaded.
* everything else — the main admin layout (`setTemplateAfter('main')`) with sidebar and top menu.

For module controllers, `prepareView()` additionally loads the `PbxExtensionModules` record, exposes it to the view as `module`, sets `globalModuleUniqueId`, and rewrites `currentPage` to `"<uniqid>/<controller>/<action>"`.

**`beforeExecuteRoute(Dispatcher $dispatcher)`** picks the correct Volt view for the dispatched route. For a module controller it picks `"Modules/{uniqid}/{Controller}/{action}"`; otherwise `"{Controller}/{action}"`. It then fires the module hook `WebUIConfigInterface::ON_BEFORE_EXECUTE_ROUTE`.

**`afterExecuteRoute(Dispatcher $dispatcher)`** is where AJAX responses are assembled. When the request is AJAX it suppresses rendering (`View::LEVEL_NO_RENDER`), sets the JSON content type, and serialises the view params. Unless the controller set `raw_response`, the helper fills in defaults:

```php
$data['success'] = $data['success'] ?? true;
$data['reload']  = $data['reload']  ?? false;
$data['message'] = $data['message'] ?? $this->flash->getMessages();
```

It also attaches the last Sentry event id and fires `WebUIConfigInterface::ON_AFTER_EXECUTE_ROUTE`.

**`saveEntity(mixed $entity, string $reloadPath = '')`** and **`deleteEntity(mixed $entity, string $reloadPath = '')`** wrap model persistence: they call `$entity->save()` / `$entity->delete()`, push validation errors to the flash service on failure, emit the `ms_SuccessfulSaved` flash on success for non-AJAX requests, and — for AJAX — set `$this->view->success` and an optional `reload` path (the `{id}` placeholder in `reloadPath` is replaced with the saved entity id).

**`sanitizeData(array $data, FilterInterface $filter)`** is a static recursive sanitiser. It walks the input array, re-encodes and sanitises embedded JSON, sanitises `http…` strings as URLs, trims and strips illegal characters from other strings, and casts numerics to int. Use it on `$this->request` data before binding it to a model.

**`isAuthenticated()`** delegates to `SecurityPlugin::isAuthenticated($this->request, $this->cookies)` so controllers and the security plugin share one authentication definition.

A `ModuleBlackListController` (in your module's `App/Controllers/`) extends `BaseController` exactly the same way:

{% code title="Extensions/ModuleBlackList/App/Controllers/ModuleBlackListController.php" %}
```php
namespace Modules\ModuleBlackList\App\Controllers;

use MikoPBX\AdminCabinet\Controllers\BaseController;
use Modules\ModuleBlackList\Models\BlackListNumbers;

class ModuleBlackListController extends BaseController
{
    public function indexAction(): void
    {
        $this->view->records = BlackListNumbers::find();
    }

    public function saveAction(): void
    {
        if (!$this->request->isPost()) {
            return;
        }
        $data   = $this->request->getPost();
        $record = BlackListNumbers::findFirstById($data['id']) ?? new BlackListNumbers();
        $record->assign($data, ['number', 'description']);
        $this->saveEntity($record); // flash + AJAX JSON handled for you
    }
}
```
{% endcode %}

{% hint style="success" %}
See the working example in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Controllers/ModuleExampleFormController.php`, which extends `BaseController` and implements `indexAction()` / `modifyAction()` / `saveAction()`.
{% endhint %}

## Layer 2 — Forms

`AdminCabinet/Forms/` holds an abstract `BaseForm extends Phalcon\Forms\Form` plus, in most cases, one `<Resource>EditForm` per controller (`ExtensionEditForm`, `GeneralSettingsEditForm`, …) and a few special-purpose forms (`LoginForm`, `CallDetailRecordsFilterForm`, `DefaultIncomingRouteForm`, the `Licensing*Form` set).

`BaseForm::initialize($entity = null, $options = null)` is the standard Phalcon form entry point and exposes typed field helpers — `addTextArea()`, `addCheckBox()`, `addSemanticUIDropdown()` — that produce markup the Fomantic UI front-end understands. A module form follows the same pattern; see the real example in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Forms/ModuleExampleFormForm.php`. For a full walkthrough of building forms, see the [Forms cookbook](../cookbook/forms/).

## Layer 3 — Views

`AdminCabinet/Views/` contains the Volt templates: one folder per controller (`Views/Extensions/index.volt`, `Views/Extensions/modify.volt`, …), shared `layouts/` (`main.volt`, `error`) and `partials/` (`leftsidebar`, `topMenu`, `mainHeader`, `submitbutton`, `tablesbuttons`, …). The Volt engine itself is configured by `ViewProvider` and `VoltProvider` (registered in `RegisterDIServices`), and `BaseController::beforeExecuteRoute()` selects which view to render.

Module views live in your module under `App/Views/<Controller>/<action>.volt` and are picked automatically — see `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Views/`.

## Layer 4 — Providers & Plugins

The web behaviour of the cabinet comes from a small set of providers and plugins.

### AssetProvider + AssetManager

`Providers/AssetProvider` (service name `assets`) builds the per-page CSS/JS bundles. It instantiates `Plugins/AssetManager` (which `extends Phalcon\Assets\Manager`) with the prefix `/admin-cabinet/assets/`, assembles header/footer/localization collections for the current controller+action, regenerates the localization cache, and — crucially for modules — fires the module hook at the very end:

{% code title="Core/src/AdminCabinet/Providers/AssetProvider.php" %}
```php
$assets->makeLocalizationAssets($di, $version);

PBXConfModulesProvider::hookModulesMethod(
    WebUIConfigInterface::ON_AFTER_ASSETS_PREPARED,
    [$assetsManager, $dispatcher]
);
```
{% endcode %}

`AssetManager` exposes the bundle-output helpers used by the layouts (`outputCss()`, `outputJs()`, `outputCombinedHeaderJs()`, `outputCombinedFooterJs()`, `outputCombinedHeaderCSS()`, `setVersion()`).

### DispatcherProvider — where the plugins attach

`Providers/DispatcherProvider` (service name `dispatcher`) builds the Phalcon dispatcher and attaches the request-lifecycle plugins to its events manager:

{% code title="Core/src/AdminCabinet/Providers/DispatcherProvider.php" %}
```php
$eventsManager = new EventsManager();

// Camelize / normalize the controller name
$eventsManager->attach('dispatch:beforeDispatch',     new NormalizeControllerNamePlugin());
$eventsManager->attach('dispatch:afterDispatchLoop',  new NormalizeControllerNamePlugin());

// Authentication + ACL gate
$eventsManager->attach('dispatch:beforeDispatch',     new SecurityPlugin());

// Forward unknown controllers/actions to the 404 page
$eventsManager->attach('dispatch:beforeException',    new NotFoundPlugin());

$dispatcher = new Dispatcher();
$dispatcher->setEventsManager($eventsManager);
```
{% endcode %}

* **`NormalizeControllerNamePlugin`** (`extends Injectable`) — `beforeDispatch()` / `afterDispatchLoop()` normalise the dispatched controller name so routing is case- and separator-insensitive.
* **`NotFoundPlugin`** (`extends Injectable`) — `beforeException()` forwards unknown controllers/actions to the 404 page; other exceptions propagate to the global error handler (and Sentry).
* **`SecurityPlugin`** — described next.

### SecurityPlugin — the JWT auth/ACL gate

`Plugins/SecurityPlugin extends Phalcon\Di\Injectable` and its `beforeDispatch(Event, Dispatcher)` method is the single gate every dispatched request passes through.

{% code title="Core/src/AdminCabinet/Plugins/SecurityPlugin.php" %}
```php
public function beforeDispatch(Event $event, Dispatcher $dispatcher): bool
{
    $isAuthenticated = $this->checkUserAuth() || $this->isLocalHostRequest();

    $action          = $dispatcher->getActionName();
    $controllerClass = $this->dispatcher->getHandlerClass();

    $publicControllers = [SessionController::class, ErrorsController::class];

    if (!$isAuthenticated && !in_array($controllerClass, $publicControllers)) {
        $this->clearAuthCookies();
        if ($this->request->isAjax()) {
            $this->response->setStatusCode(403, 'Forbidden')
                ->setContent('This user is not authorized')->send();
        } else {
            $this->forwardToLoginPage($dispatcher);
        }
        return false;
    }
    // ... authenticated path: ACL check via isAllowedAction(), 401 forward on deny
    return true;
}
```
{% endcode %}

Key facts about the gate, all from the source:

* **Authentication is JWT-based.** `SecurityPlugin::isAuthenticated()` is a static method that accepts a `RequestInterface` and `CookiesInterface`. For AJAX it validates a `Bearer` token from the `Authorization` header through `JwtProvider::validate()`; for browser page loads it accepts the presence of an encrypted `refreshToken` cookie (the front-end `token-manager.js` then exchanges it for an access token). Because the method is static, `BaseController::isAuthenticated()` reuses it verbatim.
* **Public controllers** (`SessionController`, `ErrorsController`) are reachable without authentication; everything else is gated.
* **Unauthenticated requests** get a `403` for AJAX or a forward to the login page for normal requests, after stale auth cookies are cleared to prevent redirect loops.
* **Authorization (ACL)** is enforced by `isAllowedAction(string $controller, string $action)`, which resolves the user role from the JWT (`extractRoleFromHeader()` / `extractRoleFromRefreshToken()`), defaults anonymous remote callers to `AclProvider::ROLE_GUESTS` and localhost to `AclProvider::ROLE_ADMINS`, then calls `$acl->isAllowed($role, $controller, $action)`. A denied request is forwarded to `errors/show401`.
* **Localhost is trusted** (`isLocalHostRequest()` checks `REMOTE_ADDR === '127.0.0.1'`) so internal workers and health checks bypass the gate.

`SecurityPluginProvider` (service name `securityPlugin`) also registers the plugin in the DI as a factory so it can answer ad-hoc permission questions — e.g. a module asking "is this user allowed to run `changeUserCredentials` on `UsersCredentialsController`?" — without going through a full dispatch.

## Request lifecycle (one diagram)

```
HTTP request
   │
   ▼
RouterProvider            → resolves module / controller / action
   │
   ▼
DispatcherProvider        → dispatch loop fires attached plugins:
   ├─ NormalizeControllerNamePlugin   (dispatch:beforeDispatch)
   ├─ SecurityPlugin::beforeDispatch  (auth + ACL gate; may forward/abort)
   └─ NotFoundPlugin::beforeException (unknown route → 404)
   │
   ▼
BaseController::beforeExecuteRoute     → pick view, hook ON_BEFORE_EXECUTE_ROUTE
   │
   ▼
<Resource>Controller::<action>         → controller logic, saveEntity()/deleteEntity()
   │
   ▼
BaseController::afterExecuteRoute       → AJAX JSON or rendered Volt, hook ON_AFTER_EXECUTE_ROUTE
   │
   ▼  (AssetProvider built header/footer assets earlier; hook ON_AFTER_ASSETS_PREPARED fired there)
HTTP response
```

## How a module plugs into this UI

A module that adds web pages ships **its own Phalcon module** plus a **config class** that implements the UI extension hooks. Keep the two roles distinct:

* **`App/Module.php`** — a `Phalcon\Mvc\ModuleDefinitionInterface` whose `registerServices()` only sets up the module's own `dispatcher` pointing at the module controller namespace. It does _not_ contain UI hooks. Real example — `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/App/Module.php`:

{% code title="Extensions/ModuleBlackList/App/Module.php" %}
```php
namespace Modules\ModuleBlackList\App;

use Phalcon\Di\DiInterface;
use Phalcon\Events\Manager;
use Phalcon\Mvc\Dispatcher;
use Phalcon\Mvc\ModuleDefinitionInterface;

class Module implements ModuleDefinitionInterface
{
    public function registerAutoloaders(?DiInterface $container = null): void
    {
    }

    public function registerServices(DiInterface $container): void
    {
        $container->set('dispatcher', function () {
            $dispatcher   = new Dispatcher();
            $eventManager = new Manager();
            $dispatcher->setEventsManager($eventManager);
            $dispatcher->setDefaultNamespace('Modules\ModuleBlackList\App\Controllers\\');
            return $dispatcher;
        });
    }
}
```
{% endcode %}

* **`Lib/BlackListConf`** — the config class (an implementer of `MikoPBX\Modules\Config\WebUIConfigInterface`). This is where the UI extension hooks live. The admin cabinet calls these methods through `PBXConfModulesProvider::hookModulesMethod()` at the lifecycle points shown above.

`WebUIConfigInterface` declares eleven methods in total; the four that build the page are listed below, with their **verified** signatures from `Core/src/Modules/Config/WebUIConfigInterface.php` and the constants used to invoke them:

| Hook method                                                                     | Constant                     | Fired from                              |
| ------------------------------------------------------------------------------- | ---------------------------- | --------------------------------------- |
| `onBeforeHeaderMenuShow(array &$menuItems): void`                               | `ON_BEFORE_HEADER_MENU_SHOW` | `Library/Elements` (sidebar menu build) |
| `onAfterRoutesPrepared(Router $router): void`                                   | `ON_AFTER_ROUTES_PREPARED`   | `Common\Providers\RouterProvider`       |
| `onAfterAssetsPrepared(Manager $assets, Dispatcher $dispatcher): void`          | `ON_AFTER_ASSETS_PREPARED`   | `AssetProvider`                         |
| `onVoltBlockCompile(string $controller, string $blockName, View $view): string` | `ON_VOLT_BLOCK_COMPILE`      | `VoltProvider` (Volt block compilation) |

The remaining seven cover authentication (`authenticateUser`, `getPasskeySessionData`), permissions (`onAfterACLPrepared`, `onGetControllerPermissions`), form building (`onBeforeFormInitialize`) and the request lifecycle (`onBeforeExecuteRoute`, `onAfterExecuteRoute`).

### Sidebar menu — `onBeforeHeaderMenuShow`

Add menu items by writing into the by-reference `$menuItems` array. The shape (caption + nested `submenu`) is taken directly from the real example in `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/Lib/ExampleFormConf.php`:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
public function onBeforeHeaderMenuShow(array &$menuItems): void
{
    $menuItems['module_black_list_MenuGroup'] = [
        'caption'   => 'module_black_list_MenuGroup',
        'iconclass' => '',
        'submenu'   => [
            '/module-black-list/index' => [
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

### Custom routes — `onAfterRoutesPrepared`

Receives the `Phalcon\Mvc\Router` instance so a module can register extra routes beyond the default `/<module-uniqid>/<controller>/<action>` mapping. The signature is defined in `WebUIConfigInterface::onAfterRoutesPrepared(Router $router): void`.

{% hint style="warning" %}
None of the example or shipping modules under `Extensions/` override `onAfterRoutesPrepared` — the default no-op in the base config class is sufficient because the standard module route is registered automatically. Implement this hook only when you need a non-standard URL; the signature above is the authoritative reference.
{% endhint %}

### Page assets — `onAfterAssetsPrepared`

Add CSS/JS to the current page, conditioned on the dispatched controller/action. Real example — `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`, which injects assets onto the core `Extensions:modify` page:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use MikoPBX\AdminCabinet\Providers\AssetProvider;
use Phalcon\Assets\Manager;
use Phalcon\Mvc\Dispatcher;

public function onAfterAssetsPrepared(Manager $assets, Dispatcher $dispatcher): void
{
    if ($dispatcher->getControllerName() === 'ModuleBlackList') {
        $assets->collection(AssetProvider::FOOTER_JS)
            ->addJs("js/cache/{$this->moduleUniqueId}/module-black-list-index.js", true);
    }
}
```
{% endcode %}

{% hint style="info" %}
**Asset path and file naming.** `js/cache/<moduleUniqueID>/` is not a build output directory — it is a symlink the installer creates (`PbxExtensionUtils::createAssetsSymlinks()`) from `sites/admin-cabinet/assets/js/cache/<moduleUniqueID>` to your module's `public/assets/js`. Babel compiles into `public/assets/js/`; never emit into any `cache/` path yourself. The same is done for `css/cache/…` and `img/cache/…`.

File names are **per action**: `module-<kebab-uniqueid>-<action>.js`. When you inject onto a _core_ page the name includes that page's controller and action — the shipping example is `ModuleUsersUI`, which adds `js/cache/ModuleUsersUI/module-users-ui-extensions-modify.js` on `Extensions:modify` (`Extensions/ModuleUsersUI/Lib/UsersUIConf.php:394`).
{% endhint %}

### Volt block override — `onVoltBlockCompile`

Inject a partial into a named Volt block on a core page (e.g. add a tab to the Extensions edit page). Return the view path of the partial to render for a given `"<controller>:<blockName>"`, or an empty string to leave the block unchanged. Real example — `Extensions/ModuleUsersUI/Lib/UsersUIConf.php`:

{% code title="Extensions/ModuleBlackList/Lib/BlackListConf.php" %}
```php
use Phalcon\Mvc\View;

public function onVoltBlockCompile(string $controller, string $blockName, View $view): string
{
    switch ("$controller:$blockName") {
        case 'Extensions:TabularMenu':
            return 'Modules/ModuleBlackList/Extensions/tabularmenu';
        case 'Extensions:AdditionalTab':
            return 'Modules/ModuleBlackList/Extensions/additionaltab';
        default:
            return '';
    }
}
```
{% endcode %}

{% hint style="info" %}
Each hook returns control to the admin cabinet, which merges the result (menu item, route, asset, or Volt partial) into the page it is already building. Your module never replaces the application — it augments it at these defined seams.
{% endhint %}

## See also

* [Module web interface](../module-developement/module-interface-empty.md) — full guide to building the `App/` Phalcon module, controllers, forms, views, and the `WebUIConfigInterface` hooks.
* [Forms cookbook](../cookbook/forms/) — building `BaseForm`-based edit forms.
* [Core](core.md) — how the admin cabinet relates to the other MikoPBX applications.
