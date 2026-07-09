# Next Integration Manifest

This file is the source of truth for rebuilding the `next` branch from
`next-base`. Keep it current whenever a custom patch branch or upstream PR
snapshot is added to or removed from `next`.

## Rebuild Order

Start from `next-base`, then merge the current integration set with one final
assembly merge. The branches should already merge cleanly; if the octopus merge
conflicts, fix the relevant branch first and retry.

1. Custom patch branches
2. Adapted integration branches for upstream PRs and external forks

```sh
git switch -C next next-base
git merge --no-ff \
  scroll-to-switch-tabs \
  integration/55404-detachable-items \
  integration/git-ui-improvements \
  integration/28674-tailwind-rust-completion \
  -m "Assemble next fork integrations"
```

After merging any requested upstream PR snapshots, push only when the intended
build set is complete:

```sh
git push --force-with-lease origin next
```

Pushing `next` starts the GitHub Actions build.

## Custom Patch Branches

| Branch | Source | Author | Purpose |
| --- | --- | --- | --- |
| `scroll-to-switch-tabs` | Local personal patch branch | Maciej Piasecki `<maciej@piasek.dev>` | Adds a setting and tab bar behavior for switching tabs with scroll input. |

Custom patch branches should be rebased on `upstream/main` before rebuilding
`next`:

```sh
git switch scroll-to-switch-tabs
git -c commit.gpgsign=false rebase upstream/main
git push --force-with-lease origin scroll-to-switch-tabs
```

Agents may rebase and update this branch with signing disabled unless a change
needs a product decision from Maciej.

## Adapted Integration Branches

Use adapted integration branches for upstream PRs that do not merge cleanly into
`next-base` or need compatibility changes. Keep the raw `pr/*` branch as an
attribution/source snapshot, then maintain an `integration/*` branch that
rebases on `next-base`.

