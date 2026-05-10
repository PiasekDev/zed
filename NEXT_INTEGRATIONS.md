# Next Integration Manifest

This file is the source of truth for rebuilding the `next` branch from
`next-base`. Keep it current whenever a custom patch branch or upstream PR
snapshot is added to or removed from `next`.

## Rebuild Order

Start from `next-base`, then merge entries in this order:

1. Custom patch branches
2. Upstream PR snapshot branches

```sh
git switch -C next next-base
git merge --no-ff scroll-to-switch-tabs -m "Merge scroll-to-switch-tabs into next"
git merge --no-ff pr/46478-search-modal -m "Merge upstream PR 46478 into next"
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

## Upstream PR Snapshot Branches

| Branch | Upstream PR | Upstream Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |
| `pr/46478-search-modal` | [zed-industries/zed#46478](https://github.com/zed-industries/zed/pull/46478) | `ozacod` | `54e639eab83a9a2315be9a68d084c94effa68001` | Adds a search modal for project-wide text search. | `git merge --no-ff pr/46478-search-modal -m "Merge upstream PR 46478 into next"` |

### PR #46478 Integration Notes

This PR is old relative to current `upstream/main`, so it requires local
integration work when merged:

- Resolve `crates/search/Cargo.toml` and `Cargo.lock` by keeping both
  `smol` and `text` dependencies for the `search` crate.
- In `crates/search/src/quick_search/delegate/picker_impl.rs`, handle the newer
  transient `SearchResult::WaitingForScan` and `SearchResult::Searching`
  variants by ignoring them in the quick-search result loop.

After applying the merge, run:

```sh
cargo check -p search
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

Merge the snapshot into `next`:

```sh
git switch next
git merge --no-ff pr/<number>-<short-name> -m "Merge upstream PR <number> into next"
```

Add an entry to the table above:

| Branch | Upstream PR | Upstream Author | Snapshot Commit | Purpose | Merge Command |
| --- | --- | --- | --- | --- | --- |

Use the upstream PR author from GitHub, not the local committer who imported the
branch. If a PR has multiple important authors, list them all.

## Notes For Future Agents

- Read this file and `FORK_NEXT.md` before changing `next`.
- Do not infer integrated patches only from the git graph; update this manifest
  when the intended build set changes.
- Keep upstream PR branches as merge commits in `next` so they can be reverted
  or replaced cleanly.
- Keep custom patch branches rebaseable and named directly, without a prefix.
