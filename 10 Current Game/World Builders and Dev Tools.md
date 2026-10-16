---
tags: [current-game, code-map]
date: 2026-10-04
sources:
  - ServerStorage.ZZTools.RoyalCapitalBuilder.lua
  - ServerStorage.ZZTools.SkyIslandBuilder.lua
  - ServerStorage.ZZTools.SkyIslandDecor.lua
  - ServerStorage.ZZTools.BeanstalkCrown.lua
  - ServerStorage.ZZTools.ExportParts.lua
  - ServerStorage.CloudBuilder.lua
  - ServerStorage.ForestBuilder.lua
  - ServerStorage.ShallowsBuilder.lua
  - ServerStorage.SaturationPass.lua
  - ServerStorage.DevPanel.Server.lua
  - ServerStorage.DevPanel.Client.lua
  - ServerScriptService.GardenService.DevPanel.lua
  - ServerScriptService.GardenService.lua (context only)
  - ServerScriptService.GardenStore.lua (context only)
  - ServerScriptService.GardenService.Crown.lua (context only)
  - ServerScriptService.GardenService.CrownArena.lua (context only)
  - ReplicatedStorage.GardenConfig.lua (context only)
  - ReplicatedStorage.Fruits.FruitIndex.lua (context only)
---
# World Builders and Dev Tools

Part of [[Fruit Empire - Home]]. Source for the planned features: [[Brain Dump 2026-10-04]].
All code is from the export in `~/fruit-empire/src-export-2026-10-04`. Citations use the export's file names.

Terms used below:
- **Builder**: a ModuleScript (a script that runs when another script calls `require` on it) that creates map parts. Someone runs it by hand from the Studio command bar. The game never runs it.
- **Edit mode**: Studio when the game is not being play-tested.
- **Seed**: a fixed number given to the random generator, so a re-run places everything in the same spot.
- **Attribute**: a named value stored on an instance (a part, model or player).
- **Tag**: a CollectionService label on an instance, so code can find "every part tagged X".
- **Marker**: an invisible part with a known name, so game code can find a position.
- **Remote**: a RemoteEvent, the way the server and one client send messages to each other.

## 1. What it does

This subsystem is two things. The first is nine offline tools in ServerStorage that build or restyle the map: the sky island, its decorations, the beanstalk's top, the Royal Capital, the forest ring, the stepping stones in the sea, a cloud-shape library, a colour pass, and a part exporter for Blender renders. None of them run in a live server; the forest and shallows builders say so in their headers (`ServerStorage.ForestBuilder.lua:2-4`, `ServerStorage.ShallowsBuilder.lua:4-5`). The second is the developer panel: a debug readout and a set of save-system test buttons that only one account receives (`ServerStorage.DevPanel.Server.lua:36-38`). The panel is the only part of this subsystem that runs during play. It starts with GardenService (`ServerScriptService.GardenService.lua:763-768`).

## 2. Scripts

| Script | Role | Lines |
|---|---|---|
| ServerStorage.ZZTools.SkyIslandBuilder.lua | Builds the Cloud Garden sky island's cloud: slabs, underside, drifting pieces (v3) | 660 |
| ServerStorage.ZZTools.SkyIslandDecor.lua | Dresses the Cloud Garden: castle, flags, night lamps, cherry trees, bushes, flowers, cloud bumps | 555 |
| ServerStorage.ZZTools.BeanstalkCrown.lua | Builds the top of the beanstalk (stem, climbing leaves, lookout) up through the cloud | 199 |
| ServerStorage.ZZTools.RoyalCapitalBuilder.lua | Builds the Royal Capital (courtyard, great hall, chamber, towers, arena), the Royal Gate on the main island, and the King's crown template | 1760 |
| ServerStorage.ZZTools.ExportParts.lua | Prints every part as a text line so a headless Blender can render it | 68 |
| ServerStorage.CloudBuilder.lua | Library of cloud shapes (discs, puffs, islands, stepping clouds). Not used by the v3 sky island | 271 |
| ServerStorage.ForestBuilder.lua | Builds the forest ring round the sea, one angle slice at a time, plus clearing markers | 429 |
| ServerStorage.ShallowsBuilder.lua | Replaced the sea's lily pads with stepping stones, driftwood logs and rocky islets | 466 |
| ServerStorage.SaturationPass.lua | Makes map colours richer without changing hue, and can undo it | 162 |
| ServerStorage.DevPanel.Server.lua | Dev panel server side: who gets it, the two remotes, the payload, the button actions | 312 |
| ServerStorage.DevPanel.Client.lua | Dev panel client side: draws the readout and the buttons, F4 toggles it | 272 |
| ServerScriptService.GardenService.DevPanel.lua | The panel's snapshot of game state, plus the Round trip, Reload and Seed test actions | 243 |

All twelve are ModuleScripts except DevPanel.Client, which is a LocalScript (from `_manifest.json` in the export).

## 3. How it works

### 3a. What each builder produces

