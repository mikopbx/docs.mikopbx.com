---
description: >-
  Step-by-step: build a server-paginated list grid backed by the v3 getList
  endpoint.
---

# Create datatable

A settings form holds one row. The moment your module manages **many** rows — blocked numbers, log entries, queued jobs — you need a list grid: a table that shows a page of rows, lets the admin search and sort, and fetches the next page from the server on demand instead of dumping the whole table into the browser.

This recipe wires a Fomantic-styled [DataTables](https://datatables.net/) grid to a **REST API v3 `getList` endpoint**, the same architecture the core Call Detail Records page uses. We thread it through the running example module **ModuleBlackList** (model `BlackListNumbers`, table `m_BlackListNumbers`, resource slug `module-black-list/numbers`, JS file `module-black-list-index.js`).

{% hint style="info" %}
Assets are named per **action**, not per module: the grid page (`indexAction`) loads `module-black-list-index.js` / `module-black-list-index.css`, while the edit page (`modifyAction`) loads `module-black-list-modify.js`. The production grid module follows the same rule — see the file list under `Extensions/ModulePhoneBook/public/assets/js/src/` (`module-phonebook-index.js`, `module-phonebook-settings.js`, …).
{% endhint %}

{% hint style="warning" %}
**No EXAMPLES module renders a datatable yet.** The canonical `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/` module is single-record (one form, one row), and `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/` ships the REST layer but no UI grid. This page therefore composes two _real, verified_ sources:

* the **legacy controller-action grid** pattern from the production module `Extensions/ModulePhoneBook/` (Volt table markup + DataTables wiring), and
* the **v3 `getList`** data source from `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/GetListAction.php` plus the production core CDR page that already binds a server-side DataTable to a v3 endpoint (`Core/src/AdminCabinet/Controllers/CallDetailRecordsController.php` + `Core/sites/admin-cabinet/assets/js/src/CallDetailRecords/call-detail-records-index.js`).

If you build a grid module, please contribute it back as `Extensions/EXAMPLES/WebInterface/ModuleExampleDataTable/` so the next reader has a single working anchor.
{% endhint %}

This recipe builds on the page scaffolding (controller, providers, asset collections) explained in [Module interface](../../module-developement/module-interface-empty.md) and the endpoint explained in [REST API in modules](../../module-developement/rest-api-in-modules.md). Read those first — here we only cover the _grid-specific_ parts.

## Architecture: who serves the rows

A server-paginated grid has two halves that meet over HTTP:

| Half            | Lives in                                                                       | Responsibility                                                                                                                                          |
| --------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data source** | `Lib/RestAPI/Numbers/Actions/GetListAction.php`                                | Reads `limit`/`offset`/`search`/`order` from the request, queries `BlackListNumbers`, returns a `PBXApiResult` with `data` + `pagination`.              |
| **Grid**        | `App/Views/.../index.volt` + `public/assets/js/src/module-black-list-index.js` | Renders an empty `<table>` and a DataTable that calls the v3 endpoint, maps DataTables' draw parameters to the v3 query, and renders the returned page. |

The controller's `indexAction()` does **almost nothing** for a grid — it loads assets and renders a static view. All data flows through the REST endpoint. This is exactly how the core CDR page works: its controller is "a minimal skeleton that only renders the CDR view page" (see the class docblock in `CallDetailRecordsController.php`).

{% hint style="info" %}
**Why v3 and not a controller AJAX action?** The older grids (e.g. `ModulePhoneBook::getNewRecordsAction()`) returned DataTables-native JSON from a cabinet controller action. That still works, but a v3 `getList` endpoint gives you one data source that serves **both** the grid _and_ external API clients, with OpenAPI docs, RBAC, and the standard `PBXApiResult` envelope for free. The core CDR page was migrated from the controller-action style to v3 for exactly this reason.
{% endhint %}

## Step 1 — the data source: GetListAction

The example `GetListAction` in the reference module queries the database for real, but stays deliberately simple — an optional status filter and ordering, but **no pagination**:

{% code title="Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Actions/GetListAction.php" %}
```php
public static function main(array $data): PBXApiResult
{
    $result = new PBXApiResult();

    // Bound parameters (:status:) — never string concatenation.
    $parameters = ['order' => 'priority DESC, id DESC'];
    if (!empty($data['status'])) {
        $parameters['conditions'] = 'status = :status:';
        $parameters['bind']       = ['status' => $data['status']];
    }

    $result->data    = Tasks::find($parameters)->toArray();
    $result->success = true;
    return $result;
}
```
{% endcode %}

A grid needs more than that: it must honour `limit`/`offset`, optionally filter on `search`, and report the **total row count** so the grid can render its pager. Here is the production-grade version for `BlackListNumbers`:

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/Actions/GetListAction.php" %}
```php
<?php

declare(strict_types=1);

namespace Modules\ModuleBlackList\Lib\RestAPI\Numbers\Actions;

use MikoPBX\PBXCoreREST\Lib\PBXApiResult;
use Modules\ModuleBlackList\Models\BlackListNumbers;

/**
 * GetListAction - server-paginated list of blocked numbers.
 *
 * Reads pagination/search/order from $data (populated from the request query
 * string), returns the page of rows in $result->data and the total count in
 * $result->pagination so the grid can size its pager.
 */
class GetListAction
{
    public static function main(array $data): PBXApiResult
    {
        $result = new PBXApiResult();

        // 1. Read pagination + search + ordering from the request.
        //    Defaults mirror CommonDataStructure: limit 20 (max 100), offset 0.
        $limit  = max(1, min(100, (int)($data['limit'] ?? 20)));
        $offset = max(0, (int)($data['offset'] ?? 0));
        $search = trim((string)($data['search'] ?? ''));

        // Whitelist sortable columns — never interpolate a raw column name.
        $allowedOrder = ['id', 'number', 'comment'];
        $order   = in_array($data['order'] ?? '', $allowedOrder, true)
            ? $data['order']
            : 'id';
        $orderWay = strtoupper((string)($data['orderWay'] ?? 'ASC')) === 'DESC'
            ? 'DESC'
            : 'ASC';

        // 2. Build the query parameters (Phalcon model find() syntax).
        $parameters = [];
        if ($search !== '') {
            $parameters['conditions'] = 'number LIKE :search: OR comment LIKE :search:';
            $parameters['bind']['search'] = "%{$search}%";
        }

        // 3. Count the filtered total BEFORE applying limit/offset.
        $total = BlackListNumbers::count($parameters);

        // 4. Fetch only the requested page.
        $parameters['order']  = "{$order} {$orderWay}";
        $parameters['limit']  = $limit;
        $parameters['offset'] = $offset;

        $rows = BlackListNumbers::find($parameters)->toArray();

        // 5. Return the page + pagination metadata.
        $result->data = $rows;
        $result->pagination = [
            'total'   => (int)$total,
            'limit'   => $limit,
            'offset'  => $offset,
            'hasMore' => ($offset + $limit) < (int)$total,
        ];
        $result->success = true;

        return $result;
    }
}
```
{% endcode %}

{% hint style="info" %}
The `pagination` field is a first-class property of `PBXApiResult` (`Core/src/PBXCoreREST/Lib/PBXApiResult.php`): `public ?array $pagination = null`. Its documented keys are `total`, `limit`, `offset`, `hasMore`, `lastId`. When set, `getResult()` emits it as a top-level `pagination` object in the JSON envelope.
{% endhint %}

### Wiring the pagination parameters into the Controller

The `getList` query parameters come from the shared `MikoPBX\PBXCoreREST\Lib\Common\CommonDataStructure`. Reference them on your controller's `getList()` method so they are validated and documented:

{% code title="Extensions/ModuleBlackList/Lib/RestAPI/Numbers/Controller.php" %}
```php
use MikoPBX\PBXCoreREST\Lib\Common\CommonDataStructure;

#[ApiDataSchema(schemaClass: DataStructure::class, type: 'list', isArray: true)]
#[ApiOperation(summary: 'List blocked numbers', operationId: 'getNumbersList')]
#[ApiParameterRef('limit', dataStructure: CommonDataStructure::class)]
#[ApiParameterRef('offset', dataStructure: CommonDataStructure::class)]
#[ApiParameterRef('search', dataStructure: CommonDataStructure::class)]
#[ApiParameterRef('order', dataStructure: CommonDataStructure::class, enum: ['id', 'number', 'comment'])]
#[ApiParameterRef('orderWay', dataStructure: CommonDataStructure::class)]
#[ApiResponse(200, 'OK')]
public function getList(): void {}
```
{% endcode %}

`CommonDataStructure::getPaginationParameters()` defines `limit` (integer, min 1, max 100, default 20) and `offset` (integer, min 0, default 0). `getSearchAndOrderParameters()` defines `search` (string, max 255), `order` (string — override its `enum` per resource as above), and `orderWay` (enum `ASC`/`DESC`, default `ASC`). The reference `Tasks` controller demonstrates the exact same references — see `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/Controller.php`, method `getList()`.

{% hint style="warning" %}
**Confirm where your params land.** The query string is delivered to the action as the `$data` array (`$request['data']` inside `Processor::callBack()`). Whether a given key reaches `$data` depends on it being declared via `#[ApiParameterRef]` on the controller method — undeclared query params may be dropped by sanitization. Declare every parameter your action reads.
{% endhint %}

## Step 2 — the index view: an empty table

The Volt view renders the page chrome (search box, "add" button, page-size selector) and an **empty** `<table>` with a `<thead>` only. DataTables fills the `<tbody>` from the AJAX response. This markup is adapted directly from the production phonebook grid (`Extensions/ModulePhoneBook/App/Views/ModulePhoneBook/Tabs/phonebookTab.volt`):

{% code title="App/Views/ModuleBlackList/index.volt" %}
```twig
<div class="ui grid">
    <div class="ui row">
        <div class="ui seven wide column">
            {{ link_to(
                "#",
                '<i class="add icon"></i>  ' ~ t._('module_black_list_AddNewRecord'),
                "class": "ui blue button", "id": "add-new-button"
            ) }}
        </div>
        <div class="ui nine wide column">
            <div class="ui search right action left icon fluid input" id="search-block-input">
                <i class="search link icon" id="search-icon"></i>
                <input type="search" id="global-search" name="global-search"
                       placeholder="{{ t._('module_black_list_Search') }}" class="prompt">
                <div class="results"></div>
                <div class="ui basic floating search dropdown button" id="page-length-select">
                    <div class="text">{{ t._('ex_CalculateAutomatically') }}</div>
                    <i class="dropdown icon"></i>
                    <div class="menu">
                        <div class="item" data-value="auto">{{ t._('ex_CalculateAutomatically') }}</div>
                        <div class="item" data-value="25">{{ t._('ex_ShowOnlyRows', {'rows':25}) }}</div>
                        <div class="item" data-value="50">{{ t._('ex_ShowOnlyRows', {'rows':50}) }}</div>
                        <div class="item" data-value="100">{{ t._('ex_ShowOnlyRows', {'rows':100}) }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
<br>

<table id="black-list-table" class="ui small very compact single line table">
    <thead>
    <tr>
        <th class="ten wide">{{ t._('module_black_list_ColumnNumber') }}</th>
        <th class="five wide">{{ t._('module_black_list_ColumnComment') }}</th>
        <th class="collapsing"></th>
    </tr>
    </thead>
    <tbody></tbody>
</table>
```
{% endcode %}

* `t._('key')` translates every label — see [Translations](../../module-developement/translations.md).
* `ex_CalculateAutomatically` and `ex_ShowOnlyRows` are **core** translation keys shared by every cabinet grid, so you do not have to define them yourself.
* The column count in `<thead>` must match the `columns` array in the JS (Step 4).

## Step 3 — register the grid assets in the controller

The index action attaches the DataTables vendor library (CSS + JS) and your compiled grid script, then renders the static view. The vendor asset paths below are the ones the phonebook controller registers (`Extensions/ModulePhoneBook/App/Controllers/ModulePhoneBookController.php`):

{% code title="App/Controllers/ModuleBlackListController.php" %}
```php
public function indexAction(): void
{
    $this->view->logoImagePath = $this->url->get() . 'assets/img/cache/' . $this->moduleUniqueID . '/logo.svg';
    $this->view->submitMode = null;

    // DataTables vendor CSS (shipped with the core cabinet).
    $headerCollectionCSS = $this->assets->collection(AssetProvider::HEADER_CSS);
    $headerCollectionCSS
        ->addCss('css/vendor/datatable/dataTables.semanticui.min.css', true)
        ->addCss('css/cache/' . $this->moduleUniqueID . '/module-black-list-index.css', true);

    // DataTables vendor JS + your compiled grid script.
    $footerCollectionJS = $this->assets->collection(AssetProvider::FOOTER_JS);
    $footerCollectionJS
        ->addJs('js/vendor/datatable/dataTables.semanticui.js', true)
        ->addJs('js/cache/' . $this->moduleUniqueID . '/module-black-list-index.js', true);

    $this->view->pick('Modules/' . $this->moduleUniqueID . '/ModuleBlackList/index');
}
```
{% endcode %}

`AssetProvider` constants (`HEADER_CSS`, `FOOTER_JS`, …) and the `js/cache/` / `css/cache/` compiled-asset convention are explained in [Module interface](../../module-developement/module-interface-empty.md). The core cabinet already loads the JWT helper `js/pbx/main/token-manager.js` on every page (registered in `Core/src/AdminCabinet/Providers/AssetProvider.php`), so you do **not** add it yourself — see the auth note in Step 4.

## Step 4 — the grid script: bind DataTables to v3

This is the heart of the recipe. A **server-side** DataTable (`serverSide: true`) emits, on every draw, a request describing the page it wants (`start`, `length`, `search.value`, `order`). You translate those into the v3 query parameters, and translate the `PBXApiResult` envelope back into what DataTables expects.

The two adapters are `ajax.data` (out) and `ajax.dataSrc` (in). This pattern is copied from the production CDR page (`Core/sites/admin-cabinet/assets/js/src/CallDetailRecords/call-detail-records-index.js`):

{% code title="public/assets/js/src/module-black-list-index.js" %}
```javascript
/* global globalRootUrl, globalTranslate, SemanticLocalization, TokenManager, UserMessage */

const ModuleBlackList = {
    $table: $('#black-list-table'),
    $globalSearch: $('#global-search'),
    $addNewButton: $('#add-new-button'),
    dataTable: null,

    // v3 endpoint: /pbxcore/api/v3/module-{slug}/{resource}
    apiUrl: '/pbxcore/api/v3/module-black-list/numbers',

    initialize() {
        ModuleBlackList.initializeDataTable();
        ModuleBlackList.initializeEvents();
    },

    initializeDataTable() {
        ModuleBlackList.$table.dataTable({
            serverSide: true,        // fetch one page at a time from the server
            processing: true,        // show the "loading" overlay during fetches
            paging: true,
            ordering: true,
            sDom: 'rtip',            // hide DataTables' own search box (we use #global-search)
            language: SemanticLocalization.dataTableLocalisation,

            // Column-to-field mapping. Order must match the <thead> in index.volt.
            columns: [
                { data: 'number' },
                { data: 'comment' },
                { data: null, orderable: false, defaultContent: '' },  // action buttons
            ],

            ajax: {
                url: ModuleBlackList.apiUrl,
                type: 'GET',

                // --- OUT: DataTables draw params  ->  v3 query params ---
                data(d) {
                    const params = {
                        limit:  d.length,                 // page size
                        offset: d.start,                  // first row index
                        search: (d.search && d.search.value) ? d.search.value.trim() : '',
                    };
                    // Translate the active sort column into order / orderWay.
                    if (d.order && d.order.length > 0) {
                        const col = d.columns[d.order[0].column];
                        if (col && col.data) {
                            params.order = col.data;
                            params.orderWay = d.order[0].dir.toUpperCase(); // ASC | DESC
                        }
                    }
                    return params;
                },

                // --- IN: PBXApiResult envelope  ->  DataTables rows ---
                // The envelope wire shape is { result, data, messages, pagination, ... }.
                // NOTE the key is `result` (boolean), NOT `success`.
                dataSrc(json) {
                    if (json.result === true) {
                        const pagination = json.pagination || {};
                        const total = pagination.total || 0;
                        json.recordsTotal = total;       // DataTables needs these two
                        json.recordsFiltered = total;    // counts to render its pager
                        return json.data || [];          // the array of row objects
                    }
                    if (json.messages && json.messages.error) {
                        UserMessage.showMultiString(json.messages.error);
                    }
                    return [];
                },

                // --- AUTH: attach the admin's JWT as a Bearer token ---
                beforeSend(xhr) {
                    if (typeof TokenManager !== 'undefined' && TokenManager.accessToken) {
                        xhr.setRequestHeader('Authorization', `Bearer ${TokenManager.accessToken}`);
                    }
                },
            },

            // Render the action buttons into the last column of each row.
            createdRow(row, data) {
                const buttons = `
                    <div class="ui basic icon buttons action-buttons tiny">
                        <a href="#" class="ui button delete" data-value="${data.id}">
                            <i class="icon trash red"></i>
                        </a>
                    </div>`;
                $('td', row).eq(2).html(buttons);
            },
        });

        ModuleBlackList.dataTable = ModuleBlackList.$table.DataTable();
    },

    initializeEvents() {
        // Debounced search: feed #global-search into the DataTable.
        let searchTimer = null;
        ModuleBlackList.$globalSearch.on('keyup', () => {
            clearTimeout(searchTimer);
            searchTimer = setTimeout(() => {
                ModuleBlackList.dataTable.search(ModuleBlackList.$globalSearch.val()).draw();
            }, 500);
        });

        // Delete a row, then reload the current page from the server.
        ModuleBlackList.$table.on('click', 'a.delete', (e) => {
            e.preventDefault();
            const id = $(e.currentTarget).data('value');
            ModuleBlackList.deleteRecord(id);
        });

        // "Add" opens the modify form (single-record recipe).
        ModuleBlackList.$addNewButton.on('click', (e) => {
            e.preventDefault();
            window.location = `${globalRootUrl}module-black-list/module-black-list/modify`;
        });
    },

    deleteRecord(id) {
        $.api({
            url: `${ModuleBlackList.apiUrl}/${id}`,   // DELETE /pbxcore/api/v3/.../numbers/{id}
            method: 'DELETE',
            on: 'now',
            successTest: (response) => response && response.result === true,
            onSuccess() {
                // Keep the current page/scroll position (false = do not reset paging).
                ModuleBlackList.dataTable.ajax.reload(null, false);
            },
            onFailure(response) {
                UserMessage.showMultiString(response.messages);
            },
        });
    },
};

$(document).ready(() => {
    ModuleBlackList.initialize();
});
```
{% endcode %}

### How the cabinet authenticates a v3 call from the browser

The `getList` resource is declared `#[ResourceSecurity(..., requirements: [SecurityType::LOCALHOST, SecurityType::BEARER_TOKEN])]` (see [REST API in modules](../../module-developement/rest-api-in-modules.md)). A browser is **not** localhost, so it must send a Bearer token. MikoPBX mints a short-lived JWT for the logged-in admin and exposes it through the global `TokenManager` object:

* `Core/sites/admin-cabinet/assets/js/src/main/token-manager.js` defines `TokenManager` (exposed as `window.TokenManager`). It holds the JWT in `TokenManager.accessToken` (in memory only — never `localStorage`) and refreshes it silently.
* On script load it calls `TokenManager.setupGlobalAjax()`, which installs a global jQuery `ajaxSend`-style hook that **automatically** adds `Authorization: Bearer <accessToken>` to every jQuery AJAX request that does not already carry the header.

So in practice the `beforeSend` above is belt-and-suspenders — the core CDR page sets it explicitly, and `setupGlobalAjax()` would add it anyway. Keep the explicit `beforeSend` for clarity and to make the dependency obvious.

{% hint style="danger" %}
**Do not point the grid at a v3 endpoint declared `PUBLIC`-only or one without a Bearer requirement just to dodge auth.** If the request reaches the server without a valid token it returns `401` and the grid shows an empty table with no obvious cause. The fix is always: ensure `SecurityType::BEARER_TOKEN` is in the resource's requirements and the page loads `token-manager.js` (it does, on every cabinet page).
{% endhint %}

{% hint style="info" %}
**Envelope shape: flat vs nested.** The example above returns a **flat** `data` array and a top-level `pagination` object — the natural output of `PBXApiResult` when you set `$result->data` and `$result->pagination`. The production CDR action wraps its payload one level deeper — `data: { records: [...], pagination: {...} }` — and its `dataSrc` reads `json.data.records` / `json.data.pagination` accordingly (see `Core/src/PBXCoreREST/Lib/Cdr/GetListAction.php`). Both are valid; just keep your action and your `dataSrc` in agreement.
{% endhint %}

## Step 5 — add and delete rows

* **Delete** is shown above: a trash button per row carries `data-value="${data.id}"`; clicking it issues `DELETE /pbxcore/api/v3/module-black-list/numbers/{id}`, then calls `dataTable.ajax.reload(null, false)` to refresh the current page without losing scroll position. The `DELETE` verb maps to the `delete` operation in `#[HttpMapping]`, routed by the Processor to `DeleteRecordAction`.
* **Add** can either open a dedicated modify form (shown above — `window.location = .../modify`) or `POST` a new row to the collection endpoint (`POST /pbxcore/api/v3/module-black-list/numbers`) and reload the grid. For the full create/update flow (the `SaveRecordAction` and its 7-phase pattern) see [REST API in modules](../../module-developement/rest-api-in-modules.md).

For inline (in-cell) editing — typing directly into a table cell and saving on blur — study the production phonebook grid (`Extensions/ModulePhoneBook/public/assets/js/src/module-phonebook-datatable.js`): its `buildRowTemplate()` renders editable inputs and `sendChangesToServer()` persists each changed row. That module uses a cabinet controller action rather than v3, but the **client-side editing mechanics** transfer directly.

## Step 6 — compile the JavaScript

The browser only ever loads the compiled file from `js/cache/<moduleUniqueID>/`; the `src/` file is never served directly. That URL prefix is **not** a build output directory — the installer symlinks `js/cache/<moduleUniqueID>` straight to your module's `public/assets/js` (`PbxExtensionUtils::createAssetsSymlinks()`, `Core/src/Modules/PbxExtensionUtils.php:105-114`; `css` and `img` are symlinked the same way). So Babel's `--out-dir` is `public/assets/js`, the parent of `src/`:

{% code title="bash" %}
```bash
babel public/assets/js/src/module-black-list-index.js \
  --out-dir public/assets/js \
  --source-maps inline \
  --presets airbnb
```
{% endcode %}

{% hint style="warning" %}
If your code edit does not appear in the running cabinet, you almost always forgot this step (or need to clear the browser cache). The compiled path is what the controller registers in Step 3.
{% endhint %}

## Checklist

1. **Data source** — `GetListAction` reads `limit`/`offset`/`search`/`order` from `$data`, sets `$result->data` (the page) and `$result->pagination` (`total`, `limit`, `offset`, `hasMore`).
2. **Controller (REST)** — `getList()` references the pagination/search params via `#[ApiParameterRef(..., dataStructure: CommonDataStructure::class)]`.
3. **Controller (cabinet)** — `indexAction()` attaches the DataTables vendor assets + your compiled grid script and picks the static index view.
4. **View** — an empty `<table>` with a `<thead>` whose column count matches the JS `columns` array.
5. **Grid JS** — `serverSide: true`; `ajax.data` maps DataTables → v3 query; `ajax.dataSrc` reads `json.result`/`json.pagination.total`/`json.data` and sets `recordsTotal`/`recordsFiltered`.
6. **Auth** — the page carries `token-manager.js` (every cabinet page does), and `TokenManager.accessToken` is sent as `Authorization: Bearer …`.
7. **Compile** the JS into `public/assets/js/` (which `js/cache/<id>` symlinks to) before testing.

## See also

* [Forms overview](./) — the three form recipes and how they relate.
* [Create a module form](create-module-form.md) — the single-record settings form (the "modify" page the Add button links to).
* [REST API in modules](../../module-developement/rest-api-in-modules.md) — the full v3 endpoint pattern: Controller attributes, Processor, Actions, `DataStructure`, and the 7-phase save flow.
* [Module interface](../../module-developement/module-interface-empty.md) — the controller, view, provider and asset-collection scaffolding this recipe builds on.
* Real anchors to read:
  * `Core/src/AdminCabinet/Controllers/CallDetailRecordsController.php` and `Core/sites/admin-cabinet/assets/js/src/CallDetailRecords/call-detail-records-index.js` — a production server-side DataTable bound to a v3 endpoint.
  * `Extensions/ModulePhoneBook/` — a production grid module (controller-action data source, inline editing).
  * `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/Lib/RestAPI/Tasks/` — the v3 endpoint skeleton (`Controller`, `Processor`, `GetListAction`).
