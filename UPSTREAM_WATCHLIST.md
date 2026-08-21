# Upstream Watchlist

Tracked upstream PRs, issues, and feature areas that affect this fork. The
refresh process reads this file and reports state changes in each proposal.
Update entries when their state changes; remove entries when the linked
retirement or follow-up action has been completed.

## Retirement triggers

Items that retire fork branches when they land upstream.

| Item | State to watch | Fork action when it changes |
| --- | --- | --- |
| Section-aware per-file solo diffs | Missing upstream as of 2026-08-21 | Retire `integration/59884-group-by-staging` only when upstream preserves staged `HEAD -> index`, unstaged `index -> worktree`, combined totals/fallbacks, and section-stat-only remote updates |
| [PR #55404](https://github.com/zed-industries/zed/pull/55404) move item to new window | Open (stale) | On merge or equivalent feature: re-evaluate `integration/55404-detachable-items` against the upstream implementation |
| Wheel tab switching upstream | No upstream work known | On appearance: retire `scroll-to-switch-tabs` |

## Feature areas to re-check each refresh

| Item | Why it is watched |
| --- | --- |
| [PR #59884](https://github.com/zed-industries/zed/pull/59884) and [issue #26560](https://github.com/zed-industries/zed/issues/26560) | Closed upstream; section grouping and stats landed, but per-file section bases and three correctness details remain fork-only |
| [Issue #59761](https://github.com/zed-industries/zed/issues/59761) Git panel history commit-diff actions | Closed by PR #60807, which fixed header context-menu identities only; the fork's previous/next file-history navigation remains distinct |
| Hunk-level stage/unstage reliability | Flagged as possibly buggy upstream (2026-06-20 session); re-test after each refresh that touches git_ui |
| Upstream file/folder history views (#52634, #56500) | May grow to cover the fork's file-history navigation |

## Revisit later, intended for upstream

Features Maciej wants to build or polish for upstream itself, not carry as
fork patches long-term. Do not open upstream PRs for these until he has
personally tested and approved the code.

| Item | Status | Notes |
| --- | --- | --- |
| Rename detection in git status/staged views | Not started | Upstream still uses `--no-renames` in status paths; needs status pipeline + proto + panel work. Revisit and build upstream-first when prioritized |
| `scroll-to-switch-tabs` | Shipped in fork | Oldest custom patch; Maciej wants a fresh-angle rework before proposing upstream |
| Jump-to-source from diff views | Planned | VS Code `git.openFile` parity; ship in fork first, upstream once the code is liked |
| Commit-message style few-shot (Tier 1) | Parked | Only if `agent.commit_message_instructions` (Tier 0) proves insufficient |
