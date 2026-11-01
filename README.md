# GatherMate2 (local secret-name fix)

Fork of [GatherMate2](https://github.com/Nevcairiel/GatherMate2) by Nevcairiel, with one fix for Midnight secret values. Opening some objects (e.g. Mislaid Curiosity treasures in delves) passes a secret target name to `UNIT_SPELLCAST_SENT`. GatherMate2 then errors on every cast:

```
GatherMate2.lua:527: attempted to index a table that cannot be indexed with secret keys
[GatherMate2.lua]:527: in function 'GetIDForNode'
[Collector.lua]:222: in function '?'
```

## Fix (branch `fix/secret-node-names`, commit `fbe79df`)

Based on upstream `1.52.3-6` (`fc55711`).

- `Collector.lua`, `Collector:SpellStarted`: skip the name lookup when `target` is secret, and fall back to the tooltip lookup (`GetWorldTarget`) the same way an unknown name already does.
- `GatherMate2.lua`, `GatherMate:GetIDForNode`: return `nil` for a secret or `nil` name, or an unknown node type, instead of indexing with it. This protects every caller, not just the collector.

## Upstream status

Upstream merged a fix for the same error in [#64](https://github.com/Nevcairiel/GatherMate2/pull/64) (`26d58dd`, 2026-10-07). It returns early from `SpellStarted` when the target is secret, so that cast records nothing, not even from the tooltip. As of 2026-11-01 it is on upstream `master` but not in a tagged release (latest tag `1.52.3`). It merges cleanly with this branch.

Once a release includes #64, the `Collector.lua` change here is dead code. The `GetIDForNode` guard is still worth keeping.

## Workflow

The game folder holds a **packaged** build: `Libs/` comes from `.pkgmeta` externals, `Storage/*` is moved into separate `GatherMate2Storage_*` addons, and locale `@debug@` blocks and the TOC version are substituted. So never robocopy the whole repo over it.

After an upstream update (CurseForge etc.) replaces the game folder:

1. Check whether the fix survived: `grep -n "issecretvalue(name)" GatherMate2.lua` in the game folder.
2. `git fetch upstream`, then rebase `fix/secret-node-names` onto the release tag (or merge it in).
3. `luac -p Collector.lua GatherMate2.lua`.
4. Diff those two files against the game folder (`diff --strip-trailing-cr`). The only differences should be the fix, so you don't overwrite newer upstream code. Then copy just those two files in.
5. Update the commit hashes in this README.
