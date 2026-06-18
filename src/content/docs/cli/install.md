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
[Releases](https://github.com/hydrate-sh/cli/releases) page, verify it against
its published checksum, and put `hydrate` on your `PATH`:

```sh
# Linux x86_64. Adjust the version and target for your platform.
tag=v0.1.5
target=x86_64-unknown-linux-gnu
curl -fsSLO "https://github.com/hydrate-sh/cli/releases/download/${tag}/hydrate-${tag}-${target}.tar.gz"
curl -fsSLO "https://github.com/hydrate-sh/cli/releases/download/${tag}/hydrate-${tag}-${target}.tar.gz.sha256"
sha256sum -c "hydrate-${tag}-${target}.tar.gz.sha256"
tar xzf "hydrate-${tag}-${target}.tar.gz"
./hydrate --version
```

The archives also carry signed build provenance. Verify it with the GitHub CLI:

```sh
gh attestation verify hydrate-${tag}-${target}.tar.gz --repo hydrate-sh/cli
```

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
