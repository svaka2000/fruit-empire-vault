---
tags: [roadmap, m7, verification]
date: 2026-10-06
---
# M7 World restyle: verification

M7 swaps the round logs for studded logs, puts studded lily pads in the island ponds, rebuilds the border mountains, and places landmarks. Spec: [[Plot Overhaul and Environment]] rules F to I. Code: `~/fruit-empire/src-m2` tag `m7-deployed` (51b8801), contract `M7-CONTRACT.md` (section 8 = as built). Questions 46 and 49 are built on their defaults.

**All four builders have run in Studio (Edit). Their output stays in the place, waiting for Samarth's "go"** (roadmap rule 4). Nothing is live until Samarth publishes. To take a builder back out, run its `undo()` (as a clone, like `run()`), in reverse order: BorderMountainsBuilder, LandmarksBuilder, LilyPadSwap, StudLogSwap. Everything survived two Studio reopens.

The deployed scripts match `src-m2` exactly (94 of 94 after the mountains builder, by length and hash).

## 1. Studded logs (`StudLogSwap`)
Studio printed: 534 studded logs from 534 round logs (plain 256, vines 211, hollow 67), 2,091 parts (the old logs were 1,212), every old log moved to `ServerStorage.RoundLogs_backup_2026-10-04`, 0 round logs left. Worst margins: length 3.03% (limit 5%), position 0.0000, angle 0.0000 degrees.

| Before | After |
|---|---|
| ![[m7-before-driftwood.jpg]] | ![[m7-after-driftwood.jpg]] |
| ![[m7-before-forest-vines-log.jpg]] | ![[m7-after-forest-vines-log.jpg]] |
| ![[m7-before-forest-hollow-log.jpg]] | ![[m7-after-forest-hollow-log.jpg]] |

*Driftwood in daylight shows the D-003 look: a brown studded log with green moss plates. The forest logs read darker because they stand in the forest's shade. The hollow log keeps the old hollow log's size: 9.3 studs thick and 47 long.*

## 2. Studded lily pads (`LilyPadSwap`)
Studio printed: 19 pond pads (4 in the waterfall pool, 5 in each of the 3 ponds), 5 with flowers, all 766 sea pads in the backup studded too (so an undo of the Shallows brings back studded pads), 0 pads in the sea. Every distance rule held: water 3.9 (needs 1.5), pads 4.19 apart (needs 4), stones 2.53 (needs 2.5), bridges 3.01 (needs 3), waterfall 5.09 (needs 3). Pads that overlap sit at different heights (0.04 apart or more), so they never flicker.

| Before | After |
|---|---|
| ![[m7-before-pond-bridge.jpg]] | ![[m7-after-pond-pads.jpg]] |

![[m7-after-pads-close.jpg]]
*Close up: Samarth's original notched pad mesh with studs (D-021), two overlapping pads, and a pink flower with a gold centre.*

## 3. Landmarks (`LandmarksBuilder`)
Studio printed: 869 parts (cap 900): statues 16, bridges 22, camp 479, forest mushrooms 352. Its Studio-only check found nothing in the statues' way.

**Royal Gate statues.** Two Statue With Sword copies stand on the plaza side of the gate, facing the plaza, sword fire off. Each base is 2 studs from the road edge and 1.5 from the gate. (The spec put them 8 studs from the centre line, which would have covered 4 studs of the road; contract D12.)

| Before | After |
|---|---|
| ![[m7-before-royal-gate.jpg]] | ![[m7-after-royal-gate.jpg]] |