| Builder | How to run | Produces (in Workspace unless noted) | Safe to re-run? |
|---|---|---|---|
| SkyIslandBuilder | `require` it; builds at require time | `Sky Islands > Cloud Garden > Cloud` (parts named `Slab` and `RimPiece`), `Underside` (`Under`), `Drift` (`DriftPiece`) (`ServerStorage.ZZTools.SkyIslandBuilder.lua:439-457`, `:477-478`, `:490-491`, `:546-549`, `:575-578`) | Yes, same seed (`:34`). It deletes and rebuilds only Cloud, Underside and Drift (`:451-454`). But it deletes the decor's `Bumps`, which live inside Cloud (`ServerStorage.ZZTools.SkyIslandDecor.lua:394-400`). Re-run the decor after it (`ServerStorage.ZZTools.SkyIslandBuilder.lua:28-30`). |
| SkyIslandDecor | `require` it after SkyIslandBuilder | `Cloud Garden > Castle` with up to 4 `Flag` models and 3 `NightLamp` parts, `Cloud Garden > Garden` (`CherryTree`, `Bush`, `Flower` models), `Cloud Garden > Cloud > Bumps` (`Bump` parts) (`ServerStorage.ZZTools.SkyIslandDecor.lua:122-126`, `:169-179`, `:184-204`, `:208-210`, `:398-400`) | Yes, seeded (`:21`). But it destroys and rebuilds `Castle` and `Garden` (`:36-39`), which undoes the Royal Capital's castle works. Re-run RoyalCapitalBuilder after it. |
| BeanstalkCrown | `require` it | `beanstalk > Crown` with `Stem` (`StemBlock`, `StemKnot`), `Leaves` (`BridgeLeaf`, `NorthLeaf`, `EastLeaf`, `SouthLeaf`, `ClimbLeaf`), `Lookout` (`ServerStorage.ZZTools.BeanstalkCrown.lua:30-35`, `:80-101`, `:144`, `:155-170`, `:176-189`) | Yes. It replaces `Crown` and is seeded (`:27`, `:31-32`). |
| RoyalCapitalBuilder | `require` it after the decor | `Sky Islands > Royal Capital` with `Markers`, `Courtyard`, `CastleFront`, `GreatHall`, `Towers`, `RoyalChamber`, `RoyalArena`, `ArenaPath` (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:536-541`, `:603`, `:792`, `:823`, `:1181`, `:1238`, `:1331`, `:1625`). Also `Workspace > Royal Gate` (`:1654-1724`), `ReplicatedStorage > KingAssets > RoyalCrown` (`:1729-1744`) and `ServerStorage > RoyalCapital_backup_2026-10-03` (`:554-559`). Edits the castle once (`:561-582`) and moves garden items on every run (`:585-599`). | Yes. It deletes and rebuilds Royal Capital, Royal Gate and RoyalCrown (`:532-535`, `:1650-1653`, `:1732-1733`). The castle edits are guarded by the castle's `RoyalWorks` attribute (`:561`, `:581`). |
| ForestBuilder | `require` it, then call `F.buildSector(fromDeg, toDeg)` per slice, `F.markClearings()`, `F.finish()` | `Main Island > Forest > Trees > Sector_NNN` and `Undergrowth > Sector_NNN` (`ServerStorage.ForestBuilder.lua:197-211`), `Forest > Clearings > ClearingNN` (`:380-404`) | Per sector, yes: it deletes the old sector of the same name first (`:208-210`). The result can differ from the first build (see Gotchas). `markClearings` replaces the folder (`:382-383`). `finish` skips parts it already coloured (`:416`). |
| ShallowsBuilder | `require` a clone, call `B.build(-180, 180, 20260928)` (`ServerStorage.ShallowsBuilder.lua:13-15`) | `Main Island > Shallows > Stones`, `Driftwood`, `Islets` (`:232-243`). Moves the old pads to `ServerStorage > LilyPads_backup_2026-09-28` (`:226-231`, `:386-389`) | Not as a rebuild. It moves the pads out (`:386-389`) and only looks for pads in CoastDecor2 (`:265-270`), so a second run over the same angles finds none. Full rebuild steps are in the header (`:13-15`). `B.finish` is safe to repeat (`:396-397`). |
| CloudBuilder | A library: other code calls `C.island`, `C.pad`, `C.chain` | Whatever parent the caller passes: an island Model with `Floor`, `Rim`, `Underside`, `Puffs` folders (`ServerStorage.CloudBuilder.lua:82-89`), `StepCloud` models (`:221-246`) | No. Each call makes a new model and deletes nothing (`:84-86`). |
| SaturationPass | `require(...).apply()`, `.undo()`, `.status()` in Edit mode (`ServerStorage.SaturationPass.lua:4-7`) | No new parts. Changes `Color` / `Color3` and stores the original as `SatOrigColor` (`:59`, `:108-130`) | Yes. It always works from the stored original (`:15-17`, `:112-113`). |
| ExportParts | `require(clone)(roots, run, filter)` (`ServerStorage.ZZTools.ExportParts.lua:6-9`) | Nothing. Prints one `ZZP` line per part and an `END` line (`:40-68`) | Yes, read-only. "Not used by the game; safe to delete" (`:3-4`). |

Which builders touch logs, lily pads, mountains and the forest (feature 8):
- **Forest**: only ForestBuilder.
- **Logs**: ForestBuilder places fallen logs as undergrowth. Its log templates are `Log1`, `Log2`, `VinesLog1`, `VinesLog2`, `HollowLog` and the pack's fallen trunk named `Tree` (`ServerStorage.ForestBuilder.lua:122-123`). They go in the beach, edge and deep zones (`:310-314`). ShallowsBuilder places driftwood logs in the sea from `Log1`, `Log2`, `VinesLog1`, `VinesLog2` (`ServerStorage.ShallowsBuilder.lua:87`, `:128-134`). Whether these templates are round or studded cannot be told from the code.
- **Lily pads**: no builder makes them. ShallowsBuilder removed every `Lilypad*` child of `Main Island.CoastDecor2` in the chosen angle range and moved it to the backup folder (`ServerStorage.ShallowsBuilder.lua:265-270`, `:386-389`). Its header records 766 pads, 692 pieces placed, 51 sites skipped (`:7-9`).
- **Mountains**: no builder in this subsystem makes them. ForestBuilder only puts trees and rocks on the ridge (r 1445 to 1540), sitting on terrain (`ServerStorage.ForestBuilder.lua:9-10`, `:59-66`, `:281`).

### 3b. The sky island chain, step by step

1. **SkyIslandBuilder** lays a 4-stud grid over the main island (`ServerStorage.ZZTools.SkyIslandBuilder.lua:37`, `:60-61`). It keeps cells inside a set of ellipses (`LOBES`) plus noise, outside the beanstalk opening and the arena opening (`:67-77`, `:204-210`). It keeps only the piece connected to the middle (`:213-227`).
2. It works out each cell's distance to an edge (`:230-249`). It then gives each cell a height level: rims step down (`:264-293`), hills and mounds step up (`:300-323`), the castle plinth and the Royal Capital's ground are set (`:326-345`), the walkway is flat (`:349-353`), and stairs drop into the opening (`:357-366`).
3. It merges equal cells into rectangles and makes one studded Plastic part per rectangle (`:167-200`, `:462-480`). It builds the underside on a coarser 16-stud grid (`:384-435`, `:486-494`).
4. It adds up to 150 walkable rim pieces and up to 44 non-colliding drift pieces. Each drift piece carries bob and spin numbers as attributes (`:528-589`). The client script SkyIslandLife reads them (`StarterPlayer.StarterPlayerScripts.SkyIslandLife.lua:137`).
5. It shrinks any underside block that touches the beanstalk (`ServerStorage.ZZTools.SkyIslandBuilder.lua:593-610`).
6. It writes the island's numbers as attributes (deck height, opening, arena, footprint) (`:613-655`).
7. **SkyIslandDecor** reads those attributes (`ServerStorage.ZZTools.SkyIslandDecor.lua:24-26`). It places the castle from `ServerStorage.AssetQuarantine.Q_ClassicRobloxCastle` (`:33`, `:122-126`), restyles its grey walls (`:128-137`), and adds flags, lamps, trees, bushes, bumps and flowers. It only places things where a downward ray hits the cloud (`:90-98`). It finishes by re-colouring the cherry blossom pink (`:520-548`).
8. **BeanstalkCrown** stacks 2.4-stud stem layers from y 426 to y 553 (`ServerStorage.ZZTools.BeanstalkCrown.lua:68`, `:83-101`). It adds a bridge leaf at the cloud's bottom stair height 438.2 (`:144`), a spiral of 15 leaves (`:154-170`) and a lookout (`:176-189`).
9. **RoyalCapitalBuilder** checks it is in the real place (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:35`). It needs Sky Islands, Cloud Garden and Castle to exist (`:528-531`, `:543-544`). It does the one-time castle works: the back wall moves to the backup folder, the castle is recoloured with the original kept as `OrigColor`, and the castle flags get the `RoyalBanner` tag (`:561-582`). Then it builds each area in turn (`:603-1636`), the Royal Gate (`:1650-1724`) and the crown template (`:1729-1744`). Last it parents the model and returns tag counts (`:1748-1760`).

