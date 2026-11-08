---
tags: [fruit-empire, map, world, reference]
status: current
updated: 2026-11-08
---
# Workspace map

Where everything is in the world, with heights. Roblox measures in studs; Y is up. Back to [[Knowledge Base - Start Here]].

> [!note] The shared table
> [PLACE.md](https://github.com/svaka2000/fruit-empire-handoffs/blob/main/PLACE.md) "Where things live" is the shared list of folders. This note adds coordinates and heights collected from the chats' notes. The 4 Oct picture tour is [[World Tour]] (out of date for the throne and the mountains).

## Heights, top to bottom

| Height (world Y) | What |
|---|---|
| 443 | Cloud Garden deck (top of the beanstalk) |
| 437 | The beanstalk's tip, at (118, 437, 2) |
| 374.6 | Underside of the Sky Islands: the Mega Rebirth stack must stay under it |
| 220 | Thin air starts (`Config.AltitudeFadeStart`); a fall of more than 220 studs kills (`FallKillDrop`) |
| 208.85 | Floor 14 of a Mega plot, the highest walking floor |
| about 2 | The forest floor (flat) |
| -130 | Below this, `Config.Mine.Bounds` counts you as in the mine |
| -220 | The mine hub's floor |
| -800 | The test baseplate, `Workspace["baseplate slop"]` (top of `Baseplate`) |

Sources: memory `project_fruit_empire.md` (3 Oct sky island entries), `project_fruit_empire_mega_rebirth.md`, `project_fruit_empire_pets.md`; `~/fruit-empire-mine/CAVE-OVERHAUL.md`; PLACE.md.

## Main Island: `Workspace["Main Island"]`

- **Plots:** `Plots.Plot0` to `Plot11` in a ring about 200 studs from the centre, with the beanstalk in the middle. Each plot is a walled garden (pen) on its own lot, with the owner's name on the gate, a stone hut, and the PLOT SIZE sign beside the gate. (PLACE.md; [[World Tour]]; STATE.md M6)
- **The plaza** is under the beanstalk's base (trunk radius about 60 to 80). Open ground is the sand ring at radius 129 to 173, between the grass disc and the plots. (memory `project_fruit_empire.md`, 3 Oct)
- **Mega floors** stack on the plot: floor k walks at S + 16 x (k - 1), where S is the soil's top. The beanstalk crosses over some plots, so Plot1 holds 3 floors, Plot3 6, Plot2 13, the rest 14 (`MaxFloors` on each plot). Each lot's `Lot.Tree3` moved 27 studs toward the plaza for the Sky Keep's stair tower. (STATE.md "25 to 29 Oct: Mega Rebirth")
- Also: `Paths`, `Mountains` (blocky grey peaks; the M7 replacement ring was undone on 8 Oct), `Shallows`, `Water`, `SeaRoute` (16 rest islets and 12 buoys for swimmers), `Forest` (with an invisible `BoundaryWall`). (PLACE.md)
- `workspace.CrateSpots`: where the Golden Crate can land. `Workspace.BatSafeZones`: boxes where bats can't hit. (PLACE.md; STATE.md "Open items")

## The forest

A flat floor at about Y 2 beyond the sea, with 18 clearings at radius 1,205 to 1,378 and 3,807 tree trunks. The camp and a TamerBooth are in Clearing09. The pets design puts animal nests in the clearings, rarer deeper in. (memory `project_fruit_empire_pets.md`; `~/fruit-empire-pets/ref/clearings.json`, `trunks.json`)

## The sky: `Workspace["Sky Islands"]["Cloud Garden"]`

- Deck at 443. Footprint about x -410 to 482, z -418 to 420, with an opening of radius 38 at (124, 0) where the beanstalk comes through. (memory `project_fruit_empire.md`, 3 Oct night)
- `SkyPuffs` and `SkyMushrooms` (bounce mushrooms); Sky Stars places in `ReplicatedStorage.SkyGarden`. See [[Sky Stars]].
- The `Castle` (attribute `SolidGround`) holds the `AetherFuseMachine` at the west end of its courtyard. (PLACE.md)
- **Sky ground:** a model under Sky Islands whose name is in `Config.SkyModels` lets you stand up high without dying. Names are case-sensitive; a typo once made a death zone. (PLACE.md "Gotchas")
- `Workspace.beanstalk.Crown`: the stalk's top above the cloud, rebuilt 18 Oct. (STATE.md "18 Oct: other changes")

## The mine: `Workspace.Mine`

- **The hub:** a round cave 100 studs across, floor at Y -220, roof 64 studs up since v3 (was 44). The Mine Lift is on the south wall; you land at (0, -216.5, 30.5) facing north. A timber headframe stands in the middle. (`~/fruit-empire-mine/CAVE-OVERHAUL.md`)
- **Stalls:** `Workspace.Mine.MineShop` (tagged `MineStall`): Gus's Pickaxe Shop at (12, -220, 27), Flint's Forge at (-13, -220, 27), Pip the guide at (-4.5, -220, 25). (CAVE-OVERHAUL.md; PLACE.md)
- **Three shafts**, 14 wide and 12 tall: East, North, West. Each opens into a cavern since v3: the Gold Works (east), the Crystal Geode (north), the Underground River (west). (CAVE-OVERHAUL.md section 5)
- 492 ore spots in v3, 28 ores alive at once. (memory `project_fruit_empire_mine.md`)
- **Safe pockets** (in progress, 8 Nov): `Workspace.Mine.SafePockets`, first two in the hub ring either side of the lift. See [[Mine Digging and Cave-in]].
- Built by `MineBuilder` v3; see [[Builders and ZZTools]].

## Under everything

- `Workspace["baseplate slop"]`: the test baseplate at Y -800, holding `ForestAnimalsLineup` (all 45 animals). Samarth wants it kept at publish. (PLACE.md; [[Forest Animals]])
- `Workspace._AssetViewer` at (6000, 1000, 6000): a temporary viewer of Creator Store packs. **Delete it before the next publish.** (STATE.md "Open items")

## Things to remember when building

- **Streaming is on.** Before teleporting a player from the client, call `player:RequestStreamAroundAsync(target)`. Big far scenery can be set to `ModelStreamingMode = Persistent`. (PLACE.md; memory `reference_roblox_studio_mcp.md`)
- **A teleport from the cloud deck to the ground counts as a 400-stud fall** and kills you. (PLACE.md)
- **The map pictures are drawings of the world** (21 Oct; the mine 24 Oct). Any world change needs the map redrawn and new image ids. ([[Map Panel]])
- Bat safe zones: the 4 Oct design planned them over every plot (and 240 studs below, for a mine under the plots that was never built that way). One box in `Workspace.BatSafeZones` may now sit in empty sky where the Royal Capital's arrival pad was. Not checked since 9 Oct. (old [[Decisions Log]] D-014, "Proposed"; STATE.md "Open items")

Related: [[Builders and ZZTools]], [[Mine]], [[Mega Rebirth]].
