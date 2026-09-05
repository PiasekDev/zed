# Zed Next fork

`PiasekDev/zed` is Maciej's personal downstream build of Zed. It runs editor
features upstream does not have yet on top of a fresh upstream `main`, as an
Arch package that replaces the stock Zed. Agents do most of the maintenance;
this file is written for them as much as for Maciej.

Read this file, then [NEXT_INTEGRATIONS.md](./NEXT_INTEGRATIONS.md) (the build
set) and [UPSTREAM_WATCHLIST.md](./UPSTREAM_WATCHLIST.md) (what upstream may
absorb), before touching any branch. [FEATURES.md](./FEATURES.md) says what
the build set means for someone using the editor; keep it in step when an
integration is added, narrowed, or retired.

## Principles

- **Infrastructure and behavior never share a branch.** `next-base` carries
  packaging, workflows, scripts, and these docs on top of `upstream/main`.
  Editor behavior lives on one branch per feature. `next` is only ever
  assembled from them; nobody commits to `next` directly.
- **A refresh is a proposal, not an edit.** Rebases happen on dated
  `refresh/<date>/*` copies. The live branch set moves in one step at
  promotion, after the proposal built and was reviewed. Two 2026 proposals
  were never promoted and the fork went stale for two months: promote or
  discard a proposal, never leave one hanging.
- **Raw sources stay immutable.** `pr/*` and `external/*` snapshots are
  evidence of what was imported. Adaptation happens on `integration/*`
  branches in layers: import commit, compatibility fixes, fork tailoring,
  each its own commit, so the fork's share of a feature stays inspectable.
- **Upstream absorbs, the fork retires.** Every integration has a retirement
  condition in the manifest and a watchlist entry. Every refresh checks them
  and narrows or retires what upstream now carries.
- **Agents do the mechanics, Maciej decides and pushes.** Agent commits are
  unsigned with a co-author trailer. Decisions are collected up front; a run
  does not stop to ask. Only product decisions and hardware-key actions wait
  for Maciej.
- **Everything gates on the whole workspace.** `cargo check --workspace`
  before anything is proposed; per-crate checks have let a release build fail
  on an untouched crate.

## Branches

| Pattern | Holds | Moves at |
| --- | --- | --- |
| `upstream/main` | Upstream Zed. Fetch over HTTPS; the SSH push URL would prompt the hardware key | Upstream pushes |
| `origin/main` | Mirror of `upstream/main` on the fork | Every proposal |
| `next-base` | Fork infrastructure only. GitHub default branch, so its workflow files are the ones that run | Promotion |
| `<feature>` (no prefix) | Personal patch branch rebased on `upstream/main`, for example `scroll-to-switch-tabs` | Promotion |
| `pr/<number>-<name>`, `external/<owner>-<name>` | Raw upstream-PR or external-fork snapshot; never merged directly | Only to re-snapshot a moved source |
| `integration/<number-or-owner>-<name>` | Adapted snapshot rebased on `next-base` | Promotion |
| `refresh/<date>/<any of the above>` | Dated proposal copies | Deleted at promotion |
| `next` | `next-base` plus one octopus merge of the build set. Pushing it starts the `zed_next` build | Promotion |
| `refs/backups/<date>/*` | Pre-refresh snapshot of the live set; keep two generations | `script/fork-refresh start` |

## Refresh runbook

`script/fork-refresh` does the mechanics and stops where judgment is needed.
Run it from the repository root on a clean tree; `--date` defaults to today.

1. `script/fork-refresh start` fetches, backs up the live set, creates the
   dated branches, rebases `next-base` on `upstream/main` and every build-set
   branch on the dated `next-base`. It stops on the first conflict and prints
   the commands to continue; re-running is idempotent. Pass
   `--from refresh/<previous-date>` when an earlier proposal carries newer
   adaptations than the live branches.
2. Judge. For each build-set branch, check its retirement condition and the
   watchlist against the upstream range. What upstream now carries gets
   dropped from the integration branch; compatibility fixes are named
   follow-up commits on that branch. Manifest and watchlist edits go on the
   dated `next-base`; after committing there, run `start` again so the
   integrations sit on the new tip.
3. `script/fork-refresh assemble` recreates the dated `next`. An octopus
   cannot resolve two integrations touching the same lines: fix the branch,
   or merge the colliding pair sequentially and keep the octopus for the
   rest.
4. `script/fork-refresh gate` runs `script/fork-refresh-gates` (workspace
   check, fmt, the focused tests per integration) and keeps the log under
   `target/fork-refresh/<date>/`. A failed gate is a finding for the report,
   not a stop.
5. `script/fork-refresh report` writes `.github/refresh-report.md`; fill in
   its Decisions section, then `script/fork-refresh report --commit`. The
   report becomes the PR body. Its compare links per branch are the review
   artifact; the PR diff itself is mostly upstream churn.
