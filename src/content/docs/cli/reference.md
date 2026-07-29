---
title: CLI command reference
description: Every hydrate CLI verb and its flags.
---

The complete command surface. Commands fall into five groups: orientation,
project and branch context, authoring, reading, and commit. Authoring commands
*stage* changes locally; nothing reaches the server until `hydrate commit`.

Every command is available under both the `hydrate` and `hyd` names. Run
`hydrate <command> --help` for the terse inline reference, or `hydrate guide`
for an orientation.

## Global flags

| Flag | Effect |
| --- | --- |
| `--json` | Force machine-readable JSON output. |
| `--human` | Force human-readable output. |
| `--project <name\|id>` | Select the project to act on. |

By default the CLI prints human-readable output on a terminal and JSON when its
output is piped. The two carry the same information. `--json` and `--human`
override the auto-detection and cannot be combined.

`--project` applies to `projects`, `fork`, `branches`, and `show`, and overrides
both the `HYD_PROJECT` environment variable and this directory's binding.

**Every other verb ignores `--project` without saying so.** `walk`, `pull`,
`status`, `diff`, `validate`, `commit`, and the authoring verbs act on the
branch this directory is bound to. Passing `--project` to them — even naming a
project that does not exist — is accepted silently and changes nothing:

```sh
hydrate status --project no-such-project   # succeeds, reports the bound branch
```

`hydrate status` is the reliable way to confirm what you are about to write to.
Run `hydrate projects` for names and ids.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Generic failure. |
| `4` | Conflict (the branch moved under you; `pull` and retry). |
| `5` | `hydrate validate` returned a `valid: false` verdict. |
| `6` | Network failure. |

Richer machine-readable detail rides in `--json` output, while the exit codes
stay stable. Code `5` is a pass/fail outcome rather than a failure — `validate`
reached the server and the answer was "not coherent" — which is what makes it
usable as a shell gate.

## Orientation

### `hydrate guide`

Print an orientation to authoring graphs: the loop, the concepts, a worked
example, and a pointer to these docs. Reads nothing and writes nothing; it works
before you have a key.

### `hydrate init`

Write a small pointer block into this directory's `AGENTS.md` so a coding agent
discovers the workflow — the block points at `hydrate guide`. Idempotent, and it
never clobbers your other content. A pure local file write: no network, no
branch, nothing staged.

## Project and branch context

### `hydrate projects`

List the projects on your account, with the id, language, intent, and when each
was last opened. Archived projects are flagged.

### `hydrate fork <name>`

Fork a new working branch from `main` and bind the current directory to it.
`<name>` is a slug: letters, digits, `-`, `_`. Subsequent commands in this
directory act on the bound branch.

### `hydrate branches`

List your working branches.

### `hydrate pull`

Refresh the local view of the bound branch's live graph, so you can reference
already-committed nodes by their dotted path. Run this before editing existing
nodes (`node set`, `node mv`, `node rm`, `clear`).

## Authoring

All authoring commands stage operations into a changeset. Review with `diff`,
apply with `commit`.

### `hydrate node add`

Stage a new node.

| Flag | Value | Notes |
| --- | --- | --- |
| `--kind` | `behavior` \| `boundary` \| `state` \| `io` \| `interface` | Required. |
| `--name` | name | Required. Unique within its parent scope. |
| `--description` | text | Free-text description of the node. |
| `--constraint` | text | A free-text constraint string. Repeatable. |
| `--verification` | text | A free-text verification string. Repeatable. |
| `--parent` | path | Parent node, by dotted path (e.g. `Api`). Omit for top level. |
| `--in` | `name:type` | Input port. Type required. Repeatable. |
| `--out` | `name:type` | Output port. Type required. Repeatable. |
| `--config` | `name:type` | Config port (not wired by edges). Repeatable. |
| `--external` | — | Mark the node external (a system outside the graph). |
| `--external-kind` | label | The external system's kind (requires `--external`). |
| `--protocol` | label | External-only: the system's protocol (e.g. `gRPC`). |
| `--user-kind` | label | On a boundary, classifies the boundary; on a state node, carries the state kind (e.g. `postgres-db`). Behavior nodes do not carry it. |
| `--path-prefix` | path | Boundary-only: the path prefix the boundary owns. |
| `--language` | label | Boundary-only: the codegen language (e.g. `go`, `python`). |
| `--doc-url` | url | A documentation URL (http/https). |
| `--test-node` | — | Mark the node a test node. |

