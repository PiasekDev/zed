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

Before touching branches, snapshot the current set and preview conflicts:

```sh
for b in next next-base scroll-to-switch-tabs integration/…; do
  git update-ref "refs/backups/$(date +%F)/${b//\//-}" "refs/heads/$b"
done
git merge-tree --write-tree <new-base> <branch>   # per branch; conflict dry-run
```

Backups live under `refs/backups/<date>/`, not as `backup/*` branches, so
`git branch` stays legible. Keep the two most recent generations; delete older
ones after a release built from the new set is confirmed good.

Rebase custom patch branches:

```sh
git switch scroll-to-switch-tabs
git -c commit.gpgsign=false rebase upstream/main
```

Rebase the infrastructure branch:

```sh
git switch next-base
git rebase upstream/main
```

Rebuild `next` from the infrastructure branch:

```sh
git switch -C next next-base
git merge --no-ff \
  scroll-to-switch-tabs \
  integration/55404-detachable-items \
  integration/git-ui-improvements \
  integration/28674-tailwind-rust-completion \
  integration/59884-group-by-staging \
  -m "Assemble next fork integrations"
```

Use [NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md) for the current branch list.
If the octopus merge conflicts, fix the relevant integration branch first and
retry the assembly merge. When two integration branches conflict with each
other (it has happened on shared `pane.rs` import lists), the octopus cannot
resolve it: fall back to sequential `git merge --no-ff` for the colliding pair,
resolving in that merge commit only, and keep the octopus for the rest.

Notes that keep rebases quiet:

- `README.md` carries `merge=ours` in `.gitattributes` (driver:
  `git config merge.ours.driver true`, set locally). Upstream README changes
  never conflict; cherry-pick wanted upstream README content deliberately.
- Upstream workflow files are kept, not deleted. Unwanted workflows are
  disabled in repo settings. After pushing a refreshed `next-base`, run the
  disable sweep so newly added upstream workflows do not start running:

```sh
gh workflow list --repo PiasekDev/zed --json name,path,state \
  --jq '.[] | select(.state == "active") | .path' |
  grep -v -e zed_next -e open_refresh_proposal -e promote_refresh |
  xargs -r -n1 basename | xargs -r -n1 gh workflow disable --repo PiasekDev/zed
```

For a new selected upstream PR:

```sh
git fetch upstream pull/<number>/head:pr/<number>-<short-name>
git merge --no-ff pr/<number>-<short-name> -m "Merge upstream PR <number> into next"
```

Use rebases for personal patch branches so each long-lived personal change
remains small and readable. Use the final `next` merge commit only as assembly,
not as a place to resolve integration conflicts.

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

Before importing from the snapshot, verify it actually points at the PR head
(`gh pr view <number> -R zed-industries/zed --json headRefOid`). A stale
`FETCH_HEAD` between fetch and branch creation has produced a snapshot branch
pointing at unrelated local work while the import commit message claimed the
right SHA — the import then silently brings in the wrong code.

## Publishing

Do not push `next` until it contains all intended patches and PR branches for
that build. A push to `next` starts the GitHub Actions build.

Gate before pushing: `cargo check --workspace` plus the per-integration test
lists in `NEXT_INTEGRATIONS.md`. Integration-crate checks alone have let a
release build fail on an untouched crate (`settings_ui`, 2026-06-30).

When ready:

```sh
git push origin next-base
git push --force-with-lease origin next
git push origin upstream/main:main   # keep the origin mirror current
```

`next-base` does not trigger the build workflow. `next` does. Pushes go over
HTTPS using the gh CLI credential helper configured repo-locally (see
Authentication under Scheduled Refresh Automation), so no hardware-key touch
is needed in this repository.

## Build Outputs

GitHub Actions publishes a moving prerelease named `zed-next` containing:

- `zed-linux-x86_64.tar.gz`
- `zed-remote-server-linux-x86_64.gz`
- `zed-remote-server-linux-aarch64.gz`
- `zed-next-SHA256SUMS.txt`
- `zed-next-version`

The binary Arch package reads `zed-next-version` so the package version follows
the Zed crate version and commit SHA instead of hard-coding a Zed version.

The workflow builds the app for glibc Linux and remote servers for musl. Keep
the musl target-specific compiler settings in `.github/workflows/zed_next.yml`;
using the host glibc compiler for native C dependencies can make the static
musl remote server fail to link. The x86_64 remote server is built by the Linux
bundle job. The aarch64 remote server is built by a separate native ARM GitHub
Actions job and published into the same moving release. The workflow also
enables `sccache` with the GitHub Actions cache backend through
`mozilla-actions/sccache-action` and caches Cargo registry/git sources to make
repeated builds faster. Do not set `SCCACHE_GHA_ENABLED` and `RUSTC_WRAPPER`
before that action runs; raw `sccache` needs the GitHub Actions cache URL and
runtime token that the action exposes.

## Remote Server Assets

This fork owns remote server distribution for replacement builds. Linux remote
bootstrap resolves `zed-remote-server-linux-x86_64.gz` and
`zed-remote-server-linux-aarch64.gz` from the fork's moving GitHub release:

```text
https://github.com/PiasekDev/zed/releases/download/zed-next/
```

The local cache version remains the full app version, including build metadata,
so each pushed `next` build gets its own remote-server cache entry. Official Zed
Cloud release lookup is intentionally bypassed for these Linux remote-server
assets because a `main`-tracking fork can be ahead of the official stable
release assets.

If remote bootstrap fails and a manual recovery is needed, place a matching
remote server binary in `~/.zed_server` on the remote host using the path shown
by the connection error or logs. This should be temporary; the normal path is to
publish the matching asset through the `zed_next` workflow.

