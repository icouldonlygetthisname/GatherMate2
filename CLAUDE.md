# GatherMate2 (forked third-party addon)

Upstream addon by Nevcairiel. This repo is a real GitHub fork: `origin` = github.com/icouldonlygetthisname/GatherMate2, `upstream` = github.com/Nevcairiel/GatherMate2. It carries one local fix on `fix/secret-node-names`: guarding node-name lookups against Midnight secret values. The branch also has unreleased upstream `master` merged in for WoW Forever support. See README.md for what the fix does, the re-apply steps and the Forever build.

## How this differs from the other patched addons

- Upstream's own git history is the baseline. There are no `Baseline: ... (upstream, unmodified)` commits. Rebase or merge onto upstream tags instead of copying a game folder in.
- The game folder (`_retail_\Interface\AddOns\GatherMate2`) is a packaged build, not a copy of the repo: `Libs/` comes from `.pkgmeta`, `Storage/*` is split into `GatherMate2Storage_*` addons, and locale `@debug@` blocks and `@project-version@` are substituted. Never robocopy the repo over it. Copy only `Collector.lua` and `GatherMate2.lua`, after diffing them against the game copy with `diff --strip-trailing-cr` (the game files use CRLF).
- Since the merge of upstream `master` (`7a9c7b2`), `GatherMate2.lua` localizes node names itself (upstream `afb0916`), and `Constants.lua` no longer does. Only copy the two files into a retail game folder whose release already contains `afb0916` (`git tag --contains afb0916`). Into an older release (e.g. 1.52.3) the diff is more than the fix, so don't copy.
- The fix isn't marked with a `--Local fix`-style comment. To check whether it's in the game folder: `grep -n "issecretvalue(name)" GatherMate2.lua`.
- `GatherMate2_Data` (sibling folder) is a separate addon. Don't touch it from here.

## WoW Forever

- Forever (product `wowf`, interface `16001`, `WOW_PROJECT_ID == WOW_PROJECT_CAMELOT`) runs on the mainline client. Upstream handles it in the code; check its API with the `forever` branch of wow-ui-source (`git -C E:/VScode/_tools/wow-ui-source fetch --depth 1 origin forever:refs/remotes/origin/forever`).
- `_forever_\Interface\AddOns\GatherMate2` is not a CurseForge package. It is built from this repo's committed HEAD by `E:\VScode\_tools\build-forever.ps1 -Deploy`, which also builds Routes. Commit first; uncommitted changes aren't included.
- The build needs HereBeDragons-2.0 minor 35+ (Forever world map data). The script takes it from `_tools\HereBeDragons` (upstream master), not from the retail package (minor 33).
- `@localization@` tokens stay unsubstituted, so the build is English only. The storage addons are left out (they only load on the `standard` game type). `GatherMate2_Data` is left out too: Forever uses the Classic Era uiMapIDs (`1411`-`1458`, e.g. Elwynn `1429`), and the data addon has no nodes on them. Its vanilla zones are retail map IDs with post-Cataclysm spawns, so they can't be remapped.
- Once upstream releases a version with Forever support (`git tag --contains 6b4b54b`), prefer the official package.

## Upstream overlap

Upstream PR #64 (`26d58dd`) fixes the same error with an early return in `Collector:SpellStarted`. It is merged into this branch, but check `git tag --contains 26d58dd` to see whether it's released yet. Once it is, the `Collector.lua` part of the local fix is redundant. Keep the `GetIDForNode` guard.

## Rules

- Match upstream style: tabs, no `;`, `GatherMate:`/`Collector:` methods.
- Upstream CI runs luacheck (`.luacheckrc`). Keep new globals out of the code, or add them there. Run `E:\VScode\_tools\bin\luacheck.exe . --no-color -q` before committing (currently 0 warnings).
- The fork is public. Pushing branches to `origin` is fine. Opening PRs against `upstream` is outward-facing, so ask first.
