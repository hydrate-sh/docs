---
title: Introduction
description: The hydrate.sh v1 API.
---

hydrate.sh stores a project as a graph. This documentation covers the v1 HTTP API
for reading and modifying that graph.

## The graph

A project is a graph of nodes connected through typed ports. Each node has a
description and a set of ports, and edges connect ports of matching type. See
[The graph model](/concepts/graph-model/) for the full structure.

## The v1 API

The **v1 API** is the HTTP interface for reading and modifying the graph. See the
[API reference](/api/v1/).

The API is versioned in the path; `v1` is wire-stable, and a breaking change will
rebase to `/v2/` rather than change `v1` in place.

## Next steps

- [The graph model](/concepts/graph-model/): nodes, ports, edges, and branches.
- [Authentication](/getting-started/authentication/): get a key and make a
  request.
- [API reference](/api/v1/): every endpoint over HTTP.
