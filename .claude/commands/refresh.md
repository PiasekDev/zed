---
description: Refresh the fork on current upstream, gate it, build it, publish it
---

Run a fork refresh of `PiasekDev/zed` end to end. Read `FORK_NEXT.md` first
(principles, runbook, authority), plus `NEXT_INTEGRATIONS.md` and
`UPSTREAM_WATCHLIST.md`. `$ARGUMENTS` may carry a `--date`; otherwise use
today. Work from the repository root on a clean tree, one heavy cargo job at a
time, and do not stop to ask: resolve every conflict yourself, keeping
upstream behavior plus the fork's intent.

1. `export FORK_REFRESH_COAUTHOR=""`, then `script/fork-refresh start`. On a
   conflict, read both sides, fix it, `git rebase --continue`, re-run `start`.
2. Judge, as the runbook's step 2 describes: check every retirement condition
   and watchlist entry against the upstream range, drop what upstream now
   carries, and commit manifest, watchlist, and `FEATURES.md` edits on the
   dated `next-base`. Re-run `start` afterwards.
3. `script/fork-refresh assemble`, then `script/fork-refresh gate`. Fix a
   failing gate on the integration branch that owns it as a follow-up commit,
   then start, assemble, and gate again.
4. `script/fork-refresh report`, fill in its Decisions section (watchlist
   changes, conflicts, anything dropped or adapted), then
   `script/fork-refresh report --commit`.
5. `script/package-zed-next-local --no-install` (about 30 minutes). Afterwards
   `git checkout -- packaging/arch/zed-next-local/PKGBUILD`, which makepkg
   rewrites. Maciej installs the package himself.
6. Write the summary as a page at `~/pages/zed/<date>-refresh.html`, following
   `~/.claude/skills/html-communication/SKILL.md`. Keep it short: the upstream
   range, what changed per integration, gate results, the build result, and
   the `sudo pacman -U <absolute path to the .pkg.tar.zst>` install command.
7. Publish: `script/fork-refresh promote-local`, then the push commands it
   prints, then the workflow disable sweep from `FORK_NEXT.md`, then
   `script/fork-refresh clean` to drop the dated branches. Push the fork's
   branches only, never upstream, and only after the gates and the local
   build passed.

Commits are yours: unsigned, authored by you, no co-author trailer. Fork
work never adds the README review marker (`.rules`, "Fork workflow"); never
remove one that is present.