### 3c. Forest, shallows and colour

- **ForestBuilder.buildSector** makes a jittered 8-stud grid of candidate points in the slice (`ServerStorage.ForestBuilder.lua:226-245`) and shuffles them (`:247-250`). For each point it picks a tree density by zone (`:252-265`), picks a weighted tree (`:94-103`) and rejects it if it is too close to another tree (`:274-278`). It seats the tree on the lowest terrain under its trunk (`:280-291`) and stores `X`, `Z`, `CanopyR`, `TrunkR` (and `Giant`) on it (`:292-296`). Undergrowth comes next, per zone, by rate per 1000 square studs (`:309-361`). Clearings get only grass, flowers and a few rocks (`:315`, `:363-375`).
- **ShallowsBuilder.build** groups pads closer than 7 studs into one site (`ServerStorage.ShallowsBuilder.lua:211`, `:271-286`). It picks stone, log, islet or tree islet by pad count and distance (`:335-354`). It tries up to four times and rejects any piece that reaches the beach or overlaps something (`:317-324`, `:356-384`). Then it calls `B.finish`, which lifts stones and islets clear of the water and saturates their colours (`:391`, `:426-464`).
- **SaturationPass.apply** walks every part, decal and texture in Workspace (`ServerStorage.SaturationPass.lua:87-106`). It skips fruit, neutral colours, anything below y -500, multi-colour unions and textured meshes (`:22-35`, `:77-84`). It raises saturation on a curve and trims brightness a little (`:61-66`). ForestBuilder and ShallowsBuilder reuse its `lift` function on their own clones (`ServerStorage.ForestBuilder.lua:408-427`, `ServerStorage.ShallowsBuilder.lua:446-462`).

### 3d. The dev panel, step by step

1. GardenService requires `ServerStorage.DevPanel.Server` (`ServerScriptService.GardenService.lua:32`). It runs the GardenService.DevPanel module, which puts `devSnapshot`, `devRoundTrip`, `devReload` and `devSeed` into the shared table `G` (`ServerScriptService.GardenService.lua:751-753`, `ServerScriptService.GardenService.DevPanel.lua:241-242`). Then it calls `DevPanel.Start` with those four as hooks (`ServerScriptService.GardenService.lua:763-768`).
2. `DevPanel.Start` checks all four hooks exist (`ServerStorage.DevPanel.Server.lua:290-294`). It watches joins and checks each player's UserId (`:300-309`).
3. `grantTo` makes a `DevChannel` folder with two remotes, `Feed` and `Action`, inside that player's own PlayerGui (`:242-254`). PlayerGui is only sent to its owner. It clones the client LocalScript in (`:267-268`).
4. The client builds a hidden ScreenGui named `DevPanelUI` and nine buttons (`ServerStorage.DevPanel.Client.lua:35-41`, `:99-156`). F4 shows or hides it and tells the server (`:260-272`).
5. While the panel is open, the server builds the payload every second and sends it (`ServerStorage.DevPanel.Server.lua:272-287`). The payload comes from `buildSections` (`:77-193`), which calls the snapshot hook `devSnapshot` (`ServerScriptService.GardenService.DevPanel.lua:28-156`) and the store's stats.
6. Button presses arrive on `Action`. The server re-checks the sender (`ServerStorage.DevPanel.Server.lua:259-265`) and runs `handleAction` (`:197-231`).

### 3e. Every dev panel command

The panel has no chat commands. It has F4 and nine buttons.

| Control | Action sent | What it does | Studio only? |
|---|---|---|---|
| F4 | `open` true/false | Shows or hides the panel. The server only builds snapshots while it is open (`ServerStorage.DevPanel.Client.lua:265-270`, `ServerStorage.DevPanel.Server.lua:201-204`, `:275`) | No |
| Save now | `save` | Forces a save of your profile and prints the result (`ServerStorage.DevPanel.Server.lua:206-210`) | No |
| Round trip | `roundtrip` | Runs the real leave handler, clears state on the player, checks the plot and balance really emptied, waits 1 s, then runs the real join handler (`ServerScriptService.GardenService.DevPanel.lua:189-208`) | No |
| Reload | `reload` | Throws the session away WITHOUT saving, clears state, waits 0.5 s, loads again (`ServerScriptService.GardenService.DevPanel.lua:212-218`) | No |
| Seed test | `seed` | Writes a test profile into the store: cash 1.234e12, best merge rank 40, 7 of rank 60 stored, 12 each of ranks 20, 35, 50, 64 on the plot, and 3 of a fruit name that no longer exists. Press Reload to load it (`ServerScriptService.GardenService.DevPanel.lua:224-237`) | Yes (`ServerStorage.DevPanel.Server.lua:216`, `ServerScriptService.GardenStore.lua:1049-1050`) |
| Fail read (kick) | `failread` | Arms 6 injected read failures (SaveRetryCount 5, plus 1), so the next load gives up and kicks you (`ServerStorage.DevPanel.Server.lua:217-221`, `ReplicatedStorage.GardenConfig.lua:1133`) | Yes (`ServerScriptService.GardenStore.lua:1012-1013`) |
| Fail write x1 | `failwrite` | Arms one injected write failure, so the next save retries and recovers (`ServerStorage.DevPanel.Server.lua:222-223`) | Yes (`ServerScriptService.GardenStore.lua:1012-1013`) |
| Latency 3s | `latency` | Toggles a 3-second delay on every store call (`ServerStorage.DevPanel.Server.lua:224-225`, `ServerScriptService.GardenStore.lua:147`, `:158`) | Yes (`ServerScriptService.GardenStore.lua:1022-1023`) |
| Wipe store | `wipe` | Empties the in-memory store only (`ServerStorage.DevPanel.Server.lua:226-227`, `ServerScriptService.GardenStore.lua:1038-1042`) | Yes |
| Fresh profile | `fresh` | Deletes your profile key, from memory and from the real DataStore if that is the backend (`ServerStorage.DevPanel.Server.lua:228-229`, `ServerScriptService.GardenStore.lua:1028-1035`) | Yes |

The readout has seven sections: SAVE, SPAWN, CAPACITY, INCOME, CARRY / STORAGE, PLOTS and SERVER (`ServerStorage.DevPanel.Server.lua:84-190`).

