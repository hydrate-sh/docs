---
title: Authentication
description: How to authenticate requests to the hydrate.sh v1 API.
---

Every `/v1/` request authenticates with an API key, sent as a Bearer token in
the `Authorization` header:

```bash
curl https://api.hydrate.sh/v1/projects \
  -H "Authorization: Bearer hyd_live_xxxxxxxxxxxxxxxxxxxx"
```

A request with a missing or invalid key returns `401`.

## Getting a key

API keys are issued from the hydrate dashboard. A key is shown once, when it's
created — copy it then and store it in a secrets manager. It can't be retrieved
again; if you lose it, issue a new one.

Keys come in two variants, shown by their prefix: `hyd_test_…` and `hyd_live_…`.

## Scope and revocation

A key acts with the access of the account it was issued for, and never more.
Revoking a key takes effect immediately — the next request returns `401`. See
the [API reference](/api/v1/) for the scopes each endpoint requires.
