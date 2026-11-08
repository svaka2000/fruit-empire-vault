---
tags: [fruit-empire, project, mine, digging, eggs]
status: built
updated: 2026-11-08
code: ~/fruit-empire-mine/dig
handoffs: [20, 22, 23, 27]
---
# Mine digging, the cave-in, egg races and safe pockets

Back to [[Project Status Board]]. The mine itself is [[Mine]].

## What players get

- **Dig** into the shafts' walls and floor (not the hub). Each hit chips cubes out of the rock where the pick lands; stronger pickaxes break blocks faster. (STATE.md "22 Oct: digging, the cave-in and egg races"; PR #27)
- **A cave-in every 20 minutes:** rumble, shake and a warning, then the mine caves in. Anyone still inside dies under rubble, and everyone sees "MINES HAVE BEEN RESET". The hut doors shut from 1,170 s into the cycle.
- **Egg races:** an egg of a top-3 rarity (Astral, Eternal, Transcendent) is buried in a sealed pocket 16 to 30 studs into hard rock. Chat announces it; a plate shows how many studs away it is. Since 22 Oct evening the egg also glows through the rock in its rarity colour with a "!" over it. The first to dig in and hit it wins it, straight into the save (nothing to carry, so nothing to grab).
- **Egg pace now:** one egg at a time, the first 60 to 180 s into a cycle, the next 60 to 150 s after each is taken, none after 960 s. (PR #27)
- **Safe pockets (in progress, 8 Nov):** small refuges in the rock where a player rides out the cave-in. See below.

## Samarth's asks

- 22 Oct: dig the shafts, debris scaling by pickaxe, the 20-minute reset with rumble, a deadly cave-in, and "sometimes transcdental eggs can spawn in pockets that players will all race and rush to mine too ... it should all look extremely good be really creative and use claude design to your advantage." (`~/fruit-empire-mine/DIG-AND-RACE.md`)
- 22 Oct evening: the egg "will get highlighted with its rarity color and a exclamation will be above it. you can see the hihglight through the walls", and "keep the N STUDS plate too".
- 23 Oct: an egg in every cycle. 24 Oct: "make eggs spawning more common in the mines", and fix blocks that shrank instead of chipping.
- 8 Nov: "add safe zones like little pockets people can go into, make it super aesthetic and high quality." (`~/fruit-empire-mine/pockets/MinePocketBuilder.lua`)

Source for 22-24 Oct: memory `project_fruit_empire_mine.md`.

## Where it lives

- **In the place:** `MineDigGrid`, `GardenConfig.MineDig` (`Config.Dig`), `GardenService.MineDig`, `MineDigClient` (draws the studded blocks, aim, debris, headlamp), `MineCaveIn` (warning, cave-in and reset screens, the egg plate `GardenUI.MineEggChip`, chat lines). Changed: Weilin's `Mining`, `Mine`, `PickaxeClient`, `HudScale`, `GardenStore`, `Persistence`. Save field `eggs` (a list of `{animal, tier, source, at}`, at most 25). (STATE.md; memory `project_fruit_empire_mine.md`)
- **Source of truth for the dig scripts:** `~/fruit-empire-mine/dig/src` at commit e01f6d9. Change them there, run `tests/run.sh`, install with `tools/make_patch.py` steps. (queue note "Known links between chats' scripts")
- **Plan:** `~/fruit-empire-mine/DIG-AND-RACE.md`. **Canvas:** [dig boards](https://claude.ai/artifact/3coTyLchg2MKeqXr3Nx8ZG).
- **Test door:** `RoyalDev dig state|cycle|egg|near|wall|hit|take|reset`. Egg race recipe in [[Testing in Studio]].

| PR | What |
|---|---|
| [#20](https://github.com/svaka2000/fruit-empire-handoffs/pull/20) | Digging, cave-in, egg races |
| [#22](https://github.com/svaka2000/fruit-empire-handoffs/pull/22) | The egg glows through rock, with a "!" |
| [#23](https://github.com/svaka2000/fruit-empire-handoffs/pull/23) | An egg in every cycle (`Config.Dig.EggChance` 1) |
| [#27](https://github.com/svaka2000/fruit-empire-handoffs/pull/27) | Rock chips instead of shrinking; eggs one after another |

## Safe pockets (8 Nov 2026, work in progress)

> [!warning] Being built today; nothing here is confirmed in Studio
> Another chat held the Studio turn and was building this while this note was written. The facts below come from the builder file only.

- **Builder:** `ZZTools.MinePocketBuilder` (`run`, `undo`, `status`; Edit mode only). Local copy `~/fruit-empire-mine/pockets/MinePocketBuilder.lua` (its own git repo, no commits yet on 8 Nov).
- **What it makes:** `Workspace.Mine.SafePockets`, one Model per pocket. A hub pocket replaces one slab of the hub's wall ring with rock cut round a doorway and digs a small room behind it: rough studded rock, a plank floor, timber sets with lagging, an iron-strapped portal, a "SAFE POCKET" board, two caged green beacons, a lantern, a bench, crates, a barrel and a first-aid box. Cavern pockets stand in a rock outcrop of their own.
- **First two:** `SafePocket_HubLiftEast` (bearing 162) and `SafePocket_HubLiftWest` (bearing 210), either side of the Mine Lift.
- **Contract with the game:** each pocket has an invisible part named `SafeZone` (attribute `SafePocket = true`). Whoever's HumanoidRootPart is inside one at the cave-in lives. Parts named `Beacon` and `SafeLine` are lit by `MineCaveIn` during the warning. Each pocket carries `PocketName` for the map and a guide arrow. So `GardenService.MineDig` and `MineCaveIn` must change too.
- **Re-run it after any MineBuilder run**, which rebuilds the whole mine. The replaced wall slab waits in `ServerStorage.MinePockets_backup`.

## Open items

- Samarth's seven calls on digging are in DECISIONS.md "Waiting for a person".
- Not tested with two players racing, on a phone (frame rate in a heavily dug shaft), at the five screen sizes, or with a real save and load of `eggs`.
- Weilin's OK on his changed scripts (`Mining`, `Mine`, `PickaxeClient`) and on the new screens; both people's OK on the chipping look; Samarth on the egg pace and whether the map's egg sign makes the race too easy.
- `GardenUI.Toast` overlaps the top buttons' box by 5 px at 1529 x 886.
- Eggs don't hatch until pets land ([[Pets, Eggs and Fuse Machine]]).

Sources: [STATE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/STATE.md) "Open items"; memory `project_fruit_empire_mine.md`.

Related: [[Mine]], [[Map Panel]] (shows dug tunnels and the race egg), [[Pets, Eggs and Fuse Machine]].