**Related, but not part of this subsystem.** Royal features have their own Studio test hook. It is a BindableFunction at `ServerStorage.RoyalDev`, called as `RoyalDev:Invoke(command, ...)`. It only exists in Studio when `StudioDevTools` is on (`ServerScriptService.GardenService.Crown.lua:1186-1198`, `ReplicatedStorage.CrownConfig.lua:484`). Its commands are `crown`, `vacate`, `rep`, `state` (`ServerScriptService.GardenService.Crown.lua:1199-1216`), `spar`, `bout`, `ringout`, `endBout` (`ServerScriptService.GardenService.CrownArena.lua:1410-1426`), `decree`, `decreeState` (`ServerScriptService.GardenService.CrownDecree.lua:367-376`), and `contest`, `contestNow`, `join`, `contestState` (`ServerScriptService.GardenService.CrownContest.lua:571-588`). Testing feature 10 will lean on these, not on the dev panel.

## 4. Data

### Saved profile fields

The builders save nothing to player profiles. What they make is saved in the place file. The dev panel touches profiles only through its test buttons.

| Field | Who | Read / write | Where |
|---|---|---|---|
| `cash`, `best`, `stored`, `plot` | devSeed | Writes a whole profile built from `Store.Default()` with these four set | `ServerScriptService.GardenService.DevPanel.lua:225-232`, `ServerScriptService.GardenStore.lua:1049-1056` |
| `shards`, `rebirths` and every other field | devSeed | Left at the default 0 / empty | `ServerScriptService.GardenStore.lua:259`, `:264` |
| whole profile key `p_<UserId>` | Fresh profile | Deleted | `ServerScriptService.GardenStore.lua:1028-1035`, `:62` |

### Remotes

| Name | Direction | Payload | Where |
|---|---|---|---|
| `PlayerGui.DevChannel.Feed` (RemoteEvent) | server to the dev's client | Array of sections `{ title, rows = { { label, value, tone } } }`. `tone` is `"good"`, `"warn"`, `"bad"` or nil | `ServerStorage.DevPanel.Server.lua:67-73`, `:245-247`, `:278` |
| `PlayerGui.DevChannel.Action` (RemoteEvent) | dev's client to server | `(name, value)`. `name` is one of the actions in 3e. `value` is a boolean for `open` and 0 or 3 for `latency` | `ServerStorage.DevPanel.Server.lua:249-251`, `:259-265`; `ServerStorage.DevPanel.Client.lua:147-155`, `:262` |

