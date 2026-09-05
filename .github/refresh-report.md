# Refresh next from upstream 2026-09-05

## Upstream range

- Previous fork point: `5a9b9558db01a6b906cec2fb70a797affdc58cdd`
- Refreshed upstream tip: `5a9b9558db01a6b906cec2fb70a797affdc58cdd`
- Upstream commits considered: 0

## Decisions

Docs-only follow-up to the 2026-09-03 refresh; upstream did not move
(`5a9b9558db`) and the Rust tree of `next` is byte-identical to the one the
2026-09-03 gates and local build covered, so gates and the build were not
repeated.

### Watchlist changes

- `integration/git-ui-improvements` is renamed `file-history-navigation`. Its
  outside source (Firat Ozcan's April 2026 branch) is dead, only two pieces
  were taken and both were rewritten here, so it is a fork feature branch
  now; the manifest keeps the origin. Maciej keeps the view-file arrow on
  purpose (diff on click, file on the arrow) and the previous/next commit
  navigation has no upstream equivalent.

### Obsolescence

- Nothing retired.

### Conflicts resolved

- None.

## Gates

- Not run.

## Proposal branches

| Branch | Tip | Review compare |
| --- | --- | --- |
| `refresh/2026-09-05/next-base` | `0ef48db1a7` | [upstream/main...next-base](https://github.com/PiasekDev/zed/compare/5a9b9558db01a6b906cec2fb70a797affdc58cdd...0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29) |
| `refresh/2026-09-05/scroll-to-switch-tabs` | `5d325eb00e` | [next-base...scroll-to-switch-tabs](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...5d325eb00e8fb97c16fb5bde2c7059e79fc0dc5e) |
| `refresh/2026-09-05/integration/55404-detachable-items` | `7316109500` | [next-base...integration/55404-detachable-items](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...73161095002a070395a73cfdb12c33106ba53ae1) |
| `refresh/2026-09-05/file-history-navigation` | `bc1f07d284` | [next-base...file-history-navigation](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...bc1f07d2841d34221d5cda0e74dfb7b791111226) |
| `refresh/2026-09-05/integration/28674-tailwind-rust-completion` | `a1b0877ac5` | [next-base...integration/28674-tailwind-rust-completion](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...a1b0877ac54120cf5002b81542271b073508a9fa) |
| `refresh/2026-09-05/integration/59884-group-by-staging` | `584741a968` | [next-base...integration/59884-group-by-staging](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...584741a9688edb1324bb0864b8cde5297fef62e6) |
| `refresh/2026-09-05/integration/61067-word-diff-unequal-hunks` | `395d4079af` | [next-base...integration/61067-word-diff-unequal-hunks](https://github.com/PiasekDev/zed/compare/0ef48db1a7f2b2ce69b1192fe0fbfd8fb0dbfb29...395d4079afcc2ee0a2d2284ebb7bb4ed950437a1) |
| `refresh/2026-09-05/next` | `59f5da117a` | assembled |

The raw PR diff against the live next includes upstream churn; the compare
links above are the review artifact.
