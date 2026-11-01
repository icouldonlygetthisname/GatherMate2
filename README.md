# GatherMate2 (local secret-name fix + WoW Forever build)

Fork of [GatherMate2](https://github.com/Nevcairiel/GatherMate2) by Nevcairiel, with one fix for Midnight secret values. Opening some objects (e.g. Mislaid Curiosity treasures in delves) passes a secret target name to `UNIT_SPELLCAST_SENT`. GatherMate2 then errors on every cast:

```
GatherMate2.lua:527: attempted to index a table that cannot be indexed with secret keys
[GatherMate2.lua]:527: in function 'GetIDForNode'
[Collector.lua]:222: in function '?'
```

The branch also has upstream `master` merged in (`7a9c7b2`), for WoW Forever support that isn't released yet. See [WoW Forever](#wow-forever).

## Fix (branch `fix/secret-node-names`, commit `fbe79df`)

Originally based on upstream `1.52.3-6` (`fc55711`). Now merged with upstream `master` at `da3567d`.

- `Collector.lua`, `Collector:SpellStarted`: skip the name lookup when `target` is secret, and fall back to the tooltip lookup (`GetWorldTarget`) the same way an unknown name already does.
- `GatherMate2.lua`, `GatherMate:GetIDForNode`: return `nil` for a secret or `nil` name, or an unknown node type, instead of indexing with it. This protects every caller, not just the collector.

## Upstream status

Upstream merged a fix for the same error in [#64](https://github.com/Nevcairiel/GatherMate2/pull/64) (`26d58dd`, 2026-10-07). It returns early from `SpellStarted` when the target is secret, so that cast records nothing, not even from the tooltip. It is merged into this branch, and the two checks sit side by side without conflict. As of 2026-11-01 it is on upstream `master` but not in a tagged release (latest tag `1.52.3`).

Once a release includes #64, the `Collector.lua` change here is dead code. The `GetIDForNode` guard is still worth keeping.

## Workflow (retail)

The game folder holds a **packaged** build: `Libs/` comes from `.pkgmeta` externals, `Storage/*` is moved into separate `GatherMate2Storage_*` addons, and locale `@debug@` blocks and the TOC version are substituted. So never robocopy the whole repo over it.

After an upstream update (CurseForge etc.) replaces the game folder:

1. Check whether the fix survived: `grep -n "issecretvalue(name)" GatherMate2.lua` in the game folder.
2. `git fetch upstream`, then rebase `fix/secret-node-names` onto the release tag (or merge it in).
3. `luac -p Collector.lua GatherMate2.lua`.
4. Diff those two files against the game folder (`diff --strip-trailing-cr`). The only differences should be the fix, so you don't overwrite newer upstream code. Then copy just those two files in.
5. Update the commit hashes in this README.

Because the branch now contains unreleased upstream code, step 4 only works once the installed release contains upstream `afb0916` (`git tag --contains afb0916`). That commit moved node-name localization from `Constants.lua` into `GatherMate2.lua`. Copying the new `GatherMate2.lua` over 1.52.3 would localize names twice on non-English clients.

## WoW Forever

WoW Forever (product `wowf`, interface `16001`) runs on the mainline client. Upstream already supports it in the code (`6b4b54b`, `7da8088`, `ec3a015`, `afb0916`): the TOC lists `16001`, options for other gather types are hidden, spells the client lacks are skipped, and the old vanilla node names are accepted. Comparing the API documentation of the `live` and `forever` branches of wow-ui-source shows only 7 functions/events missing on Forever. GatherMate2 uses none of them.

There is no upstream Forever release yet, so the copy in `_forever_\Interface\AddOns\GatherMate2` is built locally:

```powershell
E:\VScode\_tools\build-forever.ps1 -Deploy
```

The script builds both GatherMate2 and Routes from each repo's committed HEAD:

- Exports the repo with `git archive`, then drops `Storage/` and the dev files.
- Adds `Libs/` from the retail package, then replaces `Libs/HereBeDragons` with upstream master from `_tools\HereBeDragons`. The retail package carries HereBeDragons-2.0 minor 33, which has no Forever world map data. Minor 35 has it.
- Sets the TOC version to `<git describe>-forever` and checks that every TOC file exists.
- With `-Deploy`, moves any existing Forever copy to `%TEMP%\forever-backup\` and installs the new one. Without it, only builds into `%TEMP%\forever-build`.

Limits of the local build:

- `@localization@` tokens aren't substituted, so it is English only (the Forever client here is enUS).
- The `GatherMate2Storage_*` addons are left out. They only load on the `standard` game type.
- `GatherMate2_Data` isn't installed, so the Forever database starts empty and fills as you gather. The data addon has nothing for Forever: Forever uses the Classic Era uiMapIDs (Elwynn Forest `1429`, The Barrens `1413`), and none of its ~240,000 nodes are on those maps. Its vanilla-zone data uses retail map IDs and post-Cataclysm spawn points, so remapping the IDs wouldn't make it correct. Its TOC doesn't list `16001` either, so the Import page shows it as unavailable.

Once upstream tags a release that contains `6b4b54b`, use the official package instead.
