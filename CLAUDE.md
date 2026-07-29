# CLAUDE.md

## Project

`docs.hydrate.sh` — the public documentation site for hydrate.sh. Astro
Starlight; the API reference is generated from the vendored `openapi.json`.

## Everything here is public

Not just the rendered pages — the source, the commit messages, the pull request
titles and bodies, and the branch names are all world-readable.

So keep all of it **about the product**: what a reader can do, what the software
does, and why a change is correct. A commit message explaining a behaviour is
right; one explaining where the work sat in someone's queue is not.

Before pushing, read your commit message and PR body as a stranger who has only
ever used hydrate.sh. Anything that tells them something other than how the
product behaves does not belong in this repository.

Write the pages the same way: for someone who has never seen how any of it is
built.

## Verify claims against the running product

Documentation that describes intended behaviour rather than shipped behaviour is
worse than none, because readers build on it. Before documenting a verb, a flag,
or a failure mode, run it against a real project and confirm the output
verbatim — including the error paths. A specification is not evidence.

## Build

```
bun run build     # use bun; never npm/npx (bun.lock is the lockfile)
bun run dev
```

The build must be clean before pushing. `openapi.json` is vendored and synced by
CI — do not hand-edit it.
