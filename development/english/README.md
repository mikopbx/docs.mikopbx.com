---
description: >-
  This guide helps you to create new modules for MikoPBX and understand its
  internal structure.
---

# MikoPBX development guide

## What is MikoPBX?

MikoPBX is a free, open-source phone system for small and medium business. Under the hood it runs the [Asterisk](https://www.asterisk.org) PBX engine, wrapped in a modern web administration interface built on the [Phalcon](https://phalcon.io) MVC framework. The whole product ships as a turnkey Linux firmware that already bundles every service a phone system needs — Asterisk, Nginx, PHP-FPM, the firewall (iptables) and fail2ban, a key-value store and the background workers that glue them together.

MikoPBX is **entirely modular**. The admin GUI, the dialplan generation, the REST API surface and the background workers can all be extended by third-party modules without patching the Core. That means you can build virtually any feature you can imagine, ship it to your customers through the MIKO marketplace (or distribute it for free), and have it install cleanly on top of any compatible MikoPBX firmware.

### The architecture in one picture

| Layer                    | Technology                                                     | What lives here                                   |
| ------------------------ | -------------------------------------------------------------- | ------------------------------------------------- |
| Telephony engine         | Asterisk                                                       | Dialplan, PJSIP/SIP peers and providers, AMI, AGI |
| Admin interface          | Phalcon MVC (controllers, Volt views, forms) + ES6+ JavaScript | The web UI you and your module extend             |
| Persistent data          | SQLite (one database per module)                               | Module settings, models, relationships            |
| Runtime / cache / queues | Redis and background PHP workers                               | Inter-process state, event queues, scheduled jobs |

A module plugs into all four layers through a small set of well-defined classes — a configuration class that hooks into Core, an installer that provisions your database, Phalcon models that describe your tables, and (optionally) controllers, REST API actions, workers and AGI scripts.

## Platform target and version baseline

There are two version numbers to keep straight, and they come from two different sources. Treat them accordingly.

{% hint style="info" %}
**Core runtime platform — authoritative.** `Core/composer.json` is the source of truth for what the firmware actually runs. The current Core requires:

* `"php": "^8.4"`
* `"ext-phalcon": "^5.9.3"`

It also pins the Composer platform to PHP `8.4` (see the `config.platform` block). Your code runs inside this runtime, so it must be valid on PHP 8.x and Phalcon 5.x.
{% endhint %}

{% hint style="success" %}
**Module development baseline — recommended for new modules.** The official `mikopbx-module` skill from [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills) targets the same platform for newly generated modules:

* PHP **8.4**
* Phalcon **5.9.3**
* `min_pbx_version` **2025.1.1**
* **No** legacy-compatibility shims (no `MikoPBXVersion.php`)

In practice: write PHP 8.4-clean code, target Phalcon 5.x APIs, and set `min_pbx_version` to `2025.1.1` in your `module.json` so the firmware refuses to install your module on incompatible releases.
{% endhint %}

{% hint style="warning" %}
The shipped template at `Extensions/ModuleTemplate/module.json` still declares `"min_pbx_version": "2023.2.150"`. That value is a placeholder from an older template — for a **new** module raise it to `2025.1.1` to match the current development baseline above.
{% endhint %}

This documentation is written strictly for that baseline. You will not find Phalcon 4 APIs, PHP 7 syntax, or `MikoPBXVersion.php`-style compatibility code anywhere in this guide. When the framework matters, the relevant Phalcon 5.x reference is linked inline (for example, the model layer is covered by the [Phalcon 5 ORM documentation](https://docs.phalcon.io/5.0/en/db-models)).

## How to start module development

The path from zero to an installable module is short:

1. **Prepare your tools.** Set up your IDE, PHP, and the helper utilities used to scaffold and debug a module. See [Prepare IDE and system tools](prepare-ide-tools/) for Windows, Linux and macOS instructions.
2. **Scaffold from the template.** Clone the module template and rename its folders, files and classes with the provided script. See [How to start](module-developement/template-module-structure.md). The reference template lives in the repo at `Extensions/ModuleTemplate/`.
3. **Install it on a MikoPBX instance** and iterate. Your skeleton module is installable immediately; you then layer real behavior on top.

Throughout this guide we follow a single running example — a fictional module named **`ModuleBlackList`** that blocks unwanted callers. Its configuration class is `BlackListConf`, its main class is `BlackListMain`, its model is `BlackListNumbers` (backed by the SQLite table `m_BlackListNumbers`), and its front-end script is `module-black-list-index.js`. Every pattern in the guide is illustrated with this module and then anchored to a **real, working example module** in the repository so you can read production-quality source. The canonical examples are:

* Web UI form: `Extensions/EXAMPLES/WebInterface/ModuleExampleForm/`
* REST API (current v3 style): `Extensions/EXAMPLES/REST-API/ModuleExampleRestAPIv3/`
* AMI background worker: `Extensions/EXAMPLES/AMI/ModuleExampleAmi/`

{% hint style="info" %}
When in doubt about a pattern, read the actual source of these example modules — they are the authoritative reference, kept in sync with the current Core.
{% endhint %}

## What a module can do

The configuration and main classes give your module deep, supported access into the Core. From them you can:

* Generate Asterisk config fragments and add your own contexts to `extensions.conf`.
* Override PJSIP parameters for peers and providers.
* Register Asterisk Manager (AMI) accounts.
* Add firewall rules and fail2ban jail configuration.
* Schedule cron tasks and run long-lived PHP workers.
* Expose REST API routes and add pages to the admin interface.

Start with a clean **data model** (your SQLite tables described as Phalcon models), then write the **installer** that provisions that database at install time, then wire behavior through the **module main class**. These are covered step by step in the Module development chapter linked below.

## Orientation map

This guide is organized into five parts. Pick the one that matches what you are doing:

### 1. Module development

The end-to-end build flow: scaffolding from the template, designing your [data model](module-developement/data-model.md), writing the [installer class](module-developement/module-installer.md), the [module main class](module-developement/module-class.md), the [module interface](module-developement/module-interface-empty.md), and [translations](module-developement/translations.md). Start here: [How to start](module-developement/template-module-structure.md).

### 2. Internal structure

How MikoPBX is built underneath your module — the [Core](internal-structure/core.md), the admin interface, and the public integration surfaces: the [REST API](internal-structure/api/rest-api.md), [AMI / AJAM](internal-structure/api/ami-ajam.md), and [AGI](internal-structure/api/agi.md). Read this when you need to understand _why_ a hook exists or _where_ your data flows.

### 3. Cookbook

Task-focused recipes you can copy: building [forms](cookbook/forms/) and [datatables](cookbook/forms/create-datatable.md), [hooking on incoming calls](cookbook/asterisk/hook-on-incoming-call.md), [interacting with AMI](cookbook/asterisk/interact-with-ami.md), [modifying `extensions.conf`](cookbook/asterisk/modify-extensions.conf.md), and handling [rights and authentication](cookbook/rights-and-auth/).

### 4. Marketplace

How to publish a finished module and, if you want to, charge for it — see [Licensing](marketplace/licensing.md).

### 5. AI-assisted development

MikoPBX publishes open-standard [Agent Skills](https://agentskills.io) in the [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills) repository. The main one, `mikopbx-module`, scaffolds and augments modules from a natural-language description, following the exact conventions in this guide; it installs into Claude Code, Codex, Cursor, Gemini CLI and other agents with `npx skills add mikopbx/agent-skills`. See the [AI-assisted development](ai-assisted-development/ai-assisted-development.md) chapter for how to install and drive it.

## Make money or share it for free

When your module is ready you can publish it to the MIKO marketplace, where it reaches every MikoPBX customer. If you want to monetize it, MikoPBX provides licensing features so you can sell paid modules. See the [Licensing](marketplace/licensing.md) articles.

Develop, [debug](module-developement/debuging/), and ship — and ask our community for help along the way. Welcome to the family.
