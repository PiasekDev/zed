# Next Integration Manifest

This file is the source of truth for rebuilding the `next` branch from
`next-base`. Keep it current whenever a custom patch branch or upstream PR
snapshot is added to or removed from `next`.

## Rebuild Order

Start from `next-base`, then merge entries in this order:

1. Custom patch branches
2. Adapted integration branches for upstream PRs and external forks

```sh
git switch -C next next-base
git merge --no-ff scroll-to-switch-tabs -m "Merge scroll-to-switch-tabs into next"
git merge --no-ff integration/46478-search-modal -m "Merge search modal integration into next"
git merge --no-ff integration/55404-detachable-items -m "Merge detachable item integration into next"
git merge --no-ff integration/firatoezcan-git-ui -m "Merge firatoezcan Git UI integration into next"
git merge --no-ff integration/28674-tailwind-rust-completion -m "Merge Tailwind Rust completion integration into next"
```

After merging any requested upstream PR snapshots, push only when the intended
build set is complete:

```sh
git push --force-with-lease origin next
```

Pushing `next` starts the GitHub Actions build.

## Custom Patch Branches

| Branch | Source | Author | Purpose | Merge Command |
| --- | --- | --- | --- | --- |
| `scroll-to-switch-tabs` | Local personal patch branch | Maciej Piasecki `<maciej@piasek.dev>` | Adds a setting and tab bar behavior for switching tabs with scroll input. | `git merge --no-ff scroll-to-switch-tabs -m "Merge scroll-to-switch-tabs into next"` |

Custom patch branches should be rebased on `upstream/main` before rebuilding
`next`:

```sh
git switch scroll-to-switch-tabs
git rebase upstream/main
git push --force-with-lease origin scroll-to-switch-tabs
```

## Adapted Integration Branches

Use adapted integration branches for upstream PRs that do not merge cleanly into
`next-base` or need compatibility changes. Keep the raw `pr/*` branch as an
attribution/source snapshot, then maintain an `integration/*` branch that
rebases on `next-base`.

