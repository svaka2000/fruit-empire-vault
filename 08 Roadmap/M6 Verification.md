---
tags: [roadmap, m6, verification]
date: 2026-10-05
---
# M6 Plot pens, lots and entrance huts: verification

M6 is built, deployed to Studio and checked in Play (not yet published). Spec: [[Plot Overhaul and Environment]] rules A to E, J and K. Every script in Studio matches `~/fruit-empire/src-m2` tag `m6-deployed` (89 of 89 game scripts, by length and hash). Samarth's save was never written: before and after the Play test, `GardenProfiles` `p_1662286109` was 1,746 bytes, hash 849516707, saveCount 612. `StudioNoSave` was removed at the end.

**The builders' output is in the place now, waiting for Samarth's "go"** (roadmap rule 4). Nothing is live until he publishes. To take it all back out: run `PlotLotBuilder` undo, then `PlotKitBuilder` undo (every old fence part and the old sign are in dated backups in ServerStorage).

![[m6-plot0-plaza.jpg]]
*An empty plot from the plaza: stone border, hedges, corner and gate pillars with lanterns, the front fence, flower beds, the stone path, the lawn with trees, crates, stepping stones and the hut.*

![[m6-gate-eye-level.jpg]]
*The gate at eye height: the beam with the nameplate, lanterns on the pillars.*

![[m6-lots-overhead.jpg]]
*Two lots from above: each walled pen on its own lawn between the roads.*

![[m6-hut.jpg]]
*The underground entrance hut: studded stone, stepped roof, doors with a gold padlock, a lantern, mine-cart rails.*

## What M6 put in
- **A walled garden on every plot.** The old two-rail fence is replaced by a stone border, studded hedges on three sides, corner pillars, a low wooden fence at the front, and a stone gate with two lanterns. It is rebuilt from the soil's size every time the plot size changes.
- **A lot around every pen.** Green lawn from road to road, three studded trees, a crate stack, stepping stones and a stone path to the walkway.
- **The underground entrance hut** in the back-right corner of every lot. It stays closed until the owner has a mine floor (M13 opens it).
- **Your name and stage on your gate.** Your display name on the gate beam, and two banners in the colour of your newest rarity.
- **Lanterns at night.** The gate lanterns (and an open hut's lantern) light up when it gets dark.
- **A new Plot Size sign**, built from the Upgrade panel's Plot Size card with a live picture of your pen.

## Builders run in Studio (Edit)
- `PlotKitBuilder`: 14 kit pieces, the nameplate, a 29-part hut, the 5-part sign and the grow puff. All 10 palette colours were read from their sources (Smoky grey and Fossil from the Studed Mountain pack, the live fence wood, GrassTop), with no fallbacks. Self-check ok.
- `PlotLotBuilder`: 12 plots. Lots are 63 to 83 parts, and plot totals are 167 to 187 of the 205 cap. 468 old fence parts went to the backup. 10 trees were scaled down (0.80 to 0.95) to keep the road margin. Closest margins: 0.40 from the reserved pen box (a stepping stone), 0.526 from a road, 0.60 from the forest edge.
- A second run of `PlotLotBuilder` left the place exactly the same (1,974 parts, the same position signature) and backed nothing up twice.

## Checked in Play
| # | What | Result |
|---|---|---|
| 2 | Soil on all 12 plots | `PlotSoil` on every soil, top 0.853 |
| 3 | `DevSetPlotSize` 0 to 4 | Pen parts 60, 66, 67, 72, 73. Growing never moved a fruit. Banners and nameplate survive each rebuild. |
| 4 | Nudge | Standing 1 stud outside the right hedge at level 0, then growing to 1: moved 2.30 studs, ends 2 studs outside the new hedge, touching no pen part. |
| 6 | Dressing | Owned plot: nameplate shows the display name, banners in the stage colour. Unowned: both hidden. Rebirths 0, 2, 5, 8 give Common, Rare, Mythic and Transcendent banners, exactly `Config.RarityColors`. |
| 6b | Hut | `UndergroundFloors` 1: doors and padlock hidden and walk-through, `EntranceOpen` true. Back to 0: closed again. |
| 7 | Lamps | At night the gate lanterns are lit (screenshot below). |
| 11 | Sign | Ready: "Level 2/5", "39 x 63 studs", "$20K" in green, Buy shown. Too poor: price in red. Max: "Level 5/5", "Maxed". Its foot stands on the lawn (y 0.20). |
| Buy | A purchase (the sign's Upgrade remote) | Level 1 to 2; four grow puffs on the corner pillars; the sign moved on to "Level 3/5, 43 x 70 studs, $160K". |
| Console | Whole session | No script errors (see the one warning below). |

![[m6-own-gate-night.jpg]]
*Your gate at night: lit lanterns, the nameplate, Transcendent banners, and the Plot Size sign on the right.*

![[m6-sign.jpg]]
*The Plot Size sign.*

## Found and fixed in the Studio pass
1. **The kit builder refused to run.** Its self-check found the nameplate's name label was not scaled. In Studio, a scaled label always wraps: `TextScaled = true` turns wrapping on, and setting `TextWrapped = false` afterwards turns scaling back off. Lune (the test runner) does not model that link, so the harnesses passed. Fix: the builder no longer sets `TextWrapped`. The builder is atomic, so the failed run changed nothing.

## Still open
- **The Lock dome does not quite cover the pen's corners.** The dome is an ellipsoid sized as soil + 35 across and + 22 along. An ellipse only covers a rectangle's corners when it is about 1.4 times wider, so the front corner pillars (and the new sign) stand just outside it. This was already true of the old fence's corner posts. The fix (size the dome from the pen's outer edge times 1.41) is queued right after M8, because M8's crate work is editing the same file.
- **"Infinite yield possible on IndexState"** warnings on join. The Index and Rebirth menus wait for a folder the server only creates after the profile loads, and the load took more than 5 seconds in Studio right after the place reopened. Not caused by M6 (pens are built at server start, before anyone joins).
- Not checked live: walking into a hedge by hand, the Crown Hunt avoiding lots (it refuses anything inside a plot model, and the lots are inside plot models), TNT next to the pen, phones.

![[m6-lock-dome.jpg]]
*The Lock dome over a level-4 pen: the front corner pillars poke out.*

## Design calls for Samarth
- Look at the screenshots and reply "go" before publishing. The questions this was built on: 47 (the hut in the back-right corner) and 48 (hedges on three sides, stage banners, your display name, no statue fire, the tower slot kept free in the back-left corner), all on their recommended defaults.
- The front-yard flower beds are small. They can be made larger or swapped for something else.
