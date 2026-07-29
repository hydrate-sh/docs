---
title: CLI quickstart
description: Build your first graph from the terminal, walking the authoring loop end to end.
---

This page walks the full authoring loop: fork a branch, stage a few nodes and an
edge, review what is staged, and commit. It assumes you have
[installed the CLI](/cli/install/) and set `HYD_API_KEY`, and that you have read
[the graph model](/concepts/graph-model/).

## The authoring loop

You build a graph the way you build a change in version control: on a branch, in
stages, committed when it is ready.

1. **`hydrate fork <name>`**: create a working branch and bind this directory to it.
2. **`hydrate pull`**: sync a local view of the branch's live graph, so you can
   reference already-committed nodes by their dotted path.
3. **`hydrate node add …`** and **`hydrate edge add …`**: stage behaviors,
   boundaries, and the edges between them. Nothing has reached the server yet.
4. **`hydrate diff`**: review exactly what is staged.
5. **`hydrate validate`**: dry-run the changeset and see the server's coherence
   findings, without committing.
6. **`hydrate commit`**: apply the staged changeset to the branch as one typed
   batch.

The CLI stages edits locally and commits them as a single delta batch. The
server is the sole authority for validation, so the CLI does not mirror its
rules. If a batch is invalid, the server rejects it at commit time and reports
why.

`validate` is that same check, on demand and without the commit. It exits `5`
when there are error-severity findings, which makes it a gate:

```sh
hydrate validate && hydrate commit
```

Run it as often as you like — it never clears the stage.

## A worked example

Build a small URL shortener: an `Api` boundary holding two behaviors, wired
output to input through a shared type.

```sh
hydrate fork demo

hydrate node add --kind boundary --name Api

hydrate node add --kind behavior --name Shorten --parent Api --out url:LongUrl \
    --description 'POST /shorten: validate the body, normalize the URL, emit it.'

hydrate node add --kind behavior --name Encoder --parent Api \
    --in url:LongUrl --out code:ShortCode \
    --description 'Mint a collision-free base62 short code for a URL.'

hydrate edge add --from Api.Shorten.url --to Api.Encoder.url

hydrate diff
hydrate commit
```

What each step does:

- The boundary `Api` is a grouping. The two behaviors live inside it, so they
  are addressed `Api.Shorten` and `Api.Encoder`.
- `Shorten` has an output port `url` of type `LongUrl`, and `Encoder` has an
  input port `url` of the same type. Because the types match, the edge is valid.
- Each `--description` sets the node's description, a free-text field.

`hydrate diff` shows the staged operations, and `hydrate commit` sends them as
one batch. If a type does not line up or a path does not resolve, the commit
fails with a clear error and nothing is applied.

## Editing in place

After a `hydrate pull`, you can edit committed nodes by path:

```sh
hydrate node set Api.Encoder --description 'Mint a collision-free base62 code; 7 chars.'
hydrate node rm Api.Shorten          # removes the node (cascades its subtree)
hydrate clear                        # stage removal of every top-level node, to rebuild
```

Every edit stages the same way. Review with `hydrate diff`, then apply with
`hydrate commit`.

## Reading without pulling the whole graph

The graph you are editing may be far larger than the part you are working on.
Two verbs read *slices* of it, so a big graph stays off the wire — and out of a
coding agent's limited context.

```sh
hydrate show Api --depth 1     # Api and its direct children
hydrate walk Api.Encoder       # Encoder plus everything it touches
hydrate walk Api --boundary    # what lives inside Api
```

The two answer different questions, and the difference matters:

- **`show <path>`** answers *what is inside this*. It prints the subtree and the
  edges **interior** to it. An edge leaving the slice is not shown, so a node
  read this way can look unconnected when it is not.
- **`walk <path>`** answers *what does this touch*. It prints the node in full
  plus its 1-hop neighborhood — every node on the other end of an edge, in
  either direction.

Both are genuinely scoped: the server sends only the slice. That needs a working
copy that has been pulled, because the local index is what turns your dotted
path into the id the scoped read is addressed by. Without one the CLI falls back
to fetching the whole graph and narrowing it locally, **and tells you it did**.
The output is identical; only the transfer differs.

Neither verb mutates anything. They create no branch and stage nothing.

## Next

- [Command reference](/cli/reference/): every verb and flag.
- [The graph model](/concepts/graph-model/): the concepts behind the commands.
