---
title: Introduction
description: What hydrate.sh is and how the v1 API fits in.
---

hydrate.sh is a structured systems-thinking tool for software engineers. Your
project lives as a graph, the source of truth. The code is an artifact of it.

## The v1 API

The v1 API is the programmatic interface to your graph. Use it to read the
graph, validate proposed changes, and apply them from scripts, CI, or your own
tools.

The API is versioned in the path. `v1` is wire-stable: breaking changes will
rebase to `/v2/` rather than mutate `v1` in place.

## Next steps

- [Authentication](/getting-started/authentication/) — get a key and make your
  first request.
- [API reference](/api/v1/) — every endpoint, generated from the OpenAPI spec.