| Branch | Raw Source Branch | Upstream PR | Upstream Author | Source Anchor | Purpose |
| --- | --- | --- | --- | --- | --- |
| `integration/55404-detachable-items` | `pr/55404-detachable-items` | [zed-industries/zed#55404](https://github.com/zed-industries/zed/pull/55404) | `iam-liam` | `f7321ff6c3993eeec93d51a4953c3c9421600d24` | Adds detachable editor items, with local drag-out/reattach behavior and maximized detached windows. |
| `integration/git-ui-improvements` | `external/firatoezcan-main` | External fork branch | Firat Ozcan `<admin@firatoezcan.com>` | `bd20394908006e3d206257239df57530b32418e0` | Integrates selected Git panel diff and file-history navigation behavior originally extracted from the Firat Ozcan fork, on top of upstream's current solo diff view. |
| `integration/28674-tailwind-rust-completion` | `pr/28674-tailwind-rust-completion` | [zed-industries/zed#28674](https://github.com/zed-industries/zed/pull/28674) | `I-Info` | `19a316e2b359c1bfaa2cff1af1c57581df76bd5d` | Enables Tailwind CSS completions in Rust string contexts. |

`Source Anchor` is the immutable upstream or external revision used as the
reference point for an integration. It may be a raw source branch tip, a
specific feature commit inside a source branch, or an external fork snapshot,
depending on what was actually imported.

### Integration Branch Maintenance

Rebase adapted integration branches on `next-base` before rebuilding `next`:

```sh
git switch integration/55404-detachable-items
git rebase next-base
git push --force-with-lease origin integration/55404-detachable-items

git switch integration/git-ui-improvements
git rebase next-base
git push --force-with-lease origin integration/git-ui-improvements

git switch integration/28674-tailwind-rust-completion
git rebase next-base
git push --force-with-lease origin integration/28674-tailwind-rust-completion
```

When creating a new adapted integration branch:

```sh
git switch -C integration/<number>-<short-name> next-base
git merge --squash --no-commit pr/<number>-<short-name>
# Resolve mechanical conflicts needed to apply the source on next-base.
git commit -m "Import <short description> source snapshot for next"
# Add local fork behavior changes as named follow-up commits.
```

Include source attribution in the import commit body. Do not include GitHub
autolinks such as `owner/repo#123`, `#123`, or full pull request URLs in commit
messages, because GitHub creates timeline references on the upstream PR.

```text
Upstream-repository: zed-industries/zed
Upstream-pull-request: <number>
Original-author: <github-login>
Snapshot: <raw-pr-head-sha>
```

### Layered Integration Workflow

Integration branches should use a layered branch history:

1. Keep the raw source snapshot untouched as `pr/<number>-<short-name>` for
   upstream PRs or `external/<owner>-<short-name>` for external forks.
2. Create the adapted integration branch from `next-base`.
3. Import the source as its own commit or merge commit, including only the
   mechanical conflict resolution needed to apply it to current `next-base`.
4. Add compatibility fixes or local fork tailoring as named follow-up commits.
5. Merge the adapted integration branch into `next`.

This keeps the raw source, mechanical conflict resolution, compatibility work,
and personal tailoring easy to inspect separately. Useful inspection commands:

```sh
git log --oneline <raw-source-branch>..integration/<short-name>
git diff <import-commit>..integration/<short-name>
```

The current integration branches follow this shape. A branch may have only an
import commit when there are no local behavior changes to separate.

## Raw Upstream PR Snapshot Branches

| Branch | Upstream PR | Upstream Author | Source Anchor | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |
| `pr/55404-detachable-items` | [zed-industries/zed#55404](https://github.com/zed-industries/zed/pull/55404) | `iam-liam` | `f7321ff6c3993eeec93d51a4953c3c9421600d24` | Raw upstream source snapshot for `integration/55404-detachable-items`. | Do not merge directly into `next`; merge the adapted `integration/55404-detachable-items` branch. |
| `pr/28674-tailwind-rust-completion` | [zed-industries/zed#28674](https://github.com/zed-industries/zed/pull/28674) | `I-Info` | `19a316e2b359c1bfaa2cff1af1c57581df76bd5d` | Raw upstream source snapshot for `integration/28674-tailwind-rust-completion`. | Do not merge directly into `next`; merge the adapted `integration/28674-tailwind-rust-completion` branch. |

## Raw External Fork Snapshot Branches

| Branch | External Source | Author | Source Anchor | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |
| `external/firatoezcan-main` | `firatoezcan/zed`, branch `main` | Firat Ozcan `<admin@firatoezcan.com>` | `bd20394908006e3d206257239df57530b32418e0` | Raw external fork snapshot used as source material for `integration/git-ui-improvements`. | Do not merge directly into `next`; merge the adapted `integration/git-ui-improvements` branch. |
| `external/firatoezcan-git-ui-improvements` | `firatoezcan/zed`, branch `autoresearch/git-ui-improvements-2026-04-04` | Firat Ozcan `<admin@firatoezcan.com>` | `8ae6296bb0790506f53b3c2022429a9bb705b4d2` | Raw external exploratory branch retained as source/reference material. | Do not merge directly into `next`. |

### Retired Search Modal Integration

The previous `integration/46478-search-modal` branch is no longer part of the
`next` rebuild. Upstream merged the picker preview and Telescope-style project
search work as `ccf4058b7a` (`Add preview to pickers and make them resizable`),
which closed the upstream issue this fork originally covered. Keeping the old
branch would reintroduce stale picker and keymap code, so the fork now relies on
upstream for that behavior.

The old local branches may remain as historical source snapshots:

| Branch | Source | Status |
| --- | --- | --- |
| `pr/46478-search-modal` | Upstream PR snapshot `26ea225a7ec0a4e302ddd0c70d4427302c005570` | Retired; do not merge into `next`. |
| `integration/46478-search-modal` | Adapted fork branch | Retired; do not merge into `next`. |

### PR #55404 Integration Notes

Import basis: single upstream feature commit
`f7321ff6c3993eeec93d51a4953c3c9421600d24`.

This PR needs an adapted branch for conflict resolution against current
`next-base` and local behavior changes:

- Resolve `crates/editor/src/editor.rs` by keeping the current editor state and
  adding the PR's window activation subscription for moved editor items.
- Rename the action from moving the active item to a new window to
  `DetachActiveItem`, with user-facing text `Detach Item`.
- Add tab drag-out behavior that detaches a tab into a maximized window.
- Add drag-back behavior for detached windows. When the compositor does not
  expose reliable cross-window drop targeting, dragging the detached tab out
  reattaches it to its recorded source pane.
- Close empty detached windows after cross-window tab drops, deferring the close
  until after the current GPUI event cycle to avoid invalidating drop dispatch.
- Avoid reading a pane while it is already being updated when source and target
  panes are the same during tab drop handling.

After changing the integration branch, run:

```sh
cargo check -p workspace
cargo check -p editor
cargo test -p workspace test_handle_tab_drop_respects_is_pane_target
cargo test -p workspace test_reattach_active_item_to_source_window
cargo test -p workspace test_detach_active_item
```

### PR #28674 Integration Notes

Import basis: feature commit `19a316e2b359c1bfaa2cff1af1c57581df76bd5d`.
The raw PR branch later merged `main`, so the useful Tailwind Rust change is
anchored to the older feature commit rather than the raw branch tip.

The upstream PR changed the old Rust language config path. In current Zed, the
adapted branch applies the same Rust string override in
`crates/grammars/src/rust/config.toml` and also adds `Rust` to the built-in
Tailwind language registration list in `crates/languages/src/lib.rs`.

After changing the integration branch, run:

```sh
cargo check -p languages
```

### Git UI Improvements Integration Notes

Import basis: external fork snapshot
`bd20394908006e3d206257239df57530b32418e0`. The integration extracts only the
useful Git UI feature work from that snapshot.

The raw source is kept locally as `external/firatoezcan-main`. The adapted
`integration/git-ui-improvements` branch selectively recreates the useful Git UI
behavior on current upstream APIs rather than merging the external snapshot
directly. It intentionally omits the fork README and `build_fork` GitHub Actions
changes, because this fork already has its own README, package flow, and
`zed-next` release workflow.

Local adaptations in the current branch:

- Keep upstream's `SoloDiffView` as the whole-file Git diff view.
- Make normal Git panel file clicks open the upstream solo diff view.
- Make the secondary confirm path (`ctrl/cmd` click or the secondary confirm
  action) open the shared Project Diff multibuffer instead.
- Preserve staged-vs-unstaged row selection for partially staged files.
- Use staged/unstaged-only diff base behavior where practical: staged rows
  should show `HEAD -> index`, and unstaged rows should show
  `index -> working tree`.
- Port file-history commit navigation from the external branch, so single-file
  commit views can navigate to the previous or next commit touching that file.

The old external branch also contained `git_panel.auto_fetch_interval`,
`git_panel.diff_page_size`, and alt-click marked-entry bulk operations. Those
are intentionally not part of this fork integration unless they are requested
again later.

After changing the integration branch, run:

```sh
cargo test -p git_ui test_filtered_head_diff_uses_index_for_partially_staged_file
cargo test -p git_ui test_select_entry_by_path_prefers_staged_row_for_partially_staged_file
cargo check -p git_ui
```

When adding one, use this branch format:

```text
pr/<number>-<short-name>
```

Create or update the snapshot branch:

```sh
git fetch upstream pull/<number>/head
git switch -C pr/<number>-<short-name> FETCH_HEAD
git push --force-with-lease origin pr/<number>-<short-name>
```

If an upstream PR snapshot merges cleanly and needs no local adaptation, it may
be merged into `next` directly:

```sh
git switch next
git merge --no-ff pr/<number>-<short-name> -m "Merge upstream PR <number> into next"
```

For PRs that need adaptation, add entries to both the raw snapshot table and the
adapted integration table above. For clean PRs, add an entry to the raw snapshot
table and use it directly in the rebuild order.

| Branch | Upstream PR | Upstream Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |

Use the upstream PR author from GitHub, not the local committer who imported the
branch. If a PR has multiple important authors, list them all.

## Notes For Future Agents

- Read this file and `FORK_NEXT.md` before changing `next`.
- Do not infer integrated patches only from the git graph; update this manifest
  when the intended build set changes.
- Keep upstream PR or integration branches as merge commits in `next` so they
  can be reverted or replaced cleanly.
- Keep custom patch branches rebaseable and named directly, without a prefix.
