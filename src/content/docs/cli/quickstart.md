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
5. **`hydrate commit`**: apply the staged changeset to the branch as one typed
   batch.

The CLI stages edits locally and commits them as a single delta batch. The
server is the sole authority for validation, so the CLI does not mirror its
rules. If a batch is invalid, the server rejects it at commit time and reports
why.

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

## Next

- [Command reference](/cli/reference/): every verb and flag.
- [The graph model](/concepts/graph-model/): the concepts behind the commands.
