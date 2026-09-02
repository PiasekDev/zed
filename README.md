> [!IMPORTANT]
> Remove this line to confirm you've reviewed this PR before submitting.

# Zed Next

This is a personal downstream build of [Zed](https://github.com/zed-industries/zed)
maintained by [PiasekDev](https://github.com/PiasekDev).

This is not an official Zed build. For the upstream editor, documentation, and
issue tracker, use:

- Upstream repository: https://github.com/zed-industries/zed
- Zed website: https://zed.dev
- Zed docs: https://zed.dev/docs

## Branches

`next` is the daily-driver build, assembled from `next-base` (fork
infrastructure) and the integration branches listed in
[NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md). The maintenance workflow,
including the weekly refresh, is in [FORK_NEXT.md](./FORK_NEXT.md).

## Installing

Install the GitHub-built Arch package:

```sh
cd packaging/arch/zed-next-bin
makepkg -Csi
```

Build from this checkout and install through pacman:

```sh
script/package-zed-next-local
```

Package an already-built local bundle:

```sh
script/package-zed-next-local --no-build
```

Clean source build through makepkg:

```sh
cd packaging/arch/zed-next-git
makepkg -Csi
```

All replacement builds disable upstream auto-update and point update behavior
back to this fork/package flow.

## Building

Pushing `next` to this fork starts the `zed_next` GitHub Actions workflow, which
publishes a moving prerelease named `zed-next`.

Do not push `next` until all intended personal patches and upstream PR branches
for that build have been merged locally.

## Licensing

Zed source code is licensed primarily under GPL-3.0-or-later, with Apache-2.0
components where marked.

License information for third party dependencies must be correctly provided for
CI to pass.
