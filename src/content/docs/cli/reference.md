---
title: CLI command reference
description: Every hydrate CLI verb and its flags.
---

The complete command surface. Commands fall into four groups: branch context,
authoring, inspection, and commit. Authoring commands *stage* changes locally;
nothing reaches the server until `hydrate commit`.

Every command is available under both the `hydrate` and `hyd` names. Run
`hydrate <command> --help` for the terse inline reference, or `hydrate guide`
for an orientation.

## Global flags

| Flag | Effect |
| --- | --- |
| `--json` | Force machine-readable JSON output. |
| `--human` | Force human-readable output. |

By default the CLI prints human-readable output on a terminal and JSON when its
output is piped. The two carry the same information. `--json` and `--human`
override the auto-detection and cannot be combined.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Generic failure. |
| `4` | Conflict (the branch moved under you; `pull` and retry). |
| `6` | Network failure. |

Richer machine-readable detail rides in `--json` output, while the exit codes
stay stable.

## Branch context

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

## Inspection

### `hydrate status`

Show the bound branch and a summary of the staged operations.

### `hydrate diff`

Show the staged operations in detail. Nothing here has hit the server yet.

## Commit

### `hydrate commit`

Commit the staged changeset to the bound branch as one typed delta batch, under
optimistic concurrency control. The server validates the whole batch. If any
operation is invalid (a type mismatch, an unresolved path, a name collision),
the commit is rejected and nothing is applied. If the branch has moved under you
since your last `pull`, the commit fails with a conflict (exit code `4`); `pull`
and retry.
