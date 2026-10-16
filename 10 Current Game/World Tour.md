---
tags: [current-game, world]
date: 2026-10-04
---
# World tour: how the map looks today (4 Oct 2026)

Screenshots from the Edit view. The HUD (Inventory, Index, Shop, Rebirth, Merge All, Clear, Lock, Upgrade) is the real StarterGui. In Edit it shows $0.

## The plots
![[today-plots-ring.jpg]]
- 12 plots (`Workspace["Main Island"].Plots.Plot0…Plot11`) sit in a ring about 200 studs from the centre. The beanstalk is in the middle, inside a round forest. Brown dirt paths run between the plots like spokes.
- Each plot is a Model with a `Fence`, a `FenceFront` and a few Parts. They come in three footprints: about 49 × 79, 62 × 77 and 64 × 69 studs. Plot0 also has a `PlotSpawn`.

![[today-plot0.jpg]]
- **What a plot is today:** a flat orange-brown dirt rectangle with a two-rail wooden fence, on a pale grey studded plaza. Nothing else stands on it. This is the "default plot" that Samarth wants improved 10×. See [[Plot Overhaul]].

## The border
![[today-border-mountains.jpg]]
- `Main Island.Mountains`: 36 sector models (`Mountains_000…350`), 11,588 parts. They span 4,032 × 4,032 studs and stand up to 275 studs tall: blocky grey peaks with white snow caps.
- These are what Samarth wants replaced with the Studded Mountain pack ([[Creator Store Packs]]). That pack is a single 98 × 81 × 98 mountain with Studs on top, so it would be repeated and scaled around the ring, and recoloured.

## Water and debris
- The sea around the island is full of floating brown logs and lily pads. These are `Main Island.Shallows.Driftwood` (423 log parts) plus lily pads, all round mesh logs from "Asset Pack V1". They are among the round logs to swap for studded ones.
- The currents that carry players to the forest and back are in the code (see the code-map notes on IslandWater and Ambient). They get removed when boats arrive ([[Boats and Swimming]]).

## Loose stock in Workspace
See [[Existing VFX and Props]]: the bat tool, the effect packs under the map, the seasonal decor.

## Logs and lily pads: exact inventory (4 Oct 2026)
- **Round logs on the map: 484** (saved with position, rotation and size in `~/fruit-empire/world/round-logs-2026-10-04.json`):
  - `Forest.Undergrowth` sectors: 288 (Log1 53, Log2 59, VinesLog1 53, VinesLog2 60, HollowLog 63);
  - `Shallows.Driftwood`: 169;
  - `CoastDecor2`: 18;
  - `OuterRingV1`: 9.
  - They run 10–26 studs long; HollowLogs run 17–47.
- **Lily pads on the map today: none.** The 766 mesh lily pads (405 `Lilypad1`, 361 `Lilypad2`, meshes 6885083511 / 6885085620) were moved to `ServerStorage.LilyPads_backup_2026-09-28` on 28 Sep. The only ones left in Workspace are the two source models inside `Asset Pack V1`.
  - Samarth asked to "turn the current lily pads into a studded texture". The likely reason he still sees them: the **published** game is older than the Studio copy. Everything done in Studio since then is unpublished.
  - Plan: rebuild the lily pads as studded pads (flat Parts with Studs on top and a notch cut), placed at the 766 backed-up positions. This keeps his layout and gives the studded look. Check the boat lanes first ([[Boats and Swimming]]).
- The Shallows `Islets` (137 Islets + 11 TreeIslets) are studded rock islands with small green studded tops. They are not lily pads.

### Studded lily pad prototype
![[lilypad-options.jpg]]
*Centre pair: his original lily pad meshes with the place's existing `Studs` MaterialVariant (Material Plastic + MaterialVariant "Studs"). Right: a pad built from 1×1 studded plates. Left edge: the original pads, unchanged.*

The centre pair keeps the classic notched lily-pad shape and gets real studs, which is literally "turn the lily pads into a studded texture". The block-built pad looks jagged at this size. → [[Decisions Log|D-012]]

### Border mountains: today vs the Studed Mountain pack
![[mountain-compare.jpg]]
*Left: the Studed Mountain pack scaled ×3 (293 × 243 × 293 studs, 768 parts), recoloured by height into the island's palette (green base, Camo, grey rock, white snow). Right: today's `Mountains_000` sector (384 × 259 × 560 studs, 290 parts). Both seen from 1,000 studs, about the distance from the beach.*

Facts for the swap:
- Today's border: 36 sectors, **11,588 Plastic parts, every one already has Studs on top**. Colours: Dark grey metallic 5,737, Dark stone grey 1,967, Camo 1,600, Institutional white 811, Dark grey 712, Parsley green 399, Bright green 259, Earth orange 103. The ring sits about 1,740 studs from the centre.
- The pack is one stepped cone. At ×3 it needs about 38 copies to close the ring (about 29,000 parts). At ×4.5 (440 studs wide, 365 tall) it needs about 25 (about 19,000 parts). Mixing 2–3 sizes and rotations stops it looking stamped.
- Far scenery like this suits `ModelStreamingMode = Persistent` (always loaded) together with `LevelOfDetail = StreamingMesh` (one cheap merged mesh at distance). Both are settable from the MCP ([[World Builders and Dev Tools]]).

### Studded log prototype (three looks, same three round logs)
![[log-options.jpg]]
*Back: the round mesh logs (a Log1 and a VinesLog2) and a HollowLog. Middle rows: the Studded Pack's log, stretched to each length, first as-is (grey) and then recoloured brown. Front: a square "trunk-style" log built like the forest's studded tree trunks (Earth orange, Studs on four faces, Burlap end grain with a Dirt brown ring).*

- The pack log is a flat 2 × 1 slab with a light top strip. Its body is "Earth yellow" under the black stud texture at 85% transparency, so it renders grey and reads like a bench or stone slab.
- The trunk-style log uses the same material and colour as the studded tree trunks already in the forest (`WinterTree4` "Wood": Earth orange Plastic, 2.25 square, Studs). It reads as a log and matches the trees around it.
- Builder: `~/fruit-empire/tools/studded-logs.lua` matches each round log's length, angle and resting height (inventory: `~/fruit-empire/world/round-logs-2026-10-04.json`). Studs stay true 1-stud size at any thickness.