**Pond bridges.** The two bridges are now one stretched Stud Bridge each (dark studded planks with the pack's slight dip, no rails).

| Before | After |
|---|---|
| ![[m7-before-pond-bridge.jpg]] | ![[m7-after-pond-bridge.jpg]] |

**Forest camp** at the clearing nearest due south (Clearing09, r 1330): the Studded Pack House with its door toward the map centre, the Tent, the CampFire (its private sound swapped for a licensed Pro Sound Effects loop), log seats, a giant Mushroom, and the Stud Shop as the tamer's booth (its click target removed, no name tag). Four more giant mushrooms stand in the four nearest clearings. The stud Textures on the Tent, CampFire and logs were replaced by the Studs material (354 textures removed), because textures do not batch.

| Before | After |
|---|---|
| ![[m7-before-camp-clearing.jpg]] | ![[m7-after-camp.jpg]] |

## 4. Border mountains (`BorderMountainsBuilder`)
The first Studio run refused and changed nothing: its self-check found 932 parts "with a colour outside the six allowed ones". Measured in Studio: a part's colour is stored in 8 bits a channel and rounded DOWN (100.99/255 reads back as 100/255), while the check allowed only half a step. The offline tests modelled round-to-nearest. The check now allows one full step for stored colours (the exact values kept in attributes still use the tight check). The second run passed.

Studio printed: 8,208 parts (cap 9,000) in 36 sectors: the back row has 6 full pack copies and 30 cheaper copies, the middle row 36 cheaper copies, plus 36 ground slabs. Nothing is closer than r 1,520. The slab tops are at least 0.40 above the grass everywhere (no flicker). From the plaza, no gap shows at any of 3,600 bearings. The old ring (11,588 parts) is in `ServerStorage.Mountains_v3_backup_2026-10-04`.

| Before | After |
|---|---|
| ![[m7-before-sky-view.jpg]] | ![[m7-after-sky-view.jpg]] |
| ![[m7-before-behind-wall-b.jpg]] | ![[m7-after-behind-wall-b.jpg]] |
| ![[m7-before-behind-wall-a.jpg]] | ![[m7-after-behind-wall-a.jpg]] |

![[m7-after-plaza-skyline.jpg]]
*From the plaza: a row of separate snow-capped peaks. The old ring was one continuous jagged ridge.*

Behind the wall, the new mountains' green moss bases rise right behind it like foothills, so the "bare band" I worried about does not show from the ground. The `FrontRow` option (a low row in front, with 4 full copies instead of 6) is still there if you want the horizon fuller.

## Design calls for Samarth
1. **The pond bridges.** The old bridges were already studded plank bridges with rails, and I think they read better than the new railless dark ones. If you agree, the bridges can go back while the statues, camp and mushrooms stay. This needs a small change: today LandmarksBuilder only undoes as a whole.
2. **The statues' size.** At full pack size (29 studs) they are almost as tall as the gate and partly hide it from the plaza. They can be scaled down.
3. **Hollow logs.** They keep the old hollow logs' size (up to 9.3 thick, 47 long), which reads as a big beam. They can be made thinner.
4. **Moss plates.** Two per vines log (2,091 log parts). One per log gives 1,880, none gives 1,669.
5. **The mountain ring.** The new ring is separate peaks with green valleys (from the sky deck you can see grass between them); the old one was one solid ridge. If you want it fuller, `FrontRow` adds a low row in front (8,724 parts).

## Checked in Play (saving off)
| What | Result |
|---|---|
| ForestBuilder and ShallowsBuilder still load (Edit, as clones) | Both load; `ServerStorage.WorldKit` holds the six log templates and the pad flower |
| Stand on a studded driftwood log | Floor Plastic, the ground ray hits the log's Body, Running |
| Stand on a studded forest log | Same |
| Stand on a pond pad | Same (the pad) |
| Stand on both new bridges | Same (the bridge planks) |
| Walk across both bridges end to end | Every sample over the bridge, none over the pond's water or sand |
| Campfire sound | On the client: rbxassetid://9112780193, loaded, 36.0 s, playing, looped, volume 0.3 |
| Console | No errors |
| Samarth's save | Before and after: 1,746 bytes, hash 849516707. `StudioNoSave` removed at the end |

## Still to check
- Nothing for M7 itself. Publishing waits for your "go" on these screenshots and the design calls above.

## Update 8 Oct 2026: the M7 mountains are reverted
Samarth's call: the new mountain asset does not look good. `BorderMountainsBuilder.undo()` was run in Studio (Edit mode, through a fresh clone). It removed the M7 ring (36 sectors, 8,208 parts) and put the earlier ring back exactly as it was: `Main Island.Mountains` v3, 36 sectors, 11,588 parts, from `ServerStorage.Mountains_v3_backup_2026-10-04`. The builder never wrote Terrain, so nothing else needed putting back. The backup folder is now empty and gone (the ring is live again). Do not run `BorderMountainsBuilder.run()` again unless Samarth asks.

![[m7-mountains-reverted-before.jpg]]
*Behind the wall, before the revert: the M7 ring.*

![[m7-mountains-reverted-after.jpg]]
*The same spot after the revert: the earlier studded ring with its green ledges.*

![[m7-mountains-reverted-sky.jpg]]
*From the Cloud Garden deck: the whole earlier ring is back.*
