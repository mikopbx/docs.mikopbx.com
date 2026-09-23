---
description: >-
  Developing MikoPBX modules with AI coding agents: the mikopbx/agent-skills
  repository, the /mikopbx-module skill and how to install it into any agent.
---

# Overview

MikoPBX publishes a set of open-standard **Agent Skills** for module development in the [mikopbx/agent-skills](https://github.com/mikopbx/agent-skills) repository. The main one, **`/mikopbx-module`**, turns a plain-language description into a complete, convention-correct module. Instead of hand-copying boilerplate from a reference module, you describe what you want and the skill runs a structured discovery dialog, selects the right integration recipes, generates the full file tree from canonical templates and reference modules, and self-checks the result.

This chapter explains where the skills live, how to install them into your agent, the four modes `/mikopbx-module` operates in, how to activate it, and what guarantees it makes about the code it emits.

{% hint style="info" %}
The skills are development-time tools. They run in your AI coding environment and write source files into your checkout. They do **not** run inside MikoPBX itself, and nothing they generate depends on a skill being present at runtime — the output is ordinary module code you own.
{% endhint %}

## Where the skills live

Every skill is a folder with a `SKILL.md` file that follows the [Agent Skills](https://agentskills.io) open standard. The same folder works without modification in Claude Code, OpenAI Codex, Gemini CLI, Cursor, GitHub Copilot, OpenCode and any other agent that implements the standard. The skills were extracted from the tooling the MikoPBX core team uses daily and carry the same hook reference, recipes, anti-pattern catalogue and validation scripts.

The repository layout:

```
agent-skills/
├── README.md
├── LICENSE                      # GPL-3.0-or-later, same as MikoPBX
└── skills/
    ├── mikopbx-module/          # the module generator (this chapter)
    │   ├── SKILL.md
    │   ├── reference/           # hook-reference, recipes, anti-patterns, naming, structure
    │   ├── templates/           # per-recipe generation templates
    │   └── scripts/             # validate-rest-api-translations.php
    ├── api-client/
    ├── endpoint-validator/
    ├── openapi-analyzer/
    ├── api-test-generator/
    ├── sqlite-inspector/
    ├── log-analyzer/
    ├── asterisk-validator/
    ├── asterisk-tester/
    ├── babel-compiler/
    └── translations/
```

Whenever a page in this guide cites `reference/hook-reference.md` or `reference/anti-patterns.md`, the path is relative to `skills/mikopbx-module/` in that repository.

## Installing the skills

The quickest way is the [skills CLI](https://github.com/vercel-labs/skills), which detects the agents installed on your machine and copies the skill folders into the right place for each:

{% tabs %}
{% tab title="Interactive" %}
```bash
# pick the skills and the agents you want
npx skills add mikopbx/agent-skills
```
{% endtab %}

{% tab title="Everything" %}
```bash
# all skills, for every agent detected on this machine, no prompts
npx skills add mikopbx/agent-skills --all
```
{% endtab %}

{% tab title="Specific agents" %}
```bash
# project-local install for Claude Code and Codex only
npx skills add mikopbx/agent-skills -a claude-code -a codex

# only the skills /mikopbx-module needs, for the same two agents
npx skills add mikopbx/agent-skills -a claude-code -a codex \
  -s mikopbx-module -s translations -s babel-compiler

# user-wide instead of the current project
npx skills add mikopbx/agent-skills -g
```
{% endtab %}

{% tab title="Manual" %}
Clone the repository and copy the folders you need from `skills/` into your agent's skills directory — `.claude/skills/`, `.codex/skills/`, `.agents/skills/`, `.cursor/skills/` and so on. Each skill is self-contained.
{% endtab %}
{% endtabs %}

The CLI keeps one copy of each skill in `.agents/skills/` (the directory Codex reads) and symlinks it into every other selected agent's directory (`.claude/skills/` for Claude Code, and so on); pass `--copy` for independent copies. `npx skills update` pulls the latest revision later, and `npx skills list` shows what is installed.

Run the install from the directory where you develop modules. Most skills assume a checkout of [mikopbx/Core](https://github.com/mikopbx/Core) next to your module and a running MikoPBX (the development Docker container or a real PBX reachable over SSH); each `SKILL.md` states what it needs.

{% hint style="info" %}
**The Core repository is agent-ready too.** The root of `mikopbx/Core` and each major source area (`src/Modules`, `src/PBXCoreREST`, `src/Common/Models`, `src/AdminCabinet`, the workers, the tests) carry an `AGENTS.md` with the conventions and pitfalls an agent cannot infer from the code alone (imports, bootstrap, where Babel output goes, how to run the tests). `CLAUDE.md` files next to them simply include the `AGENTS.md`, so Claude Code and every agent that reads `AGENTS.md` see the same guidance. Keep a Core checkout available to the agent while you work: the skill reads Core source for hook signatures and the reference modules for patterns.
{% endhint %}

## The skill set

`/mikopbx-module` is the entry point; the other skills are the tools it and you reach for during generation, testing and release.

| Skill                | Use it when                                                                                                |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| `mikopbx-module`     | Creating or extending a module: structure, hooks, forms, REST endpoints, workers, translations, packaging. |
| `api-client`         | Calling the MikoPBX REST API v3 with automatic authentication, from the host or inside the dev container.  |
| `endpoint-validator` | Checking a REST endpoint implementation against its OpenAPI schema and data structure.                     |
| `openapi-analyzer`   | Extracting and inspecting the OpenAPI 3.1 specification served by a running MikoPBX.                       |
| `api-test-generator` | Generating pytest suites for REST endpoints with schema validation.                                        |
| `sqlite-inspector`   | Verifying data in the MikoPBX SQLite databases after API or module operations.                             |
| `log-analyzer`       | Reading and correlating MikoPBX logs (system, PHP, Asterisk, nginx, fail2ban).                             |
| `asterisk-validator` | Validating generated Asterisk configuration files and Asterisk logs.                                       |
| `asterisk-tester`    | Running dialplan and call-flow scenarios against a MikoPBX instance.                                       |
| `babel-compiler`     | Transpiling ES6+ admin-interface JavaScript to the ES5 bundle MikoPBX ships.                               |
| `translations`       | Managing the multilingual `Messages/` catalogs with Russian as the source language.                        |

`/mikopbx-module` calls `babel-compiler` and `translations` itself during generation, so install at least those three together.

## Target platform

The skill targets the current MikoPBX baseline and emits no legacy shims:

```
PHP:             8.4
min_pbx_version: 2025.1.1
Framework:       Phalcon 5.9.3
Legacy:          none (no MikoPBXVersion.php, no Phalcon 4 / PHP 7 patterns)
```

Generated code uses PHP 8.4 features — typed properties, constructor promotion, `match` expressions, named arguments and enums — except for Phalcon ORM model column properties, which follow the SQLite/Phalcon convention (untyped `$id`, nullable string columns). The skill knows the difference and applies each rule in the right place.

## The four modes

The skill recognizes intent from your message and works in one of four modes.

### 1. Create New Module

Generates a brand-new module from a description. This is a four-phase pipeline:

1. **Discovery dialog** — the skill parses your description, proposes a module name (`Module{Feature}`), asks whether you are building a _production module_ (its own repository, README pair and publish workflow) or an _example / learning module_ (a minimal module that demonstrates one pattern), and proposes a set of recipes.
2. **Generation** — files are emitted in dependency order (`module.json`, the `README.md` / `README.ru.md` pair and `.github/workflows/build.yml` for production modules, `Setup/PbxExtensionSetup.php`, models, the `{Feature}Conf` config class, then UI / REST / worker / AGI files as the chosen recipes require, and `Messages/` last).
3. **Post-generation checks** — `php -l` on every generated PHP file, Babel transpilation of JavaScript, a `module.json` validity check, a grep proving each `Messages/*.php` catalog is a standalone literal array, existence tests for the README pair and the build workflow (production modules only), and the REST/OpenAPI translation validator when the `rest-api` recipe is present.
4. **Report** — a summary of files created, recipes applied, check results, and next steps.

### 2. Augment Existing Module

Adds functionality to a module that already exists (for example, "add a REST API" or "add a background worker"). The skill first reads the existing tree, identifies which recipes are already present and which hooks the config class already uses, plans the new and modified files, and then applies the changes — extending the config class with new hook methods without breaking the ones already there.

### 3. Optimize Module

Scans an existing module for anti-patterns, reports findings with severity and a suggested fix for each, and — with your approval — applies the fixes to bring the code in line with the reference standards.

### 4. Consultation

Answers architecture and how-to questions about the MikoPBX module system without writing files — for example, "which hook fires when the dialplan is rebuilt?" or "where should a periodic task live?". Use this mode to plan before you build.

## Recipes

A _recipe_ is a self-contained slice of module functionality. The `base` recipe is always applied; the rest are selected automatically from signals in your description (and confirmed with you before generation).

| Recipe     | What it adds                                              |
| ---------- | --------------------------------------------------------- |
| `base`     | The minimal installable module (always applied)           |
| `ui`       | Admin settings page: controller, form, Volt view, JS, CSS |
| `rest-api` | Versioned REST API controllers and actions                |
| `dialplan` | Call-routing / dialplan integration hooks                 |
| `agi`      | AGI scripts (CallerID lookup, pre-dial logic)             |
| `workers`  | Background workers (queues, AMI/event listeners)          |
| `firewall` | Firewall ports and fail2ban rules                         |
| `acl`      | Access-control roles and permissions                      |
| `system`   | Scheduled tasks (cron) and system service config          |

See [Module recipes](../module-developement/recipes.md) for the full specification of each recipe and the files it produces.

## Beyond code: the production repository contract

For a module headed to a real repository, the skill also generates the release plumbing: an administrator-facing `README.md` / `README.ru.md` pair, a `.github/workflows/build.yml` built on the shared `mikopbx/.github-workflows/.github/workflows/extension-publish.yml@master` reusable workflow (`develop`, `master`, `workflow_dispatch`), and the `release_settings.publish_release`, `changelog_enabled` and `create_github_release` flags in `module.json`. See [What it generates](what-it-generates.md) for the full contract and what you still have to verify by hand.

## Activation phrases

The skill activates on natural-language requests in Russian or English, or on an explicit `/mikopbx-module` invocation (the slash form is how Claude Code exposes skills; other agents pick the skill up from the description in `SKILL.md`, so a plain sentence is enough):

{% tabs %}
{% tab title="English" %}
* `Create a module for ...` / `Generate a module ...`
* `Add to the module ModuleBlackList ...`
* `Optimize the module ModuleBlackList`
* `How do I make a module ...` / `Improve the module ...`
* `/mikopbx-module ...`
{% endtab %}

{% tab title="Русский" %}
* `Создай модуль для ...` / `Сгенерируй модуль ...`
* `Добавь в модуль ModuleBlackList ...`
* `Оптимизируй модуль ModuleBlackList`
* `Как сделать модуль ...` / `Доработай модуль ...`
* `/mikopbx-module ...`
{% endtab %}
{% endtabs %}

## What to expect from a session

The running example throughout this guide is a fictional spam-blocking module, **ModuleBlackList** (config class `BlackListConf`, main class `BlackListMain`, model `BlackListNumbers` backed by table `m_BlackListNumbers`, front-end script `module-black-list-index.js`). A "create" session for it might begin:

```
You:   Create a module that blocks incoming calls from a blacklist of numbers,
       with a settings page and a REST API to manage the list.

Skill: Proposed name: ModuleBlackList. Is this a production module
       (own repository, README pair, publish workflow) or an example module?
       Recipes I plan to apply: base, ui, rest-api, dialplan.
       A few questions before I generate:
         - Should blocked numbers be stored in the database? (suggests model
           BlackListNumbers / table m_BlackListNumbers)
         - Should the block happen on the incoming dialplan, or via an AGI lookup?
         - Does the REST API need full CRUD, or read-only?
```

{% hint style="warning" %}
**The skill asks a lot of questions and waits for plan approval before it writes anything.** This is by design — clarifying intent up front is cheaper than regenerating a wrong tree. Expect a discovery dialog, then a proposed plan (name, kind, recipes, file list). The skill generates only after you approve that plan.
{% endhint %}

When generation completes, the skill reports the created files and the results of its self-checks (PHP lint, Babel, `module.json` validation and the rest) so you can review before installing.

## How the code is grounded

The skill does not invent module structure. Before emitting code it reads the actual source of the reference modules MikoPBX publishes at [github.com/mikopbx](https://github.com/mikopbx) — starting with [ModuleTemplate](https://github.com/mikopbx/ModuleTemplate) — and mirrors their patterns:

* Web UI: a reference module with `App/Controllers`, `App/Forms`, `App/Views` and `public/assets/js/src`
* REST API v3: a reference module with `Lib/RestAPI/{Resource}/` controllers, processors, data structures and actions
* Workers: a reference module with an AMI event worker registered through `getModuleWorkers()`

The templates bundled with the skill only summarize the structure; the published modules are the canonical reference. They are the same modules documented elsewhere in this guide (see [Template module structure](../module-developement/template-module-structure.md)), so the generated code looks like — and links cleanly against — the rest of the MikoPBX module ecosystem.

## Next steps

* [Using the skill](using-the-skill.md) — the discovery-to-approval workflow, mode by mode, with the prompts that drive each one.
* [What it generates](what-it-generates.md) — the file tree, the per-recipe outputs, and the self-checks the skill runs before it hands the module back to you.
