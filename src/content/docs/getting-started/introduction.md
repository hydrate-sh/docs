---
title: Introduction
description: The hydrate.sh v1 API and the hydrate CLI.
---

hydrate.sh stores a project as a graph. This documentation covers the two
interfaces to that graph: the v1 HTTP API and the `hydrate` command-line client.

## The graph

A project is a graph of nodes connected through typed ports. Each node has a
description and a set of ports, and edges connect ports of matching type. See
[The graph model](/concepts/graph-model/) for the full structure.

## Interfaces

- **The `hydrate` CLI** operates on the graph from the terminal. You fork a
  working branch, stage edits, review them, and commit. See the
  [CLI quickstart](/cli/quickstart/).
- **The v1 API** is the HTTP interface for reading and modifying the graph. See
  the [API reference](/api/v1/).

Both interfaces operate on the same graph through the same delta API. The API is
versioned in the path; `v1` is wire-stable, and a breaking change will rebase to
`/v2/` rather than change `v1` in place.

## Next steps

- [The graph model](/concepts/graph-model/): nodes, ports, edges, and branches.
- [Authentication](/getting-started/authentication/): get a key and make a
  request.
- [CLI quickstart](/cli/quickstart/): author a graph from the terminal.