The builders create no remotes. They create ProximityPrompts named `RoyalPrompt`, each with a `Kind` attribute (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:495-508`). The server's Crown module connects them (`ServerScriptService.GardenService.Crown.lua:972-987`).

### Player and fruit attributes (dev panel)

| Attribute | On | Read / write | Where |
|---|---|---|---|
| `PlotName` | fruit part | Read: a fruit without a known plot counts as "untagged" | `ServerScriptService.GardenService.DevPanel.lua:38-43` |
| `CarryingFruit` | player | Read, compared with the held fruit | `ServerScriptService.GardenService.DevPanel.lua:143` |
| `Shards` | player | Set to 0 by clearPlayerResidentState | `ServerScriptService.GardenService.DevPanel.lua:166` |
| `ProfileLoaded`, `BestMerge` | player | Set to nil | `ServerScriptService.GardenService.DevPanel.lua:170`, `:172` |
| `CashPerSecond` | player | Set to 0 | `ServerScriptService.GardenService.DevPanel.lua:171` |

clearPlayerResidentState also empties the stored-fruit, tools and index folders. It resets rebirths, the King look and upgrades through the publish functions (`ServerScriptService.GardenService.DevPanel.lua:162-183`).

### Instance attributes and tags written by builders

| Attribute / tag | On | Read by | Where written |
|---|---|---|---|
| `Builder`, `Seed`, `Deck`, `HoleX`, `HoleZ`, `HoleR`, `ArenaX`, `ArenaZ`, `ArenaR`, `CenterX`, `CenterZ`, `Radius`, `MinX`, `MaxX`, `MinZ`, `MaxZ` | Cloud Garden model | SkyIslandDecor (`ServerStorage.ZZTools.SkyIslandDecor.lua:25-26`), SkyIslandLife (`StarterPlayer.StarterPlayerScripts.SkyIslandLife.lua:649`) | `ServerStorage.ZZTools.SkyIslandBuilder.lua:613-654` |
| `BobAmp`, `BobSpeed`, `BobPhase`, `SpinSpeed` | each `DriftPiece` | SkyIslandLife (`StarterPlayer.StarterPlayerScripts.SkyIslandLife.lua:137`) | `ServerStorage.ZZTools.SkyIslandBuilder.lua:582-585` |
| `NightLamp` parts (name, not attribute) | castle | SkyIslandLife switches them at dusk (`StarterPlayer.StarterPlayerScripts.SkyIslandLife.lua:181`) | `ServerStorage.ZZTools.SkyIslandDecor.lua:184-204` |
| `SolidGround` | Royal Capital model, castle | Ambient: nobody sinks into stone floors (`ServerScriptService.GardenService.Ambient.lua:195-201`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:538`, `:545` |
| `RoyalWorks` | castle | The builder itself, so castle works run once | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:561`, `:581` |
| `OrigColor` | parts of `Castle1` | Nothing in the export | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:567` |
| `MovedBy` | the moved cherry tree | Nothing in the export | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:593` |
| `Board` + tag `RoyalBoard` | SurfaceGui board parts | Crown's `setLine` (`ServerScriptService.GardenService.Crown.lua:576-596`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:307-308` |
| `CrownRole` | crown and emblem parts | Crown (`ServerScriptService.GardenService.Crown.lua:402`, `:541`), RoyalUI (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1152`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:325`, `:366` |
| `Spin`, `Bob` | spinning crowns and fruit statues | RoyalUI (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1226-1228`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:697-698`, `:1021-1022`, `:1266`, `:1494-1495`, `:1679` |
| `Style` + tag `CrownVault` | vault crowns and plates | Crown (`ServerScriptService.GardenService.Crown.lua:568`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1265-1267`, `:1270` |
| `Wave` | arena and tower flags | RoyalUI (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1229`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1201`, `:1376`, `:1522` |
| `RoyalSeat` | `ThroneSeat` | Crown (`ServerScriptService.GardenService.Crown.lua:1023`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:977` |
| `ChamberDoor` | chamber door parts | Nothing in the export (the prompt does the work) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1040` |
| `Portrait` + tag `RoyalPortrait` | portrait canvases | Crown (`ServerScriptService.GardenService.Crown.lua:611-616`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1059-1060` |
| `Kind` | each `RoyalPrompt` | Crown (`ServerScriptService.GardenService.Crown.lua:977-979`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:505` |
| `RingRadius`, `HoleRadius`, `RingTop`, `ZoneRadius`, `ZoneTop`, `BoundRadius`, `PerchRadius` | marker `ArenaCenter` | CrownArena (`ServerScriptService.GardenService.CrownArena.lua:56-60`). RoyalUI and MusicPlayer also find the marker (`StarterPlayer.StarterPlayerScripts.RoyalUI.lua:1113`, `StarterPlayer.StarterPlayerScripts.MusicPlayer.lua:214`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1583-1590` |
| `Width` | `KingAssets.RoyalCrown` | Crown scales the worn crown by it (`ServerScriptService.GardenService.Crown.lua:444`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1742` |
| `Builder`, `Seed`, `Floor`, `Pave` | Royal Capital model | Nothing in the export | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1750-1753` |
| tags `RoyalBanner`, `RoyalCushion`, `RoyalLamp`, `RoyalEmblem`, `RoyalGlow` | cloth, cushions, lamps, emblems, glows | Crown (`ServerScriptService.GardenService.Crown.lua:534-558`) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:20-27`, `:1756` |
| `X`, `Z`, `CanopyR`, `TrunkR`, `Giant` | each forest tree | ForestBuilder itself, to space later sectors | `ServerStorage.ForestBuilder.lua:292-296` |
| `Note` (folder), `Radius` (part) | `Forest.Clearings`, `ClearingNN` | Nothing yet. Meant for animals | `ServerStorage.ForestBuilder.lua:386`, `:400` |
| `Role` | each shallows piece (`SteppingStone`, `Driftwood`, `Islet`, `TreeIslet`) | Nothing in the export | `ServerStorage.ShallowsBuilder.lua:124`, `:132`, `:193` |
| `ReplacesPads`, `AddedFor` | each shallows piece | Nothing in the export | `ServerStorage.ShallowsBuilder.lua:326-327` |
| `WasIn` | each backed-up lily pad | Nothing in the export | `ServerStorage.ShallowsBuilder.lua:387` |
| `RaisedBy` | lifted stones and islets | `B.finish`, so a rerun adds nothing | `ServerStorage.ShallowsBuilder.lua:435-439` |
| `SatOrigColor` | any recoloured part, decal or texture | SaturationPass undo / status; Upgrades clones it on fence pieces (`ServerScriptService.GardenService.Upgrades.lua:66-70`) | `ServerStorage.SaturationPass.lua:124`, `ServerStorage.ForestBuilder.lua:420`, `ServerStorage.ShallowsBuilder.lua:457` |
| `Radius`, `FloorTop` + tag `CloudPuff` | CloudBuilder islands and puffs | No script in the export reads them | `ServerStorage.CloudBuilder.lua:209`, `:215-216`, `:241` |

### Markers in `Royal Capital.Markers`

`Altar`, `ReturnGate`, `CourtyardArrival` (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:725`, `:771`, `:775`); `CrownDisplay`, `ChamberDoorN`, `ChamberDoorS`, `ThroneFront`, `HallDoor` (`:1024`, `:1038`, `:1175-1176`); `StyleMirror`, `DecreeDesk`, `ChamberInside`, `ChamberExitN`, `ChamberExitS` (`:1279`, `:1290`, `:1322-1324`); `ArenaCenter`, `DuelPadA`, `DuelPadB`, `ContestSpawn1`-`8`, `KnightPost1`-`4`, `StandSpot1`-`12` (minus any in the gate arc), `ArenaGate`, `SparringPost` (`:1583-1619`). The Royal Gate's prompt sits in the Royal Gate model instead, as `PromptSpot` (`:1720-1721`).

Prompt kinds: `Altar`, `ReturnGate`, `ChamberDoor`, `StyleMirror`, `DecreeDesk`, `ChamberExit`, `Spar`, `RoyalGate` (`:726`, `:772`, `:1039`, `:1280`, `:1291`, `:1325-1326`, `:1620`, `:1721`).

### Config keys and constants read

| Key | Read by | Where |
|---|---|---|
| `Config.MaxLevel("spawnTier" / "spawnRate" / "capacity")` | panel SPAWN and CAPACITY rows | `ServerStorage.DevPanel.Server.lua:125`, `:143` |
| `Config.MaxFruitsPerPlot` | CAPACITY row | `ServerStorage.DevPanel.Server.lua:143` |
| `Config.IncomeTick`, `Config.FriendBonus` | INCOME rows | `ServerStorage.DevPanel.Server.lua:152`, `:154` |
| `Config.SaveRetryCount` | Fail read button | `ServerStorage.DevPanel.Server.lua:221` |
| `Config.SpawnTable`, `Config.SpawnInterval`, `Config.Capacity` | devSnapshot | `ServerScriptService.GardenService.DevPanel.lua:63`, `:113`, `:117` |
| `Config.KingDefault` | clearPlayerResidentState | `ServerScriptService.GardenService.DevPanel.lua:168` |
| `Config.IslandName`, `Config.FruitFolder` | SaturationPass, to skip fruit | `ServerStorage.SaturationPass.lua:69-74` |

No other builder reads GardenConfig. ForestBuilder and ShallowsBuilder hard-code `workspace["Main Island"]` and `workspace["Asset Pack V1"]` (`ServerStorage.ForestBuilder.lua:16-17`, `ServerStorage.ShallowsBuilder.lua:19-20`).

## 5. Numbers

None of these are Config keys. They are constants inside the builders, so changing one means editing the builder and re-running it.

| Value | Meaning | Where |
|---|---|---|
| 443.0 | Cloud deck height (walk surface) | `ServerStorage.ZZTools.SkyIslandBuilder.lua:39`, `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:46` |
| (124, 0) r 38 | Beanstalk opening in the cloud | `ServerStorage.ZZTools.SkyIslandBuilder.lua:45` |
| (-20, -170) r 52, flat to r 98 | Arena opening and its flat ring | `ServerStorage.ZZTools.SkyIslandBuilder.lua:52-53` |
| 1.2 / 4 | One brick step / grid cell size | `ServerStorage.ZZTools.SkyIslandBuilder.lua:37-38` |
| 150 / 44 | Max rim pieces / drift pieces | `ServerStorage.ZZTools.SkyIslandBuilder.lua:529`, `:559` |
| 46 / 5 / 5 / 90 / 140 / 80 | Max cherry trees / lip trees / back trees / bushes / bumps / flower patches | `ServerStorage.ZZTools.SkyIslandDecor.lua:250`, `:321`, `:342`, `:372`, `:406`, `:449` |
| 426 to 553 | Beanstalk crown stem height range | `ServerStorage.ZZTools.BeanstalkCrown.lua:68` |
| 438.2 | Bridge leaf height (cloud's bottom stair) | `ServerStorage.ZZTools.BeanstalkCrown.lua:144` |
| 45 degrees, 7.2 studs, from 445.4 | Spiral leaf step, rise and start | `ServerStorage.ZZTools.BeanstalkCrown.lua:154`, `:161` |
| 443.4 / 445.8 | Courtyard paving top / castle and hall floor | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:47-48` |
| 449 / 30 / 52 / 9 | Arena ring top / ring radius / opening radius / Crown Zone radius | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:57-60` |
| 51 / 6.2 | Arena bound radius / perch radius (marker attributes) | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1589-1590` |
| (0, 0.12, 150) | Royal Gate position on the main island | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1658` |
| 0.8 s / 13 | Altar prompt hold time / distance | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:726` |
| 0.4 s / 12 | Royal Gate prompt hold / distance | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1721` |
| 40 px per stud, 260 studs | Board resolution and view distance | `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:263-265` |
| 1040 / 1072 / 1140 / 1445 / 1540 | Forest beach, edge, deep, ridge, outer radii | `ServerStorage.ForestBuilder.lua:19` |
| 0.42 / 0.2, 0.06, 0.015 / 0.15 | Tree chance per candidate: edge / deep (by noise) / ridge | `ServerStorage.ForestBuilder.lua:259-264` |
| 0.6 | Chance a "mega" tree type is a giant in the deep forest | `ServerStorage.ForestBuilder.lua:269` |
| r 1190 to 1380, radius 38 to 72, every 16 to 24 degrees | Clearings | `ServerStorage.ForestBuilder.lua:136-141` |
| 7 / 381 / 1 | Pads that share one piece / no piece inside this radius / clear gap round each piece | `ServerStorage.ShallowsBuilder.lua:211-213` |
| -0.95 / -1.95 / -0.45 | Stone top / log centre / islet top | `ServerStorage.ShallowsBuilder.lua:110-112` |
| -2.2 / -1.9 / -1.6 | Ocean top / glow / sheen heights (comment) | `ServerStorage.ShallowsBuilder.lua:109` |
| 0.9 max | Total lift for a stone or islet | `ServerStorage.ShallowsBuilder.lua:408` |
| 1.6 / 0.96 / 0.08 / -500 | Saturation strength / brightness trim / neutral cut-off / skip below this y | `ServerStorage.SaturationPass.lua:48-57` |
| 1 s | Dev panel refresh while open | `ServerStorage.DevPanel.Server.lua:40` |
| 1015320113 | The only UserId that gets the panel (the place owner) | `ServerStorage.DevPanel.Server.lua:36-38`, `ReplicatedStorage.GardenConfig.lua:343` |
| 6 / 1 / 3 s | Injected read failures / write failures / latency | `ServerStorage.DevPanel.Server.lua:221-225` |

## 6. Extension points for the expansion

The dev panel touches almost every feature, because it is how the expansion gets tested. One recipe covers adding a test button:
1. Add a row to `BUTTONS` (`ServerStorage.DevPanel.Client.lua:99-109`). The tray grows on its own (`:111-112`).
2. Add a branch to `handleAction` (`ServerStorage.DevPanel.Server.lua:197-231`).
3. If it needs GardenService state, write the function in GardenService.DevPanel. Export it through `G` (`ServerScriptService.GardenService.DevPanel.lua:241-242`), add its name to the check in `DevPanel.Start` (`ServerStorage.DevPanel.Server.lua:291`), and pass it in (`ServerScriptService.GardenService.lua:763-768`).
4. Add any new state that lives on the player to `clearPlayerResidentState` (`ServerScriptService.GardenService.DevPanel.lua:162-183`). If you skip this, Round trip and Reload tests pass on old state; the comment above it says exactly this (`:158-161`).

Today no button gives cash, shards, rebirths, tools or fruit. Seed test is the only shortcut to a rich profile (`ServerScriptService.GardenService.DevPanel.lua:224-237`).

**Feature 1: rebirth-gated rarity.**
- Hook: add test buttons for "set rebirths N", "add shards", "add cash" and "fill index" with the recipe above. Add readout rows for the rarity ceiling and storage cap to SPAWN and CARRY / STORAGE (`ServerStorage.DevPanel.Server.lua:123-137`, `:162-171`).
- Change: devSeed leaves `rebirths` at 0 (`ServerScriptService.GardenStore.lua:264`) but puts rank 20, 35 and 50 fruit on the plot and best merge at rank 40 (`ServerScriptService.GardenService.DevPanel.lua:227-231`). Apple is rank 60 (`ReplicatedStorage.Fruits.FruitIndex.lua:69`). So under the new gate the seed holds fruit far above what 0 rebirths allows. That makes it a ready test for how loading clamps above-gate fruit. Decide that rule first.
- Risk: the SPAWN row lists the spawn table from `Config.SpawnTable(tierLevel)` (`ServerScriptService.GardenService.DevPanel.lua:61-79`). If the gate filters spawns somewhere else, the row will show fruit that cannot actually spawn. The arena's fruit statues show Soul Fruit, Golden Apple and Watermelon (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:63`). A new player sees locked fruit there. That is cosmetic only.

**Feature 2: bats that upgrade per rebirth.**
- Hook: a "give bat" button for knockback tests (recipe above).
- Risk: if the bat lives on the player as a tool or attribute, clearPlayerResidentState must remove it. It already empties the tools folder (`ServerScriptService.GardenService.DevPanel.lua:164`), but only for whatever lives there.

**Feature 3: explosives that drain fruit health.**
- Hook: a "give TNT tier N" button. If health is stored on fruit, add a readout row (count of damaged fruit) to CAPACITY (`ServerStorage.DevPanel.Server.lua:141-148`).
- Risk: the panel counts fruit without a valid `PlotName` as "untagged" and flags them red (`ServerScriptService.GardenService.DevPanel.lua:38-43`, `ServerStorage.DevPanel.Server.lua:147`). A health system that swaps a fruit for a damaged copy must copy `PlotName`, or the fruit drops out of plot clean-up. The panel would show that, which is useful.

**Feature 4: 100k-shard underground base with ores.**
- Hook: no builder here makes plots. Add an "add 100k shards" button (shards reset lives at `ServerScriptService.GardenService.DevPanel.lua:166`). If ores are saved, add them to devSeed and clearPlayerResidentState.
- Risk: SaturationPass skips every part below y -500 (`ServerStorage.SaturationPass.lua:57`, `:95`). That is where the asset packs are parked (`:29-30`, `ServerStorage.ShallowsBuilder.lua:404-406`). An underground base deeper than -500 will never be colour-matched by `apply()`. It could also collide with the parked packs. Their exact height is not in the code.
- ExportParts can render the underground for review, with limits (see Gotchas).

**Feature 5: forest pets.**
- Hook: the clearings were built for this. ForestBuilder keeps them free of trees "for the animals later" (`ServerStorage.ForestBuilder.lua:132`). `F.markClearings()` puts a marker part `ClearingNN` with a `Radius` attribute at each clearing's centre, under `Main Island.Forest.Clearings` (`:380-404`). A pet spawner should read these markers, not require ForestBuilder at runtime.
- Change: wild pets will walk among undergrowth. Bushes, logs, rocks and boulders keep collision on. Only grass entries (fifth field `false`) lose it (`ServerStorage.ForestBuilder.lua:309-317`, `:337-339`). Pets need pathfinding or must stay inside clearings. Each tree carries `TrunkR` and `CanopyR` (`:294-295`), which avoidance code can use.
- Risk: whether `markClearings` has been run in the live place cannot be told from the code. Check for the folder. Re-running `buildSector` does not touch it (`:206-211`).

**Feature 6: boats, swimming, no currents.**
- Hook: none in the builders. The current and drowning settings are in GardenConfig (`ReplicatedStorage.GardenConfig.lua:1291-1327`).
- Risk: the Shallows pieces (about 692, `ServerStorage.ShallowsBuilder.lua:7-9`) fill the sea from r 381 outward (`:212`). Their clone step anchors them and turns off touch, but leaves collision on (`:60`). Driftwood sits with its centre at -1.95 (`:111`), right at the ocean top (-2.2, `:109`). A boat at the waterline will hit stones, logs and islets. Plan lanes, or decide boats ride above them.
- Removing the current plates does not affect the builder. It already ignores `CurrentPlate` and `Seabed` (`:249`).
- Boats landing at the forest arrive on its beach band, r 1040 to 1072 (`ServerStorage.ForestBuilder.lua:19`). That band has collidable logs and rocks (`:310`). Clear docking spots.
- Test hook: "add shards" and "give boat" buttons.

**Feature 7: more potions.**
- Hook: a "give potion" button.
- Risk: potion timers are published as `<Effect>EndsAt` and `<Effect>Span` player attributes (`ServerScriptService.GardenService.Tools.lua:217`, `:223`). clearPlayerResidentState does not list them (`ServerScriptService.GardenService.DevPanel.lua:162-183`). Test a Round trip with a potion running. The beanstalk spiral was play-tested for the sky jump of about 16 studs (`ServerStorage.ZZTools.BeanstalkCrown.lua:150-153`). A jump or fall potion should be tried on it.

**Feature 8: plot visual overhaul, studded logs, studded lily pads, new border mountains.**
- Logs, hook: ForestBuilder `LOGS` (`ServerStorage.ForestBuilder.lua:122-123`) and ShallowsBuilder `LOGS` (`ServerStorage.ShallowsBuilder.lua:87`). Put the studded log template in both lists.
- Logs, change: do not just re-run `buildSector`. Trees are placed against whatever neighbour sectors exist at build time (`ServerStorage.ForestBuilder.lua:215-222`), so a rebuild now re-rolls the trees too. Safer: a one-off script that replaces each log in place, keeping its pivot and scale. Forest logs keep their template names (`Log1`, `Log2`, `VinesLog1`, `VinesLog2`, `HollowLog`, `Tree`) under `Forest.Undergrowth`, because `cloneScaled` keeps the name (`:43-56`). Sea logs carry `Role = "Driftwood"` in `Shallows.Driftwood` (`ServerStorage.ShallowsBuilder.lua:132`, `:330`). After swapping, run `F.finish()` and `B.finish()` so new clones get the same colour lift. Both skip parts already done (`ServerStorage.ForestBuilder.lua:416`, `ServerStorage.ShallowsBuilder.lua:453`).
- Lily pads, hook: the original pads still exist in `ServerStorage.LilyPads_backup_2026-09-28`, each with a `WasIn` attribute (`ServerStorage.ShallowsBuilder.lua:226-231`, `:387`). Live, they were replaced by Shallows pieces. "Turn the current lily pads into a studded texture" therefore has two readings: restore restyled pads, or give the stepping stones a studded look. Ask Samarth. Restoring pads would overlap the pieces that replaced them. Each piece records only how many pads it replaced (`ReplacesPads`), not which ones (`:326`).
- Mountains, hook: no builder makes them. If studded mountain models replace the terrain ridge, ForestBuilder's ground ray only sees terrain (`ServerStorage.ForestBuilder.lua:59-62`). Rebuilt ridge sectors would put trees inside the new models. Add the mountain models to `groundParams`, or delete ridge trees under them.
- Plot overhaul, risk: plots are not built here. Runtime fences are cloned from authored pieces, which carry `SatOrigColor` (`ServerScriptService.GardenService.Upgrades.lua:66-70`). New authored plot parts need `SaturationPass.apply()` after the edit. If not, they look duller than the rest. ShallowsBuilder documents this exact problem (`ServerStorage.ShallowsBuilder.lua:404-407`).
- Studs, risk: the map uses two stud methods. The main map uses Plastic with a MaterialVariant named `Studs` (a custom material look) (`ServerScriptService.GardenService.Upgrades.lua:68-69`, `ServerStorage.CloudBuilder.lua:32`). The sky builders use classic per-face `SurfaceType.Studs` (`ServerStorage.ZZTools.SkyIslandBuilder.lua:155`, `ServerStorage.ZZTools.BeanstalkCrown.lua:57`, `ServerStorage.ZZTools.RoyalCapitalBuilder.lua:119`). Pick one per area so new studded logs and pads match their surroundings.

**Feature 9: UI overhaul, especially King / Royal.**
- Hook: every Royal Capital sign is a SurfaceGui made by `board()` (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:258-310`). It already uses FredokaOne (`:292`). The emoji come from the builder's default texts: lines `:717`, `:769`, `:784-785`, `:800`, `:1084`, `:1112-1113`, `:1135-1136`, `:1148`, `:1156`, `:1295`, `:1528`, `:1618`, `:1686`. Edit them and re-run the builder.
- Change: Crown fills these boards at runtime by line name, setting `label.Text` (`ServerScriptService.GardenService.Crown.lua:592-596`). The line names (`Title`, `Caption`, `King`, `Detail`, `Hint`, `Row1`..., `Foot`, `Text`, `Status`, `Names`, `Score`, `Name`, `Eternal`, `EternalName`, `EternalTime`, `Rank`, `Reign`, `Defenses`, `Court`) are a contract between the builder and Crown.
- Risk: a baked title image must not take a line's name. An ImageLabel has no `Text`, so `setLine` would error on it. Add images as extra children with new names. Crown also caches each board list once (`ServerScriptService.GardenService.Crown.lua:577-590`), so rebuild in Edit mode, not during a play test. The dev panel is dev-only and uses the Code font (`ServerStorage.DevPanel.Client.lua:69`). Leave it out of the overhaul.

**Feature 10: royal combat.**
- Hook: the arena's shape lives in this builder. `ArenaCenter` carries ring, zone, bound and perch radii (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1583-1590`), and CrownArena reads it (`ServerScriptService.GardenService.CrownArena.lua:56-60`). Duel pads, contest spawns, knight posts and the sparring post are markers (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1595-1620`).
- Change: a weapon-skin chooser "right before fighting" can be a new marker plus `prompt()` with a new `Kind`, for example near `ArenaGate` or the duel pads. Use `marker()` and `prompt()` (`:490-508`). Crown only wires prompts on direct children of `Markers` that hold a `RoyalPrompt` child, and calls `handlers["prompt:" .. Kind]` (`ServerScriptService.GardenService.Crown.lua:972-987`). Put the marker in `Markers` and add the handler.
- Risk: slam effects and knockback must allow for the four pillars at radius 21 (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1370-1374`), the two perches at radius 41 (`:1380-1381`) and the ring edge at 30 over an opening of 52 (`:58-59`). The altar's crossed swords use the old classic sword mesh (`:687-694`). Swap them if the new weapons get a new look. Test with `RoyalDev` (`spar`, `bout`, `ringout`, `endBout`), not the dev panel (`ServerScriptService.GardenService.CrownArena.lua:1410-1426`).

## 7. Gotchas

1. **Build order is a chain.** Run SkyIslandBuilder, then SkyIslandDecor, then RoyalCapitalBuilder. The island builder deletes `Cloud`, and the decor's `Bumps` live inside it (`ServerStorage.ZZTools.SkyIslandBuilder.lua:451-454`, `ServerStorage.ZZTools.SkyIslandDecor.lua:394-400`). The decor destroys `Castle` and `Garden` (`ServerStorage.ZZTools.SkyIslandDecor.lua:36-39`). The fresh castle has no `RoyalWorks`, so its back wall is closed again, it has no `SolidGround`, and its flags are untagged until RoyalCapitalBuilder runs again (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:545`, `:561-582`). The underside's beanstalk check only sees what is in `workspace.beanstalk` at that moment (`ServerStorage.ZZTools.SkyIslandBuilder.lua:593-609`). Run BeanstalkCrown before the island builder if the crown changes.
2. **Edit mode caches `require`.** A second `require` of the same module in one Studio session returns the cached result and builds nothing. Require a clone instead (`ServerStorage.ShallowsBuilder.lua:14-15`, `:447`). This matters most for the four ZZTools builders, which build at require time.
3. **Place lock.** RoyalCapitalBuilder, ForestBuilder and ShallowsBuilder refuse to run outside place 88602028667228 (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:35`, `ServerStorage.ForestBuilder.lua:195`, `ServerStorage.ShallowsBuilder.lua:222`).
4. **BeanstalkCrown can hang.** It calls `WaitForChild("beanstalk")` with no timeout (`ServerStorage.ZZTools.BeanstalkCrown.lua:30`). If the beanstalk is missing or renamed, the command bar waits forever.
5. **The decor does not know about the Royal Capital.** Its `reserved()` keeps only the walkway, the opening and the castle clear (`ServerStorage.ZZTools.SkyIslandDecor.lua:239-244`). The courtyard, arena path, hall and arena ring came later, in island v3 (`ServerStorage.ZZTools.SkyIslandBuilder.lua:55-58`). A decor re-run can put bushes, bumps and flowers where the capital's paving and stands go. Whether that has happened in the live map is not visible from the code.
6. **RoyalCapitalBuilder's backup folder grows.** Each castle rebuild plus capital run moves another back wall and more procession-line flowers into `RoyalCapital_backup_2026-10-03` (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:554-564`, `:594-595`).
7. **RoyalCrown is only rebuilt if `ReplicatedStorage.KingAssets` exists** (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1730-1731`). If it does not, the run succeeds silently without a crown template.
8. **The Royal Gate must stay outside `Sky Islands`.** Anything inside that model counts as sky ground and turns on sky lighting (`ServerStorage.ZZTools.RoyalCapitalBuilder.lua:1647-1649`).
9. **ForestBuilder re-runs differ from the first build.** Spacing checks use the neighbour sectors that exist now (`ServerStorage.ForestBuilder.lua:215-222`). Undergrowth spacing is not shared across sectors at all (`:323`). The module also looks up every template by exact name when it loads (`:70-129`). One renamed asset in `Asset Pack V1` makes the whole module error.
10. **ShallowsBuilder quirks.** Pads move to the backup even when no piece was placed for them (`ServerStorage.ShallowsBuilder.lua:385-389`), which is the 51 skipped sites (`:8`). The comment says tree islets go at every 30-degree mark, but the loop steps by 20 (`:301`, `:303`). Islets are first parented to Workspace so raycasts can hit them (`:150-152`). If the build errors part-way, a stray `Islet` model can be left in Workspace.
11. **CloudBuilder's burst puffs have no consumer here.** It tags top puffs `CloudPuff` and names `StarterPlayerScripts.CloudPuffs` as the script that bursts them (`ServerStorage.CloudBuilder.lua:7-11`). No script in this export uses that tag. The v3 sky island does not use CloudBuilder at all.
12. **SaturationPass is global.** `apply()` lifts every eligible part in Workspace, including the sky island and the Royal Capital's exact palette (`ServerStorage.SaturationPass.lua:90-104`). `undo()` also scans ServerStorage and ReplicatedStorage (`:136`). Never use Ctrl+Z for it (`:19-20`).
13. **ExportParts loses detail.** It knows only Ball, Cylinder, Wedge, CornerWedge and box shapes (`ServerStorage.ZZTools.ExportParts.lua:26-36`), so MeshParts and unions export as boxes. It records only `SurfaceType` studs and four materials (`:12-24`). Parts studded through the `Studs` MaterialVariant render smooth. It also skips parts at 0.98 transparency or more (`:43`).
14. **The panel is for one account.** Only UserId 1015320113 gets it (`ServerStorage.DevPanel.Server.lua:36-38`). Anyone else testing must be added there. The remotes live in that player's PlayerGui, so they do not show in ReplicatedStorage (`:253-254`).
15. **Reload throws away unsaved progress, on live servers too.** It tears down with `skipSave` set (`ServerScriptService.GardenService.DevPanel.lua:214`), which abandons the session (`ServerScriptService.GardenService.lua:708-709`). Save, Round trip and Reload are not limited to Studio (`ServerStorage.DevPanel.Server.lua:206-214`).
16. **Fresh profile and Seed test hit the real DataStore in Studio.** With Studio API access on, the backend is the real `GardenProfiles` store, the same name the live game uses (`ServerScriptService.GardenStore.lua:56`, `:182-195`). Fresh profile then calls `RemoveAsync` on the dev's live key (`:1028-1035`). Seed test overwrites it (`:1049-1056`). Wipe store only clears the in-memory backend (`:1038-1042`).
17. **Fail read and Fail write only arm a counter.** Nothing happens until the next load or save uses it (`ServerScriptService.GardenStore.lua:607-609`, `:666-668`). Press Reload, Round trip or Save now afterwards.
18. **The latency button sends its old state.** The client sends 3 when latency is on and 0 when it is off, then flips (`ServerStorage.DevPanel.Client.lua:149-151`). The server sets 0 if it received more than 0, and 3 otherwise (`ServerStorage.DevPanel.Server.lua:225`). The button's colour is client-side only.
19. **clearPlayerResidentState is a fixed list.** It does not touch the `Royal*` attributes that Crown publishes (`ServerScriptService.GardenService.Crown.lua:147-166`) or potion timers. Every new player-resident system in the expansion must be added to it (`ServerScriptService.GardenService.DevPanel.lua:162-183`).
20. **Stale comment in devSeed.** It says 48 fruit against "a cap of 40" (`ServerScriptService.GardenService.DevPanel.lua:230`). Capacity now runs from 6 to 35 (`ReplicatedStorage.GardenConfig.lua:51-52`).
21. **The panel costs nothing while closed.** It builds no snapshot then (`ServerStorage.DevPanel.Server.lua:274-276`). A snapshot error shows as a red "SNAPSHOT ERROR" section rather than stopping the loop (`:280-282`).
