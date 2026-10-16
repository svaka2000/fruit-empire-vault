---
tags: [system, design]
date: 2026-10-04
status: proposed
depends_on: [Underground and Ores, Rarity Gating and Rebirth]
---
# Plot Overhaul and Environment

Every plot becomes a studded walled garden on its own green lot, built from Samarth's packs. Round logs, lily pads and the border mountains get the same studded look. Everything here is a proposal unless it cites live code.

## Why

From [[Brain Dump 2026-10-04]], verbatim:

> also improve the look of the default plots by 10x, make them look extremely highquality. and more appealing.

> switch out all round logs for the studded logs that you already have. then turn the current lily pads into a studded texture.

> heres some more stuff for design and structure

> https://create.roblox.com/store/asset/105342635739603/Studed-Mountain - you can figure out the uses for all the stuff im sending. for example these can be used to replace the mountains you made on the border of the map.

> make this super planned and tell me exactly what assets you need dont make slop unless it looks clean and studded.

> you get access to an underground portion of your base. the underground should look super good.

> the king feature also showed me that the castle you made looks really AI.

> it should be obvious to them that to unlock more fruits to merge and more fruits in the index they need to rebirth.

What the system is for:
- Today a plot is a flat orange dirt rectangle with a two-rail fence on a pale grey plaza ([[World Tour]]). This system turns each of the 12 plots into a garden that looks finished: a stone-edged raised bed, hedges, a gate with lanterns, a path and a lawn.
- It gives every plot a physical, locked entrance to the underground, so the base upgrade is visible before anyone buys it.
- It shows each owner's stage on their gate (banners in the rarity colour), so rebirthing changes how your base looks.
- It makes the rest of the world match: studded logs, studded lily pads, studded mountains, and uses for every pack Samarth sent. Every shape and colour comes from his packs or the live map, not from invented art, which is the fix for "looks really AI".

![[today-plot0.jpg]]
*Today's plot: flat dirt, two-rail fence, grey plaza.*

![[today-plots-ring.jpg]]
*The 12 plots in their ring, with roads between them.*

**Words used in this note**
- **Plot model**: `Main Island.Plots.PlotN` (N = 0 to 11). One player owns one.
- **Soil**: the flat part fruit sit on. Today the code finds it as the biggest part in the plot model (`PlotRegistry.lua:21-30`).
- **Pen**: the soil plus everything that hugs it (border, hedges, fence, gate, path, beds). The pen is rebuilt in code every time the plot size changes.
- **Lot**: the green area around one pen, between the two roads on either side of it. Its decor never moves.
- **Wedge**: the triangle of lot between the central lawn and a road. Each lot has a left wedge and a right wedge.
- **Reserved pen zone**: the rectangle the pen fills at its biggest size (level 4). Nothing static may stand in it.
- **Stage**: the newest rarity a player has unlocked ([[Stage Model]]). Stage = rebirths + 1. Stage 1 is Common, stage 9 is Transcendent.
- **F_s**: the full-plot $/s of stage s. **R_s**: the rebirth cash requirement of stage s. Both from the [[Stage Model]] table.
- **PlotKit**: a folder of prefab parts in ServerStorage that the pen and lot are cloned from.
- **Builder**: a ModuleScript in `ServerStorage.ZZTools` that someone runs by hand in Studio Edit mode. The game never runs it ([[World Builders and Dev Tools]]).
- **Seed**: a fixed number for the random generator, so a re-run places everything in the same spot.
- **LOD copy** (level of detail): a cheaper copy of a model with fewer parts, used where the player is far away. A **hero copy** is a full-detail copy.
- **Plot frame** used for all positions below: origin at the soil centre. **x** = studs to the right of a visitor who walks in from the plaza (negative = left). **a** = distance from the map centre along the plot's axis, so small a = the plaza side (front) and big a = the forest side (back). The soil centre sits at a = 200.8 (Plot0's soil centre is (51.98, 194.00), NOTES.md 2026-09 plot-size entry). All 12 plots share this geometry ("Same geometry on all 12 plots", same entry).
- **S**: the soil top height, world y 0.50 (NOTES.md map entry). Heights below are given as S plus or minus a number.

## Player view

1. You join. You appear on your plot's spawn pad, 5 studs inside the front of your soil, as today (`PlotRegistry.lua:112-139`).
2. You see your garden: furrowed soil inside a two-tone stone border, green studded hedges on three sides, studded stone pillars at the corners, a low wooden fence at the front, and a stone gate with two lanterns on top. Your display name is on a plaque on the gate beam. Two cloth banners hang on the gate pillars in your stage's rarity colour (grey-blue at stage 1).
3. You walk out through the gate. A stone path leads to the grey walkway ring that circles the plaza. Flower beds sit on both sides of the gate. Your lot is green lawn from road to road, with studded trees in the side wedges.
4. Just right of the gate stands a studded wooden sign. Only you see it (as today). It shows the Plot Size card from the Upgrade panel, a live picture of your pen, the level, the next size and the price. You press Buy. Your pen grows: the border, hedges, pillars and fence rebuild one size out, with a short dust puff at each corner. Fruit stay where they are.
5. At the back right of your lot, stepping stones lead to a small stone hut with locked wooden doors and a gold padlock. That is the underground entrance, and the Mine Lift of [[Underground and Ores]] stands inside it (decision D-023). After you buy the underground the doors stand open and the lantern above them is lit.
6. At night (30 s of every 330 s cycle, `GardenConfig.lua:1475-1479`) the gate lanterns and the hut lantern glow on every plot.
7. You rebirth. Your banners turn the next rarity's colour (green at stage 2). Your pen keeps its size (Rule E1, decision D-020).
8. You look past the forest. The horizon is a ring of stepped, studded mountains from the Studed Mountain pack with snow caps. The logs in the forest and the sea are square studded logs. The ponds on the island have studded lily pads. Two sword statues guard the Royal Gate.

## Rules

### A. Plot structure
1. Each plot model keeps its soil part, `PlotSpawn` and owner icon. It gains three children: `Pen` (Model, rebuilt at runtime), `Lot` (Model, static), `UndergroundEntrance` (Model, static).
2. The soil part carries the attribute `PlotSoil = true`. Every part this system adds carries `PlotDecor = true`.
3. Soil detection (`PlotRegistry.lua:21-30` and its copy in `PenUpgradeSign.lua:238-247`) returns the descendant with `PlotSoil = true`. If none has it, it returns the largest-area BasePart that does NOT have `PlotDecor`. A lawn bigger than the soil must never become the soil.
4. No part of `Lot` or `UndergroundEntrance` may lie inside the reserved pen zone: x within ±29.1 and a within 200.8 ± 44.1, plus 1.0 stud of clearance. The lot builder checks this and refuses to finish if any part breaks it.
5. Every part of `Lot` stays inside its wedge-shaped area: at least 0.5 studs from each road edge (roads are 10 wide), between a = 146 and a = 250, and at radius 258 or less from the map centre (the forest edge is r 259.5, `GardenConfig.lua:104-107`).
6. The grey walkway ring from r 129 to r 146 stays bare plaza. Nothing in this system is built there except the two Royal Gate statues (Rule I1).

### B. The pen
1. The pen is rebuilt from the soil's current size by `buildPen(plot)`. It runs when `applyPlotSize` changes the level (`Upgrades.lua:242-260`), and once for all 12 plots at server start. It runs at no other time.
2. `buildPen` deletes the old `Pen` model and the legacy `Fence`, `FenceFront` and non-soil BaseParts named `Part` (the same rule as `Upgrades.lua:168-173`). It never touches `Lot`, `UndergroundEntrance`, `PlotSpawn` or the soil.
3. `buildPen` does not yield. A rebuild finishes inside one server step, so no other code sees a half-built pen.
4. The pen pieces, sizes and heights are exactly those in the Numbers table "Pen pieces". The pen's part count per level is at most the "Pen parts" column of the "Pen per level" table.
5. The gate opening is 19.36 studs wide, the same as today (`GATE_HALF` 9.68, `Upgrades.lua:61`). The gate step top is S + 0.30, so walking in feels like today's kerb.
6. Hedges, pillars, the front fence and the gate beam are solid (`CanCollide` true). Furrows, banners, the nameplate and lantern glass have `CanCollide`, `CanQuery` and `CanTouch` all false.
7. After a rebuild, any character whose root part is inside, or within 1.5 studs of, a new hedge, pillar or fence part is moved to the nearest point 2 studs outside that part, on the side its root was on. Characters standing on the soil are never moved.
8. Every pen part gets the CollectionService tag `PenPiece`. Lantern glass parts also get `PlotLamp`.

### C. The lot
1. The lawn is 3 parts: one central rectangle and two flat wedge parts that fill the triangles next to the roads (Numbers table "Lot layout"). The lawn top is S − 0.30.
2. Lot decor positions are fixed by the lot builder with seed 20261004 + the plot's index. A re-run gives the same parts in the same places (within 0.001 studs).
3. The left wedge's back corner (x −37 to −55, a 230 to 248) is reserved for the future upward base upgrade ("towers upwards", brain dump). Until then it holds lawn and one studded tree.