6. `script/fork-refresh propose` pushes the dated branches and the `main`
   mirror, then opens the PR labeled `refresh-proposal` against `next`.
7. Maciej reviews and comments `/promote` on the PR. The `promote_refresh`
   workflow moves every live branch to its dated copy, force-pushes `next`
   (the build starts), and deletes the dated branches. `script/fork-refresh
   sync` then updates the local live branches. Without GitHub,
   `script/fork-refresh promote-local` moves the local branches and prints
   the push commands.
8. After a promoted `next-base` reaches GitHub, disable upstream workflows
   that arrived with the refresh. Upstream workflow files are kept so rebases
   stay quiet; unwanted ones are disabled, not deleted:

   ```sh
   gh workflow list --repo PiasekDev/zed --json name,path,state \
     --jq '.[] | select(.state == "active") | .path' |
     grep -v -e zed_next -e promote_refresh |
     xargs -r -n1 basename | xargs -r -n1 gh workflow disable --repo PiasekDev/zed
   ```

Adding or retiring an integration is described in the manifest.

## Why the pieces are shaped this way

- `README.md` has `merge=ours` in `.gitattributes` (driver:
  `git config merge.ours.driver true`, set per clone), so upstream README
  churn never conflicts; wanted upstream README content is cherry-picked.
- The proposal PR is opened with the owner's token, not by a workflow:
  GitHub Actions may not create pull requests in this repository (a
  repository setting), and the workflow that tried failed on its first run.
  Promotion is a `/promote` comment rather than an approval because an author
  cannot approve their own PR.
- The promotion workflow dispatches `zed_next.yml` explicitly: pushes made
  with `GITHUB_TOKEN` never trigger other workflows.
- Before importing a `pr/*` snapshot, verify its tip against
  `gh pr view <number> -R zed-industries/zed --json headRefOid`. A stale
  `FETCH_HEAD` once produced a snapshot pointing at unrelated local work
  while the commit message claimed the right SHA.
- One heavy cargo job at a time on Maciej's machine; parallel full builds
  and test suites have frozen it (out of memory).

## Building and installing

The `zed_next` workflow (`.github/workflows/zed_next.yml`) builds the Linux
app with glibc and the remote servers with musl, and publishes the moving
`zed-next` prerelease that `packaging/arch/zed-next-bin` installs. Keep the
musl-specific compiler settings in the workflow: the host glibc compiler makes
the static remote server fail to link. `sccache` comes from
`mozilla-actions/sccache-action`; do not set `SCCACHE_GHA_ENABLED` or
`RUSTC_WRAPPER` before that action runs.

Local alternatives live in `packaging/arch/` (`zed-next-local`,
`zed-next-git`) and `script/package-zed-next-local`, which builds the current
checkout as a stable-channel replacement (it temporarily writes `stable` to
`crates/zed/RELEASE_CHANNEL`) and packages it with `makepkg`. Local builds
add `-C target-cpu=native`; set `ZED_NEXT_TARGET_CPU=` to disable that. Every
replacement build sets `ZED_UPDATE_EXPLANATION`, which turns upstream
auto-update off.

The fork owns Linux remote-server distribution: bootstrap resolves the remote
server from the `zed-next` release instead of Zed's release service, because
a `main`-tracking fork is ahead of the stable assets. The cache key is the
full app version, so every `next` build gets its own entry. Manual recovery:
place a matching binary in `~/.zed_server` on the remote host.

## Authentication

Pushes use HTTPS with the gh credential helper configured **in this
repository only**; the rest of the machine stays on SSH with the hardware
key. Re-provision a clone with:

```sh
git remote set-url --push origin https://github.com/PiasekDev/zed.git
git config credential."https://github.com".helper ''
git config --add credential."https://github.com".helper '!/usr/bin/gh auth git-credential'
```

Never run `gh auth setup-git` here; it writes the helper globally. The token
needs the `workflow` scope (`gh auth refresh -h github.com -s workflow`)
because pushes carry workflow files. The token lives in the desktop keyring,
so headless runs fail authentication by design.

## Commit authorship

Agents commit and rebase with `git -c commit.gpgsign=false` and add a
co-author trailer naming the agent, on fixups and rewritten integration
commits too. Merge commits on `next` stay plain. The repository `.rules`
require the README review marker whenever an agent modifies source; Maciej
removes it after review.

## Scheduling

No weekly schedule is live (decision pending). `packaging/systemd/user/`
holds a user timer that runs a local Claude Code agent through this runbook
every Friday evening inside the desktop session, where the keyring is
available:

```sh
systemctl --user link "$PWD/packaging/systemd/user/zed-fork-refresh.service"
systemctl --user link "$PWD/packaging/systemd/user/zed-fork-refresh.timer"
systemctl --user enable --now zed-fork-refresh.timer
```

Manual runs: open a Claude Code session in this repository and ask for the
refresh runbook.