## Commit Authorship

Agent-created infrastructure, integration, and custom patch branch updates are
usually committed with signing disabled to avoid YubiKey prompts:

```sh
git -c commit.gpgsign=false commit ...
git -c commit.gpgsign=false rebase ...
```

When an agent creates or rewrites such commits, include a co-author trailer
for the agent that authored the work, for example:

```text
Co-authored-by: Codex <codex@openai.com>
Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
```

Use the same trailer for follow-up fixup commits, amended commits, and rewritten
integration-branch commits. Merge commits on `next` can remain simple merge
markers.

## Working Style

Frontload decisions: collect everything that needs Maciej's input at the start
of a maintenance run, then create, commit, and build without mid-run prompts.
Stop only for true product decisions or actions that need his hardware key.
Finish by handing back a summary plus the exact push/install commands.

## Scheduled Refresh Automation

The weekly refresh runs as a local Codex scheduled automation on Maciej's
machine (Friday evening), inside the desktop session so the gh keyring
credential is available. It never touches `next` directly; it builds a
proposal and promotion happens only through Maciej's PR approval.

### Proposal flow

1. Fetch `upstream` and `origin`; read `UPSTREAM_WATCHLIST.md`.
2. Build the refresh on fresh dated branches, never on the live ones:
   `refresh/<YYYY-MM-DD>/next-base`, one `refresh/<YYYY-MM-DD>/<integration>`
   per manifest branch, and the assembled `refresh/<YYYY-MM-DD>/next`.
3. Gate: `git merge-tree` dry-runs, `cargo check --workspace`, the manifest's
   per-integration test lists. Gate failures do not block the proposal; they
   are reported prominently in the summary instead.
4. Write the summary to `.github/refresh-report.md` on the proposal `next`
   branch: upstream range, watchlist state changes, conflicts and how they
   were resolved, obsolescence candidates, gate results, and per-integration
   GitHub compare links (the raw PR diff includes upstream churn and is not
   the review artifact).
5. Push the dated branches, then dispatch the PR opener:
   `gh workflow run open_refresh_proposal.yml -f branch=refresh/<date>/next -f title="Refresh next from upstream <date>"`.
   The PR is opened by `github-actions[bot]` because GitHub forbids approving
   your own PR and the local automation authenticates as the repo owner.

### Promotion

Approving the proposal PR is the go-ahead. The `promote_refresh` workflow
(trigger: PR review submitted; guarded to approvals by the repo owner on
`refresh/*`-headed, `refresh-proposal`-labeled PRs against `next`) then:

1. Force-pushes `next` to the proposal head.
2. Dispatches `zed_next.yml` explicitly — required because `GITHUB_TOKEN`
   pushes never trigger other workflows.
3. Comments on the PR and deletes the proposal branches. The PR closes as
   merged on its own once `next` contains the head commits.

Steering instead of approving: comment on the PR; the next automation run (or
an on-demand run) reads open proposal-PR comments and rebuilds the proposal
accordingly. Rejecting: close the PR; nothing was changed.

After promotion, the next scheduled run fast-forwards the live branch set
(`next-base`, integrations) to what was promoted and pushes `origin/main` to
mirror `upstream/main`.

### Schedule

The schedule lives as the Codex Desktop automation `zed-fork-refresh-proposal`
(weekly, Friday 19:00 Europe/Warsaw). Manage it through a Codex Desktop
thread using Codex's automation tool; it is not exposed via the Codex CLI or
app-server API. This machine's scheduler has previously mispersisted
`TZID=Europe/Warsaw` rules as UTC — after any schedule change, verify
`next_run_at` in `~/.codex/sqlite/codex-dev.db` converts to the intended
local time.

### Authentication

Git pushes use HTTPS with the gh CLI credential helper, configured
**repo-locally only** — the rest of the machine stays on SSH with Maciej's
hardware key, and this repository is the deliberate exception so agents and
the automation can push without key touches. No separate PAT. To re-provision
(new machine or new clone):

```sh
git remote set-url --push origin https://github.com/PiasekDev/zed.git
git config credential."https://github.com".helper ''
git config --add credential."https://github.com".helper '!/usr/bin/gh auth git-credential'
```

Do not run `gh auth setup-git` for this — it writes the helper into the
global git config. The gh token needs the `workflow` scope
(`gh auth refresh -h github.com -s workflow`) because pushes here routinely
carry `.github/workflows/` changes. The token lives in the desktop keyring,
so headless runs outside the desktop session will fail auth — this is
accepted, not a bug to fix with a plaintext token.

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

1. Read this file, `NEXT_INTEGRATIONS.md`, and `UPSTREAM_WATCHLIST.md`.
2. Fetch `upstream` and `origin`; snapshot the branch set to
   `refs/backups/<date>/` and dry-run conflicts with `git merge-tree`.
3. Rebase `next-base` on `upstream/main`, using unsigned agent commits with
   the authoring agent's co-author trailer.
4. Rebase custom patch branches (agents may do this unsigned; involve Maciej
   only for product decisions).
5. Rebase or rebuild the integration branches listed in
   `NEXT_INTEGRATIONS.md`; verify `pr/*` snapshot tips against the upstream
   PR head SHA before any import.
6. Recreate `next` from `next-base` and assemble with the octopus merge.
7. Gate: `cargo check --workspace` plus the manifest's per-integration tests.
8. Update `NEXT_INTEGRATIONS.md` and `UPSTREAM_WATCHLIST.md` if the build set
   or watched items changed.
9. Validate package metadata with `makepkg --printsrcinfo` for packages touched.
10. Do not push `next` until the intended set is complete; after pushing
    `next-base`, run the workflow disable sweep; also push the `main` mirror.
