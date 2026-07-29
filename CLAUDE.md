# CLAUDE.md

## Project

`docs.hydrate.sh` — the public documentation site for hydrate.sh. Astro
Starlight; the API reference is generated from the vendored `openapi.json`.

## This repository is PUBLIC

Everything here is world-readable: the rendered pages, the source, the commit
messages, the pull request titles and bodies, and the branch names.

- **No internal references.** Do not introduce internal planning paths, plan or
  milestone codenames, phase or ticket ids, internal review labels or reviewer
  names, or issue/PR numbers from any other repository. This applies to code,
  comments, commit messages, PR titles and bodies, branch names, and tests —
  not only to the rendered page.
- **No internal process vocabulary.** How work is planned, reviewed, sequenced,
  or staffed is not public. Describe *what changed and why it is true*, never
  which phase it belongs to or which review caught it.
- **No cross-repo links.** Other hydrate-sh repositories are private. Naming one,
  or citing an issue number in one, discloses both its existence and its
  contents.
- **Write for someone who has never seen the internals.** Public-facing text is
  user documentation. If a sentence only makes sense to someone who has read an
  internal document, rewrite it.

A useful check before pushing: read the commit message and PR body as a
stranger. If either tells them something about how this project is run rather
than about the product, cut it.

## Verify claims against the product, not against intent

Documentation that describes intended behaviour rather than shipped behaviour is
worse than no documentation: readers build on it. Before documenting a verb, a
flag, or a failure mode, run it against a real project and confirm the output
verbatim. Specifications and plans are not evidence.

## Build

```
bun run build     # use bun; never npm/npx (bun.lock is the lockfile)
bun run dev
```

The build must be clean before pushing. `openapi.json` is vendored and synced by
CI — do not hand-edit it.