### D. Dressing (what changes with the owner)
1. Owned plot: the gate nameplate shows the owner's `DisplayName`. Unowned plot: the nameplate is hidden. It updates within 1 s of a claim or release.
2. Owned plot: both gate banners take `Config.RarityColors[tier]` (`GardenConfig.lua:582-592`), where tier is the owner's stage (`min(Rebirths + 1, 9)`). Unowned plot: banners are hidden (Transparency 1). They update within 1 s of the owner's `Rebirths` attribute changing.
3. The underground entrance is closed unless the owner owns at least one mine floor. The attribute name is `Config.PlotEntranceAttr`, set to `"UndergroundFloors"`, the attribute [[Underground and Ores]] publishes (one name for every system, [[Integration Review]] conflict C15). Closed: doors and padlock visible and solid, lantern off. Open: doors and padlock hidden and not solid, lantern on. It updates within 1 s.
4. Lamps: on each client, every 1 s, every `PlotLamp` part's PointLight is on when the night amount is above 0.3, and off otherwise. The night amount uses the same formula as `SkyIslandLife.lua:482-488`. The lantern glass is Neon while on and Glass while off.

### E. Plot size and price (tied to the [[Stage Model]])
1. **The plot size level survives rebirth** (integrated default, decision D-020; [[Rarity Gating and Rebirth]] rule 15 and its code plan now say the same). Rebirth keeps `upgrades.plotSize` and does not shrink the pen. This amends the reset list in [[Decisions Log]] D-009 and Stage Model rule 4. The reason is measured in Numbers, table "Plot Size price": under today's reset, from stage 4 on the whole ladder is bought again inside the first 0.1 to 0.8 minutes of the stage, so the reset is churn with no decision in it.
2. With E1, prices stay as today: 2.5K, 20K, 160K, 1.28M (`GardenConfig.lua:143`, `:157`). Each level is bought once per career, in stages 1 and 2.
3. **Fallback if D-009 stays as written:** the price of level n in stage s is `round3(f_n × R_s)`, with f = (0.001, 0.003, 0.008, 0.02). `round3` is today's 3-figure rounding (`GardenConfig.lua:161-165`). Stage 9 has no rebirth requirement, so it uses R_9 = round3(F_9 × R_8 / F_8) = 281Oc (the same 11,298 seconds of full-plot income as stage 8's gate).
4. The pen sign shows exactly the price the server will charge. The server stays the only judge of price, balance and max level (`Upgrades.lua:301-343`).

### F. Studded logs
1. Every round log in Workspace is replaced, except the parked templates in `Workspace["Asset Pack V1"]`. A round log is any instance named `Log1`, `Log2`, `VinesLog1`, `VinesLog2` or `HollowLog`, plus fallen trunks named `Tree` inside `Main Island.Forest.Undergrowth` (ForestBuilder adds them as logs, `ForestBuilder.lua:122-123`).
2. Each replacement is a `StudLog` (Numbers, "Logs") with: length within ±5% of the old log's longest side; pivot position within 0.1 studs; yaw within 1 degree; a flat studded face on top; the same `Anchored`, `CanCollide`, `CanQuery` and `CanTouch` as the old log. Forest logs sit with their bottom on the ground under them. Driftwood keeps its centre height (about −1.95, `ShallowsBuilder.lua:111`).
3. Every old log moves to `ServerStorage.RoundLogs_backup_2026-10-04` with a `WasIn` attribute. After the swap, the count of round logs outside `Asset Pack V1` is 0.
4. `ForestBuilder` and `ShallowsBuilder` read their `LOGS` lists from `PlotKit.StudLog*` instead of the round templates (`ForestBuilder.lua:122-123`, `ShallowsBuilder.lua:87`). A future re-run builds studded logs. The `Asset Pack V1` templates are not renamed or removed, because ForestBuilder looks them up by exact name when it loads (`ForestBuilder.lua:70-129`).
5. This follows D-003 exactly: a square studded "trunk-style" log, Earth orange body with `Studs` on its four long faces, Burlap end grain, one per round log at the same length, angle and resting height. D-003 picked this over the Studded Pack's `Logs` after a side-by-side ([[World Tour]], log-options.jpg: the pack log rendered grey and read like a bench). Plain Parts with the `Studs` MaterialVariant, no `Texture` objects, because textures do not batch ([[Roblox Platform Constraints for the Expansion]], section 3). The builder flag that switches to the pack log stays (D-003). (This note first rebuilt the pack log instead; aligned with D-003 in the [[Integration Review]], conflict C44.)

