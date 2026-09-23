---
description: >-
  How module licensing and the MikoPBX marketplace work, at a high level — the
  author-facing model for commercial modules and version-filtered distribution.
---

# Licensing

This page explains, at an overview level, how module **licensing** and the **MikoPBX marketplace** fit together from a module author's point of view. It deliberately stays high-level: the license server, its endpoints, and the release infrastructure are internal MIKO services. What you need to know as an author is which keys go into your `module.json`, what the Core does with them at install and enable time, and how the marketplace decides which version of your module to offer a given PBX.

Throughout this page the running example is the fictional **ModuleBlackList** module (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers`). For a real, working commercial module you can read end to end, see `Extensions/ModuleAmoCrm/`. For a free module with no license keys at all, see `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`.

{% hint style="info" %}
**Free vs. commercial.** Licensing is entirely opt-in. A module is "free" simply by **not** declaring `lic_product_id` in its `module.json`. The Core treats `lic_product_id = 0` (the default when the key is absent) as "no license required" and skips every licensing step described below. `ModuleExampleForm` is exactly such a module — its `module.json` has no `lic_product_id` at all.
{% endhint %}

## The two license keys in module.json

A commercial module declares its licensing identity with two integer keys in [`module.json`](../module-developement/module-json.md):

| Key              | Meaning                                                                                 |
| ---------------- | --------------------------------------------------------------------------------------- |
| `lic_product_id` | The **product** this module is sold as. Drives trial activation at install time.        |
| `lic_feature_id` | The **feature** that must be present in the license key for the module to run (enable). |

Here is the relevant slice of the real `ModuleAmoCrm/module.json` (`Extensions/ModuleAmoCrm/module.json`):

{% code title="module.json" %}
```json
{
  "developer": "MIKO",
  "moduleUniqueID": "ModuleAmoCrm",
  "support_email": "help@miko.ru",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2022.2.1",
  "lic_product_id": 81,
  "lic_feature_id": 45
}
```
{% endcode %}

For **ModuleBlackList** you would declare your own MIKO-assigned IDs, for example:

{% code title="Extensions/ModuleBlackList/module.json" %}
```json
{
  "developer": "Your Company",
  "moduleUniqueID": "ModuleBlackList",
  "support_email": "support@example.com",
  "version": "%ModuleVersion%",
  "min_pbx_version": "2025.1.1",
  "lic_product_id": 1234,
  "lic_feature_id": 5678
}
```
{% endcode %}

{% hint style="warning" %}
**Product and feature IDs are issued by MIKO.** You do not invent them. To sell a module through the marketplace you register the product with MIKO and receive the `lic_product_id` / `lic_feature_id` values to put in your `module.json`. Contact MIKO (`help@miko.ru`) to obtain them.
{% endhint %}

## What the Core does at install time: `activateLicense()`

When a module is installed, `PbxExtensionSetupBase::installModule()` runs a fixed sequence of steps; the second of them is `activateLicense()`. See the base class at `Core/src/Modules/Setup/PbxExtensionSetupBase.php`.

The logic is short and worth reading in full:

{% code title="Core/src/Modules/Setup/PbxExtensionSetupBase.php" %}
```php
public function activateLicense(): bool
{
    if ($this->lic_product_id > 0) {
        $lic = PbxSettings::getValueByKey(PbxSettings::PBX_LICENSE);
        if (empty($lic)) {
            $this->messages[] = $this->translation->_("ext_EmptyLicenseKey");
            return false;
        }

        // Get trial license for the module
        $this->license->addtrial($this->lic_product_id);
    }
    return true;
}
```
{% endcode %}

In plain terms:

1. If `lic_product_id` is `0` (a free module), `activateLicense()` does nothing and returns `true`. No license key is required to install.
2. If `lic_product_id > 0`, the PBX must already hold a license key (`PbxSettings::PBX_LICENSE`). With no key the install **fails** with the `ext_EmptyLicenseKey` message, and because `activateLicense()` returns `false`, `installModule()` aborts the whole installation.
3. With a key present, the Core asks the license service for a trial of the product via `$this->license->addtrial($this->lic_product_id)`. This requests a time-limited trial entitlement against the PBX's license key so the customer can evaluate the module immediately after install.

The `$this->license` worker is the `MarketPlaceProvider` shared service (`MikoPBX\Common\Providers\MarketPlaceProvider`), injected in the setup-base constructor. The underlying `addtrial` call talks to the MIKO license service — authors never call it directly and never need its internals.

{% hint style="info" %}
`activateLicense()` only requests a **trial**. It does not, by itself, guarantee the module will run. The real gate that decides whether a commercial module may be **enabled** is the feature check described next.
{% endhint %}

For the full install pipeline (`checkCompatibility()` → `activateLicense()` → `installFiles()` → `installDB()` → `fixFilesRights()`), see the [module installer](../module-developement/module-installer.md) page.

## What happens at enable time: the feature gate and `DISABLED_BY_LICENSE`

Installing a module only places it on the system in a **disabled** state. The licensing decision that matters for day-to-day operation happens when the module is **enabled**, in `PbxExtensionState::enableModule()` (`Core/src/Modules/PbxExtensionState.php`).

If the module declares a `lic_feature_id > 0`, the Core asks the license service whether that feature is currently available **before** enabling anything:

{% code title="Core/src/Modules/PbxExtensionState.php" %}
```php
if ($this->lic_feature_id > 0) {
    // Try to capture the feature if it is set
    $result = $this->license->featureAvailable($this->lic_feature_id);
    if ($result['success'] === false) {
        $textError = (string)($result['error'] ?? '');
        $reasonText = $this->license->translateLicenseErrorMessage($textError);
        $this->messages['license'][] = $reasonText;

        // Find and update the module's disabled flag in the database
        $module = PbxExtensionModules::findFirstByUniqid($this->moduleUniqueID);
        if ($module !== null) {
            $module->disabled = '1';
            $module->disableReason = self::DISABLED_BY_LICENSE;
            $module->disableReasonText = $reasonText;
            $module->save();
        }
        return false;
    }
}
```
{% endcode %}

When the feature is **not** available, the module stays disabled and the Core records _why_: it sets `disableReason` to the constant `PbxExtensionState::DISABLED_BY_LICENSE` and stores the human-readable explanation (translated via `translateLicenseErrorMessage()`) in `disableReasonText`. This is the mechanism by which a module that **loses** its license — for example, when the customer's license expires or the feature is revoked — is automatically taken out of service: the next enable attempt fails and the module is marked `DISABLED_BY_LICENSE`.

`DISABLED_BY_LICENSE` is one of a small set of disable-reason constants defined on `PbxExtensionState`:

{% code title="Core/src/Modules/PbxExtensionState.php" %}
```php
public const string DISABLED_BY_EXCEPTION  = 'DisabledByException';
public const string DISABLED_BY_USER       = 'DisabledByUser';
public const string DISABLED_BY_LICENSE    = 'DisabledByLicense';
public const string DISABLED_BY_CRASH_LOOP = 'DisabledByCrashLoop';
```
{% endcode %}

{% hint style="info" %}
**Two keys, two roles.** `lic_product_id` gates **installation** (it triggers the trial request in `activateLicense()`); `lic_feature_id` gates **enabling** (it triggers the runtime feature check in `enableModule()`). A commercial module typically declares both, as `ModuleAmoCrm` does.
{% endhint %}

So for **ModuleBlackList**: declaring `lic_product_id` means a customer cannot install it without a license key on the PBX, and a trial is requested on install; declaring `lic_feature_id` means each time the module is enabled the Core confirms the license still grants that feature, and quietly parks the module as `DISABLED_BY_LICENSE` if it does not.

## Marketplace distribution and version filtering

The marketplace side answers a different question: _which build of your module should a given PBX be offered?_

Modules are published through the MIKO **release pipeline**, configured by the `release_settings` block in `module.json`. `ModuleAmoCrm/module.json` shows the real shape:

{% code title="Extensions/ModuleAmoCrm/module.json" %}
```json
{
  "release_settings": {
    "publish_release": true,
    "changelog_enabled": true,
    "create_github_release": true
  }
}
```
{% endcode %}

| `release_settings` key  | Effect                                                     |
| ----------------------- | ---------------------------------------------------------- |
| `publish_release`       | Whether the build is published to the MIKO release server. |
| `changelog_enabled`     | Whether a changelog is generated for the release.          |
| `create_github_release` | Whether a corresponding GitHub release is created.         |

Once published, the **MIKO release server** lists the module and serves compatibility metadata for each release, including `min_pbx_version` and `max_pbx_version`. When a PBX queries the marketplace, the server uses that metadata to **filter** which releases are offered to that specific PBX version: a release whose compatibility window does not include the PBX's version is simply not presented.

{% hint style="warning" %}
**Version filtering is server-side.** The compatibility window (`min_pbx_version` / `max_pbx_version`) is enforced by the release server when it decides what to _offer_. The Core itself only checks the **lower** bound at install time: `PbxExtensionSetupBase::checkCompatibility()` compares the running PBX version against the module's `min_pbx_version` and refuses to install anything older than required. **The Core does not enforce `max_pbx_version` locally** — there is no `max_pbx_version` check in the install path. The upper bound exists purely as marketplace metadata for filtering offers.
{% endhint %}

For reference, here is the Core's only local version gate — note it is a one-sided `min_pbx_version` check:

{% code title="Core/src/Modules/Setup/PbxExtensionSetupBase.php" %}
```php
public function checkCompatibility(): bool
{
    $currentVersionPBX = PbxSettings::getValueByKey(PbxSettings::PBX_VERSION);
    $currentVersionPBX = str_replace('-dev', '', $currentVersionPBX);
    if (version_compare($currentVersionPBX, $this->min_pbx_version) < 0) {
        $this->messages[] = $this->translation->_(
            "ext_ModuleDependsHigherVersion",
            ['version' => $this->min_pbx_version]
        );
        return false;
    }
    return true;
}
```
{% endcode %}

In practice this means: set `min_pbx_version` honestly (the Core will hold you to it on every install), and rely on the release server's `max_pbx_version` metadata to stop offering an old build to PBX versions it was never tested against.

## Author checklist

For a free module (like `ModuleExampleForm`):

* Do **not** add `lic_product_id` or `lic_feature_id`. Installation and enabling skip all licensing.
* You may still publish through the marketplace via `release_settings`.

For a commercial module (like `ModuleAmoCrm`, or our `ModuleBlackList`):

1. Obtain `lic_product_id` and `lic_feature_id` from **MIKO** (`help@miko.ru`).
2. Add both keys to your [`module.json`](../module-developement/module-json.md).
3. Set an accurate `min_pbx_version` — the Core enforces it on install.
4. Add a `release_settings` block so the build is published and version-filtered by the release server.
5. Test the loss-of-license path: with the feature revoked, your module should end up disabled with reason `DISABLED_BY_LICENSE` and remain functional otherwise (no fatal errors in your config/main classes).

## See also

* [module.json reference](../module-developement/module-json.md) — every key, including `lic_product_id`, `lic_feature_id`, `min_pbx_version`, and `release_settings`.
* [Module installer](../module-developement/module-installer.md) — the full `installModule()` pipeline and where `activateLicense()` and `checkCompatibility()` run.
* Real commercial module: `Extensions/ModuleAmoCrm/`.
* Real free module: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`.
