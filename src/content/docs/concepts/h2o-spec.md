---
title: The h2o spec
description: The canonical format for a hydrate project — node, port, edge, and verification schemas, the delta mutation grammar, and the invariants the server enforces.
---

**h2o** is the canonical format for a hydrate project. It is the single source
of truth: the browser editor, the [`hydrate` CLI](/cli/quickstart/), the
[v1 API](/api/v1/), and every code-generation step all read and write the same
shapes defined here.

This page is the formal reference. If you want the concepts first, read
[The graph model](/concepts/graph-model/); if you want a task-oriented tour,
read the [Quickstart](/getting-started/quickstart/). This page is what those
pages describe precisely, field by field.

## Conventions

- **Identifiers are UUIDs.** Every node, port, edge, and verification has a
  `id` that is a UUID string.
- **Field naming is mixed.** Most fields are snake_case (`user_kind`,
  `path_prefix`, `is_external`, `source_decisions`, `parent_id`). A small set of
  fields cross the wire in camelCase: the delta reference fields `nodeId` and
  `edgeId`, the edge endpoints `sourceHandle` and `targetHandle`, and
  `isTestNode`. Notably, `parent_id` stays snake_case even inside a delta whose
  own reference field (`nodeId`) is camelCase.
- **Types are nominal strings.** A port type such as `HotDog` or `Rating` is
  just a name. Two types are compatible when their strings are exactly equal.
- **Unknown keys are ignored, not rejected.** A payload may carry extra keys;
  the server drops them silently. Missing *required* keys are rejected.
- **Empty string means null.** For nullable string fields (`parent_id`,
  `user_kind`, `path_prefix`, `external_kind`, `protocol`, `documentation_url`,
  and edge handles), the server coerces `""` to `null` at parse time.

## The graph document

A project is a graph of **nodes** and **edges**:

```json
{
  "nodes": [ /* Node objects */ ],
  "edges": [ /* Edge objects */ ]
}
```