```sh
hydrate node add --kind behavior --name Rater --in raw:HotDog --out score:Score \
    --description 'Score a hot dog photo from 0–100.'
```

### `hydrate node set <path>`

Stage an edit to an existing node. Only the fields you pass change; everything
else is left as is. Addressed by dotted path (e.g. `Api.Rater`).

**Identity and description:**

| Flag | Value | Notes |
| --- | --- | --- |
| `--name` | name | Rename the node (its leaf name within its parent scope). |
| `--description` | text | New description. |
| `--clear-description` | — | Set the description empty. |
| `--constraint` | text | Replace constraints with these (repeatable). |
| `--clear-constraints` | — | Remove all constraints. |
| `--verification` | text | Replace verifications with these (repeatable). |
| `--clear-verifications` | — | Remove all verifications. |

**Ports.** Add, remove, or retype ports on each channel. Retyping keeps the
port's identity:

| Flag | Value | Notes |
| --- | --- | --- |
| `--add-in` / `--add-out` / `--add-config` | `name:type` | Add a port. Repeatable. |
| `--rm-in` / `--rm-out` / `--rm-config` | `name` | Remove a port by name. Repeatable. |
| `--retype-in` / `--retype-out` / `--retype-config` | `name:newtype` | Change a port's type, keeping its identity. Repeatable. |

**Boundary fields:**

| Flag | Value | Notes |
| --- | --- | --- |
| `--user-kind` | label | Set the boundary classifier. |
| `--clear-user-kind` | — | Clear it. |
| `--path-prefix` | path | Set the boundary path prefix. |
| `--clear-path-prefix` | — | Clear it. |
| `--language` | label | Set the boundary codegen language. |
| `--clear-language` | — | Clear it. |

**External fields:**

| Flag | Value | Notes |
| --- | --- | --- |
| `--external` | — | Mark external. |
| `--no-external` | — | Unmark external. |
| `--external-kind` | label | Set the external kind. |
| `--clear-external-kind` | — | Clear it. |
| `--protocol` | label | Set the protocol. |
| `--clear-protocol` | — | Clear it. |

**Other:**

| Flag | Value | Notes |
| --- | --- | --- |
| `--doc-url` | url | Set the documentation URL. |
| `--clear-doc-url` | — | Clear it. |
| `--test-node` | — | Mark the node a test node. |
| `--no-test-node` | — | Unmark it. |

```sh
hydrate node set Api.Rater --description 'Score 0–100, reject non-food images.' \
    --add-out reason:Text --retype-in raw:FoodPhoto
```

### `hydrate node mv <path>`

Reparent a node under a new boundary, or move it to the top level.

| Flag | Value | Notes |
| --- | --- | --- |
| `--parent` | path | New parent boundary, by dotted path. |
| `--top` | — | Move to the top level (no parent). Mutually exclusive with `--parent`. |

### `hydrate node rm <path>…`

Stage the removal of one or more nodes, by dotted path. Removing a node cascades
its subtree. The path argument is repeatable.

```sh
hydrate node rm Api.Rater Api.Encoder
```

### `hydrate edge add`

Stage an edge between two typed ports, from an output port to an input port.
The two types should match; a mismatch is accepted and reported as a
`type_mismatch` finding by `hydrate validate` rather than rejected.

