# Zed Next Fork Workflow

This repository is a personal downstream build of Zed. The goal is to keep a
small infrastructure branch on top of upstream `main`, then build the daily
driver `next` branch by merging selected personal patch branches and selected
upstream pull requests.

## Branches

- `upstream/main`: read-only remote from `zed-industries/zed`.
- `origin/main`: optional mirror of upstream `main` in `PiasekDev/zed`.
- `next-base`: fork infrastructure only. This contains packaging, release, and
  local build support. It should not contain editor behavior patches.
- `next`: daily-driver branch. Start from `next-base`, then merge custom patch
  branches and selected upstream PR branches.
- Custom patch branches: named directly, for example `scroll-to-switch-tabs`.
  Keep these rebaseable on top of `upstream/main`.
- Raw upstream PR snapshot branches: use `pr/<number>-<short-name>`.
- Raw external fork snapshot branches: use `external/<owner>-<short-name>`.
- Adapted integration branches: use `integration/<number>-<short-name>` for
  upstream PRs and `integration/<owner-or-feature>-<short-name>` for external
  forks.

The current intended merge set is tracked in
[NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md). Update that file whenever a
custom patch branch or upstream PR snapshot is added to or removed from `next`.

## Updating From Upstream

Fetch upstream first:

```sh
git fetch upstream --prune
git fetch origin --prune
```

Rebase custom patch branches:

```sh
git switch scroll-to-switch-tabs
git rebase upstream/main
```

Rebase the infrastructure branch:

```sh
git switch next-base
git rebase upstream/main
```

Rebuild `next` from the infrastructure branch:

```sh
git switch -C next next-base
git merge --no-ff scroll-to-switch-tabs -m "Merge scroll-to-switch-tabs into next"
```

Use [NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md) for the full ordered merge
list.

Then merge any selected upstream PR or integration branches:

```sh
git fetch upstream pull/<number>/head:pr/<number>-<short-name>
git merge --no-ff pr/<number>-<short-name> -m "Merge upstream PR <number> into next"
```

Use merge commits for upstream PRs so they are easy to revert or replace. Use
rebases for personal patch branches so each long-lived personal change remains
small and readable.

If an upstream PR needs conflict resolutions or compatibility fixes, keep the
raw `pr/*` branch untouched and create an adapted
`integration/<number>-<short-name>` branch on top of `next-base`. Merge the
adapted integration branch into `next`.

For external fork work, keep a raw `external/*` branch as the source snapshot
and create an adapted `integration/*` branch for the version that should merge
into this fork. Prefer layered integration history for new work: import the raw
source as its own commit or merge commit, then add compatibility fixes and
personal tailoring as follow-up commits. This makes it easy to inspect what was
changed beyond the original source. Some current integrations predate this
workflow and may be split into layered commits in a future cleanup.

When importing an upstream PR, push a personal snapshot branch to this fork so
the exact tested code remains available even if the upstream PR branch is
force-pushed or deleted:

```sh
git fetch upstream pull/<number>/head
git switch -C pr/<number>-<short-name> FETCH_HEAD
git push --force-with-lease origin pr/<number>-<short-name>
```

## Publishing

Do not push `next` until it contains all intended patches and PR branches for
that build. A push to `next` starts the GitHub Actions build.

When ready:

```sh
git push origin next-base
git push --force-with-lease origin next
```

`next-base` does not trigger the build workflow. `next` does.

## Build Outputs

GitHub Actions publishes a moving prerelease named `zed-next` containing:

- `zed-linux-x86_64.tar.gz`
- `zed-remote-server-linux-x86_64.gz`
- `zed-next-SHA256SUMS.txt`
- `zed-next-version`

The binary Arch package reads `zed-next-version` so the package version follows
the Zed crate version and commit SHA instead of hard-coding a Zed version.

The workflow builds the app for glibc Linux and the remote server for musl. Keep
the musl target-specific compiler settings in `.github/workflows/zed_next.yml`;
using the host glibc compiler for native C dependencies can make the static
musl remote server fail to link. The workflow also enables `sccache` with the
GitHub Actions cache backend through `mozilla-actions/sccache-action` and caches
Cargo registry/git sources to make repeated builds faster. Do not set
`SCCACHE_GHA_ENABLED` and `RUSTC_WRAPPER` before that action runs; raw `sccache`
needs the GitHub Actions cache URL and runtime token that the action exposes.

## Installing

Install the GitHub-built binary package:

```sh
cd packaging/arch/zed-next-bin
makepkg -Csi
```

Build and install from this checkout while reusing the local `target` directory:

```sh
script/package-zed-next-local
```

Package an already-built local bundle:

```sh
script/package-zed-next-local --no-build
```

Clean source-build package:

```sh
cd packaging/arch/zed-next-git
makepkg -Csi
```

## Local Build Notes

`script/package-zed-next-local` temporarily writes `stable` to
`crates/zed/RELEASE_CHANNEL`, builds the Linux bundle, restores the previous
file contents, and packages the tarball through `makepkg`.

The local and source-build packages add `-C target-cpu=native` by default. To
disable that:

```sh
ZED_NEXT_TARGET_CPU= script/package-zed-next-local
ZED_NEXT_TARGET_CPU= makepkg -Csi
```

All replacement builds set `ZED_UPDATE_EXPLANATION`, which disables upstream
auto-update and points updates back to this fork/package flow.

## Agent Checklist

When asked to update this fork:

1. Read this file.
2. Fetch `upstream` and `origin`.
3. Rebase `next-base` on `upstream/main`.
4. Rebase personal patch branches on `upstream/main`.
5. Recreate `next` from `next-base`.
6. Merge the branches listed in `NEXT_INTEGRATIONS.md` in order.
7. Fetch any requested upstream PR branches as `pr/<number>-<name>`.
8. Fetch any requested external fork branches as `external/<owner>-<name>`.
9. Update `NEXT_INTEGRATIONS.md` if the intended build set changed.
10. Validate package metadata with `makepkg --printsrcinfo` for packages touched.
11. Do not push `next` until the requested PR set is complete.
