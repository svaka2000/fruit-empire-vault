---
tags: [fruit-empire, project, mine]
status: built
updated: 2026-11-08
code: ~/fruit-empire-mine
handoffs: [11, 12, 13, 14, 15, 16]
---
# Mine: pickaxes, ores and the cave

The shared mine under the island: shard ores, nine pickaxes, a shop and a forge, and the cave itself. Digging, the cave-in, egg races and safe pockets are in [[Mine Digging and Cave-in]]; the mine's purpose (geodes) is [[Crystal Fruit]]. Back to [[Project Status Board]].

## What players get

- Ride the Mine Lift down to the hub at Y -220. Mine studded shard ores with your pickaxe; they crack at 2/3 and 1/3 HP and burst into chunks, the shards flying into you. (STATE.md "21 Oct: shard ore models")
- **Nine pickaxes,** one per rarity, Stone Pick to Halo Pick, unlocked by rebirths, bought from **Gus** at the Pickaxe Shop, upgraded (Strength, Speed) at **Flint**'s Forge. **Pip** guides at the lift. Pickaxes and upgrades are kept through a rebirth. (STATE.md "22 Oct: pickaxes, Pickaxe Shop and Forge")
- **The cave (v3):** a taller hub with a real timber headframe, and three caverns at the shaft ends: the Gold Works, the Crystal Geode, the Underground River. (STATE.md "22 Oct: cave overhaul")

## History

- Weilin built the first mine after 9 Oct (MineBuilder v2). It was not written up in a handoff (STATE.md "Changed after 9 Oct").
- 20 Oct: Samarth asked for pickaxes "like the ones in the video" plus NPC models. On 22 Oct he answered MINE-PLAN section 8 with every default. (memory `project_fruit_empire_mine.md`)
- 21 Oct: he turned down smooth crystal ores for studded voxel ones. ([[Samarth's Taste]])
- 22 Oct, cave overhaul: "fixes all dark spots of the cave. you dont have to add more lanterns ... make the wooden structure 10x better make it super realistic ... make the cave bigger". (`~/fruit-empire-mine/CAVE-OVERHAUL.md`)

| PR | What |
|---|---|
| [#11](https://github.com/svaka2000/fruit-empire-handoffs/pull/11) | 9 pickaxe, 3 NPC and 2 stall models in `ServerStorage.MineModels` |
| [#12](https://github.com/svaka2000/fruit-empire-handoffs/pull/12) | 12 studded shard ore models |
| [#13](https://github.com/svaka2000/fruit-empire-handoffs/pull/13) | Ores use the models; cracks, break, shards fly in |
| [#14](https://github.com/svaka2000/fruit-empire-handoffs/pull/14) | A near-white light over each ore's crystals |
| [#15](https://github.com/svaka2000/fruit-empire-handoffs/pull/15) | Pickaxe Tools, Pickaxe Shop, Forge, `MineMenu` |
| [#16](https://github.com/svaka2000/fruit-empire-handoffs/pull/16) | MineBuilder v3 (Timber, Caverns), the light fix |

## Where it lives

- **In the place:** `Workspace.Mine` (built by `ServerStorage.ZZTools.MineBuilder` v3), `Workspace.Mine.MineShop`, `ServerStorage.MineModels` (sources), `ServerStorage.Pickaxes` + `ReplicatedStorage.PickaxeModels`, `GardenService.Mining` (Weilin's), `GardenService.Pickaxes`, `GardenConfig.MinePickaxes`, `GardenConfig.MineOreLook`, `ReplicatedStorage.MineOreKit`, client `OreClient`, `PickaxeClient`, `MineMenu`, `MineNPCs`. Save field `pickaxes`. (PLACE.md; memory `project_fruit_empire_mine.md`)
- **Local:** `~/fruit-empire-mine` holds the plans (`MINE-PLAN.md`, `MINE-PURPOSE.md`, `CAVE-OVERHAUL.md`, `PICKAXE-ECONOMY.md`) and sub-repos, each its own git: `code/` (pickaxes, shop, forge), `cave/` (v3 builder, before/after, shots), `crystal/`, `dig/`, `pockets/`. Models: `~/fruit-empire-mine/models/tools` (pickaxes, NPCs, props, ores, all voxel builders run in Blender).
- **Canvases:** [cave boards](https://claude.ai/artifact/LfLzAVMycG4NvNpiRVK3VA). Palette: [[Colour Palettes]] (mine section). Map: [[Workspace Map]] (mine section). Builder traps: [[Builders and ZZTools]].

## Open items

- **Ores never get tougher,** so pickaxes from about the Comet Pick up one-hit almost anything. Tougher tunnel ores are "phase 2", not built. (STATE.md "Open items")
- Prices and stats are first guesses with no economy check. `~/fruit-empire-mine/PICKAXE-ECONOMY.md` exists; whether its numbers were applied is not recorded. `[unclear]`
- Up to 35 ore lights plus 59 cave lights: check the frame rate on a phone before publishing.
- 28 ores at once in a cave twice as big: Samarth to say if it feels empty.
- Weilin's OK on MineBuilder v3, `Mining` changes and both panels.
- Not tested with two players or on a phone. Delete the backups (`Mine_backup_2026-10-22`, `Backup_2026-10-22_before_cave_overhaul`, `Backup_2026-10-22_before_pickaxe_shop`) only after both partners accept.

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "Open items"; memory `project_fruit_empire_mine.md`.

Related: [[Mine Digging and Cave-in]], [[Crystal Fruit]], [[Map Panel]].
