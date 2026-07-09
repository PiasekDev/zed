# Zed Next

This is a personal downstream build of [Zed](https://github.com/zed-industries/zed)
maintained by [PiasekDev](https://github.com/PiasekDev).

This is not an official Zed build. For the upstream editor, documentation, and
issue tracker, use:

- Upstream repository: https://github.com/zed-industries/zed
- Zed website: https://zed.dev
- Zed docs: https://zed.dev/docs

## Branches

- `next-base`: fork infrastructure, packaging, release workflow, and fork docs.
- `next`: daily-driver build branch. It starts from `next-base`, then uses one
  assembly merge for personal patch branches and selected upstream PR branches.
- `scroll-to-switch-tabs`: personal patch branch currently included in `next`.
- `pr/<number>-<short-name>`: raw local snapshots of upstream Zed PRs.
- `integration/<number>-<short-name>`: adapted PR branches that carry local
  conflict resolutions or compatibility fixes.

See [FORK_NEXT.md](./FORK_NEXT.md) for the full branch, update, and release
workflow. See [NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md) for the current
patch/PR integration manifest used to rebuild `next`.

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
