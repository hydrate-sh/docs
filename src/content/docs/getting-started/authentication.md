---
title: Authentication
description: How to authenticate the hydrate.sh CLI and the v1 API.
---

Both the [`hydrate` CLI](/cli/quickstart/) and the [v1 API](/api/v1/)
authenticate with the same kind of API key. One key works for both surfaces.

## Getting a key

API keys are issued from the hydrate dashboard. A key is shown once, when it is
created, so copy it then and store it in a secrets manager. It cannot be
retrieved again; if you lose it, issue a new one.

Keys come in two variants, shown by their prefix: `hyd_test_…` and `hyd_live_…`.

## The v1 API

Every `/v1/` request sends the key as a Bearer token in the `Authorization`
header:

```bash
curl https://api.hydrate.sh/v1/projects \
  -H "Authorization: Bearer hyd_live_xxxxxxxxxxxxxxxxxxxx"
```

A request with a missing or invalid key returns `401`.

## The CLI

The CLI reads your key from the **`HYD_API_KEY`** environment variable, or from
a `.env` file in the working directory:

```bash
export HYD_API_KEY=hyd_live_xxxxxxxxxxxxxxxxxxxx
hydrate branches
```

The CLI does not write the key to disk, and it does not print or log it,
including in `--json` output and at debug level.

To point the CLI at a different service URL for local development, set
`HYD_BASE_URL`.

## Scope and revocation

A key acts with the access of the account it was issued for, and no more.
Revoking a key takes effect immediately: the next request returns `401`. See the
[API reference](/api/v1/) for the scopes each endpoint requires.
