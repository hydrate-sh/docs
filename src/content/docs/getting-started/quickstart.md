---
title: Quickstart
description: What hydrate.sh is, what you can do today, and the concepts to get started.
---

hydrate.sh lets you design a software system as a **graph of typed decisions**
rather than as files. The graph is the source of truth; code is an artifact
generated from it. A clear graph makes the code that follows almost free.

This guide covers what you can do in the current alpha and the concepts you
need to be productive.

## What you can do today

Alpha is focused on **authoring and validating a system graph**:

- Build the graph — in the browser editor, or from the terminal with the
  [CLI](/cli/quickstart/).
- Describe each node in plain language; a node's description is the instruction
  its code is generated from.
- Connect nodes through **typed ports** and **validate** that the pieces fit
  before any code exists.
- Author with the **Chat agent** in the browser.

Two things are **not in the app yet**, and this guide points to where they land:

- **Hydration** — turning the graph into code — runs in the app soon. For now
  you generate code **locally, with your own LLM**, from the graph you've
  authored. See [Getting code out](#getting-code-out).
- **Code import** — bringing an existing codebase back into the graph — is
  coming.

## API keys

You use up to two independent keys, depending on what you're doing:

| To… | You need | Where it goes |
|---|---|---|
| Use the **Chat agent** in the browser | your **Anthropic** key | Settings → Account |
| Use the **CLI** and hydrate **locally** | a **Hydrate** key **and** your **Anthropic** key | `HYD_API_KEY` for the CLI; your Anthropic key for local generation |

Your **Hydrate** key (`hyd_live_…`) authenticates you to your hydrate.sh
account — see [Authentication](/getting-started/authentication/). Your
**Anthropic** key is your own: hydrate.sh runs LLM calls on *your* key, never a
shared one. The Chat agent uses your Anthropic key to author the graph — and it
can only edit the graph; it cannot hydrate or generate code.

## Node types

A graph is built from five kinds of node. Each has a distinct shape, so the
structure reads at a glance:

- **Behavior** *(rectangle)* — a unit of logic: a transform, a rule, a
  calculation. The default kind.
- **Boundary** *(frame)* — a concept grouping that contains other nodes.
  Boundaries nest, forming scopes over the graph.
- **State** *(cylinder)* — a place shared, mutable data lives (a store, a cache,
  a table). Its input ports are writes; its output ports are reads.
- **I/O** *(parallelogram)* — the program's edge with its environment: where
  data enters or leaves (standard input, command-line arguments, the caller;
  standard output, an exit code, a response). An I/O node carries **exactly one
  typed port on one side** — a **source** (an output port, data flowing *into*
  the program) or a **sink** (an input port, data flowing *out*).
- **Interface** *(hexagon)* — the program's edge with the callers that depend on
  it: a piece of the public API surface, such as a function, type, or parameter
  that outside code links against. Where I/O is the program's edge with its
  environment, interface is its edge with its consumers.

Any node can also be marked **external** — a system outside your graph that you
depend on (a payment gateway, a database) — labelled with its kind and an
optional protocol.

## Descriptions are the prompt

Every node has a **description** in plain language. It is not documentation — it
is the instruction the node's code is generated from. *"Validate an RFC 5322
email address and return whether it is deliverable"* is a spec that generates;
*"email stuff"* is not. Two fields sharpen it:

- **Constraints** — rules the node must obey (edge cases, invariants), in human
  terms.
- **Verifications** — checks that confirm it; each becomes a test specification.

Writing good descriptions is the real work: a well-specified node generates
reliable code.

## Typed ports and connections

A node exposes **ports**, each written `name:type` (the type is required). There
are three channels: **inputs** (consumed), **outputs** (produced), and **config**
(a third channel, not wired by edges).

An **edge** connects an output port to an input port of the **same type** —
that's how you declare that two nodes fit together. **Validate** checks every
edge: compatible connections turn green, mismatches red, *before any code
exists*. **Clear validation** resets the tint. A type is just a name (`HotDog`,
`Rating`); two ports match when their type strings are equal.

## Editing the graph

In the browser editor you can:

- **Add** nodes and boundaries, and **wire** an output port to an input port by
  dragging between them.
- **Drill into** a boundary to work at that altitude, then step back out.
- **Edit a node's contract** (double-click): its name, description, ports,
  constraints, and verifications.
- **Group**, **reparent** (move a node under a boundary), **delete**, and
  **auto-organize** the layout.
- **Author with Chat**: describe what you want, and the agent adds and wires
  nodes for you.

Your work lives on a **branch** and autosaves. You fork a working branch and
build on it; `main` is not edited directly. The same graph is reachable from the
[CLI](/cli/quickstart/) and the [API](/api/v1/).

## Getting code out

App-side hydration is coming. Today the path is local:

1. Author and validate your graph (browser or CLI).
2. Pull it with the CLI (`hydrate pull`), or read it via the [API](/api/v1/).
3. Each node's **description is the generation prompt**. Generate code from the
   graph with **your own LLM** (your Anthropic key) — node by node, or a
   boundary at a time.

First-class hydration in the app — and **code import**, which brings an existing
codebase back into the graph — are on the roadmap.

## Next steps

- [The graph model](/concepts/graph-model/) — the precise structure of nodes,
  ports, edges, and branches.
- [CLI quickstart](/cli/quickstart/) — build a graph from the terminal.
- [Authentication](/getting-started/authentication/) — get your Hydrate key.
- [API reference](/api/v1/) — the same graph over HTTP.
