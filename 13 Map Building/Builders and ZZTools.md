---
tags: [fruit-empire, builders, tools, studio]
status: current
updated: 2026-11-08
---
# Builders and ZZTools

A **builder** is an Edit-mode tool script that builds or changes part of the world. The game never runs it. Most live in `ServerStorage.ZZTools`; some live on Samarth's computer and are pasted into `execute_luau`. Back to [[Knowledge Base - Start Here]].

> [!note] The shared list
> [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Builders" lists which tools in ZZTools have `undo()` and which do not. Read it before running any of them.

## The builder contract

Every new builder follows this:

1. **`run()`, `undo()`, `status()`.** `status()` changes nothing. Requiring the module does nothing by itself.
2. **Refuses outside Edit mode and outside place 88602028667228.**
3. **Moves what it replaces into a dated backup folder** in ServerStorage, so `undo()` can put it back exactly.
4. **Builds everything unparented and checks first.** A failed check changes nothing.
5. **Seeded and re-runnable.** Running it again gives the same result.

Why re-runnable matters: everything done through the MCP lives only in the open Studio session until someone saves. On 29 Sep a Studio restart rolled the world back mid-build; only seeded, re-runnable builders saved the day. After any big build, ask Samarth to save. (memory `reference_roblox_studio_mcp.md`, 29 Sep)

Sources: the header of `~/fruit-empire-mine/pockets/MinePocketBuilder.lua` (8 Nov) states the full contract; PLACE.md "Builders".

## Calling a builder

Studio's command bar keeps the first `require` of a module for the whole session. So always call a tool through a fresh copy (from PLACE.md):

```lua
local SS = game:GetService("ServerStorage")
local m = SS.ZZTools.SkyGardenBuilder:Clone(); m.Parent = SS.ZZTools
local T = require(m); m:Destroy()
print(T.status())   -- or T.run() / T.undo()
```

- Run one builder at a time. Undo several in the reverse order they ran.
- Say in the handoff which builders ran.

> [!danger] Builders that act the moment they are required
> `BeanstalkCrown`, `BuildExplosives`, `BuildShopTabs`, `BuildUIKitPrefabs`, `SkyIslandBuilder`, `SkyIslandDecor` have no `undo()`. Never require one unless you mean to rebuild that thing. `SkyIslandBuilder` would bring back the King's Throne version of the cloud. Never run `BorderMountainsBuilder.run()` again unless Samarth asks. (PLACE.md)

## MineBuilder v3 and its children

- `ServerStorage.ZZTools.MineBuilder` is Weilin's builder; v3 since the cave overhaul (22 Oct, [PR #16](https://github.com/svaka2000/fruit-empire-handoffs/pull/16)). It builds `Workspace.Mine` with two child modules: **`Timber`** (headframe, drift sets, portals, footbridge, carts and track) and **`Caverns`** (Gold Works, Crystal Geode, Underground River).
- `run()` keeps the old mine in `ServerStorage.Mine_backup_2026-10-22` (only if that folder does not exist yet) and copies `Workspace.Mine.MineShop` into the new mine. `undo()` puts the backup back. (PLACE.md)
- **Traps found while building it** (memory `project_fruit_empire_mine.md`, 22 Oct):
  - A fresh-copy `require` destroys the module's children, so v3 loads `Timber` and `Caverns` while it loads.
  - Every random draw shifts everything built after it. A late fix therefore has to be a post-process edit (delete a post, split a rail), not one more random draw.
  - Asset Pack V1's "Rock 1" and "Rock 2" are Parts, not Models, so `ScaleTo` fails on them.
- Source copies: `~/fruit-empire-mine/cave/v3/` (`MineBuilder.lua`, `.Timber.lua`, `.Caverns.lua`), with before/after shots in `~/fruit-empire-mine/cave/`.
- **`MinePocketBuilder`** (8 Nov, work in progress) adds the safe pockets to `Workspace.Mine.SafePockets`. Because MineBuilder rebuilds the whole mine, **re-run MinePocketBuilder after any MineBuilder run.** (`~/fruit-empire-mine/pockets/MinePocketBuilder.lua` header)

## Other builders worth knowing

| Builder | What it does | Where |
|---|---|---|
| `ZZTools.RebirthPanelBuilder` | Moved the Rebirth panel's own pieces into layout A; saves old values as `Was<Property>` attributes | in the place ([[Rebirth Panel]]) |
| `ZZTools.RobuxPageBuilder` | The shop's Robux page | in the place ([[Robux Shop]]) |
| `ZZTools.GuiFitCheck` | Checks a panel fits at the five screen sizes | in the place |
| `build_bat_tools.luau` | Makes the 9 bat Tools and hit effects from `ServerStorage.BatModels`; its LOOK table dims them for the place's light | `~/fruit-empire-bats/code/tools/` ([[Nine Bats]]) |
| `build_pickaxe_tools.luau`, `place_mine_shop.luau` | The 9 pickaxe Tools; the stalls and NPCs in the hub | `~/fruit-empire-mine/code/tools/` ([[Mine]]) |
| `lineup_builder.luau` | The 45-animal display under the map | `~/fruit-empire-pets/forest-animals/tools/` ([[Forest Animals]]) |
| `PlotPenLayout` + `PlotPenDecor` | Every pen and its castle level; pens rebuild, so change the decor module, never a pen by hand | in the place ([[Plot Castle]]) |

The local builders are pasted into `execute_luau` with `MODE = "status"`, `"run"` or `"undo"` on the first line. (PLACE.md)

## Pasting a big builder safely

Put the file inside a `[===[ ... ]===]` string, compare its length and fingerprint with the local file, and only then `loadstring` it with `MODE` set. A one-character slip then refuses instead of running. (memory `reference_roblox_studio_mcp.md`, 22 Oct) For whole scripts, see [[Deploy Method]].

## Placing things well

- Map the room first with `GetPartBoundsInBox` grids at player height plus a list of the walls; screenshots alone misread doors and stairs. After placing, check every part with `GetPartsInPart`. Move clashing decor to a dated backup instead of deleting it. (memory `reference_glb_to_roblox_parts.md`)
- Raycasts only hit parts that are already in the workspace, so parent a new model before snapping things onto it. (memory `reference_roblox_studio_mcp.md`, 29 Sep)
- Move a model with `PivotTo`, never by setting a Part's `CFrame` (welded children stay behind). (memory `reference_roblox_studio_mcp.md`, 6 Oct)

Related: [[Workspace Map]], [[Deploy Method]], [[Studio MCP Traps]].