| Branch | Raw Source Branch | Upstream PR | Upstream Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- | --- |
| `integration/46478-search-modal` | `pr/46478-search-modal` | [zed-industries/zed#46478](https://github.com/zed-industries/zed/pull/46478) | `ozacod` | `54e639eab83a9a2315be9a68d084c94effa68001` | Adds a search modal for project-wide text search. | `git merge --no-ff integration/46478-search-modal -m "Merge search modal integration into next"` |
| `integration/55404-detachable-items` | `pr/55404-detachable-items` | [zed-industries/zed#55404](https://github.com/zed-industries/zed/pull/55404) | `iam-liam` | `f7321ff6c3993eeec93d51a4953c3c9421600d24` | Adds detachable editor items, with local drag-out/reattach behavior and maximized detached windows. | `git merge --no-ff integration/55404-detachable-items -m "Merge detachable item integration into next"` |
| `integration/firatoezcan-git-ui` | `external/firatoezcan-main` | External fork branch | Firat Ozcan `<admin@firatoezcan.com>` | `bd20394908c4b4e1d8e7cbb0ca669e68d511314a3` | Adds Git panel single-file diff/history improvements, with local defaults and staged/unstaged fixes. | `git merge --no-ff integration/firatoezcan-git-ui -m "Merge firatoezcan Git UI integration into next"` |
| `integration/28674-tailwind-rust-completion` | `pr/28674-tailwind-rust-completion` | [zed-industries/zed#28674](https://github.com/zed-industries/zed/pull/28674) | `I-Info` | `19a316e2b359c1bfaa2cff1af1c57581df76bd5d` | Enables Tailwind CSS completions in Rust string contexts. | `git merge --no-ff integration/28674-tailwind-rust-completion -m "Merge Tailwind Rust completion integration into next"` |

### Integration Branch Maintenance

Rebase adapted integration branches on `next-base` before rebuilding `next`:

```sh
git switch integration/46478-search-modal
git rebase next-base
git push --force-with-lease origin integration/46478-search-modal

git switch integration/55404-detachable-items
git rebase next-base
git push --force-with-lease origin integration/55404-detachable-items

git switch integration/firatoezcan-git-ui
git rebase next-base
git push --force-with-lease origin integration/firatoezcan-git-ui

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

| Branch | Upstream PR | Upstream Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |
| `pr/46478-search-modal` | [zed-industries/zed#46478](https://github.com/zed-industries/zed/pull/46478) | `ozacod` | `54e639eab83a9a2315be9a68d084c94effa68001` | Raw upstream source snapshot for `integration/46478-search-modal`. | Do not merge directly into `next`; merge the adapted `integration/46478-search-modal` branch. |
| `pr/55404-detachable-items` | [zed-industries/zed#55404](https://github.com/zed-industries/zed/pull/55404) | `iam-liam` | `f7321ff6c3993eeec93d51a4953c3c9421600d24` | Raw upstream source snapshot for `integration/55404-detachable-items`. | Do not merge directly into `next`; merge the adapted `integration/55404-detachable-items` branch. |
| `pr/28674-tailwind-rust-completion` | [zed-industries/zed#28674](https://github.com/zed-industries/zed/pull/28674) | `I-Info` | `19a316e2b359c1bfaa2cff1af1c57581df76bd5d` | Raw upstream source snapshot for `integration/28674-tailwind-rust-completion`. | Do not merge directly into `next`; merge the adapted `integration/28674-tailwind-rust-completion` branch. |

## Raw External Fork Snapshot Branches

| Branch | External Source | Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |
| `external/firatoezcan-main` | `firatoezcan/zed`, branch `main` | Firat Ozcan `<admin@firatoezcan.com>` | `bd20394908c4b4e1d8e7cbb0ca669e68d511314a3` | Raw external fork snapshot used to extract `integration/firatoezcan-git-ui`. | Do not merge directly into `next`; merge the adapted `integration/firatoezcan-git-ui` branch. |
| `external/firatoezcan-git-ui-improvements` | `firatoezcan/zed`, branch `autoresearch/git-ui-improvements-2026-04-04` | Firat Ozcan `<admin@firatoezcan.com>` | `8ae6296bb0790506f53b3c2022429a9bb705b4d2` | Raw external exploratory branch retained as source/reference material. | Do not merge directly into `next`. |

### PR #46478 Integration Notes

This PR is old relative to current `upstream/main`, so the adapted
`integration/46478-search-modal` branch carries local integration work:

- Resolve `crates/search/Cargo.toml` and `Cargo.lock` by keeping both
  `smol` and `text` dependencies for the `search` crate.
- In `crates/search/src/quick_search/delegate/picker_impl.rs`, handle the newer
  transient `SearchResult::WaitingForScan` and `SearchResult::Searching`
  variants by ignoring them in the quick-search result loop.

After changing the integration branch, run:

```sh
cargo check -p search
```

### PR #55404 Integration Notes

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

The upstream PR changed the old Rust language config path. In current Zed, the
adapted branch applies the same Rust string override in
`crates/grammars/src/rust/config.toml` and also adds `Rust` to the built-in
Tailwind language registration list in `crates/languages/src/lib.rs`.

After changing the integration branch, run:

```sh
cargo check -p languages
```

### firatoezcan Git UI Integration Notes

The raw source is kept locally as `external/firatoezcan-main`. The adapted
`integration/firatoezcan-git-ui` branch extracts the useful Git UI feature work
and intentionally omits the fork README and `build_fork` GitHub Actions changes,
because this fork already has its own README, package flow, and `zed-next`
release workflow.

Local adaptations in the current branch:

- Enable `git.single_file_diff` by default so Git panel file clicks open a
  whole-file single-file diff tab instead of the shared Project Diff
  multibuffer excerpt view.
- Fix duplicated staged/unstaged Git panel rows for partially staged files so
  selecting a path can preserve whether the user clicked the staged row or the
  unstaged row.
- Fix filtered Head diffs for partially staged files so staged rows show
  `HEAD -> index` and unstaged rows show `index -> working tree`, instead of
  collapsing to the final `HEAD -> working tree` comparison.
- Center newly opened single-file diff tabs on the first diff hunk.
- Add coverage for partially staged staged/unstaged selection and diff content.

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
