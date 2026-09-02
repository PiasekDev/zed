# Upstream watchlist

What upstream may absorb or has partly absorbed. Each refresh checks these
against the upstream range and records state changes in the proposal report.
Remove an entry when its fork action is done.

## Retirement triggers

| Watch | State (2026-09-03) | Fork action on change |
| --- | --- | --- |
| Wheel tab switching upstream | None known | Retire `scroll-to-switch-tabs` |
| [PR 55404](https://github.com/zed-industries/zed/pull/55404) move item to new window | Open, last activity 2026-08-25 | Re-evaluate `integration/55404-detachable-items` against the upstream implementation |
| Inline view-file button and previous/next file-history navigation | Missing upstream. Upstream has file history views (PRs 52634 and 56500 merged), a `file_filter` on `CommitView`, and a Git Graph view (`crates/git_ui/src/git_graph.rs`) | Retire `integration/git-ui-improvements` when both land; narrow it as pieces land |
| Rust registered for Tailwind completions | Missing upstream | Retire `integration/28674-tailwind-rust-completion` |
| Section-aware per-file solo diffs | Missing upstream. Grouping and section stats landed (PRs 59884, 60976, 60815); hunk operations moved onto `BufferDiff` (PR 63556, 2026-09) | Retire `integration/59884-group-by-staging` when upstream opens section-base diffs with the same stat semantics |

## Re-check each refresh

| Area | Why |
| --- | --- |
| Git panel multi-select with bulk stage, unstage, and discard (PR 60340, 2026-08) | Same panel code as the staging integration; watch for regressions in grouped sections |
| Hunk-level stage and unstage reliability | Flagged as possibly buggy upstream (2026-06); re-test after each refresh that touches `git_ui` |
| Upstream tab features in `pane.rs` (file permalinks, PR 62177) | Shares the import lines that `scroll-to-switch-tabs` touches; the usual rebase conflict is there |

## Intended for upstream later

Features Maciej wants in upstream, not as long-term fork patches. No upstream
PR until he has personally tested and approved the code.

| Item | Status |
| --- | --- |
| `scroll-to-switch-tabs` | Shipped in the fork; wants a fresh-angle rework first |
| Rename detection in git status and staged views | Not started; upstream status paths still use `--no-renames` |
| Jump-to-source from diff views (VS Code `git.openFile` parity) | Planned |
| Word-level diff highlighting, commit list per branch, review comments, AI rename suggestions | Planned 2026-09-03; see the plan page under `~/pages/zed/` |
| Commit-message style few-shot | Parked unless `agent.commit_message_instructions` proves insufficient |
