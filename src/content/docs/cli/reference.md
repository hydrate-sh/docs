---
title: CLI command reference
description: Every hydrate CLI verb and its flags.
---

The complete command surface. Commands fall into five groups: orientation,
branch context, authoring, reading, and commit. Authoring commands *stage*
changes locally; nothing reaches the server until `hydrate commit`.

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

`--project` applies to the project-scoped verbs (`projects`, `fork`,
`branches`, `show`) and overrides both the `HYD_PROJECT` environment variable
and this directory's binding. The binding-only verbs (`pull`, `status`,
`validate`, `commit`, and the authoring verbs) act on the bound branch and
ignore it. Run `hydrate projects` for names and ids.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Generic failure. |
| `4` | Conflict (the branch moved under you; `pull` and retry). |
| `5` | `hydrate validate` found error-severity findings. |
| `6` | Network failure. |

Richer machine-readable detail rides in `--json` output, while the exit codes
stay stable.

Code `5` is a pass/fail outcome rather than a failure: `validate` reached the
server and got an answer, and the answer was "there are errors." That is what
makes `hydrate validate && hydrate commit` a usable gate in a shell.

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

## Branch context

### `hydrate projects`

List the projects on your account. Archived projects are flagged.



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

Stage a new node, either a behavior or a boundary.

| Flag | Value | Notes |
| --- | --- | --- |
| `--kind` | `behavior` \| `boundary` | Required. |
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
| `--user-kind` | label | Boundary-only: the user-facing kind label. |
| `--path-prefix` | path | Boundary-only: the path prefix the boundary owns. |
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

Stage an edge between two typed ports. The edge runs from an output port to an
input port of the **same type**.

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
| `--depth` | `N` | Read only this many levels below `path`. `1` is direct children. Requires `path`. |

```sh
hydrate show                    # the whole graph
hydrate show Api                # Api and everything under it
hydrate show Api --depth 1      # Api and its direct children only
```

`--depth` is not just a filter. Without it the CLI fetches the whole branch and
narrows the result locally; with it, the server sends only that slice, so a
large graph never crosses the wire and never enters an agent's context.

That scoped read needs a working copy that has been pulled — the local index is
what turns your dotted path into the id the server's read is addressed by. If
there is no index, or the path is not in it, the CLI falls back to fetching the
whole graph and filtering locally, **and says so**. The output is the same
either way; only the transfer differs.

**A subtree view shows only the edges interior to it.** An edge from a node in
the slice to a node outside it is not in the output — so `show Api.Encoder`
prints that node with no edges at all, even when several connect to it. `show`
answers *what is inside this*; use `walk` to ask *what does this touch*. The
scoped and fallback paths agree on this, so the answer does not change with the
state of your index.

### `hydrate walk <path>`

Read one node's scoped context: the node itself plus its 1-hop neighborhood —
every node on the other end of an edge touching it, in either direction. The
node is rendered in full.

| Flag | Effect |
| --- | --- |
| `--boundary` | Read the boundary's scope instead: its children and the edges interior to it. |

```sh
hydrate walk Api.Encoder             # the node and everything it touches
hydrate walk Api --boundary          # what lives inside Api
```

`--boundary` only applies to a boundary node. Pointed at anything else, the CLI
says so up front rather than issuing a read that cannot succeed:

```
'Api.Encoder' is not a boundary — this working copy's index has it as a
behavior. Run `hydrate walk Api.Encoder` for its neighborhood, or
`hydrate pull` if the index is behind.
```

That verdict comes from your local index, which is why the message says so: a
node's kind can change on the branch. If the index is older than the branch, or
predates a kind this build knows, the CLI defers to the server rather than
guessing.

Like `show --depth`, `walk` is a genuinely scoped read when a pulled index is
present, and falls back to a whole-graph fetch with a note when one is not.

## Validate and commit

### `hydrate validate`

Dry-run the staged changeset against the bound branch and report the server's
coherence findings — without committing, and without clearing the stage. Run it
as many times as you like.

Exits `5` when there are error-severity findings, so an agent can gate the
commit in a shell:

```sh
hydrate validate && hydrate commit
```

Human output resolves each finding to a dotted path:

```
99 coherence findings:
  [error] unsatisfied_input  Api.Encoder.url: input port Api.Encoder.url has no incoming edge
```

JSON output carries the raw `locator` — the port or node id — beside the `code`,
`severity`, and `message`. The path resolution is the CLI's, from your pulled
index, so a stale working copy can yield a stale path; the CLI compares branch
versions and warns when it detects one.

Structural coherence
(a dangling edge, a missing input, a cycle, a kind that cannot hold what it
holds) is a **hard gate**. Type compatibility across an edge is **advisory** —
a mismatch is reported as a hint, not an error, and will not block a commit.

### `hydrate commit`

Commit the staged changeset to the bound branch as one typed delta batch, under
optimistic concurrency control. The server validates the whole batch. If any
operation is invalid (an unresolved path or a name collision, for example),
the commit is rejected and nothing is applied. If the branch has moved under you
since your last `pull`, the commit fails with a conflict (exit code `4`); `pull`
and retry.
