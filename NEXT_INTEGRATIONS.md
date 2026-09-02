# Next integration manifest

The build set of `next`. `script/fork-refresh` reads the **Build set** table
below: one row per branch, the branch name in backticks in the first column;
keep that shape. The gate commands per integration live in
`script/fork-refresh-gates`.

## Build set

| Branch | Source | Carries | Retire when |
| --- | --- | --- | --- |
| `scroll-to-switch-tabs` | Personal patch by Maciej Piasecki | `tab_bar.scroll_to_switch_tabs`: the wheel over the tab bar switches tabs, Shift inverts, no wrap, horizontal scroll ignored, the pinned row handled on its own | Upstream ships wheel tab switching. Intended for upstream after a fresh-angle rework and Maciej's own testing |
| `integration/55404-detachable-items` | `pr/55404-detachable-items`, upstream PR 55404 by `iam-liam` (open) | `DetachActiveItem` ("Detach Item"), tab drag-out into a maximized window, drag-back reattaches to the source pane, emptied detached windows close after the drop | The PR merges or upstream ships detach and reattach |
| `integration/git-ui-improvements` | `external/firatoezcan-main` by Firat Ozcan, rebuilt on current APIs | A view-file arrow button on git panel entries; previous and next commit navigation in single-file commit views, with `(commit, file)` tab identity | Both land upstream. Upstream already has file history views and a `file_filter` on `CommitView`; only the navigation and the per-file dedup remain fork-only |
| `integration/28674-tailwind-rust-completion` | `pr/28674-tailwind-rust-completion`, upstream PR 28674 by `I-Info` (closed unmerged) | Tailwind completions inside Rust string and raw-string literals; Rust registered for the built-in Tailwind server | Upstream registers Rust for Tailwind completions |
| `integration/59884-group-by-staging` | Fork tailoring on upstream's merged PR 59884; `pr/59884-group-by-staging` is the historical snapshot | Section-aware per-file diffs from the staging-grouped panel: staged rows open `HEAD -> index` read-only, unstaged rows open `index -> worktree`; combined footer totals with per-section fallback; remote updates for section-stat-only changes | Upstream opens per-file diffs against the section base with the same stat semantics |
| `integration/61067-word-diff-unequal-hunks` | `pr/61067-word-diff-unequal-hunks`, upstream PR 61067 by `aetosdios27` (open) | Word-level diff highlights on hunks whose base and buffer line counts differ: lines are paired by similarity and word-diffed per pair, so an edited line next to an added or removed one keeps its intra-line highlight | The PR merges or upstream relaxes the equal-line-count gate in `buffer_diff` |

Paired user settings live in Maciej's dotfiles, not in code:
`git_panel.group_by: "staging"` and
`git_panel.entry_primary_click_action: "file_diff"` for the staging
integration, `tab_bar.scroll_to_switch_tabs: true` for the patch branch.

## Constraints that are not obvious from the code

- Detachable items: keep upstream's editor activation behavior. The source
  PR's per-editor window-activation subscription was removed upstream after
  blink and redraw regressions; detachment does not need it. When source and
  target panes are the same during a tab drop, do not read the pane while it
  is being updated; close an emptied detached window only after the current
  GPUI event cycle.
- Git UI improvements: upstream's `CommitView::open` takes the file filter
  and streams history through `LogSource::Path`; the fork adds older/newer
  navigation on top of `Repository::file_history_shas`.
- Tailwind in Rust: the override lives in
  `crates/grammars/src/rust/config.toml` plus the language list in
  `crates/languages/src/lib.rs`; the PR's original config path no longer
  exists.
- Group by staging: `SoloDiffView` takes a `DiffBase`. Staged views use
  upstream's `StagedDiffHunkRenderer`, unstaged ones
  `UnstagedDiffHunkRenderer`; hunk staging operations belong to the
  `BufferDiff` since upstream PR 63556. A regression test pins that toggling
  a hunk in a section diff does not duplicate it.

## Snapshots

Immutable source anchors. Never merge them into `next`.

| Branch | Anchor | Source |
| --- | --- | --- |
| `pr/55404-detachable-items` | `f7321ff6c3993eeec93d51a4953c3c9421600d24` | zed-industries/zed PR 55404 |
| `pr/28674-tailwind-rust-completion` | `19a316e2b359c1bfaa2cff1af1c57581df76bd5d` (the feature commit; the branch tip later merged main) | zed-industries/zed PR 28674 |
| `pr/59884-group-by-staging` | `035c1b6378f4f285167c0236b7b4621932e72b53` | zed-industries/zed PR 59884, merged upstream 2026-07-11 |
| `pr/61067-word-diff-unequal-hunks` | `d829a6eb8e53a038a7af1a3adee278d2ca803a13` | zed-industries/zed PR 61067 |
| `external/firatoezcan-main` | `bd20394908006e3d206257239df57530b32418e0` | firatoezcan/zed `main` |
| `external/firatoezcan-git-ui-improvements` | `8ae6296bb0790506f53b3c2022429a9bb705b4d2` | firatoezcan/zed `autoresearch/git-ui-improvements-2026-04-04`, reference only |

`integration/46478-search-modal` and `pr/46478-search-modal` are retired;
upstream `ccf4058b7a` covers them. They stay in git as history only.

## Adding an integration

1. Snapshot the source and verify it is the PR head before anything else:

   ```sh
   git fetch upstream pull/<number>/head
   git switch -C pr/<number>-<short-name> FETCH_HEAD
   gh pr view <number> -R zed-industries/zed --json headRefOid   # must equal the tip
   git push --force-with-lease origin pr/<number>-<short-name>
   ```

2. Create `integration/<number>-<short-name>` from `next-base`. Import with
   `git merge --squash --no-commit pr/<number>-<short-name>`, resolve only
   the mechanical conflicts, and commit with the trailers below. Fork
   behavior goes in named follow-up commits.
3. Add the row above, the gate lines in `script/fork-refresh-gates`, and a
   watchlist entry for the retirement trigger.

Import commit trailers (no GitHub autolinks in the message; they create
timeline noise on the upstream PR):

```text
Upstream-repository: zed-industries/zed
Upstream-pull-request: <number>
Original-author: <github-login>
Snapshot: <raw-pr-head-sha>
```

External forks use `external/<owner>-<short-name>` as the snapshot and
`integration/<owner-or-feature>-<short-name>` as the adapted branch.

## Retiring an integration

Remove the row, its gate lines, and its watchlist entry; leave the branches
in git. Name the upstream commit that replaced it in the refresh report.
