---
title: Install the CLI
description: Install the hydrate command-line client.
---

`hydrate` is the command-line client for hydrate.sh. It authors your system
graph from the terminal. The binary is `hydrate`, and `hyd` is a short alias.

## Prebuilt binaries

Prebuilt binaries ship with each tagged release, so you do not need a Rust
toolchain. Each release publishes archives for Linux (x86_64, aarch64), macOS
(x86_64, aarch64), and Windows (x86_64), each with a `.sha256` checksum.

Download the archive for your platform from the
[Releases](https://github.com/hydrate-sh/cli/releases) page, verify it, and put
`hydrate` on your `PATH`:

```sh
# Linux x86_64. Check Releases for the current tag; adjust the target for your platform.
tag=v0.1.17
target=x86_64-unknown-linux-gnu
curl -fsSLO "https://github.com/hydrate-sh/cli/releases/download/${tag}/hydrate-${tag}-${target}.tar.gz"
curl -fsSLO "https://github.com/hydrate-sh/cli/releases/download/${tag}/hydrate-${tag}-${target}.tar.gz.sha256"

# Integrity: did the download arrive intact?
sha256sum -c "hydrate-${tag}-${target}.tar.gz.sha256"

# Provenance: was it really built from this repo? Verify BEFORE you run it.
gh attestation verify "hydrate-${tag}-${target}.tar.gz" --repo hydrate-sh/cli

tar xzf "hydrate-${tag}-${target}.tar.gz"
./hydrate --version
```

The two checks do different jobs, and only the second is adversarial. The
archive and its checksum come from the same host over the same channel, so
anyone able to serve you a bad archive could serve a matching checksum —
`sha256sum` catches a truncated or corrupted download, not a substituted one.
The attestation is a signed statement that the artifact was built by this
repository's release workflow, which is why it belongs before `tar`.

`gh attestation` needs GitHub CLI 2.49 or newer. On Windows the asset is a
`.zip` rather than a `.tar.gz`.

## Authenticate

The CLI reads your API key from the environment:

```sh
export HYD_API_KEY=hyd_live_xxxxxxxxxxxxxxxxxxxx
```

See [Authentication](/getting-started/authentication/) for where keys come from
and how the CLI handles them.

## Orient yourself

The CLI ships a built-in orientation. Run it any time:

```sh
hydrate guide
```

It prints the authoring loop, the core concepts, and a worked example. The
[quickstart](/cli/quickstart/) below expands on it.