### G. Studded lily pads
1. **The look (decision D-021, from D-012):** a studded pad is Samarth's original lily pad mesh (`Lilypad1` / `Lilypad2`, meshes 6885083511 / 6885085620) with `Material = Plastic` and `MaterialVariant = "Studs"`. The [[World Tour]] side-by-side showed that this keeps the notched pad shape and gets real studs, while a pad built from studded plates looks jagged. Any `Lilypad1` or `Lilypad2` under Workspace (outside `Asset Pack V1`) gets that material in place.
2. Every pad in `ServerStorage.LilyPads_backup_2026-09-28` (766 pads, `ShallowsBuilder.lua:226-231`) gets the same material inside the backup. An undo of the Shallows then restores studded pads, never plain ones.
3. **Where (amends D-012's "restore at their backed-up positions"):** the 766 sea pads stay in the backup. Samarth asked on 29 Sep to remove them from the sea and replace them with something better (NOTES.md 29 Sep), the sea is now the swim and boat route ([[Boats and Swimming]]), and the Shallows stones, driftwood and islets took their place. The live count is 0. As the default (Open question 1), the builder places 19 studded pads (the same mesh and material) in the island's calm water: 5 in each of the 3 `PondWater` parts and 4 in the `FallPool`. Pads are at least 4 studs apart, 2.5 studs from the pond stones and 3 studs from bridges.

### H. Border mountains
1. `Main Island.Mountains` (36 sector models, 11,588 parts) moves to `ServerStorage.Mountains_v3_backup_2026-10-04`. It is not deleted.
2. The new ring is built from the Studed Mountain pack (`ServerStorage.AssetQuarantine.Q_105342635739603`). It has at most 9,000 parts in total.
3. Every mountain part has `CanCollide`, `CanQuery`, `CanTouch` and `CastShadow` false, and is anchored. Players cannot reach them: the forest's invisible wall at r 1470 stays (`GardenConfig.lua:1268`).
4. Nothing of the new ring is closer than r 1520 to the map centre. No part is named `CurrentPlate`, `Seabed` or `ReturnPlate` (`PlayerVision.lua:141-152` reads those names).
5. Copies are grouped into 36 models of 10 degrees each, `ModelStreamingMode = Persistent`, as today (`tools-mountains.lua:269`), so the horizon never streams out.
6. From the plaza, the beach at 4 bearings and the sky island deck, no gap shows empty void below the skyline.

### I. Landmarks
1. Two Statue With Sword copies stand on the plaza side of the Royal Gate at (0, 0.12, 150) (`RoyalCapitalBuilder.lua:1658`), 8 studs either side of the road's centre line, at r 141. They live in `Main Island.Landmarks`, never inside the `Royal Gate` model, because a RoyalCapitalBuilder re-run deletes and rebuilds that model (`RoyalCapitalBuilder.lua:1650-1653`). Their `Fire` effect is disabled by default.
2. The pond bridges in `Main Island.Nature` are replaced by Stud Bridge copies scaled to the same span. The builder lists every `Nature` child whose name contains "Bridge" and stops without changing anything if it finds a number other than 2 (NOTES.md 29 Sep records four bridge ends).
3. Proposed sites for the other packs (the owning system can move them): a forest camp (Studded Pack House, Tent, CampFire, Logs, one Mushroom, one Stud Shop as the tamer's booth) for [[Pets]]; four more Mushrooms in the forest clearings nearest the camp; one Stud Shop as the dock booth and one Stud Bridge as the gangway for [[Boats and Swimming]].

### J. Colour and building
1. Every new colour is read from a named source part at build time (Numbers, "Palette"). The fallback RGB is used only if the source is missing, and the builder prints a warning when it does.
2. Every new colour goes through `SaturationPass.lift` and the part stores `SatOrigColor`, like `ShallowsBuilder.lua:446-462`. Nobody runs `SaturationPass.apply()` over the whole map (NOTES.md 29 Sep: it would saturate some parts twice).
3. Every new part uses `Material = Plastic` with `MaterialVariant = "Studs"`, like the map's fences (`Upgrades.lua:68-69`). The mountains keep the pack's own stud surfaces.
4. Every builder: refuses to run outside place 88602028667228; uses a fixed seed; stamps `Builder`, `Seed` and `Version` attributes; moves what it replaces into a dated ServerStorage backup with `WasIn`; prints its part counts; and has an `undo()`.

### K. UI
1. This system adds no ScreenGui, no HUD element and no ProximityPrompt.
2. It adds two in-world SurfaceGuis: the pen sign (owner only, rebuilt from kit images) and the gate nameplate. Both use FredokaOne, white text and a black stroke.

## Numbers

### Palette (every colour has a source)
| Token | Used for | Source read at build time | Fallback RGB (cited) |
|---|---|---|---|
| Wood | posts, rails, beam, gate step, sign | Studded Pack `Logs`, side colour | 119, 76, 47 (authored fence, `Upgrades.lua:116`) |
| WoodDark | doors, crate bands, nameplate back, bed boxes | Studded Pack `Logs`, darkest colour | 84, 49, 28 (authored kerb, `Upgrades.lua:118`) |
| EndGrain | log end caps | BrickColor Burlap (D-003) | 199, 172, 120 |
| Leaf | hedges, lily pads | Studded Pack studded tree, leaf colour | 78, 138, 52 (MOSS, `tools-mountains.lua:196`) |
| LeafLight | hedge trim | Studded Pack studded tree, lighter leaf | 63, 166, 28 (GRASS, `tools-mountains.lua:195`) |
| StoneDark | border base, pillar bodies, hut walls, path edges | Studed Mountain "Smoky grey" parts | 88, 86, 92 (ROCK 2, `tools-mountains.lua:194`) |
| StoneLight | border cap, pillar caps, path, stepping stones | Studed Mountain "Fossil" parts | 198, 199, 202 (SNOWC at 80% value, proposal) |
| Lawn | lot lawn | `Main Island.Base.GrassTop` | 63, 166, 28 |
| Soil | soil (unchanged) | the soil itself | unchanged |
| Furrow | furrow ridges | Soil at 82% value | derived |
| Lamp | lantern glass, padlock | `Config.UpgradeStatColor` | 255, 214, 92 (`GardenConfig.lua:269`) |
| Banner | gate banners | `Config.RarityColors[tier]` | `GardenConfig.lua:582-592` |
| Flowers | bed flowers, pad flowers | `Config.RarityColors` MYTHIC, LEGENDARY, ETERNAL, TRANSCENDENT | same |

### Pen per level (computed from `Config.PlotSizes`, `GardenConfig.lua:117-123`)
"Pen outer" includes the pillar caps (5.1 studs outside the soil on every side). Clearances are measured to the forest edge (r 259.5) and to the nearest road edge. Today's part count (soil + pad + posts + rails + kerbs) comes from running today's `buildFence` logic (`Upgrades.lua:160-228`).

| Level | Soil (W x L) | Pen outer | Pen front a | Pen back a | Clear to forest | Clear to road at front corner | Front yard depth | Gate path length | Furrows | Front posts per side | Pen parts | Today's parts | New plot total (pen + lot + soil + pad) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 0 | 29.5 x 47.1 | 39.7 x 57.3 | 172.2 | 229.5 | 29.2 | 20.4 | 26.4 | 29.7 | 6 | 0 | 60 | 37 | 186 |
| 1 | 34.1 x 54.8 | 44.3 x 65.0 | 168.3 | 233.3 | 25.2 | 17.2 | 22.5 | 25.8 | 8 | 1 | 66 | 37 | 192 |
| 2 | 38.8 x 62.6 | 49.0 x 72.8 | 164.4 | 237.2 | 21.0 | 13.9 | 18.6 | 21.9 | 9 | 1 | 67 | 46 | 193 |
| 3 | 43.4 x 70.3 | 53.6 x 80.5 | 160.6 | 241.1 | 17.0 | 10.7 | 14.8 | 18.1 | 10 | 2 | 72 | 46 | 198 |
| 4 | 48 x 78 | 58.2 x 88.2 | 156.7 | 244.9 | 12.9 | 7.4 | 10.9 | 14.2 | 11 | 2 | 73 | 52 | 199 |

Today's level-4 pen measured 15.2 studs clear of the forest and 10.0 clear of the nearest road (NOTES.md, plot-size entry). The new pen is about 2.3 studs wider per side because of the hedges and pillars, so both clearances shrink but stay positive.

Formulas (hw = W/2, hl = L/2, all in the plot frame):
- Pen outer half-extents = (hw + 5.1, hl + 5.1).
- Front posts per side = max(1, round((hw − 9.98) / 4.5)) − 1.
- Furrows = floor((W − 2) / 4), evenly spaced across the soil, each running the soil's length minus 2.
- Pen parts = 54 + 4 × (front posts per side) + furrows.

### Pen pieces
| Piece | Count | Size | Where | Top | Colour |
|---|---|---|---|---|---|
| Border base | 5 (2 sides, back, 2 front halves) | 1.6 thick | inner face on the soil edge; the front is split by the gate gap | S + 0.6 (bottom S − 0.5) | StoneDark |
| Border cap | 5 | 2.0 thick, 0.4 tall | 0.2 over the soil, 0.2 past the base | S + 1.0 | StoneLight |
| Gate step | 1 | 19.36 x 1.6 deep | in the gate gap of the border | S + 0.3 | Wood |
| Furrow ridge | 6 to 11 | 0.6 wide, (L − 2) long | across the soil, every ~4 studs | S + 0.12 | Furrow |
| Hedge | 3 (left, right, back) | 2.4 thick; sides L + 4.6 long, back W + 4.6 | inner face 2.4 outside the soil edge | S + 2.9 | Leaf |
| Hedge trim | 3 | 2.6 thick, 0.4 tall | on each hedge | S + 3.3 | LeafLight |
| Corner pillar (body + cap) | 4 x 2 | body 2.6 x 2.6; cap 3.0 x 0.5 x 3.0 | centres at (±(hw + 3.6), front and back) | S + 4.4 | StoneDark / StoneLight |
| Gate pillar (body + cap) | 2 x 2 | body 2.6 x 2.6; cap 3.0 x 0.5 x 3.0 | centres at x ±10.98, on the front fence line | S + 7.4 | StoneDark / StoneLight |
| Gate lantern (glass + roof) | 2 x 2 | glass 1.4 x 1.6 x 1.4; roof 1.8 x 0.3 x 1.8; PointLight | on each gate pillar cap | S + 9.3 | Lamp / WoodDark |
| Gate beam | 1 | 20.56 x 1.4 x 1.0 | between the gate pillars, ends 0.6 into them | S + 6.7 (bottom S + 5.3, world 5.8) | Wood |
| Nameplate | 1 | 8.4 x 1.35 x 0.2 | flush on the beam's plaza face, centred | S + 6.65 | WoodDark |
| Banner | 2 | 2.2 x 4.0 x 0.15 | flush on each gate pillar's plaza face | S + 5.4 (bottom S + 1.4) | Banner |
| Front rail | 4 | 0.6 x 0.8 | gate pillar to corner pillar, each side, two heights | centres S + 0.8 and S + 2.0 | Wood |
| Front post (body + cap) | 0 to 2 per side, x 2 parts | body 1.0 x 1.0; cap 1.3 x 0.4 x 1.3 | evenly along each front run | S + 3.2 | Wood / WoodDark |
| Gate path + 2 edges | 3 | path 8 wide; edges 0.6 wide | from the gate step to a = 146 | path S − 0.18, edges S − 0.08 | StoneLight / StoneDark |
| Front bed (box + 2 flowers) | 2 x 5 | box 6 x 0.9 x 3 deep; flower = stem 0.4 x 1.0 x 0.4 + bloom 1.0 x 0.5 x 1.0 | x ±19.5 to ±25.5, starting 1 stud in front of the front fence | box S + 0.4 | WoodDark / Flowers |

The world gap under the gate beam is 5.8 studs. Path and stepping-stone tops (world 0.32) match the road tops (NOTES.md: roads top 0.32), and sit 0.12 above the lawn, which avoids the flicker that 0.02 to 0.05 gaps caused on the old ring paths (NOTES.md 20 Sep).

### Lot layout (static, per plot)
| Item | Parts | Where (x, a) | Notes |
|---|---|---|---|
| Central lawn | 1 | x −33.4 to +33.4, a 146 to 250 (66.9 x 104) | top S − 0.30 |
| Wedge lawns | 2 (WedgeParts lying flat) | right angle at (±33.4, 250); legs 104 along a and 27.9 across | the long edge runs parallel to the road, 0.5 inside its edge |
| Studded trees | 3, at most 25 parts each | (+41, 200), (−41, 200), (−46, 239) | picked from the Studded Pack's 3 studded trees by seed, yaw jitter ±6 degrees |
| Harvest crates | 3 x 3 | cluster centred at (−42, 222) | box, band, fruit-red top |
| Stepping stones | 8 | x +31.7 at a 156, 168, 180, 192, 204, 216; then (+36, 222) and (+40, 226) | 2.4 x 0.3 x 2.4, top S − 0.18 |
| Underground entrance | at most 29 | x +40 to +50, a 229 to 243; door on the front face (a 229) | see "Underground entrance" |
| Reserved tower slot | 0 | x −37 to −55, a 230 to 248 | holds the third tree until a tower exists |
| **Lot total** | **at most 124** | | |

Lot lawn area: 9,851 square studs per plot. The whole plot band is about 144,000 square studs after roads, so lawns cover about 82% of it. The rest stays grey plaza (the walkway ring and the slivers by the roads).

```
                 forest edge, r 259.5
   road \   _______________________________________   / road
         \ | tower slot |   back hedge      | hut  | /      a 250
          \| [tree]     |+-----------------+| door |/
           \ [crates]   || furrowed soil   || [==] /
            \           || (fruit)         ||  o  /        o = stepping stones
             \ [tree]   ||                 || o [tree]
              \         |+--P   [gate]   P--+ o /          P = gate pillar + lantern
               \  [bed]  plaque on beam      [sign] [bed]
                \           == stone path ==     /
                 \___________________________________/    a 146
                    grey walkway ring, r 129 to 146
```

### Underground entrance (static, per plot)
| Part group | Parts | Notes |
|---|---|---|
| Walls (back, left, right, two front pieces) | 5 | StoneDark, 5.5 tall |
| Door frame (2 posts + lintel) | 3 | Wood |
| Roof (3 stepped slabs) | 3 | Wood, WoodDark |
| Doors (2) + padlock (1) | 3 | WoodDark doors, Lamp-coloured padlock; hidden when open |
| Lantern over the door (bracket, glass, roof) | 3 | glass tagged `PlotLamp`; lit only when open and at night |
| Mine-cart rail hint (2 rails, 3 sleepers) | 5 | StoneDark, Wood |
| Interior floor + top stair step | 2 | dark floor |
| `LiftAnchor` marker + `DoorAnchor` attachment holder | 2 | invisible; [[Underground and Ores]] places its Mine Lift cage, depth lamps, MINE sign and prompts here (D-023) |
| Spare | 3 | |
| **Total** | **29** | |

The look follows [[Mining and Ore Merging - Reference Games]] section 7 and proposal 8: studded stone walls, a hanging lantern with a warm PointLight, no shadows on the light.

### Part budget per plot and for the whole environment
| Item | Today | New | Change |
|---|---|---|---|
| One plot at level 0 | 37 | 186 | +149 |
| One plot at level 4 | 52 | 199 | +147 |
| 12 plots, all level 4 (worst case) | 624 | 2,388 (cap 2,460 = 205 x 12) | +1,764 |
| Border mountains (Persistent) | 11,588 | 8,208 (cap 9,000) | −3,380 |
| Round logs | about 1,082 parts ([[Creator Store Packs]]) | about 1,700 (cap) | +618 |
| Lily pads (live) | 0 expected | 19 mesh pads, 19 parts | +19 |
| Landmarks (statues 16, bridges at most 44, camp at most 500, mushrooms at most 300) | 0 | at most 900 | +900 |
| **Net** | | | **about −79 parts; Persistent parts −3,380** |

The lot builder fails if one plot's static parts plus its level-4 pen pass 205.

### Plot Size price, tied to the [[Stage Model]]
Measured with a copy of the model (`~/fruit-empire/sim/economy.py`, `simulate_stage`) that adds plot size as a pure cash sink, bought with the same "cheap next to the next Spawn Rarity level" rule the model uses for capacity and spawn rate. Median of 7 careers, attentive player, pets and ores on. The model has no area limit, so this measures cost only. The scratch script is `plotsize_sim.py` in this session's scratchpad.

| Option | Career length | Stage 1 | Stage 2 | When levels are bought | From stage 4 on |
|---|---|---|---|---|---|
| No plot size at all (baseline) | 10.84 h | 14.8 min | 34.3 min | - | - |
| Today: flat prices, reset every rebirth | 10.89 h | 16.2 min | 38.2 min | every stage | whole ladder re-bought in 0.1 to 0.8 min |
| **E1: flat prices, kept through rebirth (recommended)** | **10.89 h** | **16.2 min** | **38.1 min** | L1 at 3.8 min and L2 at 10.9 min of stage 1; L3 at 9.6 min and L4 at 12.4 min of stage 2 | never re-bought |
| E3 fallback: f = (0.001, 0.003, 0.008, 0.02) x R_s, reset | 11.06 h | 14.9 min | 34.6 min | from about a quarter to a third of each stage | about 30 to 45 min into each stage |
| Rejected: f = (0.01, 0.03, 0.08, 0.20) x R_s, reset | 12.87 h | 15.9 min | 36.2 min | | adds about 2 h to a career |

E1 prices against the stage table: L1 2.5K = 4.4% of R_1 (56.5K); L2 20K = 35% of R_1; L3 160K = 0.15% of R_2 (104M); L4 1.28M = 1.2% of R_2.

Why the reset is churn, as seconds of full-plot income (F_s):

| Stage | F_s | L1 2.5K | L2 20K | L3 160K | L4 1.28M |
|---|---|---|---|---|---|
| 1 | 707 | 3.5 s | 28 s | 226 s | 1,810 s |
| 3 | 3.74B | under 0.001 s | under 0.001 s | under 0.001 s | under 0.001 s |
| 6 | 2.49Qi | under 0.001 s | under 0.001 s | under 0.001 s | under 0.001 s |
| 9 | 24.9Sp | under 0.001 s | under 0.001 s | under 0.001 s | under 0.001 s |

E3 fallback worked values (`round3(f_n x R_s)`):

| Stage | R_s | L1 (0.1%) | L2 (0.3%) | L3 (0.8%) | L4 (2%) |
|---|---|---|---|---|---|
| 1 | 56.5K | 56.5 | 170 | 452 | 1.13K |
| 3 | 8.11T | 8.11B | 24.3B | 64.9B | 162B |
| 6 | 13Sx | 13Qi | 39Qi | 104Qi | 260Qi |
| 9 | 281Oc (derived, Rule E3) | 281Sp | 843Sp | 2.25Oc | 5.62Oc |

### Banner colour by stage (`Config.RarityColors`, `GardenConfig.lua:582-592`)
| Stage | Rarity | Banner RGB |
|---|---|---|
| 1 | Common | 151, 169, 189 |
| 2 | Uncommon | 78, 194, 101 |
| 3 | Rare | 78, 164, 230 |
| 4 | Epic | 170, 93, 222 |
| 5 | Legendary | 242, 171, 48 |
| 6 | Mythic | 232, 66, 60 |
| 7 | Astral | 71, 222, 217 |
| 8 | Eternal | 237, 81, 195 |
| 9 | Transcendent | 252, 247, 227 |

### Border mountains
The pack mountain is 98 x 81 x 98 studs with 768 parts ([[Creator Store Packs]]). `tools-mountains.lua` is `~/fruit-empire/tools-mountains.lua`, the script that built today's ring. Rows reuse two of its three rows (`tools-mountains.lua:36-40`). Today's front row (heights 30 to 75) is dropped: the forest's terrain ridge already rises 30 to 110 studs at r 1440 to 1540 (NOTES.md, forest entry), so those peaks were hidden behind it.

| Row | Copies | Radius | Height h | Scale s = h / 81 | Footprint 98 x s | Detail | Parts |
|---|---|---|---|---|---|---|---|
| Back | 36 (one per 10-degree sector, bearing 10k + 5 ± 3 degrees) | 1700 to 1770 | 180 to 300 | 2.22 to 3.70 | 218 to 363 | the 6 tallest are hero copies; 30 LOD | 6 x 768 + 30 x 54 = 6,228 |
| Middle | 36 (bearing 10k ± 3 degrees) | 1590 to 1650 | 90 to 170 | 1.11 to 2.10 | 109 to 206 | LOD | 36 x 54 = 1,944 |
| Ground skirt | 36 slabs, one per sector | 1530 to 2000 | tops 2.0 and 2.4, alternating so neighbours never share a plane | - | 10 degrees | - | 36 |
| **Total** | | | | | | | **8,208** |

- **LOD copy (proposal):** the builder slices the recoloured pack into 27 bands of 3 studs (pack space). Each band becomes 2 studded boxes, one square and one turned 45 degrees, sized to 92% of the band's footprint, in the band's main colour. That keeps the pack's exact stepped silhouette at 54 parts. The builder prints the real count; if it is not 54, it recomputes the row counts to stay at or under 9,000.
- **Colours** use today's ring so the skyline stays familiar (`tools-mountains.lua:194-204`): moss (78, 138, 52) below world height 34 ± 24; snow (248, 249, 252) above 142 ± 36; between them the four rock shades (74, 80, 92), (88, 86, 92), (99, 95, 98), (115, 111, 116), darker lower down. The ± ranges come from the same noise as today.
- Back-row spacing is about 303 studs of arc against an average footprint of about 290, so neighbours meet near the base. The skirt covers the ground the old columns covered, so the sky island never looks down onto void.

### Logs
| Source folder | Round-log parts today | Kept from the old log |
|---|---|---|
| `Forest.Undergrowth` | about 600, plus fallen `Tree` trunks | bottom on the ground below |
| `Shallows.Driftwood` | 423 (175 logs, NOTES.md 29 Sep) | centre height about −1.95 |
| `CoastDecor2` | 37 | ground contact |
| `OuterRingV1` | 22 | ground contact |
| `NaturePack` and anywhere else in Workspace | counted by the builder | ground contact |

`StudLog` = a body (L x T x T, T = the old log's thickness, at least 1.2; BrickColor Earth orange, `Studs` on the four long faces, D-003) + 2 end caps (0.2 thick, inset 0.2, EndGrain) = 3 parts. D-003's darker ring is left out to hold the 3-part budget (about 1,700 parts for all logs); it can be added as one more part per end if the side-by-side shows it is needed. `VinesLog` sources add 2 moss plates (1.6 x 0.3 x T, Leaf). `HollowLog` sources (20 studs long) add 1 dark inset plate on one end (WoodDark at 50% value). Long or short variant by old length: 8 studs or more uses the long proportions.

### Lily pads
A studded pad is the original mesh pad (`Lilypad1` or `Lilypad2`) with `Material = Plastic` and `MaterialVariant = "Studs"` (D-021, the prototype in [[World Tour]]). Scale 0.9 to 1.3 by seed for the 19 pond pads. 25% of pond pads also get a flower (bloom 1.0 x 0.5 x 1.0 in Flowers pink or white, plus a 0.4 cube in Lamp gold). 1 to 3 parts each. Pond pads sit with their top 0.15 above the water part's top and keep `CanCollide` on, like the old pads. The earlier crossed-plates `StudLilyPad` is dropped.

### Lights
| Light | Count | Range | Brightness | Shadows | On when |
|---|---|---|---|---|---|
| Gate lanterns | 24 (2 per plot) | 10 | 1 | off | night amount above 0.3 |
| Entrance lanterns | 12 | 12 | 1 | off | entrance open and night amount above 0.3 |

Night is 30 s of each 330 s day (`GardenConfig.lua:1475-1479`), so lamps are on for roughly 9% of play.

## UI

This system adds nothing to the HUD. No ScreenGui, button or prompt is added, so nothing can overlap the HUD zones in [[UI Kit Inventory]]. Both new pieces are SurfaceGuis on 3D parts, built from kit images and the kit font, following [[Consistent Image-Based UI Kits in Roblox]] (A2: clone kit pieces, never draw Frames).

### 1. Pen Upgrade sign (owner only)
Today's sign is code-drawn: flat Frames, a gradient button and hand-drawn bracket icons (`PenUpgradeSign.lua:90-170`; the code map lists it as "not kit"). It is rebuilt from the Plot Size card the Upgrade panel already owns.

| Element | Kit component | Size and place on a 640 x 998 canvas | States |
|---|---|---|---|
| Board (3D) | new studded `PenSignBoard` in PlotKit, saved as `ReplicatedStorage.PenSignTemplate` | face 3.84 x 5.99 studs (aspect 320/499, `Config.UpgradeCardAspect`, `GardenConfig.lua:254`); 2 Wood posts; thin on local X, display face = Right, WorldPivot at the foot, no PrimaryPart (same contract as today, NOTES.md sign entry) | - |
| Card | Plot Size card art `rbxassetid://105075774667649` (`Config.UpgradeCards.plotSize`, `GardenConfig.lua:214`) | full canvas | always |
| Window | ViewportFrame, the card's open middle | position (0.5, 0.45), size (0.84, 0.42), same as `UpgradeMenu.lua:101-102`; shows only the owner's soil and `PenPiece` parts, framed with the camera maths in `UpgradeMenu.lua:150-173` | rebuilt when `Upgrade_plotSize` or `PlotName` changes |
| Level row | FredokaOne, white (`Config.UpgradeLevelColor`), black stroke 6 px | y 0.655, h 0.052 (`Config.UpgradeRows`, `GardenConfig.lua:255-259`); fixed TextSize 44, no TextScaled | `Config.UpgradeLevelText`, e.g. "Level 2/5" |
| Stat row | same font, gold (`Config.UpgradeStatColor`) | y 0.712, h 0.052; TextSize 44 | next size from `Config.UpgradeStat("plotSize", level + 1)`, e.g. "39 x 63 studs"; current size at max |
| Price row | same font, green (`Config.UpgradePriceColor`) or red (`Config.UpgradeUnaffordable`) | y 0.769, h 0.055; TextSize 46 | green when cash covers it, red when not, hidden at max |
| Buy | ImageButton, `Config.UpgradeButtonImage` `rbxassetid://112539925904292` | centre (0.5, 0.885), size (0.8719, 0.1463) (`Config.UpgradeButtonRect`) | press scale 0.92, hover 1.05, the same tween numbers as today (`PenUpgradeSign.lua:214-224`) |
| Maxed | ImageLabel, `Config.UpgradeMaxedButton` `rbxassetid://76326716689640` | same centre, width 0.8719, height 0.1605 (`Config.UpgradeMaxedHeight`) | shown only at max, Buy hidden |

- Placement: 6.5 studs out from the soil's front edge (unchanged) and 16 studs to the visitor's right (was 13). That keeps the board 1.8 studs clear of the right gate pillar and its banner, and 1.6 studs clear of the right flower bed.
- The ground raycast in `PenUpgradeSign.lua:286-288` lands on the lawn at world y 0.20, inside its "within 1.5 studs of 0" test, so the foot is still trusted.
- Hidden state: no plot, or the soil has not streamed in (unchanged behaviour, `PenUpgradeSign.lua:257-265`).
- Overlap: the sign is in the world, 1.6 studs in front of the fence line. It does not block the gate or the path.

### 2. Gate nameplate
| Element | Kit component | Size | States |
|---|---|---|---|
| Plate background | `CarryInfo.NameBar` `rbxassetid://123470450245311` (168 x 27, [[UI Kit Inventory]]) | full canvas 840 x 135 on the 8.4 x 1.35 stud plate (same 6.2 aspect) | - |
| Name | TextLabel, FredokaOne, white, black UIStroke 5 px | inset 6% left and right, 80% tall; TextScaled with a UITextSizeConstraint of 40 to 80 (one label, so every name fills the plate the same way) | owner's DisplayName; SurfaceGui disabled when unowned |
- Two SurfaceGuis: one on the plate's plaza face, one on the beam's pen face, so the owner reads it from inside too.
- `LightInfluence` 0.2, `MaxDistance` 120. The owner headshot (25 studs above the soil centre, `GardenConfig.lua:298`) floats far above the plate, so the two never stack on screen from the plaza.

### 3. Prompts
- None added here. The entrance's purchase and descent prompt belongs to [[Underground and Ores]]. Recommendation for that note: use a key other than E, because every fruit uses E (`FruitFactory.lua:238-250`) and only one prompt per key shows by default ([[Roblox Platform Constraints for the Expansion]], section 9).

### 4. New art
None. Every image above is already uploaded and in use.

## World and assets

![[pack-studded-pack.jpg]]
![[pack-studded-logs-closeup.jpg]]
![[pack-studded-mountain.jpg]]

| Thing | Source (already in the place) | Copies | Built by | What must be built |
|---|---|---|---|---|
| Pen pieces (border, hedges, pillars, fence, gate, beam, plate, banners, path, beds, furrows) | new studded Parts in `ServerStorage.PlotKit`, colours from the Palette sources | 60 to 73 per plot | runtime `buildPen` | the PlotKit templates |
| Lot lawn, crates, stepping stones | new studded Parts in PlotKit | 3 + 3 + 8 per plot | `PlotLotBuilder` | templates |
| Lot trees | Studded Pack (`AssetQuarantine.Q_86601833587701`), 3 studded trees | 3 per plot | `PlotLotBuilder` | nothing |
| Underground entrance hut | new studded Parts in PlotKit | 1 per plot | `PlotLotBuilder` | the hut template |
| Pen sign board | new studded Parts in PlotKit, saved over `ReplicatedStorage.PenSignTemplate` | 1 (cloned per owner client) | `PlotKitBuilder` | the board |
| Studded logs | Studded Pack `Logs` (shape and colours) | one per round log | `StudLogSwap` | `StudLog` templates |
| Studded lily pads | the original pad meshes with the `Studs` MaterialVariant (D-021) | 19 live in the ponds + 766 restyled in the backup | `LilyPadSwap` | nothing new |
| Border mountains | Studed Mountain (`AssetQuarantine.Q_105342635739603`) | 72 copies + 36 skirt slabs | `BorderMountainsBuilder` | the LOD template |
| Royal Gate statues | Statue With Sword (`AssetQuarantine.Q_4697087599`) | 2 | `LandmarksBuilder` | nothing; `Fire` disabled |
| Pond bridges | Stud Bridge (`AssetQuarantine.Q_71814553242028`) | 2 | `LandmarksBuilder` | nothing |
| Forest camp | Studded Pack House, Tent, CampFire, Logs, Mushroom; Stud Shop (`AssetQuarantine.Q_12340318191`) | 1 set | `LandmarksBuilder` | a 45-stud clearing; trees there move to backup |
| Forest mushrooms | Studded Pack Mushroom | 4 | `LandmarksBuilder` | nothing |
| Grow puff | `Anime` effect pack, `Smoke-01` emitters ([[Existing VFX and Props]]) | 4 bursts per purchase | copied to `ReplicatedStorage.PlotKitFx.GrowPuff` by `PlotKitBuilder` | nothing |
| Lantern light | PointLight on lantern glass | 36 | PlotKit | nothing |

- Sounds: none new. The garden stays quiet; the music system is unchanged.
- The packs are read from `ServerStorage.AssetQuarantine`, never from `Workspace._AssetViewer`, which is a temporary viewer copy ([[Creator Store Packs]]).
- The Studded Pack's 3 basic (non-studded) trees are not used.
- The camp's default site is due south (bearing 180 degrees, the side where the old dock stood), at r about 1,110, inside the forest edge band (r 1,072 to 1,140). If `Main Island.Forest.Clearings` exists, the builder uses the clearing nearest that bearing instead (`ForestBuilder.lua:380-404`). [[Pets]] and [[Boats and Swimming]] may move it.

![[pack-statue-with-sword.jpg]]
![[pack-stud-bridge.jpg]]
![[pack-stud-shop.jpg]]

## Asset requests for Samarth

- **Images:** none. Every UI piece reuses an uploaded kit image (the Plot Size card, the Buy and Maxed buttons, the Name bar).
- **Animations to publish:** none.
- **Robux products:** none.
- **Publishing (only he can do it):** after the builders run and he has looked at the screenshots, delete `Workspace._AssetViewer`, then File > Save to Roblox, then File > Publish to Roblox.
- **Decisions:** the open questions at the end, especially 1 (which lily pads) and 2 (plot size survives rebirth).

## Code plan

The pattern: GardenService modules are `return function(G)`, loaded in order, sharing state through `G` (`GardenService.lua:88-98`, `:398-415`). Builders are Edit-mode ModuleScripts in `ServerStorage.ZZTools`.

### New runtime scripts
| Script | Kind | Loaded | What it does |
|---|---|---|---|
| `ServerStorage.PlotPenLayout` | ModuleScript library (plain table) | required by `GardenService.PlotPen` and by the builders | `Layout.pieces(soilCFrame, soilSize, rc, cfg)` returns a list of `{kit, name, size, cframe}` for one pen (the Numbers tables). `Layout.build(parent, pieces, kit)` clones kit templates. One file, so the Edit-mode preview and the live pen can never differ. |
| `ServerScriptService.GardenService.PlotPen` | `return function(G)` | new line after FruitFactory (`GardenService.lua:409`), before Upgrades (`:413`) | Moves `plotFrame` and `soilExtents` here from `Upgrades.lua:142-158` (still exported as `G.plotFrame`, `G.soilExtents`; RebirthLock reads `G.soilExtents`, `RebirthLock.lua:26`). Adds `buildPen(plot)` (Rules B1 to B8) and `nudgeCharacters(plot)` (B7). At load, runs `pcall(buildPen, plot)` and `G.placeSpawn(plot)` for all 12 plots and warns on failure. If `ServerStorage.PlotKit` is missing, keeps today's fence code as `legacyFence` so the game still runs. Exports `G.buildPen`. |
| `ServerScriptService.GardenService.PlotDressing` | `return function(G)` | new line right after Upgrades (`GardenService.lua:413-415`) | `refreshPlotDressing(plot)`: nameplate text, banner colour, entrance state (Rules D1 to D3). On claim it connects the owner's `Rebirths` and `Config.PlotEntranceAttr` change signals and stores them on `plot.dressConns`; on release it disconnects them. `setEntranceOpen(plot, open)`. Exports `G.refreshPlotDressing`, `G.setEntranceOpen`. |
| `StarterPlayer.StarterPlayerScripts.PlotLife` | LocalScript | client start | Every 1 s: `CollectionService:GetTagged("PlotLamp")`, skip parts with no Parent (streamed out), set lights and glass (Rule D4). On `Upgrade_plotSize` going up: clone `ReplicatedStorage.PlotKitFx.GrowPuff` to the 4 corner pillars of your own pen, `Emit(12)`, remove after 2 s. No Heartbeat loop. |

### Changes to live scripts
| File and lines | Change |
|---|---|
| `ServerScriptService.GardenService.PlotRegistry.lua:21-30` (`soilOf`) | Return the `PlotSoil` part first; else the largest-area part without `PlotDecor` (Rule A3). |
| `ServerScriptService.GardenService.Upgrades.lua:50-228` | Delete the fence shape constants, the majority-vote `FENCE_STYLE`, `FALLBACK_COLOR`, `fencePart`, `plotFrame`, `soilExtents` and `buildFence`. They move to PlotPen (fallback only) and PlotPenLayout. Read `G.buildPen` and `G.soilExtents` in the header (`:15-22`). |
| `Upgrades.lua:255-259` (`applyPlotSize`) | `buildFence(plot)` becomes `buildPen(plot)`. After `placeSpawn` add `if G.refreshPlotDressing then G.refreshPlotDressing(plot) end`, because the rebuild made a new nameplate and new banners. Keep the early return at `:244`; the startup rebuild calls `buildPen` directly. |
| `Upgrades.lua:426-430` (exports) | Remove `G.soilExtents` (now exported by PlotPen). |
| `ServerScriptService.GardenService.lua:409-415` | Add `require(script.PlotPen)(G)` before Upgrades and `require(script.PlotDressing)(G)` after it. |
| `GardenService.lua:439` (`claimPlot`) and `:458` (`releasePlot`) | After `setOwnerIcon`, call `G.refreshPlotDressing(plot)`. In `releasePlot` call it after the owner fields are cleared (`:457-460`), so it reads "unowned". |
| `ServerScriptService.GardenService.RebirthLock.lua:95-97` (Rule E1 is the integrated default, D-020; the change is listed once, in [[Rarity Gating and Rebirth]]'s code plan, which rewrites rebirth) | `publishUpgrades(player, nil)` becomes `publishUpgrades(player, { plotSize = levelOf(player, "plotSize") })`, and `applyPlotSize(plot, 0)` is removed. |
| `Upgrades.lua:309-310` (only for fallback E3) | For `plotSize`, read the price from a new `Config.PlotSizePrice(level, stage)` instead of `Config.UpgradePrice`. `PenUpgradeSign.lua:187` reads the same function. |
| `StarterPlayer.StarterPlayerScripts.PenUpgradeSign.lua:48-170` | Replace the drawn Frames, gradient and bracket icon with the kit card, rows, Buy and Maxed images and the Window (UI section 1). |
| `PenUpgradeSign.lua:185-207` (`refresh`) | Fill Level, Stat and Price rows; swap Buy and Maxed; price colour by cash. |
| `PenUpgradeSign.lua:238-247` (`soilOf`) | Same rule as PlotRegistry (A3). |
| `ReplicatedStorage.GardenConfig.lua:2099` | `Config.PenSignSide` 13 becomes 16. |
| `GardenConfig.lua:2102` | `Config.PenSignCanvas` becomes `Vector2.new(640, 998)`. |
| `GardenConfig.lua:2090-2093`, `:2104-2115` | Delete the `PenSign*` title, text and colour keys. Only `PenUpgradeSign.lua` reads them (searched the export). |
| `GardenConfig.lua` new block in the plot section (`:276-300`) | `Config.PlotKitFolder = "PlotKit"`; `Config.PlotLot = { FrontA = 146, BackA = 250, RoadHalf = 5, RoadMargin = 0.5 }`; `Config.PlotPen = {...}` (every number in the "Pen pieces" table); `Config.PlotEntranceAttr = "UndergroundFloors"`; `Config.PlotLampRange = 10`; `Config.PlotNameplateMaxDistance = 120`; `Config.RarityOrder` comes from the [[Rarity Gating and Rebirth]] block (it adds the ordered list), so this note does not define it. |
| `ServerStorage.ForestBuilder.lua:122-123` | `LOGS` reads `PlotKit.StudLogLong` and `PlotKit.StudLogShort`. |
| `ServerStorage.ShallowsBuilder.lua:87` | Same. |

### Builders (Edit mode only, `ServerStorage.ZZTools`)
Run order matters. Each is required as a clone, because Studio caches `require` in Edit mode ([[World Builders and Dev Tools]], gotcha 2):
```lua
local function run(name)
	local m = game.ServerStorage.ZZTools[name]:Clone()
	m.Parent = game.ServerStorage
	local ok, res = pcall(require, m)
	m:Destroy()
	print(name, ok, res)
end
run("PlotKitBuilder") run("PlotLotBuilder") run("StudLogSwap")
run("LilyPadSwap") run("BorderMountainsBuilder") run("LandmarksBuilder")
```
| Builder | Seed | Builds | Backup and undo |
|---|---|---|---|
| `PlotKitBuilder` | none (no randomness) | `ServerStorage.PlotKit` (pen, lot, hut, sign board, `StudLogLong`, `StudLogShort`, pad flower), `ReplicatedStorage.PenSignTemplate`, `ReplicatedStorage.PlotKitFx.GrowPuff`. Samples every Palette source and prints any fallback it used. | old `PenSignTemplate` to `ServerStorage.PenSignTemplate_backup_2026-10-04` |
| `PlotLotBuilder` | 20261004 + plot index | per plot: `PlotSoil` on the soil, `Lot`, `UndergroundEntrance`, and a level-0 `Pen` preview made by `PlotPenLayout` (so Edit mode shows the real look); stamps `LotVersion`, `LotSeed`. Asserts Rules A4, A5 and the 205-part cap. | old `Fence`, `FenceFront` and kerbs to `ServerStorage.PlotFences_backup_2026-10-04` |
| `StudLogSwap` | 20261005 | Rule F | `ServerStorage.RoundLogs_backup_2026-10-04` |
| `LilyPadSwap` | 20261006 | Rule G (material swap on the backup pads, 19 new pond pads) | stores each pad's old `Material` and `MaterialVariant` as attributes for undo |
| `BorderMountainsBuilder` | 20261002 (the same seed as today's ring, `tools-mountains.lua:33`) | Rule H; prints parts per sector and the total | `ServerStorage.Mountains_v3_backup_2026-10-04` |
| `LandmarksBuilder` | 20261007 | Rule I; `Main Island.Landmarks` | replaced bridges and cleared trees to `ServerStorage.Landmarks_backup_2026-10-04` |

### Remotes
None new. The sign keeps firing the existing `Upgrade` remote with `"plotSize"` (client to server). The server checks the key, profile, price and balance in `buyUpgrade` (`Upgrades.lua:301-343`).

### Save-document fields
None new. The look lives in the place file. The plot size level is already saved as `upgrades.plotSize` (`Persistence.lua:92-94`) and restored before fruit (`Persistence.lua:276-277`). With Rule E1, old saves keep the level they have now; nothing migrates. Players who lost levels to earlier rebirths keep their current level.

### Attributes and tags
| Name | On | Written by | Read by |
|---|---|---|---|
| `PlotSoil` (bool) | soil part | PlotLotBuilder | `soilOf` (server and sign) |
| `PlotDecor` (bool) | every new pen, lot and hut part | PlotPenLayout, builders | `soilOf` |
| `LotVersion` (number), `LotSeed` (number) | plot model | PlotLotBuilder | builders (skip or rebuild) |
| `PlotSizeLevel` (number) | plot model | existing (`Upgrades.lua:259`) | dev panel |
| `UndergroundFloors` (number 0 to 3, name set by `Config.PlotEntranceAttr`) | player | [[Underground and Ores]] | PlotDressing |
| `Rebirths` (number) | player | existing (`Progression.lua:96`) | PlotDressing |
| `ReplacedLog` (string), `WasIn` (string) | new log, old log | StudLogSwap | undo |
| `ReplacedPad` (string) | new pad | LilyPadSwap | undo |
| `Version`, `Builder`, `Seed`, `Detail` ("Hero" or "LOD") | Mountains folder and copies | BorderMountainsBuilder | builder |
| tag `PenPiece` | every pen part | PlotPenLayout | sign Window |
| tag `PlotLamp` | lantern glass | PlotPenLayout, PlotLotBuilder | PlotLife |

## Edge cases and abuse

- **Stealing is gone and stays gone.** Hedges and gates are scenery, not security. No piece of this system moves, hides or reads anyone's fruit, pets or currency.
- **Buying a size while standing next to the pen.** The hedge moves out by up to 2.3 studs per side per level. Rule B7 moves anyone it would trap. Tested in Test plan 4.
- **Fruit near the edge.** The soil grows about its centre, so fruit stay inside (`Upgrades.lua:236-241`). The border cap overhangs the soil by 0.2 studs, less than the 0.6-stud spawn margin (`GardenConfig.lua:288`), so no fruit sits under it.
- **Lock dome.** It is sized to the soil plus 35 across and 22 along (`RebirthLock.lua:168-175`). The pen's outer edge is 5.1 studs past the soil, so the dome still covers it.
- **Leave or rejoin mid-rebuild.** `buildPen` does not yield, so a leave cannot land in the middle. Release rebuilds the pen at level 0 (`GardenService.lua:456`) and hides the nameplate and banners. The next claim fills them in.
- **Rebirth mid-action.** With E1 the pen does not change on rebirth; only the banners recolour. With the E3 fallback, rebirth runs `applyPlotSize(plot, 0)` (`RebirthLock.lua:97`), which rebuilds the pen and nudges characters.
- **A plotless player** (servers hold 50 players and there are 12 plots, [[Roblox Platform Constraints for the Expansion]]) sees all 12 lots. No dressing, no sign.
- **Exploiters.** No remote is added. The sign is local to its owner; a client that deletes it only hides it from itself. A client firing `Upgrade` directly is checked by the server as today.
- **Crown Hunt decree.** It refuses spots on anything inside `Plots` (`CrownDecree.lua:177-179`). Lawns live inside plot models, so the crown is never hidden on a lot. The hunt still has the plaza ring, the roads and the island grass, and it tries 40 random spots (`CrownDecree.lua:171`).
- **Explosives.** Pen and lot parts are not damageable. TNT targets fruit by distance and `standingOn` (`FruitFactory.lua:79-85`), which still use the soil. Any `Explosion` must keep `DestroyJointRadiusPercent = 0` ([[Roblox Platform Constraints for the Expansion]], section 7) so it never breaks pen models.
- **Mobile.** No new HUD. The sign's Buy area is 3.3 x 0.9 studs, readable and tappable from 15 studs. SurfaceGuis in PlayerGui take taps (today's pattern, `PenUpgradeSign.lua:60-61`).
- **Streaming.** Lot decor streams with its plot model. The sign already re-seats when the soil streams back as a new part (`PenUpgradeSign.lua:303-313`). PlotLife skips streamed-out lamps. Mountains are Persistent.
- **Server authority.** All geometry is built by the server or by builders. Clients only build their own sign and switch their own lamp visuals.
- **Studio require cache and place lock.** Builders must be required as clones and refuse other places, like the existing builders.
- **Re-running SaturationPass.** Forbidden over the whole map (Rule J2). Builders lift their own colours.
- **Place file size.** Backups add parts to ServerStorage (not sent to clients) and count toward the 100 MB place limit ([[Roblox Platform Constraints for the Expansion]], section 3). Delete them one release after Samarth approves.
- **ForestBuilder re-runs.** Re-running a forest sector re-rolls its trees ([[World Builders and Dev Tools]], feature 8), so the log swap replaces logs in place and never re-runs `buildSector`.

## Performance

Limits from [[Roblox Platform Constraints for the Expansion]]: about 500 draw calls and 500,000 triangles in view on 90% of phones (community budget); under 50 KB/s download per client; Persistent models only "in very rare circumstances"; no official part cap; no official numbers for light counts.

| Area | Cost |
|---|---|
| Parts | Net about −79 parts across the environment. Persistent parts (sent to every client at join) drop by 3,380 because the mountains shrink. Per plot: at most 205 parts, 12 plots at most 2,460. |
| Draw calls | All new parts are Plastic with the same `Studs` variant, so identical shapes batch. No new `Texture` or `Decal` objects (the log swap avoids the pack's 672 stud textures). New SurfaceGuis: 1 sign per client, 24 nameplate faces in the world. |
| Lights | 36 PointLights, shadows off, range 10 to 12, on about 9% of the time. |
| Particles | Only the grow puff: 4 one-shot bursts of 12 particles on the buyer's client, at most 4 times per career with E1. |
| Server per frame | Nothing added. `buildPen` runs only on resize, load, release and server start. |
| Client per frame | Nothing added. PlotLife runs a 1 Hz loop over 36 parts. The sign keeps its existing 1 Hz poll. |
| Network | A pen rebuild destroys and creates at most 73 parts once (today about 50 at level 4). Banner colours change once per rebirth. Nameplate text changes on claim and release. No streaming of positions. |
| Physics | Every new part is anchored. Mountains, furrows, banners, nameplate and lantern glass are non-colliding and non-queryable. |

## Test plan

Before any Play test, run `workspace:SetAttribute("StudioNoSave", true)` in the command bar. Without it, a playtest writes the tester's real save (`GardenStore.lua:568-572`).

1. **Builders (Edit mode).** Run the six builders in order. Each prints counts; compare with the Numbers budgets. Run each a second time: counts and positions must be identical (Rule C2, J4). Run each `undo()` once on a copy of the place and confirm the backup comes back.
2. **Soil detection.** In Play (server), for all 12 plots print `plot.soil:GetAttribute("PlotSoil")` and `plot.topY`. Expect true and 0.50 on every plot.
3. **Pen per level.** Use the Studio door `DevSetPlotSize` (`GardenService.lua:657-683`) for levels 0 to 4. After each: count `Pen` descendants (expect the "Pen parts" column); every fruit is inside the soil; the spawn pad is 5 studs inside the soil front; the sign's foot is at y 0.20 ± 0.05 and 16 studs to the right.
4. **Nudge.** Stand 1 stud outside the current hedge, then raise the size one level. Expect your root to be outside every `PenPiece` (`workspace:GetPartsInPart` on the root returns none) and you can walk away.
5. **Walls and gate.** Walk in and out through the gate. Try to walk through a side hedge; you are blocked.
6. **Dressing.** Press Round trip on the dev panel (F4). During the leave the nameplate hides; after the rejoin it shows your display name. Set `Rebirths` on your player to 0, 2, 5 and 8 from the server command bar; the banners show the stage 1, 3, 6 and 9 colours in the table within 1 s. Set `UndergroundFloors` to 1 and back to 0; the doors open and close within 1 s.
7. **Lamps.** Temporarily set `Config.DayCycleOffset` in Studio so night starts within a minute (or wait one 330 s cycle). The 36 lights turn on at night and off by day.
8. **Lock dome.** Press Lock while on your plot; the dome covers the whole pen including pillars.
9. **TNT.** Use today's TNT on your own plot. Fruit behave as before; no pen or lot part is removed.
10. **Crown Hunt.** `RoyalDev:Invoke("decree", "CrownHunt")` ten times ([[World Builders and Dev Tools]], 3e). No crown lands inside a plot model's bounding box.
11. **Sign states.** On the client only, flip `Upgrade_plotSize` and `Cash` to show ready, too poor and max (as NOTES.md did for the old sign), then restore them. A tester presses Buy by hand once (a script cannot press a SurfaceGui button).
12. **Logs.** Command bar: count instances named `Log1`, `Log2`, `VinesLog1`, `VinesLog2`, `HollowLog` (and `Tree` in `Forest.Undergrowth`) outside `Asset Pack V1`; expect 0. Stand on a studded driftwood log and a forest log. `require` a clone of ForestBuilder; it loads without error.
13. **Lily pads.** Count pads with `MaterialVariant = "Studs"` in the ponds (19) and in the backup (766); count pads in the sea (0). Stand on a pond pad.
14. **Mountains.** Count descendants of the new `Mountains` folder (at most 9,000). Screenshot from Plot0, Plot6, the beach at bearings 0, 90, 180 and 270, and the sky island deck: no void under the skyline, no floating pieces. Walk to r 1470; the wall still blocks you.
15. **Landmarks.** Statues stand outside the `Royal Gate` model. Re-run RoyalCapitalBuilder on a copy of the place; the statues survive. Walk across both new pond bridges.
16. **Performance.** At Plot0 facing the plaza, record Studio's render stats (draw calls, triangles) and `Stats` instance count before and after. MicroProfiler shows no new script work above 0.1 ms per frame.
17. **Devices.** Device Emulator at Actual Resolution on a phone and a tablet: the sign is readable at 15 studs and its Buy area can be tapped; the HUD is unchanged.

## Dependencies and conflicts

| System | How it touches this one | What to agree |
|---|---|---|
| [[Rarity Gating and Rebirth]] | Banner colour reads the stage. Rule E1 changes the rebirth reset list (D-009, Stage Model rule 4). | Agreed in the [[Integration Review]]: that note's code plan carries the `RebirthLock.lua:95-97` change and adds `Config.RarityOrder` (D-020). |
| [[Bats]] | Knockback can throw players into hedges and pillars. They are solid and harmless. | Knockback must not move players through solid pen parts; it uses the arena's client push pattern ([[Roblox Platform Constraints for the Expansion]], "Bats, swords and duels"). |
| [[Explosives and Fruit Health]] | TNT on plots. Fruit health bars are BillboardGuis over fruit, inside the pen. | Pen and lot parts are never damaged; any `Explosion` uses `DestroyJointRadiusPercent = 0`; soil detection now uses `PlotSoil`. |
| [[Weapons and Artillery]] | Shells may hit hedges, pillars or trees on lots. Mountains do not collide. | Shells explode on lot parts without damaging them. |
| [[Underground and Ores]] | The entrance hut, its `LiftAnchor` and `DoorAnchor`, and the `UndergroundFloors` attribute. | Agreed in the [[Integration Review]] (D-023): the hut is the one entrance; that note puts its Mine Lift inside it and owns the prompt, the descent and the underground volume. The volume stays under its own lot (a neighbour's mine must never connect, per [[Mining and Ore Merging - Reference Games]], "What to avoid"). |
| [[Pets]] | Pets wander on the soil. The border cap overhangs the soil edge by 0.2 at height S + 0.6. The forest camp site. | Pets keep at least 1 stud from the soil edge. The Pets note confirms or moves the camp. |
| [[Boats and Swimming]] | Studded driftwood keeps the old logs' collision, so boats meet the same obstacles. The dock booth (Stud Shop) and gangway (Stud Bridge) are reserved for it. Lily pads go in the ponds, not the sea. | Boat lanes through the shallows; dock site (the `FillsDockGap` pieces mark the old one, NOTES.md 29 Sep). |
| [[Potions]] | None direct. Feather Falling players may land on gate lanterns or hedges. | Nothing. |
| [[King UI Overhaul]] | Same kit and font rules. The Royal Gate statues stand next to the gate the King UI points at. | The overhaul does not restyle the statues; the statues stay outside the `Royal Gate` model. |
| [[Royal Combat]] | Statue With Sword is also a candidate for the arena entrance ([[Creator Store Packs]]). | Copies are unlimited; this note only uses 2 at the Royal Gate. |

## Open questions for Samarth

1. **Which lily pads do you mean?** All 766 sea lily pads were swapped for stepping stones, driftwood and islets on 28 to 29 Sep after you asked to remove them, so the live count should be 0 (the published game may still show them, because Studio changes since then are unpublished). **Default (D-021):** your original pad meshes with the Studs material, as in the [[World Tour]] prototype; the 766 stay in the backup (restyled), and 19 go in the island's 3 ponds and the waterfall pool. If you want them back in the sea, say so: they would become extra swim rest spots, and the boat channels would need checking.
2. **Keep plot size through rebirth?** This changes D-009 and Stage Model rule 4. Under the reset, from stage 4 on the whole ladder is re-bought in under a minute, and the pen visibly shrinks and regrows every rebirth for nothing. **Default:** keep it (Rule E1, logged as D-020 by the [[Integration Review]]). If you want the reset, the fallback price is 0.1%, 0.3%, 0.8% and 2% of the stage's rebirth cash, which adds about 13 minutes to a career.
3. **Hedges or wooden fence on the sides and back?** **Default:** studded hedges, with the low wooden fence kept at the front so fruit stay visible from the plaza.
4. **Statue fire on or off?** **Default:** off (calm, and one less effect).
5. **Underground entrance location.** **Decided (D-023):** a stone hut in the back-right corner of every lot, outside the pen, with the Mine Lift of [[Underground and Ores]] inside it.
6. **Studded logs: trunk-style (D-003) or the pack's logs?** D-003 already chose the trunk-style log from a side-by-side, and the pack's logs carry 672 stud textures that do not batch. **Default:** the D-003 trunk-style log as plain studded parts; the builder flag can switch to the pack log if you prefer it.
7. **Mountains: 6 full-detail pack copies plus cheaper copies of the same shape, or something else?** **Default:** 6 hero + 66 LOD + skirt (8,208 parts, 3,380 fewer than today). Option to test later: Roblox's model level-of-detail imposters, which would allow more full copies [inferring, not cited].
8. **Stage banners on the gate?** **Default:** yes, two banners in the rarity colour of your newest stage, hidden on empty plots.
9. **Name on the gate: display name or username?** **Default:** display name.
10. **Forest camp and mushrooms.** **Default:** camp due south at r about 1,110 (or the nearest clearing), 4 mushrooms in nearby clearings; [[Pets]] and [[Boats and Swimming]] may move them.
11. **Future tower slot.** **Default:** reserve the back-left corner of every lot (x −37 to −55, a 230 to 248) for the upward base upgrade you mentioned, with a placeholder tree there until it exists.