| Flag | Value | Notes |
| --- | --- | --- |
| `--from` | `node.port` | Source (output) port, by dotted path. |
| `--to` | `node.port` | Target (input) port, by dotted path. |

```sh
hydrate edge add --from Maker.dog --to Rater.raw
```

### `hydrate edge rm`

Stage the removal of the edge between two ports.

| Flag | Value | Notes |
| --- | --- | --- |
| `--from` | `node.port` | Source port of the edge to remove. |
| `--to` | `node.port` | Target port of the edge to remove. |

### `hydrate boundary flatten <path>`

Flatten a boundary: promote its children to its parent and remove the boundary
itself. Addressed by dotted path (e.g. `Api`).

### `hydrate clear`

Stage the removal of *every* top-level node, to wipe the branch and rebuild it
in place. The cascade removes each node's subtree. Requires a prior
`hydrate pull`.

## Reading

`status` and `diff` read your *staged* work. `show` and `walk` read the
*committed* graph on the branch. None of them mutate anything: they create no
branch and stage nothing.

`show --depth` and `walk` are **scoped reads**: the server sends only the slice,
so a large graph never crosses the wire or enters an agent's context. Turning
your dotted path into the id such a read is addressed by takes a pulled working
copy, so the CLI falls back to fetching the whole branch and narrowing it here
when it cannot. It falls back if there is no working copy, if the index has not
been pulled, if the path is not in the index, or if you asked for a branch other
than the bound one.

**A fallback is not equivalent.** It prints a note on **stderr** saying what
happened, and — for `show` — it drops `--depth` entirely, returning the whole
subtree. Asking for one level of a 96-node graph without an index gets you all
96 nodes. On stdout the tell is structural: a scoped `show` carries `scoped`,
`root`, `depth`, and `truncated` keys that a fallback does not.

### `hydrate status`

Show the bound branch and a summary of the staged operations.

### `hydrate diff`

Show the staged operations in detail. Nothing here has hit the server yet.

### `hydrate show [path]`

Print a read-only view of a branch's graph — nodes, ports, and edges. With a
dotted `path`, the view is limited to that node and its subtree; omit it for the
whole graph.

| Flag | Value | Notes |
| --- | --- | --- |
| `--branch` | name | Which branch to show. Defaults to this directory's bound branch, else the project's main branch. |
| `--depth` | `1`–`32` | Read only this many levels below `path`. `1` is direct children. Requires `path`; a value outside `1..=32` is rejected. |

```sh
hydrate show                    # the whole graph
hydrate show Api                # Api and everything under it
hydrate show Api --depth 1      # Api and its direct children only
```

Only `--depth` makes this a scoped read. A bare `show <path>` always fetches the
whole branch and narrows it here.

A depth-limited read is a partial answer, and says so when it is:

```
(cut at depth 1 — there are more nodes below; raise --depth to see them)
```

JSON sets `"truncated": true` in the same case.

**A subtree view shows only the edges interior to it.** An edge from a node in
the slice to a node outside it is not in the output, so `show Api.Encoder` can
print a node with no edges at all while several connect to it. The CLI counts
what it left out rather than hiding it:

```
1 edge cross out of this subtree — run `hydrate show` for the full graph
```

JSON carries the same count as `cross_boundary_edges`; a `0` there means the
slice is genuinely self-contained. `show` answers *what is inside this*; use
`walk` to ask *what does this touch*. Scoped and fallback reads agree on which
nodes and edges come back, so this part of the answer does not change with the
state of your index.

### `hydrate walk <path>`

Read one node's scoped context: the node itself plus its 1-hop neighborhood —
every node on the other end of an edge touching it, in either direction. The
node is rendered in full.

| Flag | Effect |
| --- | --- |
| `--boundary` | Read the boundary's scope instead: its children and the edges interior to it. |

`walk` reads the branch this directory is bound to and requires a binding. There
is no `--branch`; use `show --branch` to read another one.