When the graph is exported as a `.h2o` file (what the CLI works with), it is
wrapped in an envelope carrying a `schemaVersion`, the project name, and the
node and edge lists. See [Versioning](#versioning) below.

## Node

A node is a vertex in the graph.

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Required. |
| `kind` | enum | Required. One of `behavior`, `boundary`, `state`, `io`. |
| `parent_id` | UUID \| null | The enclosing boundary, or `null` for a top-level node. |
| `data` | NodeData | The payload; see below. Defaults to an empty payload. |

`kind` is immutable once a node exists. The only exception is
[`flatten_boundary`](#the-seven-deltas), which removes a boundary. A client may
echo `kind` inside `data`, but it must match the node's real kind.

### The four kinds

- **behavior** — a unit of work (a transform, a rule, a calculation). The
  default kind.
- **boundary** — a container that holds other nodes. Boundaries nest, forming a
  tree of containment (scopes) over the graph.
- **state** — a place shared, mutable data lives. Input ports are writes;
  output ports are reads.
- **io** — the program's edge with its environment: where data enters or leaves
  (standard input, arguments, the caller; standard output, an exit code, a
  response). An io node is **one-directional** — see the [io rules](#node-kind-field-matrix).

## NodeData

`data` carries the node's contract. Which fields are legal depends on the node's
`kind` and whether it is external — see the [field matrix](#node-kind-field-matrix).
All fields are optional at the type level; the cross-field rules are enforced
server-side.

| Field | Type | Default | Notes |
|---|---|---|---|
| `name` | string | `""` | Unique within the parent scope. |
| `description` | string | `""` | Plain-language spec; the prompt code is generated from. |
| `inputs` | Port[] | `[]` | Ports the node consumes. |
| `outputs` | Port[] | `[]` | Ports the node produces. |
| `config` | Port[] | `[]` | A third channel, never connected by edges. |
| `constraints` | string[] | `[]` | Free-text rules the node must obey. |
| `verifications` | Verification[] | `[]` | Test specifications (behavior nodes only). |
| `parent_id` | UUID \| null | `null` | The enclosing boundary; mutable via update. |
| `is_test_node` | bool | `false` | Wire alias `isTestNode`. Strict boolean. |
| `status` | string | `"idle"` | |
| `user_kind` | string \| null | `null` | Boundary label / state's `state_kind`. |
| `path_prefix` | string \| null | `null` | Boundary only. |
| `is_external` | bool | `false` | Strict boolean. |
| `external_kind` | string \| null | `null` | E.g. `rest-api`. External nodes only. |
| `protocol` | string \| null | `null` | E.g. `gRPC`. External nodes only. |
| `documentation_url` | string \| null | `null` | Must be `http://` or `https://`. |
| `source_decisions` | UUID[] | `[]` | Server-authoritative; see [invariants](#invariants). |

On **update**, key-presence semantics apply: a field *present* with a null value
is set to null; an *omitted* field is left unchanged. This is how a partial
update expresses "clear this" versus "don't touch this."

### Port

A port is one input, output, or config endpoint on a node.

| Field | Type | Default | Notes |
|---|---|---|---|
| `id` | UUID | — | Required. |
| `name` | string | `""` | Written `name:type` in tooling. |
| `type` | string | `""` | Nominal type name. |
| `description` | string | `""` | |

Port ids must be unique across a node's `inputs`, `outputs`, and `config`
combined.

### Verification

A verification is a test specification attached to a behavior node.

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Required. |
| `author` | enum | Required. One of `user`, `agent`. |
| `text` | string | Required. The check, in human terms. |
| `type` | string \| null | Optional. |

## Edge

An edge connects one node's output port to another node's input port.

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Required. |
| `sourceHandle` | UUID \| null | The source **output** port id. |
| `targetHandle` | UUID \| null | The target **input** port id. |

An edge runs **output → input**, and the two ports must have the **same type**.
Config ports are never edge endpoints. See the [edge rules](#edge-rules) for how
edges interact with state and io nodes.

## The delta mutation grammar

The graph is never overwritten wholesale. It changes through **deltas** — small,
typed mutations submitted as an ordered batch. The server validates the whole
batch as a dry run, and applies it atomically under optimistic concurrency: you
send the `version` you based your edits on, and the write is rejected if the
branch has moved since (see [invariants](#invariants)).

Each delta is a JSON object with a `type` discriminator. There are exactly
seven.

### The seven deltas

| `type` | Fields | Effect |
|---|---|---|
| `add_node` | `node` | Insert a node. |
| `delete_node` | `nodeId` | Delete a node and cascade to its descendant subtree. |
| `update_node_data` | `nodeId`, `after` | Update a node's data (key-presence partial semantics). |
| `reparent_node` | `nodeId`, `parent_id` | Move a node to a new parent, or to top-level (`null`). |
| `add_edge` | `edge` | Insert an edge. |
| `delete_edge` | `edgeId` | Remove an edge. |
| `flatten_boundary` | `nodeId` | Delete a boundary and promote its children to its parent. |

Notes on the partial-mutation deltas:

- `update_node_data.after` is a `NodeData` object, but only the fields *present*
  in it are changed — see [NodeData](#nodedata) key-presence semantics.
- `reparent_node.parent_id` must be *present* on the wire. Use `null` for
  top-level; an absent key is rejected, so a client bug surfaces as a clean
  parse error rather than a silent no-op.

A delta whose `type` is missing or not one of these seven is rejected with an
`unknown_delta_type` error. A known type with a malformed field is rejected with
a `validation_error` carrying the offending field path.

## Invariants

These are the semantic rules the server always enforces. They are what makes a
graph well-formed — and what a code-generation step can rely on.

### Node-kind field matrix

Each kind admits a specific set of fields:

| | behavior | boundary | state | io |
|---|:---:|:---:|:---:|:---:|
| `user_kind` | ✗ | ✓ | ✓ *(state_kind)* | ✗ |
| `path_prefix` | ✗ | ✓ | ✗ | ✗ |
| `config` ports | ✓ | ✓ | ✗ | ✗ |
| `verifications` | ✓ | — | ✗ | ✗ |
| may be `external` | ✓ | ✓ | ✗ | ✗ |

- **state** is internal-only, carries no `path_prefix`, no `config` ports, and
  no `verifications`. Its `user_kind` holds its `state_kind`.
- **io** is internal-only, carries no `user_kind`, no `path_prefix`, no `config`
  ports, and no `verifications`. It must carry **exactly one typed port on one
  side** — an output makes it a *source*, an input makes it a *sink*; it can
  never carry both (it is a source XOR a sink). Fan-out to many behaviors is
  expressed with *edges* on that one port, not with extra ports.

### External nodes

- An internal node may not carry `external_kind` or `protocol`.
- An external **behavior** requires an `external_kind`.
- An external **boundary** may not carry `external_kind` or `protocol`.

### Edge rules

- An edge runs **output → input**; endpoints must be **type-compatible** (equal
  type strings). Config ports are never endpoints.
- A **state or io edge** must connect exactly one behavior node and exactly one
  state-or-io node — never state↔state, io↔io, or state↔io. The direction
  carries the read/write semantic: a read is state-output → behavior-input; a
  write is behavior-output → state-input; a source io is io-output →
  behavior-input; a sink io is behavior-output → io-input.

### Structure

- **Names are unique within a scope.** A node's `name` is unique among its
  siblings; a port's `name` is unique on its node.
- **Ids are unique.** Node, port, and edge ids do not collide.
- **No self-parenting and no cycles.** `parent_id` cannot equal the node's own
  id, must reference an existing node on the branch, and a reparent may not
  create a containment cycle. An edge node may not be parented to a behavior,
  state, or io node.
- **`source_decisions` is server-authoritative.** It is accepted on `add_node`
  but cannot be mutated by `update_node_data`; an incoming list must match the
  current set exactly.
- **Optimistic concurrency.** A write carries the branch `version` it was based
  on; if the branch has advanced, the write is rejected and you re-pull.

## Versioning

Two version numbers matter, and they are independent:

- **`schemaVersion`** — the `.h2o` **file format** version. It advances when the
  format gains structure: boundary nodes arrived at `3`, state at `4`, and io at
  `5` (the current version). A reader upgrades older files forward.
- **The v1 API** is versioned in its path (`/v1`). The shapes on this page are
  what that version accepts; a breaking change to the wire shape would land under
  a new path, never by mutating `/v1` under you.

## See also

- [The graph model](/concepts/graph-model/) — the same structure, explained
  conceptually.
- [Quickstart](/getting-started/quickstart/) — a task-oriented tour of the
  editor and CLI.
- [API reference](/api/v1/) — the live, generated reference for the HTTP
  surface, including the delta endpoint.
