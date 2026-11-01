# GatherMate2 (forked third-party addon)

Upstream addon by Nevcairiel. This repo is a real GitHub fork: `origin` = github.com/icouldonlygetthisname/GatherMate2, `upstream` = github.com/Nevcairiel/GatherMate2. It carries one local fix on `fix/secret-node-names`: guarding node-name lookups against Midnight secret values. See README.md for what the fix does and the re-apply steps.

## How this differs from the other patched addons

- Upstream's own git history is the baseline. There are no `Baseline: ... (upstream, unmodified)` commits. Rebase or merge onto upstream tags instead of copying a game folder in.
- The game folder (`_retail_\Interface\AddOns\GatherMate2`) is a packaged build, not a copy of the repo: `Libs/` comes from `.pkgmeta`, `Storage/*` is split into `GatherMate2Storage_*` addons, and locale `@debug@` blocks and `@project-version@` are substituted. Never robocopy the repo over it. Copy only `Collector.lua` and `GatherMate2.lua`, after diffing them against the game copy with `diff --strip-trailing-cr` (the game files use CRLF).
- The fix isn't marked with a `--Local fix`-style comment. To check whether it's in the game folder: `grep -n "issecretvalue(name)" GatherMate2.lua`.
- `GatherMate2_Data` (sibling folder) is a separate addon. Don't touch it from here.

## Upstream overlap

Upstream PR #64 (`26d58dd`) fixes the same error with an early return in `Collector:SpellStarted`. Check `git tag --contains 26d58dd` to see whether it's released yet. Once it is, the `Collector.lua` part of the local fix is redundant. Keep the `GetIDForNode` guard.

## Rules

- Match upstream style: tabs, no `;`, `GatherMate:`/`Collector:` methods.
- Upstream CI runs luacheck (`.luacheckrc`). Keep new globals out of the code, or add them there. Run `E:\VScode\_tools\bin\luacheck.exe . --no-color -q` before committing (currently 0 warnings).
- The fork is public. Pushing branches to `origin` is fine. Opening PRs against `upstream` is outward-facing, so ask first.
