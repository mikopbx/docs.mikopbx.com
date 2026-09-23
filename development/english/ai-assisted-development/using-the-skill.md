---
description: >-
  How to drive the /mikopbx-module skill: prompting, the discovery dialog and
  recipe selection.
---

# Using the skill

The `/mikopbx-module` skill turns a natural-language description into a complete, convention-correct MikoPBX module. It runs in one of four modes — create a new module, augment an existing one, optimize one against the reference standards, or simply answer an architecture question without writing files — and it works best when you feed it the _problem_ rather than a file list. This page shows how to prompt it, what questions it will ask, and how the words you use map to the code it generates.

For the bigger picture see [What the skill is](ai-assisted-development.md); for the file inventory each recipe produces see [What it generates](what-it-generates.md).

{% hint style="info" %}
The skill is defined in `skills/mikopbx-module/SKILL.md` of the [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills) repository and its recipe specifications live in `skills/mikopbx-module/reference/recipes.md` next to it. Everything on this page traces back to those two files. Install the skill first — see [Installing the skills](ai-assisted-development.md#installing-the-skills).
{% endhint %}

## How the skill activates

The skill recognizes both an explicit invocation and a set of natural-language triggers. From `SKILL.md` ("Task Activation Patterns"):

* `/mikopbx-module ...` — explicit invocation (Claude Code; other agents match on the `SKILL.md` description, so the phrases below are enough)
* "Create a module ..." / "Создай модуль ..."
* "Generate a module ..." / "Сгенерируй модуль ..."
* "Add to the module ..." / "Добавь в модуль ..."
* "Optimize the module ..." / "Оптимизируй модуль ..."
* "How do I make a module ..." / "Как сделать модуль ..."
* "Improve the module ..." / "Доработай модуль ..."

The verb you choose selects the mode: _create/generate_ → Mode 1, _add/improve_ → Mode 2, _optimize_ → Mode 3.

## Good prompting patterns

### State the problem, not the file list

The discovery dialog (Mode 1, Phase 1) parses your description to infer purpose, name, and the set of recipes to apply. Give it the _behavior_ you want and let it choose the structure.

{% tabs %}
{% tab title="Good" %}
> Create a module that blocks inbound calls from a configurable list of phone numbers. Admins manage the list on a settings page, and an external system can sync numbers over a REST API.

This tells the skill: it needs a settings model, a web page (`ui`), a REST endpoint (`rest-api`), and call interception (`dialplan` + `agi`). All four recipes flow from one sentence.
{% endtab %}

{% tab title="Weak" %}
> Make me a `BlackListConf.php` and a `BlackListNumbers.php` and a controller.

Naming files by hand bypasses the recipe selection. You lose the auto-discovery, the matching JS/CSS pair, the README and workflow files, and the post-generation checks — and you will likely miss a file the recipe would have created for you.
{% endtab %}
{% endtabs %}

### Use trigger vocabulary that maps to recipes

The skill selects recipes from keywords in your description. Use the words on the left and the recipe on the right is added automatically. This table is the recipe-selection trigger table from `SKILL.md` (Mode 1, Phase 1, step 4):

| Recipe          | Trigger words / signals                                                        |
| --------------- | ------------------------------------------------------------------------------ |
| `base` (always) | —                                                                              |
| `ui`            | settings, page, form, interface, UI / настройки, страница, форма, интерфейс    |
| `rest-api`      | API, REST, endpoint, CRUD / эндпоинт                                           |
| `dialplan`      | calls, routing, IVR, incoming, outgoing / звонки, маршрут, входящие, исходящие |
| `agi`           | AGI, script, lookup, CallerID, "before dial" / скрипт, перед набором           |
| `workers`       | background, worker, queue, events / фоновый, воркер, очередь, события          |
| `firewall`      | firewall, port, fail2ban, security / порт, фаервол, безопасность               |
| `acl`           | ACL, permissions, roles, access / доступ, права, роли                          |
| `system`        | cron, nginx, scheduled, periodic / периодический, запуск                       |

`base` is always included — it generates `module.json`, the `README.md` / `README.ru.md` pair, `.github/workflows/build.yml`, `Setup/PbxExtensionSetup.php`, at least one model under `Models/`, the `Lib/{Feature}Conf.php` config class, and `Messages/ru.php`.

{% hint style="success" %}
**Running example.** "A module that **blocks** inbound **calls** from numbers on a managed **list**, with a **settings page** and a **REST** sync **endpoint**" selects `base + ui + rest-api + dialplan + agi`. That is exactly the recipe set the skill reports for **ModuleBlackList** in the sample run inside `SKILL.md` (Phase 4 report).
{% endhint %}

### Let naming flow from the feature name

Do not hand-name every class. Give the skill a feature concept — "BlackList" — and it derives every identifier from the naming-conventions table in `SKILL.md`:

| Entity             | Pattern                        | ModuleBlackList value         |
| ------------------ | ------------------------------ | ----------------------------- |
| Module ID          | `Module{Feature}`              | `ModuleBlackList`             |
| Namespace          | `Modules\{ModuleID}\...`       | `Modules\ModuleBlackList\Lib` |
| Config class       | `{Feature}Conf`                | `BlackListConf`               |
| Main class         | `{Feature}Main`                | `BlackListMain`               |
| Model              | `{Entity}`                     | `BlackListNumbers`            |
| DB table           | `m_{Entity}`                   | `m_BlackListNumbers`          |
| Controller         | `Module{Feature}Controller`    | `ModuleBlackListController`   |
| Worker             | `Worker{Feature}{Type}`        | `WorkerBlackListAMI`          |
| JS file            | `module-{kebab-case}-{action}` | `module-black-list-index.js`  |
| CSS file           | `module-{kebab-case}-{action}` | `module-black-list-index.css` |
| Translation prefix | `module_{feature}_`            | `module_black_list_`          |

If you supply only "BlackList", the skill proposes `ModuleBlackList` and every dependent name follows. Override a single name only if you have a strong reason — the convention is what keeps the module consistent with the Core and with other modules.

### Answer discovery questions precisely

The skill is instructed to **ask many questions** rather than guess. Short, decisive answers keep the dialog fast. If asked "Will the module have its own settings page in the admin panel?" answer "Yes, one page listing the blocked numbers with add/remove" — not just "yes" — so the skill can shape the form and view correctly.

### Confirm the plan before generation

Mode 1, Phase 1 ends with **plan confirmation**: the skill presents the proposed structure (location, recipes, file list) and waits for your approval before writing anything. Read it. This is the cheap moment to add a missing recipe or correct the location — adjusting now is one sentence, adjusting after generation means editing files.

## Mode 1: the discovery dialog, step by step

Mode 1 ("Create New Module") runs four phases. Phase 1 is the dialog you steer; the rest are mechanical. The dialog order from `SKILL.md` is:

1. **Purpose** — what problem the module solves. Inferred from your description; the skill confirms it back to you.
2. **Name** — the skill suggests `Module{Feature}` (e.g. `ModuleBlackList`) following the conventions above.
3. **Target kind** — you choose what you are building:
   * **production module** — a module with its own repository, README pair and publish workflow.
   * **example / learning module** — a minimal module that demonstrates one pattern; no README pair and no per-module workflow (the repository that hosts it builds it with a shared workflow).
4. **Recipes** — the skill proposes the recipe set using the trigger table, and asks clarifying questions like "Should the module react to call events?" or "Is there data to store in the database?"
5. **Plan confirmation** — the skill prints the kind, location, recipes, and file list and waits for your "go".

```
You: Create a module that blocks inbound calls from a list of numbers,
     with a settings page and a REST API to sync the list.

Skill: Purpose — block inbound calls from a managed deny-list. Correct?
You:   Yes.

Skill: Suggested name: ModuleBlackList. OK?
You:   Yes.

Skill: Production module (own repository, README pair, publish workflow)
       or example module?
You:   Production.

Skill: Recipes I propose: base, ui, rest-api, dialplan, agi.
       - ui: settings page to manage the list
       - rest-api: external sync endpoint
       - dialplan + agi: intercept inbound calls and look up the number
       Should blocked calls just hang up, or route to an announcement?
You:   Hang up with a busy tone.

Skill: Plan —
       Kind:     production
       Location: ModuleBlackList/
       Recipes:  base, ui, rest-api, dialplan, agi
       Files:    Setup/PbxExtensionSetup.php, Lib/BlackListConf.php,
                 Lib/BlackListMain.php, Models/BlackListNumbers.php,
                 App/Controllers/ModuleBlackListController.php, ...
                 agi-bin/check-blacklist.php, module.json
       Generate?
You:   Go.
```

After you approve, Phase 2 generates files in a fixed order (metadata → setup → models → config → main → web → REST → workers → AGI → translations), Phase 3 runs the post-generation checks (`php -l` on every PHP file, Babel transpilation for JS, `module.json` JSON validation, the standalone-catalog grep on `Messages/`, the README-pair and workflow tests for production modules, and the REST/OpenAPI translation validator when `rest-api` is present), and Phase 4 prints a report listing the files created and the check results.

{% hint style="info" %}
The skill reads the actual source of a published reference module before generating each recipe: a module with an admin page for `ui` and `base`, the REST API v3 reference module for `rest-api`, a module with an AMI worker for `workers`. Reference modules, including [ModuleTemplate](https://github.com/mikopbx/ModuleTemplate), are published at [github.com/mikopbx](https://github.com/mikopbx). Pointing the skill at a checkout of the module you want it to imitate never hurts — for example the reference modules under `Extensions/EXAMPLES/` referenced throughout this guide.
{% endhint %}

## Mode 2: augment an existing module

Trigger Mode 2 with "Add ... to ModuleBlackList" or "Improve ModuleBlackList". It runs four phases of its own:

1. **Analysis** — the skill reads the module directory, identifies which recipes are already present, **which hooks are used in the `Conf.php` class**, and counts models, controllers, and workers. It also scans for anti-patterns.
2. **Plan changes** — it decides which new files to create and which existing files to modify, and presents the diff plan.
3. **Implementation** — it applies the change using the same patterns as Mode 1.
4. **Optimization** (if you ask) — runs the anti-pattern checker on the touched code.

The critical guarantee, stated in `SKILL.md` Phase 3, is that **when modifying `Conf.php` the skill adds new hook methods without breaking existing ones**. If your `BlackListConf` already implements `extensionGenContexts()` and you ask to add a worker, the skill appends `getModuleWorkers()` and leaves your existing dialplan hook untouched. Prompt accordingly — name the capability you want added, and let the skill weave it into the existing class:

> Add a Beanstalk worker to ModuleBlackList that revalidates the deny-list every minute.

The skill will add a `getModuleWorkers()` method returning the worker registration (with `'type' => WorkerSafeScriptsCore::CHECK_BY_BEANSTALK`) and create `bin/WorkerBlackListMain.php`, without disturbing the dialplan hooks already in `BlackListConf`.

## Mode 3: optimize against anti-patterns

Trigger Mode 3 with "Optimize ModuleBlackList". The skill:

1. **Reads all module files.**
2. **Checks anti-patterns** against the reference standards (`skills/mikopbx-module/reference/anti-patterns.md` in the agent-skills repository).
3. **Reports findings with severity** and a fix suggestion for each.
4. **Applies fixes** only if you approve.

This is where the modern-baseline rules are enforced: PHP 8.4 idioms (typed properties on non-model classes, constructor promotion, `match`, enums), the Phalcon ORM exception for model column properties (untyped `$id`, nullable string defaults like `public ?string $enabled = '0';`), the import rule (`use Phalcon\Di\Di;`, never `use Phalcon\Di;`), and the file-header rule (`declare(strict_types=1);`, no closing `?>`). Read each finding's severity before approving a bulk fix — apply the high-severity ones first.

The reference catalogue has two halves, and the second is easy to overlook. Alongside 31 numbered code anti-patterns (`MikoPBXVersion.php` in a new module, `shell_exec` instead of the PBX helpers, monolithic classes, phantom model fields, `die()` in a worker, memory leaks in long-running workers, file-based IPC instead of Redis, `@` suppression, direct SQL instead of the ORM, a typed `int $id` on a model primary key, unconditional required-field validation in a save action, a form that skips `parent::initialize()`, a dialplan context with bare-digit extensions included into `[internal]`, and the idiom rules above) it carries **14 security anti-patterns**, `S1`–`S14`, five of them CRITICAL:

| ID          | Severity     | Issue                                                                                                                                      |
| ----------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `S1`        | CRITICAL     | Unauthenticated endpoints exposing sensitive data or actions                                                                               |
| `S2`        | CRITICAL     | SQL injection via string interpolation in `find()` / `findFirst()`                                                                         |
| `S3`        | CRITICAL     | Command injection via unescaped shell arguments                                                                                            |
| `S4`        | CRITICAL     | Path traversal / arbitrary file read                                                                                                       |
| `S5`        | CRITICAL     | Reflected XSS                                                                                                                              |
| `S6`–`S10`  | HIGH         | Dynamic dispatch from user input, insecure deserialization, cron injection, disabled TLS verification, credentials leaked in API responses |
| `S11`–`S14` | MEDIUM / LOW | SSRF via admin-configurable URLs, information disclosure via error output, `postMessage("*")`, predictable temp paths                      |

Running Mode 3 on a module you inherited is therefore a cheap first security pass, not only a style pass. Sort the findings by severity and fix `S1`–`S5` before anything cosmetic.

{% hint style="warning" %}
Mode 3 changes existing, possibly production code. Always review the proposed diff before approving. The skill will not apply fixes without your explicit go-ahead, but it is your job to confirm the change is safe for the module's release.
{% endhint %}

## Mode 4: consultation

The skill also answers architecture and how-to questions without writing a single file — "Как сделать ... в модуле?" / "How to ... in a module?". It answers from the same reference set it generates from (`reference/hook-reference.md` for the Core hook catalogue, `reference/recipes.md`, `reference/anti-patterns.md`, `reference/module-structure.md` and `reference/naming-conventions.md`). Use this before Mode 1 when you are not yet sure which recipes your feature needs; nothing is created until you ask for it explicitly.

## Anti-prompts: what slows the skill down

* **Listing files instead of behavior** — you bypass recipe selection and lose the generated asset pair, release plumbing and post-generation checks.
* **Skipping the kind answer** — the skill must know production vs example before it decides whether to lay down the README pair and the publish workflow.
* **Approving the plan without reading it** — the plan is the last cheap checkpoint before files hit disk.
* **Renaming individual generated classes ad hoc** — breaks the naming-convention chain that ties the module together.

## Where to go next

* [What it generates](what-it-generates.md) — the exact file set each recipe produces.
* [Best practices](../module-developement/best-practices.md) — the standards Mode 3 enforces, written out as guidance you can apply by hand.
* [AI-assisted development overview](ai-assisted-development.md) — when to reach for the skill at all.
