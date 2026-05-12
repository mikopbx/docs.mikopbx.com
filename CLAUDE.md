# docs.mikopbx.com repository

## Branch layout — critical

This repository keeps **two independent branches**, one per language:

- `russian` — Russian documentation (RU GitBook space).
- `english` — English documentation (EN GitBook space).

**These branches are unrelated and must never be merged.** Each branch
is self-contained: `russian` has the Russian files, `english` has the
English files. It is **not** a single tree with translations — content
diverges in structure, screenshots and wording.

### What this means in practice

- **Never `git merge russian` ↔ `english`** (or the other way around),
  not even partially. That mixes languages and breaks both versions.
- **Do not synchronise the branches via rebase/cherry-pick across
  languages.** When the same change has to land in both versions, apply
  it **separately** in each branch with the appropriate translation.
- **PRs are opened per branch**: `russian` → `origin/russian`,
  `english` → `origin/english`. No cross-language merges.
- **GitBook publishing** is configured per branch to its own space.

### Typical workflow for bilingual updates

1. `git checkout russian` — apply the RU edits, commit.
2. `git checkout english` — apply the EN equivalent **manually** (or
   copy from pre-prepared drafts), commit.
3. Push both branches as independent commits.

Drafts are often prepared upfront in a separate repository
(e.g. `mikopbx/Core/sessions/tasks/.../docs-drafts/{ru,en}/`) and copied
into the matching branch following an `APPLY.md` runbook.

## Other notes

- Pages are GitBook-flavoured Markdown: `{% hint %}` blocks,
  `{% content-ref %}` references, `<figure>` for images, assets live in
  `.gitbook/assets/`.
- The navigation root is `SUMMARY.md` (edited in both branches in
  parallel, but with different heading text).
- Default branch on GitHub is `english`.
