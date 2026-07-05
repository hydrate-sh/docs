---
title: The graph model
description: Nodes, typed ports, edges, boundaries, and branches.
---

A hydrate project is a graph. This page defines its structure, which both the
[CLI](/cli/quickstart/) and the [v1 API](/api/v1/) operate on. For the formal,
field-by-field reference — the canonical **h2o** format, its delta grammar, and
the invariants the server enforces — see [The h2o spec](/concepts/h2o-spec/).

## Nodes

A node is a vertex in the graph. There are four kinds:

- **Behavior**: a unit of work. The default kind.
- **Boundary**: a node that contains other nodes. Boundaries nest, forming a
  tree of containment over the graph.
- **State**: a place shared, mutable data lives. Its input ports are writes and
  its output ports are reads.
- **I/O**: the program's edge with its environment — where data enters or leaves
  (standard input, arguments, the caller; standard output, an exit code, a
  response). An I/O node carries exactly one typed port on one side: an output
  port makes it a source (data flowing into the program), an input port a sink
  (data flowing out).

A behavior or boundary may be marked external, representing a system outside the
graph. External nodes carry an external-kind label (for example, `rest-api`) and
may name a protocol (for example, `gRPC`).

See [Node types](/getting-started/quickstart/#node-types) in the Quickstart for
the shapes and when to reach for each.

### Node fields

Every node has a free-text **description**. It may also carry:

- **Constraints**: a list of free-text strings.
- **Verifications**: a list of free-text strings.
- **Doc URL**: an optional documentation link.

Boundary nodes additionally have a user-kind label, a path prefix, and an
optional **language** — the codegen language the boundary declares, which the
nodes inside it inherit.

## Ports and types

A node exposes ports. Every port has a name and a type, written `name:type`; the
type is always required. There are three channels:

- **Inputs**: ports a node consumes.
- **Outputs**: ports a node produces.
- **Config**: a third channel that is not connected by edges.

A type is a nominal string, such as `HotDog`, `ShortCode`, or `Rating`. Two
ports are compatible when their type strings are equal.

## Edges

An edge connects an output port to an input port. Both ports must have the same
type. The server rejects an edge whose endpoints have different types, and config
ports cannot be edge endpoints.

## Addressing: dotted paths

Nodes and ports are addressed by dotted path, scoped by the boundaries that
contain them.

- `Api.Rater` is the node `Rater` inside the boundary `Api`.
- `Api.Rater.score` is the port `score` on that node.
- `Rater` is a top-level node with no enclosing boundary.

A node's name is unique within its parent scope.

## Branches

A project has a `main` branch and any number of working branches. You create a
working branch with `hydrate fork` and make edits on it; `main` is not edited
directly. The graph is stored on the server, and the CLI keeps a local view that
`hydrate pull` refreshes.

## See also

- [The h2o spec](/concepts/h2o-spec/): the formal format reference.
- [CLI quickstart](/cli/quickstart/): the commands that build a graph.
- [CLI command reference](/cli/reference/): every verb and flag.
- [API reference](/api/v1/): the same operations over HTTP.