```sh
hydrate walk Api.Encoder             # the node and everything it touches
hydrate walk Api --boundary          # what lives inside Api
```

Running plain `walk` on a boundary is fine — it returns the neighborhood and
points out that `--boundary` will show the interior.

`--boundary` applies only to a boundary node. Pointed at anything else, the CLI
refuses before issuing a read that cannot succeed, and exits `1`:

```
hydrate: 'Api.Encoder' is not a boundary — this working copy's index has it as
a behavior. Run `hydrate walk Api.Encoder` for its neighborhood, or
`hydrate pull` if the index is behind.
```

The kind comes from your local index, which is why the message says so — a
node's kind can change on the branch. Without an index the CLI cannot mention
one, so the wording is shorter and omits the `pull` advice. When the index
records no kind for that node, the check is skipped and the read is sent, with a
note explaining why.

## Validate and commit

### `hydrate validate`

Dry-run the staged changeset against the bound branch and report the server's
coherence findings, without committing and without clearing the stage. Run it as
often as you like.

**The verdict is on the resulting branch as a whole, not on your changeset.**
Findings that were already on the branch are reported alongside anything your
staged edits introduce, and an empty stage is a valid input — it asks "is this
branch coherent right now?". That makes `validate` a branch-health probe, but it
also means the verdict is not a judgement on your work alone.

```sh
hydrate validate && hydrate commit
```

This gate is only usable on a branch that is already clean. On a branch carrying
pre-existing findings — the normal state of a large imported graph — it will
refuse to commit no matter how correct your change is. Run `validate` **before**
staging to get a baseline, then compare.

Human output resolves each finding to a dotted path and ends with the verdict:

```
2 coherence findings:
  [error] unsatisfied_input  Api.Encoder.url: input port Api.Encoder.url has no incoming edge
  [error] type_mismatch  Api.Shorten.url -> Api.Encoder.url: edge connects a port of type 'LongUrl' to one of type 'ShortCode'

Invalid: 2 coherence errors on branch 'demo'; not safe to commit.
```

A clean branch reports `Valid: no coherence errors on branch 'demo'.`, or
`No coherence findings.` when there are none at all.

`validate` reports exactly three codes — `unsatisfied_input`, `dangling_wire`,
and `type_mismatch` — and the server currently emits **all three at `error`
severity**, so any of them fails the gate above. Note in particular that a type
mismatch does *not* block `hydrate commit` on its own: the commit endpoint
accepts mismatched edges, because port types are advisory hints rather than a
hard contract. `validate` is the stricter of the two. If you need to commit a
graph whose only findings are type mismatches, run `hydrate commit` directly
rather than through the `&&` gate.

Exit code `5` follows the server's `valid` verdict, not the CLI's own count of
findings; when the two disagree the CLI trusts the server and says so loudly.

JSON output carries the server's `findings` verbatim — each with `code`,
`severity`, `message`, and a raw `locator` id — plus a `valid` boolean and a
`located` array that adds the resolved dotted `path` for each finding. Prefer
`located`: `path_complete` is `false` when only part of a path could be
resolved, which lets a consumer tell a complete answer from a partial one
without inspecting ids.

Path resolution is the CLI's, from your pulled index, so a stale working copy
can yield a stale path. The CLI compares the branch version it pulled at against
the branch's current version and warns when they differ — even when every id
resolved, because a confidently wrong path is worse than a raw id. It also warns
when the index is unreadable or when some ids cannot be placed at all; in those
cases findings are shown by id.

### `hydrate commit`

Commit the staged changeset to the bound branch as one typed delta batch, under
optimistic concurrency control. The server validates the whole batch. If any
operation is invalid (an unresolved path or a name collision, for example),
the commit is rejected and nothing is applied. If the branch has moved under you
since your last `pull`, the commit fails with a conflict (exit code `4`); `pull`
and retry.
