# docs.mikopbx.com — English checkout

## Layout

This working tree is the **English** documentation. The repository is
laid out on disk as two independent checkouts side-by-side:

```
docs.mikopbx.com/
├── english/   ← you are here, branch `english`
└── russian/   ← sibling checkout, branch `russian`
```

Each folder is a complete clone with its own `.git/` directory, both
pointing at `git@github.com:mikopbx/docs.mikopbx.com.git`. **No branch
juggling.** You never need to `git checkout` to switch language — just
`cd` to the sibling folder.

## Working rules

- Stay in this folder. Edit only the English files here; never touch
  `../russian/`. Each side commits to its own branch independently.
- **Never `git merge russian`** (or cherry-pick across folders). The
  two branches are unrelated trees with diverging structure, wording
  and screenshots — merging mixes languages and breaks both versions.
- Commits and pushes from this folder go to `origin/english` only.
- For bilingual updates: apply the English edit here, ask the user (or
  a sibling agent) to apply the Russian equivalent in `../russian/`.
  Do not synchronise the two via git plumbing.
- GitBook publishing is wired per branch to its own space; nothing
  here triggers the Russian space.

## GitBook conventions

- Pages are GitBook-flavoured Markdown: `{% hint %}` blocks,
  `{% content-ref %}` references, `<figure>` for images. Assets live
  in `.gitbook/assets/` (binary; do not rewrite by mistake).
- The navigation root is `SUMMARY.md`.
- Default branch on GitHub is `english`.

## Path notes

- This checkout's path: `/Volumes/DevDisk/Developement/docs.mikopbx.com/english`
- Russian sibling: `/Volumes/DevDisk/Developement/docs.mikopbx.com/russian`
