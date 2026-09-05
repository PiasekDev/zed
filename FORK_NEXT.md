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
- **Agents do the mechanics and publish them; Maciej decides and tests.**
  The agent running a refresh authors its commits and pushes the fork's
  branches once the gates and a local build pass. Decisions are collected up
  front; a run does not stop to ask. Only product decisions wait for Maciej.
- **Every session refreshes, few sessions ship.** Any session that touches
  the fork begins with the cheap half of the runbook (steps 1 and 3 plus
  `cargo check --workspace`, roughly five minutes, ideally in a subagent that
  reports conflicts back). The expensive half (gate, local build, package,
  push) runs at ship points only: a feature landed, or Maciej asked.
- **Everything gates on the whole workspace.** `cargo check --workspace`
  before anything is proposed; per-crate checks have let a release build fail
  on an untouched crate.

## Branches

| Pattern | Holds | Moves at |
| --- | --- | --- |
| `upstream/main` | Upstream Zed. Fetch over HTTPS; the SSH push URL would prompt the hardware key | Upstream pushes |
| `origin/main` | Mirror of `upstream/main` on the fork | Every proposal |
| `next-base` | Fork infrastructure only. GitHub default branch, so its workflow files are the ones that run | Promotion |
| `<feature>` (no prefix) | Fork feature branch rebased on `upstream/main`: Maciej's own patches (`scroll-to-switch-tabs`) and features the fork maintains itself once their outside source is dead (`file-history-navigation`) | Promotion |
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
   its Decisions section, then `script/fork-refresh report --commit`. Its
   compare links per branch are the review artifact; the full diff against
   the previous state is mostly upstream churn.
6. `script/fork-refresh promote-local` moves the live local branches onto the
   dated ones and prints the push commands; run them. Pushing `next` starts
   the `zed_next` build. Fork branches only, never upstream.
7. After the promoted `next-base` reaches GitHub, disable upstream workflows
   that arrived with the refresh. Upstream workflow files are kept so rebases
   stay quiet; unwanted ones are disabled, not deleted:

   ```sh
   gh workflow list --repo PiasekDev/zed --json name,path,state \
     --jq '.[] | select(.state == "active") | .path' |
     grep -v -e zed_next -e promote_refresh |
     xargs -r -n1 basename | xargs -r -n1 gh workflow disable --repo PiasekDev/zed
   ```
8. `script/fork-refresh clean` deletes the dated branches, locally and on
   `origin`; the live branches now carry their content and
   `refs/backups/<date>/*` keeps the pre-refresh state.

Adding or retiring an integration is described in the manifest.

## Why the pieces are shaped this way

- `README.md` has `merge=ours` in `.gitattributes` (driver:
  `git config merge.ours.driver true`, set per clone), so upstream README
  churn never conflicts; wanted upstream README content is cherry-picked.
- `script/fork-refresh propose` and `.github/workflows/promote_refresh.yml`
  (a PR labeled `refresh-proposal`, promoted by a `/promote` comment) stay in
  the tree for a future automated setup; nothing uses them today.
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

On Maciej's desktop the local package is what gets installed:
`script/package-zed-next-local` builds the current checkout as a
stable-channel replacement (it temporarily writes `stable` to
`crates/zed/RELEASE_CHANNEL`) and packages it with `makepkg`, with
`-C target-cpu=native` (set `ZED_NEXT_TARGET_CPU=` to disable). Agents run it
with `--no-install` and report the package path; Maciej installs with
`sudo pacman -U <path>`. `makepkg` rewrites `pkgver` in the PKGBUILD, so
`git checkout -- packaging/arch/zed-next-local/PKGBUILD` afterwards. Other
local variants live in `packaging/arch/` (`zed-next-local`, `zed-next-git`).
Pushing `next` still matters even though the desktop builds its own: GitHub
produces the portable package for other machines and the remote-server assets
below. Every replacement build sets `ZED_UPDATE_EXPLANATION`, which turns
upstream auto-update off.

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

From 2026-09-05 on, the agent doing the work is the author of what it
commits (earlier fork commits carry Maciej as author with a co-author
trailer): unsigned
(`git -c commit.gpgsign=false -c user.name=... -c user.email=...`), no
co-author trailer, and the same for rebases of Maciej's own patch branches
such as `scroll-to-switch-tabs`. Keep `FORK_REFRESH_COAUTHOR` empty so the
script adds no trailer either. Maciej signs only the commits he cares about
personally. The README review marker required by `.rules` applies only to
changes headed for an upstream pull request; fork-only work never adds it (see
the "Fork workflow" section of `.rules`), and only Maciej removes one.

`.claude/commands/refresh.md` is the runbook as a Claude Code `/refresh`
command; agents that are not Claude Code run the instructions in that same
file. `AGENTS.md` is upstream's symlink to `.rules` and is never edited here.

## Scheduling

Refreshes are manual: this desktop is the only builder, so a refresh happens
when a session touches the fork. `packaging/systemd/user/` keeps a user timer
for later, which would run a local Claude Code agent through this runbook
every Friday evening inside the desktop session, where the keyring is
available:

```sh
systemctl --user link "$PWD/packaging/systemd/user/zed-fork-refresh.service"
systemctl --user link "$PWD/packaging/systemd/user/zed-fork-refresh.timer"
systemctl --user enable --now zed-fork-refresh.timer
```

Manual runs: open a Claude Code session in this repository and type
`/refresh`.
